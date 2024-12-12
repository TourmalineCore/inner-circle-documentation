# Enum as string in the database

## Status
Proposed

## Context
We need to specify language for the book, which we will add to the system. We have to decide how we will store this information in the database.

## Alternatives

### One alternative
Create a “Language” table and add a language to the book as language_id.
#### Pros
- Following the third normal db form - if the data of one table changes, the data in the related tables will also change.
#### Cons
- There are no more than 2 languages of books at the moment and in the near;
- Languages do not change, are not edited.
### Two alternative
Use enum for this task.
#### Pros
- No additional actions are required with the database.
#### Cons
- The language is stored not as a value, but as an id, and there is no decryption of this id.
## Decision
We decided to use enum, because we found the way to store enum as its value instead of its id: https://learn.microsoft.com/en-us/ef/core/modeling/value-conversions?tabs=data-annotations
