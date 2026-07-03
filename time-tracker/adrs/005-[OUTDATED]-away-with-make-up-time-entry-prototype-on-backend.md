# 005: Away With Make-Up Time Entry Prototype (Backend)

## Status
Outdated 

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

[Away With Make-Up Time Endpoints](../backend-contract.md#away-with-make-up-time)

## Validation

1. Add new migrations with updated overlap constraint.
2. Make sure Make-Up Time is equal to Away With Make-Up Time. These checks will be made on the level of Handler.

## Testing Strategy

### E2E Karate Tests

Cases:
1. No Permissions Lead to Unauthorized (403 error)

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

Exception message: `Total make-up time must equal your away time. Please check and adjust your entries.`

## Use Cases for e2e Tests

Cypress Happy path
1. 
```
GIVEN time-tracker page, 
WHEN add Away With Make-Up Time card with one Make-Up Time slot, 
SHOULD create this card and one Make-Up Time card in time-tracker 
```
or
```
I was away one hour
I will Make-Up for it today after my working hours
I did as it planned
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
AND update date of this Make-Up Time slot
SHOULD display the Task on the same slot and move Make-Up Time card to the new date 
```

Karate Happy path
```
I was away two hours
I want Make-Up 30 minutes tomorrow and 1:30 day after tomorrow
Then i realized that i can't Make-up tomorrow and want to move this 30 minutes Make-Up to another day 
```

```
I plan to be away tommorow from for 4PM to 5PM
I will Make-Up in advance today
I Made-Up as planned
But my away reason was cancelled and thus I will not be away (I delete it)
And my task tracked yesterday remains untouched  
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
