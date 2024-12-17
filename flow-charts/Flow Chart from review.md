# Flow chart (create book, then edit it by adding new author, then rename author)
## Creating a Book
```mermaid
graph TD;
    A[Send request to create a book with body: title, annotation, authors names list, language, artwork url] --> B[Call CreateBookCommand];
    B --> C[Iterate through the list of authors];
    C --> D{Is the author in the database?};
    D --> |Yes| E[Add author to book's authors list];
    D --> |No| F[Add author to database, save, and add author to book's authors list];
    E --> G[Add book to the database];
    F --> G[Add book to the database];
```
## Adding a New Author to the Book
```mermaid
graph TD;
    H[While editing book, add new author to authors field] --> I[Send request to update the book];
    I --> J[Iterate through the new list of authors];
    J --> K{Is the author in the database?};
    K --> |Yes| L[Add author to book's authors list];
    K --> |No| M[Add author to database, save, and add author to book's authors list];
    L --> N[Save new book data to the database];
    M --> N[Save new book data to the database];
```
## Editing Author's Name
```mermaid
graph TD;
    O[Open author name edit on book edit] --> P[Send request to change author name];
    P --> Q[Change author name];
    Q --> R[Save change to the database];
```
# A prototype for editing a book/authors, on which the implemented solution is based
![](update%20book%20modal.png)
