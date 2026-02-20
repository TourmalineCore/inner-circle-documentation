# 002: Strategy of Validation for Time Intervals Overlapping and Table Structure

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
| **Task** | <span style="background-color: #e6ffe6; padding: 5px 10px; display: inline-block;">**\-**</span> | **\+** | **\+** | **\-** | **\-** | **\+** | **\-** | **\+** | **\+** | **\-** |
| **Overtime** | **\+** | <span style="background-color: #e6ffe6; padding: 5px 10px; display: inline-block;">**\-**</span> | **\-** | **\-** | **\-** | **\+** | **\-** | **\+** | **\+** | **\-** |
| **Make-up time** | **\+** | **\-** | <span style="background-color: #e6ffe6; padding: 5px 10px; display: inline-block;">**\-**</span> | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** |
| **Lunch** | **\-** | **\-** | **\-** | <span style="background-color: #e6ffe6; padding: 5px 10px; display: inline-block;">**\-**</span> | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** |
| **Unwell** | **\-** | **\-** | **\-** | **\-** | <span style="background-color: #e6ffe6; padding: 5px 10px; display: inline-block;">**\-**</span> | **\-** | **\-** | **\-** | **\-** | **\-** |
| **Day-off** | **\+** | **\+** | **\-** | **\-** | **\-** | <span style="background-color: #e6ffe6; padding: 5px 10px; display: inline-block;">**\-**</span> | **\-** | **\-** | **\-** | **\-** |
| **Late** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | <span style="background-color: #e6ffe6; padding: 5px 10px; display: inline-block;">**\-**</span> | **\-** | **\-** | **\-** |
| **Sick leave** | **\+** | **\+** | **\-** | **\-** | **\-** | **\-** | **\-** | <span style="background-color: #e6ffe6; padding: 5px 10px; display: inline-block;">**\-**</span> | **\-** | **\-** |
| **Vacation** | **\+** | **\+** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | <span style="background-color: #e6ffe6; padding: 5px 10px; display: inline-block;">**\-**</span> | **\-** |
| **Away** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | **\-** | <span style="background-color: #e6ffe6; padding: 5px 10px; display: inline-block;">**\-**</span> |

## Decision

Validate time overlaps only at the database level using constraints. This solves race conditions and data integrity issues. This solution has a technical limitation: range intersection constraints only work within a single table. To achieve this behavior, we used the  [TPH (table-per-hierarchy)](https://learn.microsoft.com/en-us/ef/core/modeling/inheritance#table-per-hierarchy-and-discriminator-configuration) inheritance type, which has one base table from which all others inherit.

*Example: Task cannot intersect with Unwell.*

|  | Task | Unwell |
| :---- | :-----: | :-----: |
| **Task** | \- | \- |
| **Unwell** | \- | \- |

```
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
ALTER TABLE tracked\_entries

ADD CONSTRAINT exclude\_type12\_overlap

EXCLUDE USING GIST (

    tsrange(start\_time, end\_time, '\[)') WITH &&

)

WHERE (type IN (1, 2));

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
