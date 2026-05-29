# 005: Away With Make-Up Time Entry Prototype (Backend)

## Context

Need to design tracking of ***Away With Make-Up Time*** entry feature. 

## Entries model
We will have a separate model for Make-Up Time entry which will be validated at the database level.
It will have different rules of validation in comparison with other entries, e.g. it can overlap with a Task entry, whereas Away entry cannot overlap with a Task entry (for the full list see [ADR on overlap validation](003-time-api-overlap-validation.md)).
Make-Up Time entry will not have its own endpoints, because it will be created in a related entry (in this case Away With Make-Up Time entry) when calling this related entry's endpoints for create/update.

### Make-Up Time Entry

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

    public long RelatedEntryId { get; set; }
}
```

### Away With Make-Up Time Entry

```c#
public class AwayWithMakeUpTimeEntry : TrackedEntryBase
{
    // EntityFrameworkCore related empty default constructor
#pragma warning disable CS8618 // Non-nullable field must contain a non-null value when exiting constructor. Consider adding the 'required' modifier or declaring as nullable.
    public AwayWithMakeUpTimeEntry()
#pragma warning restore CS8618 // Non-nullable field must contain a non-null value when exiting constructor. Consider adding the 'required' modifier or declaring as nullable.
    {
        Type = EntryType.AwayWithMakeUpTime;
    }

    public string Description { get; set; }

    public List<MakeUpTimeEntry> MakeUpTimeList { get; set; }
}
```

## Endpoints

1. **POST** `/api/tracking/away-with-make-up-time-entries` - add Away With Make-Up Time entry

**Request body:**
```c#
{
  // we discarded the idea to call this field "awayReason" in favor of consistency across all entry types
  description: string, 
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
  newAwayWithMakeUpTimeEntryId: long
}
```

2. **POST** `/api/tracking/away-with-make-up-time-entries/{id}` - update Away With Make-Up Time entry

**Request body:**
```c#
{
  // we discarded the idea to call this field "awayReason" in favor of consistency across all entry types
  description: string,
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
2. Make sure Make-Up Time is equal to Away With Make-Up Time. These checks will be made on the level of Handler (or maybe separate validator?).

## Testing Strategy

### E2E Karate Tests

Cases:
1. No Permissions Lead to Unauthorized

Steps:
- Authorization under an account without permissions
- Call **POST** `/api/tracking/away-with-make-up-time-entries` and check response status it should be 403
- Call **POST** `/api/tracking/away-with-make-up-time-entries/{id}` and check response status it should be 403

2. Happy path 

Steps:
- Authorization under an account with all permissions
- Call **POST** `/api/tracking/away-with-make-up-time-entries` to add Away With Make-Up Time entry
- Call **POST** `/api/tracking/away-with-make-up-time-entries/{id}` to update Away With Make-Up Time entry
- Call **GET** `/api/tracking/entries` to verify added Away With Make-Up Time entry with related Make-up Time entry/entries
- Call **DELETE** `/api/tracking/entries/{id}/hard-delete` to hard delete the Away With Make-Up Time entry with related Make-up Time entry/entries
- Call **GET** `/api/tracking/entries` to verify that Away With Make-Up Time entry with related Make-up Time entry/entries were deleted

### Integration tests

Cases:
1. Away With Make-Up Time cannot overlap with Unwell entries
2. Away With Make-Up Time cannot overlap with Task entries 
3. Make-up Time can overlap with Task entries
4. Make-up Time cannot overlap with Unwell entries
5. Make-up Time cannot overlap with Away With Make-Up Time entries

### Unit tests

Cases:
1. Check exception message in case Make-up Time slot (or sum of slots if there are more than one Make-up Time slot) is not equal to Away With Make-Up Time time.

Exception message: `Away time must be equal to Make-up time.`

## Use Cases for e2e Tests

1. 
```
GIVEN time-tracker page, 
WHEN add Away With Make-Up Time card with one Make-Up Time slot, 
SHOULD create this card and one Make-Up Time card in time-tracker 
```

2. 
```
GIVEN time-tracker page, 
WHEN add Away With Make-Up Time card with two Make-Up Time slots, 
SHOULD create this card and two Make-Up Time cards in time-tracker 
```

Maybe it is worth uniting use cases 1 & 2.

3.
``` 
GIVEN an Away entry with two Make-Up Time slots,
WHEN track Task into first Make-Up Time slot,
THEN update date of this Make-Up Time slot
SHOULD display the Task on the same slot and move Make-Up Time card to the new date 
```

4.
```
GIVEN an Away entry with two Make-Up Time slots,
WHEN update Away entry start time and end time without changing its duration,
SHOULD move Away card to the new time and leave Make-Up Time slots untouched
```

## Out Of Scope
Update personal report
Why?