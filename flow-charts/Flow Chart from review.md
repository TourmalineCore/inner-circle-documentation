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
## Adding a New Author to the Book, delete old one from it, rename old one
```mermaid
graph TD;
    A[While editing a book, add a new author in the form] --> B[Remove the old author];
    B --> C[Enter the new name in the old author's name edit field];
    C --> D[Send request to change the existing author's name];
    D --> E[Finish editing the book by submitting the form];
    E --> F[Call UpdateBookCommand];
    F --> G[Iterate through the new list of authors];
    G --> H{Is the author in the database?};
    H --> |Yes| I[Add to the book's authors list];
    H --> |No| J[Create author, add to database, associate with book];
    I --> K[Save updated book information];
    J --> K[Save updated book information];
```
## Editing Author's Name
```mermaid
graph TD;
    O[Open author name edit on book edit] --> P[Send request to change author name];
    P --> Q[Change author name];
    Q --> R[Save change to the database];
```
# A prototype for creating a book, on which the implemented solution is based
![](createBookModalWindowPrototype.png)

# A prototype for editing a book/authors, on which the implemented solution is based
![](updateBookModalWindowPrototype.png)
