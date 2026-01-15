# 002: Time Zone

## Status
Accepted (2025-12-18)

## Context
As we are developing a time-tracker for distributed teams across multiple time zone. We need to store both tracked time (past events) and planned activities (future events) while accommodating:

- employees working in different time zones
- future scheduling (time-off, make-up time)
- time zone changes (travel, relocation)
- possible Daylight Saving Time transitions

## Decision
According to the [DDR on timezones](../../time-tracker/ddrs/time-zone.md), the time zone is set and can be changed in an employee's profile.

When an employee tracks time in the time-tracker, we will save the **time without time zone** (i.e., **wall time**, what the clock says) as **timestamp without time zone** in the database. </br>
The user's time zone will be stored in a separate database column, based on what was set in the profile (e.g. "America/New_York"). 

_A **timestamp** is a specific point in time, equal to the number of milliseconds since 1970. It's not tied to a time zone, so if User A from Chelyabinsk logged time into the database at the same time as User B from Moscow (in real time, at the same moment, though it might be 2:00 PM for one and 4:00 AM for the other), the timestamp would be the same._

### Implementation Details

1. **Database Schema:**
- `start_time` and `end_time` columns will store the wall time
- separate timezone column (e.g., "America/New_York", "Europe/Moscow")

2. **Time Zone Management:**
- time zone is configured in each employee's profile
- can be updated when employees travel or relocate
- uses IANA time zone identifiers (e.g., "Europe/Moscow", not "MSK+3")

3. **Application Logic:**
- employees see and enter time in their local time zone
- managers see time entries converted to their own time zone
- reports aggregate time entries correctly across time zones

## Consequences

**Positive**
- Future Events: Planned activities remain at their intended local times
- Time Zone Changes: Employees can update time zones without corrupting historical data
- DST Safety: Daylight Saving transitions don't shift scheduled events
- Payroll Integrity: Work is recorded and paid according to when it was actually performed
- Manager Reports: Time entries display correctly in each manager's local time zone

**Negative**
- Storage Overhead: Extra column for time zone storage
- Application Logic: Must handle time zone conversions explicitly

Overall, storing **timestamp without time zone** will give us an opportunity to convert to UTC or any other time zone of the user and aviod discrepancies inherent in the alternative solution with **timestamptz** (see below). </br>
PostgreSQL has an **AT TIME ZONE** operator that can convert wall time to UTC and back, knowing the time zone ID, which is exactly what we need, see [here](https://www.enterprisedb.com/postgres-tutorials/postgres-time-zone-explained).


## Alternatives
###  Saving in UTC along with the time zone (timestamptz)
The option with UTC suits well for saving past events - the recent past can safely be saved as UTC.</br>
However, the time-tracker will feature planning future events (adjustments, such as make-up time and time-off), and in this case it's better to store the wall time and time zone separately.</br>
This [article](https://www.creativedeletion.com/2015/03/19/persisting_future_datetimes.html) dwells in detail why it is no good to store in UTC.
In this [article's](https://habr.com/ru/articles/772954) comment section there's a good discussion against this option.

## Consequences
- can cause disrepancies between timesheets and actually tracked time, e.g. a person tracked time in the Moscow time zone, while a manager in the Chelyabinsk time zone sees this as a next day entry.
- a change in time zone rules, e.g. when your country stopped using Daylight Saving Time, can cause future entries to shift (e.g. a 1-hour lunch shift for everyone).
- since PostgreSQL takes into account the time zone of the connection, using UTC (timestamptz) with PostgreSQL can cause weird effects - one time will be displayed from DBeaver, another from psql, and a third from the application.

## Other Useful Links

- https://stackoverflow.com/questions/19626177/how-to-store-repeating-dates-keeping-in-mind-daylight-savings-time#19627330
- https://stackoverflow.com/questions/2532729/daylight-saving-time-and-time-zone-best-practices#2532962
- https://stackoverflow.com/questions/19166995/java-calendar-date-and-time-management-for-a-multi-timezone-application/19170823#19170823

## Comparison Table


| Scenario | Wall Time Only (no TZ) | Wall Time \+ Timezone ✅ | UTC Time Only (timestamptz) | UTC Time \+ Timezone |
| :---- | :---- | :---- | :---- | :---- |
| 1\. **Past Time Tracking & Monthly Reporting**.</br> Employee tracks worked hours; manager makes a monthly report across time zones | ❌ No timezone context → can't correctly aggregate across timezones | ✅ Local times preserved; easy conversion to manager's TZ for reporting | ⚠️ Careful manual conversion to local time is required  | ✅ Can convert to local time for reporting |
| 2\. **Future Time-Off & Make-up Time Scheduling**.</br> Employee schedules future absence; manager needs to know local work hours | ❌ Without TZ future times are ambiguous  | ✅ Future local times preserved regardless of DST changes | ❌ Future events can shift when DST rules change | ❌ Future events can shift when DST rules change |
| 3\. **Employee Time Zone Change**.</br> Employee travels, updates TZ in profile, needs to adjust yesterday's entry | ❌ Can't distinguish between  original and current TZ | ✅ Historical entries keep original TZ; new entries use new TZ | ❌ All time entries appear wrong after TZ change | ⚠️ Can preserve original TZ but complex to manage |
| 4\. **Fixed Local Time Events**.</br> "Lunch at 12 PM" means local noon, regardless of location | ❌ No TZ → "12 PM" is meaningless | ✅ 12 PM local time preserved in each TZ | ❌ UTC noon shifts with location/DST | ⚠️ Can store but requires conversion |
| 5\. **DST-Proof Scheduling**.</br> Make-up time at 10 AM Saturday shouldn't become 9 AM after DST transition | ❌ No TZ → can't handle DST | ✅ Local time preserved; DST handled by TZ database | ❌ UTC fixed → local time shifts with DST | ❌ UTC fixed → local time still can shift if TZ rules change |
| 6\. **Local Date Boundary Integrity**.</br> Work at 1 AM Jan 1st local time counts as Jan 1st, even if manager sees Dec 31st | ❌ Date depends on viewer's TZ | ✅ Date determined by employee's local time | ❌ Date depends on conversion TZ | ✅ Can determine local date with TZ |
| 7\. **Cross Time Zone Team Collaboration**.</br> A pair programming session with two people in different time zones working simultaneously. After the event, a single account logs the session, specifying participants from both St. Petersburg and Chelyabinsk. | ❌ Can't represent different TZs | ✅ Each entry has its own local time \+ TZ. Each participant gets their OWN local time entry | ❌ Can't represent different TZs | ⚠️ One UTC period with multiple TZ can be tricky:</br> 1\. Individual time adjustments affect all</br> 2\. Local dates must be calculated</br> 3\. Confusion between "instant" vs "local experience"</br> 4\. Payroll assignment requires conversion |
| 8\. **Overnight Work & Payroll Alignment**.</br> Employees in different TZs work same instant; payroll aligns with local dates | ❌ Date ambiguity causes payroll errors | ✅ Local dates preserved → correct payroll  | ❌ UTC may place work in wrong pay period | ✅ Can calculate local dates for payroll |
| 9\. **Time Sheet Closure Edge Case**.</br> Work logged after the monthly timesheet is closed, but appears before due to TZ difference. Such work can be recorded but not paid. | ❌ Unresolvable without TZ | ✅ Local date determines timesheet assignment | ❌ May assign to closed timesheet incorrectly | ✅ Can assign to correct timesheet with TZ |
| 10\. **Historical Data After TZ Rule Changes**.</br> Country changes DST rules; historical entries should remain correct | ❌ Times become ambiguous | ✅ Original local times are preserved  | ❌ Historical local times recalculated incorrectly | ⚠️ Can preserve but requires TZ versioning |

### Critical Scenarios Where Only Wall Time \+ TZ Works:

1. **Future Scheduling**: Only this approach guarantees DST-proof future events  
2. **Payroll Compliance**: Legal requirement to pay based on when work was performed locally  
3. **Cross Time Zone Team Collaboration**: Each team member sees times in their local context
