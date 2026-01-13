# 002: Time Zone

## Status
Accepted (2025-12-18)

## Context
As we are developing a time-tracker for distributed teams, we need to chose a way to store the tracked time and the time of planned activities taking into account employees' time zones.

## Decision
According to the [DDR on timezones](../../time-tracker/ddrs/time-zone.md), the time zone is set and can be changed in an employee's profile.

When an employee tracks time in the time-tracker, we will save the **time without time zone** (i.e., **wall time**, what the clock says) as **timestamp without time zone** in the database. </br>
The user's time zone will be stored in a separate database column, based on what was set in the profile (e.g. "America/New_York"). 

_A **timestamp** is a specific point in time, equal to the number of milliseconds since 1970. It's not tied to a time zone, so if User A from Chelyabinsk logged time into the database at the same time as User B from Moscow (in real time, at the same moment, though it might be 2:00 PM for one and 4:00 AM for the other), the timestamp would be the same._

## Consequences
Storing **timestamp without time zone** will give us an opportunity to convert to UTC or any other time zone of the user and aviod doscrepancies inherent in the alternative solution with **timestamptz** (see below). </br>
PostgreSQL has an **AT TIME ZONE** operator that can convert wall time to UTC and back, knowing the time zone ID, which is exactly what we need, see [here](https://www.enterprisedb.com/postgres-tutorials/postgres-time-zone-explained).

## Alternatives
###  Saving in UTC along with the time zone (timestamptz)
The option with UTC suits well for saving past events - the recent past can safely be saved as UTC.</br>
However, the time-tracker will feature planning future events (adjustments, such as make-up time and time-off), and in this case it's better to store the wall time and time zone separately.</br>
This [article](https://www.creativedeletion.com/2015/03/19/persisting_future_datetimes.html) dwells in detail why it is no good to store in UTC.
In this [article's](https://habr.com/ru/articles/772954) comment section there's a good discussion against this option.

## Consequences
- can cause disrepancies between timesheets and actually tracked time, e.g. a person tracked time in the Moscow time zone, while a manager in the Chelyabinsk time zone sees this as a next day entry.
- a change in time zone rules, e.g. your country stopped using Daylight Saving Time, can cause future entries to shift (e.g. a 1-hour lunch shift for everyone).
- since PostgreSQL takes into account the time zone of the connection, using UTC (timestamptz) with PostgreSQL can cause weird effects - one time will be displayed from DBeaver, another from psql, and a third from the application.

## Other Useful Links

- https://stackoverflow.com/questions/19626177/how-to-store-repeating-dates-keeping-in-mind-daylight-savings-time#19627330
- https://stackoverflow.com/questions/2532729/daylight-saving-time-and-time-zone-best-practices#2532962
- https://stackoverflow.com/questions/19166995/java-calendar-date-and-time-management-for-a-multi-timezone-application/19170823#19170823

## Comparison Table

| Scenario | Wall Time  | Wall Time \+ Timezone | UTC Time | UTC Time \+ Timezone |
| :---- | :----: | :----: | :----: | :----: |
| An employee tracks time \- the manager makes a monthly report. |  | ✅  |  |  |
| An employee adds time-off and make-up time \- the manager checks the employee's working time. |  | ✅ |  |  |
| Today I'm tracking Task A in Chelyabinsk, tomorrow I'm flying to St. Petersburg, changing the time zone in my profile and changing the time of yesterday's Task A. |  | ✅ |  |  |
| Lunch is always at 12, regardless of whether the employee is in Chelyabinsk or on a business trip. |  | ✅ |  |  |
| If make-up time is scheduled for 10 a.m. on Saturday, it should not start at 9 a.m. due to daylight saving time. |  | ✅ |  |  |
| If an employee tracked that they were working at 1 a.m. on January 1st, then this time should be counted as January 1st, even if the manager in St. Petersburg was still working on December 31st at that time. |  | ✅ |  |  |
| We plan only individual make-up time (or other individual adjustments). The case of a pair or mob session will be tracked after the event through a single account, specifying people from both St. Petersburg and Chelyabinsk. They were working during the same time period, from Instant to Instant, but each person's data will be saved in their time zone. |  | ✅ |  |  |
| Two employees were debugging something overnight together. In Chelyabinsk, Employee A worked from 12 a.m. to 2 a.m., and in St. Petersburg, Employee B worked from 10 a.m. to 12 a.m. The accounting will be as follows: Employee A in Chelyabinsk will be logged for the next day's work, and Employee B in St. Petersburg will be logged for the previous day's work. In other words, the work will be recorded and paid for as it was logged. But if you store it in UTC, there will be a problem: you logged it for one day, but got paid for another day/month/year. |  | ✅ |  |  |
| When storing in UTC, it's possible for work to be recorded but not paid. The November timesheet is already closed and sent to the client before the weekend. An employee in Vladivostok tracks the work on December 1st, but due to the time zone difference, this time interval appears to be November for the manager. As a result, this work won't be recorded in November (the timesheet is already closed) or in December. |  | ✅ |  |  |
