# 007: Sick Leave Period: Date-Only or DateTime

## Status
Accepted (2026-07-10)

## Context
We need to decide what data format to return for sick leave entries in the GET endpoint that returns all entries for a period.

Two options were proposed:

1. Return only the date range (start and end dates without time)

```c#
sickLeaveEntries: [
  {
    id: long,
    entryType: int,
    period: {
      startDate: DateOnly,
      endDate: DateOnly
    }
  }
]
```

2. Return dates with time

```c#
sickLeaveEntries: [
  {
    id: long,
    entryType: int,
    startTime: DateTime,
    endTime: DateTime
  }
]
```

## Decision
Return the period as dates only (startDate/endDate), without time. It's the frontend's responsibility to display sick leaves as full-day events. The period is inclusive on both sides. For example, July 10–12 means 3 full days of sick leave.

## Rationality
1. Sick leave always covers full days — including time doesn't make sense.
2. The create/update endpoints expect the API to accept DateOnly format.

## Consequence
1. Frontend must interpret the received dates as full-day events when displaying sick leave in the time tracker.