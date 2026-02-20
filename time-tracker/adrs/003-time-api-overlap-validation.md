# 003: Strategy of Validation for Time Intervals Overlapping and Table Structure

## Status

Accepted (2026-02-13)

## Context

We need to validate the overlaps of time intervals between different types of entries (task, overtime, make-up time, lunch, away, sick leave, day off, unwell, etc.) in the time tracker.

### Requirements:

* Validation must ensure data integrity and consistency.  
* Clear and precise errors must be returned to the client.

### Intersection Rules:

| | Task | Overtime | Make-up time | Lunch | Unwell | Day-off | Late | Sick leave | Vacation | Away |
| :---- | :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: |
| **Task** | ${\color{blue}-}$ | **\+** | **\+** | **\-** | **\-** | **\+** | **\-** | **\+** | **\+** | **\-** |
| **Overtime** | **\+** | ${\color{blue}-}$ | **\-** | **\-** | **\-** | **\+** | **\-** | **\+** | **\+** | **\-** |
| **Make-up time** | **\+** | **\-** | ${\color{blue}-}$| **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** |
| **Lunch** | **\-** | **\-** | **\-** | ${\color{blue}-}$ | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** |
| **Unwell** | **\-** | **\-** | **\-** | **\-** | ${\color{blue}-}$| **\-** | **\-** | **\-** | **\-** | **\-** |
| **Day-off** | **\+** | **\+** | **\-** | **\-** | **\-** | ${\color{blue}-}$| **\-** | **\-** | **\-** | **\-** |
| **Late** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | ${\color{blue}-}$ | **\-** | **\-** | **\-** |
| **Sick leave** | **\+** | **\+** | **\-** | **\-** | **\-** | **\-** | **\-** | ${\color{blue}-}$ | **\-** | **\-** |
| **Vacation** | **\+** | **\+** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | ${\color{blue}-}$ | **\-** |
| **Away** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | ${\color{blue}-}$ |

## Decision

Validate time overlaps only at the database level using constraints. This solves race conditions and data integrity issues. This solution has a technical limitation: range intersection constraints only work within a single table - we cannot add a constraint that will validate overlaps acreoss multiple tables. Initially, we had separate tables for work entries , i.e. tasks, and for adjustments (e.g., overtime, make-up time), so it wouldn't be possible to use range intersection constraints. So we had to unite all entries to a single table using the [TPH (table-per-hierarchy)](https://learn.microsoft.com/en-us/ef/core/modeling/inheritance#table-per-hierarchy-and-discriminator-configuration) inheritance type, which has one base table from which all others inherit.

*Example: Task cannot intersect with Unwell.*

|  | Task | Unwell |
| :---- | :-----: | :-----: |
| **Task** | \- | \- |
| **Unwell** | \- | \- |

```
// Constraint 1: Block overlaps within {type 1, type 2}
ALTER TABLE tracked\_entries

ADD CONSTRAINT exclude\_type12\_overlap

EXCLUDE USING GIST (

    tsrange(start\_time, end\_time, '\[)') WITH &&

)

WHERE (type IN (1, 2));
```

*Example: Task can overlap with Overtime, but Unwell cannot overlap with any other entry.*

|  | Task | Unwell | Overtime |
| :---- | :-----: | :-----: | :-----: |
| **Task** | \- | \- | \+ |
| **Unwell** | \- | \- | \- |
| **Overtime** | \+ | \- | \- |

```
// Constraint 1: Block overlaps within {type 1, type 2}
ALTER TABLE tracked\_entries

ADD CONSTRAINT exclude\_type12\_overlap

EXCLUDE USING GIST (

    tsrange(start\_time, end\_time, '\[)') WITH &&

)

WHERE (type IN (1, 2));

// Constraint 2: Block overlaps within {type 2, type 3}
ALTER TABLE tracked\_entries

ADD CONSTRAINT exclude\_type23\_overlap

EXCLUDE USING GIST (

    tsrange(start\_time, end\_time, '\[)') WITH &&

)

WHERE (type IN (2, 3));
```

### Generating User-Friendly Errors

To generate user-friendly errors, an additional query is made to the database at the code level to obtain information about the conflicting record and pass its type to the error text.

## Consequences

### Positive

* Data integrity is guaranteed at the database level which protects from concurrent writes.  
* No need to create too many constraints.  
* Errors for the client remain flexible and meaningful because we additionally check them in the code.  
* Cross-table overlap checks (which is impossible with constraints) are not required.

### Negative

* All validated records must be stored in a single table.  
* The logic for grouping types in constraints requires careful maintenance.  
* An additional query is added to handle constraint errors.
* It is tricky to test all variations of range overlaps, and the complexity will grow when new types will be added.  

## Alternatives

### Only database constraints without additional processing

Relying solely on database errors without additional analysis.

#### Cons:

* Unable to generate clear error messages for the client.

### Validation at the code level

All overlap checks are performed only in the backend code.

#### Cons:

No protection against concurrent writes.

### Separate record types across multiple tables

Storing different record types in different tables.

#### Cons:

* Unable to validate overlaps between tables using constraints.  
* Data model and queries become more complex.

### Optimistic Locking with Daily Aggregation

Instead of complex database constraints and TPH inheritance, we can fundamentally rethink the storage schema so that a single row stores data for an employee's entire day. Then we can remove the complex jumble of constraints in the database.

#### Cons:

* Cross-day overlap validation will require complex validation logic.
