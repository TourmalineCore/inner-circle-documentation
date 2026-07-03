# 006: Make-Up Entry Relations with Other Entries (Backend)

## Status
Accepted (2026-06-17)

## Context
Make-up entry can be related to several different entries (e.g. Away With Make-Up Time, Day-Off, Late).
That's why we need to find a way how to organize it in the backend models to be able to link a make-up entry to its related entry by id (foreign key).

## Decision 
Add MakeUpTimeList to TrackedEntryBase, which is our shared base class from which all other entries are inherited by principle of Table-Per-Hierarchy (TPH).

```c#
public class TrackedEntryBase : EntityBase, IOwnedByEmployee, ICanBeDeleted
{
    public TrackedEntryBase()
    {
    }

    public long EmployeeId { get; set; }

    public DateTime StartTime { get; set; }

    ...

    public List<MakeUpTimeEntry> MakeUpTimeList { get; set; }
}
```

This way we'll be able to add RelatedEntry to MakeUpTimeEntry:

```c#
using System.ComponentModel.DataAnnotations.Schema;
using Core.Entities;

public class MakeUpTimeEntry : TrackedEntryBase
{
    public MakeUpTimeEntry()
    {
        EntryType = EntryType.MakeUpTime;
    }

    public long RelatedEntryId { get; set; }

    [ForeignKey(nameof(RelatedEntryId))]
    public TrackedEntryBase RelatedEntry { get; set; }
}
```

## Consequences
In this case EFCore will authomatically add **ForeignKey** and this way we will understand to which other entry a MakeUpTimeEntry is related. 

## Advantages
1. We can benefit from EFCore features (built-in mechanism of linking) which will make the work with MakeUpEntry easier - we'll be able to use Include to extract MakeUpTime, ForeignKey will be automatlcally added, cascade deletion will work.
1. Database won't contain any additional columns for all related entry types (see Alternative #1 Disadvantages). And adding new entry types with make-up time will not add such extra columns. 
1. Database will be kept integral - you won't be able to add a MakeUpTimeEntry which is not linked to an existing related entry.

## Disadvantages
TrackedEntryBase is "contaminated" with a field that is not 100% shared between all entries.

## Alternatives
### Alternative 1
Separate ForeignKey for each entry type.

```c#
public class MakeUpTimeEntry : TrackedEntryBase
{
    public MakeUpTimeEntry()
    {
        EntryType = EntryType.MakeUpTime;
    }

    public long AwayWithMakeUpTimeEntryId { get; set; }

    public AwayWithMakeUpTimeEntry AwayWithMakeUpTimeEntry { get; set; }


    public long DayOffToMakeUpForEntryId { get; set; }

    public DayOffToMakeUpForEntry DayOffToMakeUpForEntry { get; set; }
}
```

```c#
public class AwayWithMakeUpTimeEntry : TrackedEntryBase
{
    public AwayWithMakeUpTimeEntry()
    {
        EntryType = EntryType.AwayWithMakeUpTime;
    }

    public string Description { get; set; }

    public List<MakeUpTimeEntry> MakeUpTimeList { get; set; }
}
```

#### Advantages
1. Base class will be kept clean.
1. We can benefit from EFCore features (see above).

#### Disadvantages
1. Database will contain a lot of empty columns. In future if we add new entry types with make-up time, the number of such columns will grow.

### Alternative 2
Manually linking MakeUpTimeEntry to its related entry by NotMapped tag bypassing EFCore features.

```c#
public class MakeUpTimeEntry : TrackedEntryBase
{
    public long RelatedEntryId { get; set; }
}

public class AwayWithMakeUpTimeEntry : TrackedEntryBase
{
    [NotMapped]
    public List<MakeUpTimeEntry> MakeUpTimeList { get; set; }
}
```

#### Advantages
Seemingly most minimalistic approach - no extra columns in DB, no contamination of base class.

#### Disadvantages
1. We won't be able to use EFCore features (see Decision Advantages).
