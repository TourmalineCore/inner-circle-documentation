# Time Tracker Backend Strategy 2025-11-20

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

- **POST** /api/work-entries - add
- **POST** /api/work-entries/{id} - update
- **GET** /api/work-entries?startTime={startTime}&endTime={endTime} - get list by period
- **GET** /api/work-entries/{id} - get one by id
- **DELETE** /api/work-entries/{id} - soft delete

#### adjustments

- **POST** /api/adjustments - add
- **POST** /api/adjustments/{id} - update
- **GET** /api/adjustments?startTime={startTime}&endTime={endTime} - get list by period
- **GET** /api/adjustments/{id} - get one by id
- **DELETE** /api/adjustments/{id} - soft delete


## db diagram

```mermaid
erDiagram
    work_entries ||--|| projects : "1-to-1"
    adjustments ||--o{ adjustments : "1-to-many"
    work_entries {
      id int PK "int or long?"
      tenant_id long FK
      employee_id long FK
      project_id int FK "internal request projects list, int or long?"
      start_time timestamp
      end_time timestamp
      timezone text
      duration interval "(calculated)"
      type int
      title text
      task_id text "Nullable"
      description text "Nullable"
      is_deleted boolean
    }
    projects {
      id int PK "int or long?"
      tenant_id long FK
      name text
    }
    adjustments {
      id int PK "int or long?"
      tenant_id long FK
      employee_id long FK
      parent_id int FK "Nullable. ссылка сама на себя на ориг карточку из этой же таблицы, int or long?"
      start_time timestamp
      end_time timestamp
      timezone text
      duration interval "положительное для переработок, отрицательное для отгулов"
      type int
      description text "Nullable"
      is_deleted boolean
      sick_leave_reason boolean "Nullable. причина плохого самочувствия"
      is_approved boolean
      is_billable boolean
      is_full_day boolean "?"
    }
```
