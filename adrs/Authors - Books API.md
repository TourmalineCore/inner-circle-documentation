# How to store book authors in the database?

## Status
Proposed

## Context
We have a book and we have an author/authors of this book. How should we store information about author/authors?

## Alternatives
### One alternative
As string in the “Book” table - store fullnames as a string array.
#### Pros
- There will be no need to implement cascading deletion/editing when deleting/editing a book;
- Few people write more than 1 book from technical authors, a dictionary with authors **for our** library will not be reused often.
#### Cons
- If necessary, it will be more difficult to put the authors in a separate table;
- It will be more difficult to search for books by author;
- If there is a mistake in the author's name, you will have to correct it in each book separately.
### Two alternative
As a different table named “Author” with M:M relation to “Book” table.
#### Pros
- If there is a feature for selecting an author from the list, it will be easier to implement;
- When searching for an author through the search bar, we will not depend on the database provider;
- If you need to correct a mistake in the name of an author who owns several books, just edit his name in the "Author" table.

#### Cons
- It is necessary to correctly implement the deletion of entities in order not to delete books by deleting the author and vice versa;
- It is still unclear whether a filter by the author will be needed;
- The list of authors is not planned to be managed in the first iteration.
## Decision
?