# URL structure 

## Status
Proposed

## Context
We have an API, we need to define the URL of its endpoints. Each service describes an entity/entities and the URL starts with /api/entity-name. In which part of the URL can we specify the id of one of the instances in order to perform deletion/editing operations or other operations related to ONE instance of the entity?

## Alternatives

### One alternative
Specify the id at the end, after the name of the operation performed on the instance. For example: /api/books/edit/{id}
#### Pros
?
#### Cons
?
### Two alternative
Specify the id before the name of the operation performed on the instance. For example: /api/books/{id}/edit
#### Pros
- A more understandable and logical structure. We turn to books, then to a specific copy of the book, then specify the name of the operation that we perform on this particular instance
#### Cons
?
## Decision
We decided to use the second alternative due to the fact that it is more logical and displays the sequence of actions.
