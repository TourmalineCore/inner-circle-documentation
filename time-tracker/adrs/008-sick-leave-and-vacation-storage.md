# 008: Sick Leave And Vacation Storage

## Status
Accepted (2026-07-10)

## Context
It is necessary to make a decision about what time will be stored in `start_time` and` end_time` for the sick leave in Db, and how to ensure that the timesheet matches up. More about sick leave in [DDR](../ddrs/sick-leave.md)

### What does it mean for a timesheet to "match"?
Typically, this means that the total number of hours recorded by an employee on their timesheet equals the number of hours they are required to work under their employment contract (e.g., 8 hours per day or 40 hours per week).

However, the system is not built around a fixed daily rate. Instead, it operates on the basis of a production calendar and formal work schedules. The timesheet must reflect actual working days — the days the employee was scheduled to work according to their personal or company-wide work schedule.

From the total number of working days in a given period, we subtract:

- Days actually worked
- Days on sick leave
- Vacation days

The remaining balance should be zero (or accounted for by other absence types).

For a full‑time employee with a standard 5‑day workweek, a working day equals 8 hours. But this is a configurable setting — the system will support different work schedules (e.g., part‑time, shift work, flexible hours) in the future.

## Decision
### What to store in startTime and endTime in DB
Store an interval covering full calendar days. For example if sick leave from July 2 to July 4, then:

`start_time = 2026-07-02T00:00:00` and `end_time = 2026-07-05T00:00:00`

### How to get a correct timesheet
For a full‑time employee with a standard 5‑day workweek, it will be considered like this, since our database stores a single full day entry record with a period (start and end), we will need to calculate the number of sick leave days, excluding non-working days (for example if the sick leave falls on weekends), and multiply that by 8.

Example: `startTime=2026-07-06T00:00:00` and `endTime=2026-07-11T00:00:00` then `5 (full days) × 8 = 40 hours`

## Consequence
## Advantages
1. Database-level validations will work good and without major issues.
2. Less load on the database
3. Easy removal of a multi‑day event

## Alternatives 

## Store each day as a separate database record

Instead of storing a single interval (e.g., 2026-07-02T00:00:00 – 2026-07-05T00:00:00), we would create one record per calendar day of sick leave.

Example:

Sick leave from July 2 to July 4 -> 3 separate records:

1. 2026-07-02T00:00:00 – 2026-07-03T00:00:00
2. 2026-07-03T00:00:00 – 2026-07-04T00:00:00
3. 2026-07-04T00:00:00 – 2026-07-05T00:00:00

## Advantages
1. Database-level validations will work good and without major issues.
2. It's easier to do calculations for reports

## Disadvantages
1. Complexity of deleting an entire multi‑day event 
2. Increased database overhead (more records in the database)

## Store only dates in a separate table

This is not suitable for us because it violates our validation strategy. Read [Strategy of Validation for Time Intervals Overlapping and Table Structure](./003-time-api-overlap-validation.md)