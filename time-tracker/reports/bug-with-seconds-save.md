# Report: Bug With Seconds Save

## Context

A bug appeared in production: seconds were saved to the database in time fields. The bug was noticed after one user tracked time from 07:00 to 16:00, but the metrics showed 8 hours 59 minutes. In the network request, we could see that the start time had 10 seconds. We could not reproduce the bug by creating entries through the UI.

When we checked the production database backup, we found 4 entries with seconds (at the time of this report, the database has 9835 entries):

| StartTime | EndTime | Type |
|---|---|---|
| 2026-03-23 23:45:00 | 2026-03-23 23:55:19 | TaskEntry |
| 2026-03-24 00:00:36 | 2026-03-24 00:15:00 | TaskEntry |
| 2026-07-15 00:00:39 | 2026-07-15 01:40:00 | TaskEntry |
| 2026-10-01 07:00:10 | 2026-10-01 16:00:00 | UnwellEntry |

## Cause of error

The main reason is that the server saves seconds if they come in the request and does not validate them. According to business rules, the system does not work with seconds, but this rule is not written anywhere.

One of the triggers was a function on the UI. This function joins the date and time before sending them to the server, but it does not reset seconds if they are in the input data.

```js
export function concatDateAndTime({
  date,
  time,
}: {
  date: Date,
  time: Date,
}) {
  return moment(date)
    .hours(moment(time)
      .hours())
    .minutes(moment(time)
      .minutes())
    .seconds(moment(time)
      .seconds())
    .format(`YYYY-MM-DDTHH:mm:ss`)
}
```

## Decision

The system does not work with seconds — this is a business rule. That's why it was decided to add a constraint that checks that time with seconds cannot be saved. This will allow validating start_time and end_time at the database level and will close all possible ways of saving incorrect data.

Before adding the new constraint, we need to write a migration that will reset the seconds to zero in all existing records on prod.

The server-side solution fully fixes the problem. In addition, we should also change the UI function so that it resets seconds before sending data to the server. This will prevent sending time with seconds.

```js
export function concatDateAndTimeToMinute({
  date,
  time,
}: {
  date: Date,
  time: Date,
}) {
  return moment(date)
    .hours(moment(time)
      .hours())
    .minutes(moment(time)
      .minutes())
    .startOf('minute')
    .format(`YYYY-MM-DDTHH:mm:ss`)
}
```

We also discussed adding extra attributes with validation to all requests that have startTime and endTime.
But in the end, we decided not to add them, because testing these attributes would require building testing infrastructure that would have to be maintained, and for an optional validation layer this is unnecessary.

## Alternatives
Reset seconds at the domain level when we create `TrackedEntryBase` and all its child classes.

```c#
public class TrackedEntryBase : EntityBase, IOwnedByEmployee, ICanBeDeleted
{
    private DateTime _startTime;

    private DateTime _endTime;

    public DateTime StartTime
    {
        get => _startTime;
        set => _startTime = TrimToMinutes(value);
    }

    public DateTime EndTime
    {
        get => _endTime;
        set => _endTime = TrimToMinutes(value);
    }

    private static DateTime TrimToMinutes(DateTime dateTime)
    {
        return new DateTime(
            dateTime.Year,
            dateTime.Month,
            dateTime.Day,
            dateTime.Hour,
            dateTime.Minute,
            0,
            dateTime.Kind
        );
    }
}
```

### Disadvantages
- It does not protect against adding incorrect data directly through the database.