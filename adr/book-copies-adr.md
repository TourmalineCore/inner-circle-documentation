# Book copies
## Status
Proposed

## Context
In the books api service, we are going to store information about books that are in the office. We have several copies of several books. We need to store information about the number of copies of books in general and about copies that are not in the hands of an employee (that is, the current number of copies of the book available for reading).

## Alternatives
### Store all books in the database in separate rows - when we add two or more instances, we store each in a separate row of the table.
#### Pros
1. Fewer columns in a row with a book;
2. Simpler logic of the "take a book" function.
#### Cons
1. Duplicate lines;
2. Complicating the request to get a list of all books (you need to use a function that excludes copies of lines);
3. If we want to add two copies of the book, we will have to fill out the "Add a book" form twice or add a field with the number of copies on the frontend and send a request to the backend several times.

### Add the "total number of instances" and "current number of instances" fields - store books in one row, and manage instances in the row fields of this book.
#### Pros
1. No line duplication;
2. There is no need to exclude duplicate rows when getting a full list of books.
3. If we suddenly decide to change the decision to the first one, it will be easier to remove the field than to add it and update the data about the copies 
#### Cons
1. Additional columns for the book row;
2. For a small number of books, this alternative seems more complicated.

## Decision