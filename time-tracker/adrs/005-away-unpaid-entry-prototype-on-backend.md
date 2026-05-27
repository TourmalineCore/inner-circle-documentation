# 005: Away Unpaid Entry Prototype (Backend)

## Context

Need to design tracking of Away (Unpaid) Entry feature. 

## Entries model
We will have a separate model for Make Up Time entry which will be validated at the database level.
It will have different rules of validation in comparison with other entries, e.g. it can overlap with a Task entry, whereas Away entry cannot overlap with a Task entry (for the full list see [ADR on overlap validation](003-time-api-overlap-validation.md)).
Make Up Time entry will not have its own endpoints, because it will be created in a parent entry (in this case Away entry) when calling this parent's endpoints for create/update.

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

    public List<MakeUpTimeEntry> MakeUpTimeList { get; set; }
}
```

## Endpoints

1. **POST** `/api/tracking/away-unpaid-entries` - add Away (unpaid) entry

**Request body:**
```c#
{
  description: string, // or awayReason
  startTime: DateTime,
  endTime: DateTime,
  makeUpTimeList: [
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

2. **POST** `/api/tracking/away-unpaid-entries/{id}` - update Away (unpaid) entry

**Request body:**
```c#
{
  description: string, // or awayReason
  startTime: DateTime,
  endTime: DateTime,
  makeUpTimeList: [
    {
      startTime: DateTime,
      endTime: DateTime,
    }
  ]
}
```

## Validation

1. Add new migrations with updated overlap constraint.
2. Make sure make up time is equal to away unpaid time. These checks will be made on the level of Handler (or maybe separate validator?).

## Testing strategy

### E2E karate tests

Cases:
1. No Permissions Lead to Unauthorized

Steps:
- Authorization under an account without permissions
- Call **POST** `/api/tracking/away-unpaid-entries` and check response status it should be 403
- Call **POST** `/api/tracking/away-unpaid-entries/{id}` and check response status it should be 403

2. Happy path 

Steps:
- Authorization under an account with all permissions
- Call **POST** `/api/tracking/away-unpaid-entries` to add Away unpaid entry
- Call **POST** `/api/tracking/away-unpaid-entries/{id}` to update Away unpaid entry
- Call **GET** `/api/tracking/entries` to verify added Away unpaid entry with related Make-up entry/entries
- Call **DELETE** `/api/tracking/entries/{id}/hard-delete` to hard delete the Away unpaid entry with related Make-up entry/entries
- Call **GET** `/api/tracking/entries` to verify that Away unpaid entry with related Make-up entry/entries were deleted

### Integration tests

Cases:
1. Away unpaid cannot overlap with unwell entries
2. Away unpaid cannot overlap with task entries 
3. Make up time can overlap with task entries
4. Make up time cannot overlap with unwell entries
5. Make up time cannot overlap with away unpaid entries

### Unit tests

Cases:
1. Check exception message in case Make up time slot (or sum of slots if there are more than one Make up time slot) is not equal to Away unpaid time.

Exception message: `Away time must be equal to Make-up time.`

## Out Of Scope
Update personal report
