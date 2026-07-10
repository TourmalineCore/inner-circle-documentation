# 008: Sick leave hours storage

## Status
Proposed

## Context
It is necessary to make a decision about what time will be stored in `start_time` and` end_time` for the sick leave in Db, and how to ensure that the timesheet matches up. More about sick leave in [DDR](../ddrs/sick-leave.md)

### What does it mean for a timesheet to "match"?
Typically, this means that the total number of hours recorded by an employee on their timesheet equals the number of hours they are required to work under their employment contract (e.g., 8 hours per day or 40 hours per week).

In the timesheet, sick leave is recorded in working hours — i.e., the hours the employee was scheduled to work according to their work schedule. In our case, we have an 8-hour workday. Because for reporting purposes and widgets, sick leave needs to be accounted for as 8 hours.

## Decision
### What to store in startTime and endTime in DB
Store an interval covering full calendar days. For example if sick leave from July 2 to July 4, then:

`start_time = 2026-07-02T00:00:00` and `end_time = 2026-07-05T00:00:00`

### How to get a correct timesheet
Since our database stores a single sick leave record with a period (start and end), we will need to calculate the number of sick leave days, excluding non-working days (for example if the sick leave falls on weekends), and multiply that by 8.

Example: `startTime=2026-07-06T00:00:00` and `endTime=2026-07-11T00:00:00` then `5 (full days) × 8 = 40 hours`

## Consequence
1. Database-level validations will work good and without major issues.
2. For reports, we can assume that one sick day equals 8 hours.