# Report: Time Tracking Calculation Error For Invoices

## Context
After the implementation of the new logic for calculating tracked task hours, there were discrepancies in the invoices service for April. For example, the tracked time of the user (Yulia) changed from 7.16 hours (confirmed by manual calculation) to 7 hours in the system.

## Cause of error
```c#
// Old code - causes data loss
TrackedHours = TrackedMinutes / 60  // int / int = int (decimal part discarded)
```

When converting minutes to hours, both operands were integers, causing C# to perform integer division (floor truncation).

For example: 430 minutes = 7.16 hours → integer division gave 7 hours (lost 0.16 hours)

## Decision
Changed the data type from int to decimal throughout the calculation chain to preserve fractional minutes.

```c#
// Old code 
public int GetDurationInMinutes()
{
    return (int)(EndTime - StartTime).TotalMinutes;
}
```

```c#
// New code
public decimal GetDurationInMinutes()
{
    return (decimal)(EndTime - StartTime).TotalMinutes;
}
```