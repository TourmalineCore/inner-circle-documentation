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

One of the reasons was a function on the UI. This function joins the date and time before sending them to the server, but it does not reset seconds if they are in the input data.

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

The system does not work with seconds — this is a business rule. So we need to reset seconds at the domain level when we create `TrackedEntryBase` and all its child classes. This will fix the problem at the root and protect us from any way of creating entries with seconds.

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
