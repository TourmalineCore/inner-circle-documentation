# CRUD operations
All endpoints are closed by authorization.
## Create (or add) a book
- **Url:** /api/books; 
- **Type - POST**; 
- **Headers:** Authorization - Bearer ***
- **Permission:** CanManageBooks

**Request body:**
```json
{
  "title": "Пример названия книги", (required)
  "annotation": "Пример аннотации книги", (required)
  "authors": [ (required, count of values should be more than zero)
    {
      "fullName": "Иванов Иван"
    }
  ],
  "language": "ru", (required)
  "bookCoverUrl": "http://images-for-test.com/book-image.jpg" (optional, default: null)
}
```
**Response body:**
```json
{
  "newBookId": 1
}
```
## Update a book
- **Url:** /api/books/{id}/edit; 
- **Type - POST**; 
- **Headers:** Authorization - Bearer ***
- **Permission:** CanManageBooks

**Request body:**
```json
{
  "title": "Пример обновленного названия книги", (required)
  "annotation": "Пример обновленной аннотации книги", (required)
  "language": "en", (required)
  "authors": [ (required, count of values should be more than zero)
    {
      "fullName": "Иванов Иван"
    },
    {
      "fullName": "Петров Петр"
    }
  ],
  "bookCoverUrl": "http://images-for-test.com/book-image-updated.jpg" (optional, default: null)
}
``` 
**Response body:** 200 OK

## Delete a book (soft)
- **Url:** /api/books/{id}/soft-delete; 
- **Type - DELETE**; 
- **Headers:** Authorization - Bearer ***
- **Permission:** CanManageBooks

**Response body:** 
```json
{
  "isDeleted": true
}
```

## Delete a book (hard)
- **Url:** /api/books/{id}/hard-delete; 
- **Type - DELETE**; 
- **Headers:** Authorization - Bearer ***
- **Permission:** IsBooksHardDeleteAllowed

**Response body:** 
```json
{
  "isDeleted": true
}
```

## Get all of existing books
- **Url:** /api/books; 
- **Type - GET**; 
- **Headers:** Authorization - Bearer ***

**Response body:** 
```json
{
  "books": [
    {
      "id": 1,
      "title": "Пример названия книги",
      "annotation": "Пример аннотации книги",
      "language": "ru",
      "authors": [
        {
          "fullName": "Иванов Иван"
        }
      ],
      "bookCoverUrl": "http://images-for-test.com/book-image.jpg"
    },
    {
      "id": 2,
      "title": "Пример названия книги 2",
      "annotation": "Пример аннотации книги 3",
      "language": "en",
      "authors": [
        {
          "fullName": "Иванов Иван"
        },
        {
          "fullName": "Петров Петр"
        }
      ],
      "bookCoverUrl": "http://images-for-test.com/book-image-2.jpg"
    }
  ]
}
```

## Get existing book by id
- **Url:** /api/books/{id}; 
- **Type - GET**; 
- **Headers:** Authorization - Bearer ***

**Response body:** 
```json
{
  "id": 1,
  "title": "Пример названия книги",
  "annotation": "Пример аннотации книги",
  "language": "ru",
  "authors": [
    {
      "fullName": "Иванов Иван"
    }
  ],
  "bookCoverUrl": "http://images-for-test.com/book-image.jpg"
}
```
