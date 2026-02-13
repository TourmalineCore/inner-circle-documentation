# Time Tracker Backend Strategy 2025-11-20, 2025-11-25

## Entities

### 1. work_entries

- задача
- отсутствие (отраб)
- отсутствие (не отраб)
- опоздание
- плохое самочувствие
- отработка
- переработка
- обед

**события на день** (возможно)

- дей-офф (отраб)
- дей-офф (не отраб)
- больничный
- отпуск

#### sql notes

```sql
SELECT *
FROM work_entries
WHERE startTime > @startTime
AND endTime < @endTime
AND tenantId = @tenantId
AND employeeId = @employeeId
AND type = @type

CREATE INDEX a ON work_entries
```

## endpoints

#### work-entries

1. **GET** /api/time/tracking/work-entries?startDate={startDate}&endDate={endDate} - get list by period

**Response body:**
```ts
{
  workEntries: [
    {
      id: long,
      title: string,
      taskId: string,
      projectName: string,
      description: string,
      startTime: DateTime,
      endTime: DateTime,
      type: long,
    },
  ]
}
```

2. **POST** /api/time/tracking/work-entries - add

**Request body:**
```ts
{
  title: string,
  taskId: string,
  projectId: long,
  description: string,
  startTime: DateTime,
  endTime: DateTime,
  timeZoneId: string
  type: long,
}
```

**Response body:**
```ts
{
  newWorkEntryId: long
}
```

3. **POST** /api/time/tracking/work-entries/{id} - update

**Request body:**
```ts
{
  title: string,
  taskId: string,
  projectId: long,
  description: string,
  startTime: DateTime,
  endTime: DateTime,
  timeZoneId: string
  type: long,
}
```

**Response body:** 200 OK

4. **GET** /api/time/tracking/work-entries/projects?date={date} - get employee's projects

**Response body:**

```ts
{
  projects: [
    {
      id: long,
      name: string;
    },
  ]
}
```

5. **DELETE** /api/time/tracking/work-entries/{id}/soft-delete - soft delete  

## db for add task & get all tasks & update task (1 iteration)

```ts
{
  id long PK
  tenant_id long FK
  employee_id long FK
  project_id long FK "internal request projects list"
  start_time timestamp
  end_time timestamp
  time_zone_id text
  duration interval "(calculated)"
  type int // get default value from emun for now
  title text
  task_id text
  description text
  is_deleted boolean
}
```

## db diagram

```mermaid
erDiagram
    work_entries ||--|| projects : "1-to-1"
    work_entries {
      id long PK
      tenant_id long FK
      employee_id long FK
      project_id long FK "internal request projects list"
      parent_id long FK "? Nullable. a self-reference to the original card from the same table"
      start_time timestamp
      end_time timestamp
      time_zone_id text
      duration interval "(calculated)"
      amount interval "?"
      type int
      title text "Nullable."
      task_id text "Nullable."
      description text "Nullable."
      is_deleted boolean
      is_paid boolean "?"
      is_full_day boolean "?"
    }
    projects {
      id long PK
      tenant_id long FK
      name text
    }
```
