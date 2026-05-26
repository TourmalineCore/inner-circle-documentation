# 005: Away Unpaid Entry Prototype (Backend)

## Context

Need to design tracking of Away (Unpaid) Entry feature. 

## Entries model
Дописать причину почему для Make Up Time требуется отдельная модель (валидация на уровне БД). Не будет иметь собственных эндпоинтов, будет создаваться в других. 

### Make Up Time Entry

```c#
public class MakeUpTimeEntry : TrackedEntryBase
{
    // EntityFrameworkCore related empty default constructor
#pragma warning disable CS8618 // Non-nullable field must contain a non-null value when exiting constructor. Consider adding the 'required' modifier or declaring as nullable.
    public MakeUpTimeEntry()
#pragma warning restore CS8618 // Non-nullable field must contain a non-null value when exiting constructor. Consider adding the 'required' modifier or declaring as nullable.
    {
        Type = EntryType.MakeUpTime;
    }


    public long ParentEntryId { get; set; }
}
```

### Away Unpaid Entry

```c#
public class AwayUnpaidEntry : TrackedEntryBase
{
    // EntityFrameworkCore related empty default constructor
#pragma warning disable CS8618 // Non-nullable field must contain a non-null value when exiting constructor. Consider adding the 'required' modifier or declaring as nullable.
    public AwayUnpaidEntry()
#pragma warning restore CS8618 // Non-nullable field must contain a non-null value when exiting constructor. Consider adding the 'required' modifier or declaring as nullable.
    {
        Type = EntryType.AwayUnpaid;
    }

    public string Description { get; set; }

    public List<MakeUpTimeEntry> MakeUpTime { get; set; }
}
```

## Endpoints

1. **POST** /api/tracking/away-unpaid-entries - add away unpaid entries

**Request body:**
```c#
{
  description: string, // or awayReason
  startTime: DateTime,
  endTime: DateTime,
  makeUpTime: [
    {
      startTime: DateTime,
      endTime: DateTime,
    }
  ]
}
```

**Response body:**
```c#
{
  newAwayUnpaidEntryId: long
}
```

2. **POST** /api/tracking/away-unpaid-entries/{id} - update away unpaid entry

**Request body:**
```c#
{
  description: string, // or awayReason
  startTime: DateTime,
  endTime: DateTime,
  makeUpTime: [
    {
      startTime: DateTime,
      endTime: DateTime,
    }
  ]
}
```

## Validation

1. Add new migrations with updated overlap constraint.
2. Make sure make up time is equal to away unpaid time.

## Testing strategy

### E2E karate tests

Cases:
1. No Permissions Lead to Unauthorized

Steps:
- Authorization under an account without permissions
- Call **POST** /api/tracking/away-unpaid-entries and check response status it should be 403
- Call **POST** /api/tracking/away-unpaid-entries/{id} and check response status it should be 403

2. Happy path 

Steps:
- Authorization under an account with all permissions
- Call **POST** /api/tracking/away-unpaid-entries to add away paid entry
- Call **POST** /api/tracking/away-unpaid-entries/{id} to update away paid entry
- Call **GET** /api/tracking/entries to verify added away paid entry
- Call **DELETE** /api/tracking/entries to hard delete the away unpaid entry
- Call **GET** /api/tracking/entries to verify that away unpaid entry was deleted

### Integration tests

Cases:
1. Away unpaid cannot overlap with unwell entries
2. Away unpaid cannot overlap with task entries 
3. Make up time can overlap with task entries
4. Make up time cannot overlap with unwell entries
5. Make up time cannot overlap with away unpaid entries

## Out Of Scope
Update personal report