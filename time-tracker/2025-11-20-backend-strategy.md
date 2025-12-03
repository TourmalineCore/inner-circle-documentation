# Time Tracker Backend Strategy 2025-11-20, 2025-11-25

## Entities

### 1. work_entries

- задача

#### sql notes

```sql
SELECT *
FROM work_entries
WHERE startTime > @startTime
AND endTime < @endTime
AND tenantId = @tenantId
AND employeeId = @employeeId

CREATE INDEX a ON work_entries
```

### 2. adjustments

**другие события**

- отсутствие (отраб)
- отсутствие (не отраб)
- опоздание
- плохое самочувствие
- отработка
- переработка
- отгул
- обед

**события на день**

- дей-офф (отраб)
- дей-офф (не отраб)
- больничный
- отпуск

## endpoints

#### work-entries

1. **GET** /api/time/tracking/work-entries?startTime={startTime}&endTime={endTime} - get list by period

**Response body:**
```ts
{
  workEntries: [
    {
      id: long,
      title: string,
      taskId?: string,
      startTime: Date,
      endTime: Date,
    },
  ]
}
```

2. **POST** /api/time/tracking/work-entries - add

**Request body:**
```ts
{
  title: string,
  taskId?: string,
  startTime: Date,
  endTime: Date,
  timeZoneId: string
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
  taskId?: string,
  startTime: Date,
  endTime: Date,
  timeZoneId: string
}
```

**Response body:** 200 OK

4. **DELETE** /api/time/tracking/work-entries/{id}/soft-delete - soft delete   

#### adjustments

- **POST** /api/time/tracking/adjustments - add
- **POST** /api/time/tracking/adjustments/{id} - update
- **GET** /api/time/tracking/adjustments?startTime={startTime}&endTime={endTime} - get list by period
- **DELETE** /api/time/tracking/adjustments/{id}/soft-delete - soft delete

## db for add task & get all tasks & update task (1 iteration)

```ts
{
  id long PK
  tenant_id long FK
  employee_id long FK
  // project_id long FK "internal request projects list"
  start_time timestamp
  end_time timestamp
  time_zone_id text
  duration interval "(calculated)"
  type int // get default value from emun for now
  title text
  task_id text "Nullable"
  description text "Nullable"
  is_deleted boolean
}
```

## db diagram

```mermaid
erDiagram
    work_entries ||--|| projects : "1-to-1"
    adjustments ||--o{ adjustments : "1-to-many"
    work_entries {
      id long PK
      tenant_id long FK
      employee_id long FK
      project_id long FK "internal request projects list"
      start_time timestamp
      end_time timestamp
      time_zone_id text
      duration interval "(calculated)"
      type int
      title text
      task_id text "Nullable"
      description text "Nullable"
      is_deleted boolean
    }
    projects {
      id long PK
      tenant_id long FK
      name text
    }
    adjustments {
      id long PK
      tenant_id long FK
      employee_id long FK
      parent_id long FK "Nullable. a self-reference to the original card from the same table"
      start_time timestamp
      end_time timestamp
      time_zone_id text
      duration interval "positive for overtime, negative for time off"
      amount interval "to think about whether time is deducted from working hours or not"
      type int
      description text "Nullable"
      is_deleted boolean
      sick_leave_reason int "Nullable. причина плохого самочувствия"
      is_approved boolean
      is_need_comment "? check the box to indicate whether or not you should write the reason for feeling unwell"
      is_paid boolean
      is_full_day boolean "?"
    }
```
