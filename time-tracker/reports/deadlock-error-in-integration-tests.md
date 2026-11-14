# Report: Deadlock (40P01) when running integration tests in parallel

## Context

When running integration tests in parallel, a PostgreSQL deadlock error occurs: `40P01: deadlock detected`. The error happens during the insertion (`INSERT`) of records into the `tracked_entries` table, which causes the tests to fail before executing the main logic being tested.

Below are the tests that constantly conflict when run in parallel.

```c#
[Fact]
public async Task UpdateTaskEntryAsync_ShouldThrowInvalidTimeRangeExceptionIfStartTimeIsGreaterEndTime()
{
    var context = CreateTenantDbContext();

    var mockClaimsProvider = MockClaimsProviderFactory.CreateMock(EMPLOYEE_ID, TENANT_ID);

    var updateTaskEntryCommand = new UpdateTaskEntryCommand(context, mockClaimsProvider);

    var taskEntry = await AddEntityAndSaveAsync(
        context,
        new TaskEntry
        {
            TenantId = TENANT_ID,
            EmployeeId = EMPLOYEE_ID,
            Title = "Task 1",
            StartTime = new DateTime(2025, 11, 24, 9, 0, 0),
            EndTime = new DateTime(2025, 11, 24, 10, 0, 0),
            TaskId = "#2231",
            ProjectId = 1,
            Description = "Task description",
        }
    );

    var updateTaskEntryRequest = new UpdateTaskEntryRequest
    {
        Id = taskEntry.Id,
        Title = "Task 2",
        StartTime = new DateTime(2025, 11, 25, 12, 0, 0),
        EndTime = new DateTime(2025, 11, 25, 11, 0, 0),
        TaskId = "#22",
        ProjectId = 2,
        Description = "Task description",
    };

    var exception = await Assert.ThrowsAsync<InvalidTimeRangeException>(
        async () => await updateTaskEntryCommand.ExecuteAsync(updateTaskEntryRequest)
    );

    Assert.Contains("ck_entries_end_time_is_greater_than_start_time", exception.InnerException!.Message);
    Assert.Equal("End time must be greater than start time", exception.Message);
}
```

```c#
[Fact]
public async Task GetTrackedEntriesWithNonExistentEmployeeId__ShouldReturnEmptyTrackedEntriesList()
{
    var context = CreateTenantDbContext();

    var getEmployeeTrackedEntriesQuery = new GetEmployeeTrackedEntriesQuery(context);

    var taskEntry = new TaskEntry
    {
        TenantId = TENANT_ID,
        EmployeeId = 1,
        StartTime = new DateTime(2025, 11, 24, 9, 0, 0),
        EndTime = new DateTime(2025, 11, 24, 10, 0, 0),
    };

    await AddEntityAndSaveAsync(context, taskEntry);

    var nonExistentEmployeeId = -1;

    var result = await getEmployeeTrackedEntriesQuery
        .GetAsync(
            nonExistentEmployeeId,
            new DateOnly(2025, 11, 01),
            new DateOnly(2025, 11, 30)
        );

    Assert.Empty(result);
    }
```

## Cause of error

1. **Identical data in parallel tests:** Both tests create a `TaskEntry` record with exactly the same time interval
(`StartTime = 09:00:00`, `EndTime = 10:00:00`) for the same employee and tenant.

2. **Exclusion Constraint:** The `tracked_entries` table has a constraint configured that prohibits overlapping time intervals for the same employee (`ck_entries_1_2_3_no_time_overlap`).

3. **Locking mechanism:** When executing `INSERT`, PostgreSQL must check this constraint. To do this, it acquires locks (`ShareLock`) on tuples and GiST index pages to compare the new interval with existing ones.

4. **Circular dependency (Deadlock):**
- `Transaction A` sees an uncommitted row from `Transaction B`.
- To check if its own insert will break any rules after `Transaction B` finishes, `Transaction A` waits for `Transaction B` to complete.
- At the same time, ` Transaction B` sees `Transaction A's` uncommitted row and also tries to acquire a ShareLock on it.
- Result: Deadlock. PostgreSQL terminates one of the transactions with a `40P01` error.

**Important note on isolation:** Transactions are logically isolated, but they are not isolated from physical resources (index rows and pages on disk). Parallel queries compete for the same physical locks.

For diagnostics, extended logging was enabled:
```sql
ALTER SYSTEM SET log_statement = 'all';
ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM SET deadlock_timeout = '1s';
ALTER SYSTEM SET logging_collector = on;
SELECT pg_reload_conf();
```

**Key log fragment:**
```
2026-10-15 10:20:00.380 UTC [5285] ERROR:  deadlock detected
2026-10-15 10:20:00.380 UTC [5285] DETAIL:  Process 5285 waits for ShareLock on transaction 4004; blocked by process 5286.
    Process 5286 waits for ShareLock on transaction 4002; blocked by process 5285.
    Process 5285: INSERT INTO tracked_entries (...) VALUES (...)
    Process 5286: INSERT INTO tracked_entries (...) VALUES (...)
2026-10-15 10:20:00.380 UTC [5285] CONTEXT:  while checking exclusion constraint on tuple (0,28) in relation "tracked_entries"
```

**Why is the constraint name `ck_entries_1_2_3_no_time_overlap` not in the logs?**

The constraint name `ck_entries_1_2_3_no_time_overlap` does not appear anywhere in the error logs because the deadlock occurs during the constraint checking phase, not during the violation phase. Both processes get stuck trying to acquire index locks to verify the data. PostgreSQL kills one of the processes as a deadlock victim before it can determine whether the constraint would have been violated. Therefore, the log only records the fact of locking at the tuple level, not the name of the rule itself.

**Is there a problem in production?**

This issue is very unlikely to happen in production. The chance of two users creating the exact same overlapping record for the same employee at the exact same millisecond is almost zero.

