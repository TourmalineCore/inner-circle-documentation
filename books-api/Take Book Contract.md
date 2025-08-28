# Take Book Contract

### Take book request - POST /api/books/take

Request body:

```ts
{
  bookCopyId: 1,
  scheduledReturnDate: `2025-11-22`
}
```

Response - void

### Get book by id request - GET /api/books/{id}

Now getting a book looks like this

```ts
{
  id: 1,
  title: `ChatGPT мастер подсказок или как создавать сильные промты  для нейросети`,
  annotation: `annotation`,
  language: `ru`,
  authors: [
    {
      fullName: `authors`, 
    },
  ],
  coverUrl: ``,
  bookCopiesIds: [ 
    11, 
    12,
  ],
}
```

But after adding take book flow, the page also needs to display who has the book in their hands (now it will be list of fullNames), and if the user who has the book in their hands has visited the book page, this also needs to be understood

```ts
{
  id: 1,
  title: `ChatGPT мастер подсказок или как создавать сильные промты  для нейросети`,
  annotation: `annotation`,
  language: `ru`,
  authors: [
    {
      fullName: `authors`, 
    },
  ],
  coverUrl: ``,
  bookCopiesIds: [ 
    11, 
    12, 
  ],
  readers: [
    {
      employeeId: 1,
      fullName: "Ivanov Ivan", // for now to display who has the book in their hands
      // photo: // later to display who has the book in their hands instead of fullName
    },
  ],
}
```

### Get employees by ids internal request - POST {EmployeesServiceUrl}/internal/get-employees-by-ids

Request body:

```ts
{
    employeesIds: [
      21, 
      32
    ]
}
```
 
Response:

```ts
{
    employees: [
        {
            employeeId: 21,
            fullName: "Ivanov Ivan"
        },
        {
            employeeId: 32,
            fullName: "Petrov Petr"
        },
    ]
}
```

### Sequence Diagram for take book contract

```mermaid
sequenceDiagram
    actor User
    participant BookUI
    participant BookAPI
    participant EmployeesAPI

    User->>BookUI: 1. visit book copy page after scan qr code <br> /books/1?copyId=11
    BookUI->>BookAPI: 2. get book by id request <br> GET /api/books/1
    BookAPI->>EmployeesAPI: 3. get employees by ids internal request <br> POST {EmployeesServiceUrl}/internal/get-employees-by-ids
    EmployeesAPI->>BookAPI: 4. return employees data
    BookAPI->>BookUI: 5. return book data
    BookUI->>User: 6. render book page with book data
    User->>BookUI: 7. take book using "Take Book" button
    BookUI->>BookAPI: 8. take book request <br> POST /api/books/take
    BookAPI->>BookUI: 9. success take book response
    BookUI->>BookAPI: 10. get book by id request to update book data on the page <br> (because of the addition of a new reader) <br> GET /api/books/1
    BookAPI->>EmployeesAPI: 11. get employees by ids internal request <br> POST {EmployeesServiceUrl}/internal/get-employees-by-ids
    EmployeesAPI->>BookAPI: 12. return employees data
    BookAPI->>BookUI: 13. return book data with new reader data
    BookUI->>User: 14. reload book page after taking book
```
