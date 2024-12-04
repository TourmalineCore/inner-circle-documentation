# CRUD operations

**Create (or add) a book**</br>
**Url:** /api/books/create; **Type - POST**
```json
{
  "title": "5 пороков команды",
  "annotation": "Работа команды в силу своего потенциала и неповторимости является основным звеном деятельности компании",
  "author_id": 1,
  "artworkUrl": "https://cdn.litres.ru/pub/c/cover_415/620095.webp",
  "numberOfCopies": 1,
  "statusId": 1,
  "categoryId": 1,
  "tags": [
    "Менеджмент",
    "Тимбилдинг"
  ],
  "languageId": 1
}
```

**Update a book**</br>
**Url:** /api/books/edit; **Type - PUT**
```json
{
  "id": 1,
  "title": "6 пороков команды",
  "annotation": "Работа команды в силу своего потенциала и неповторимости является основным звеном деятельности компании(изменено)",
  "artworkUrl": "https://ir-2.ozone.ru/s3/multimedia-1-n/wc1000/7094131403.jpg",
  "numberOfCopies": 1,
  "tags": [
    "Тимбилдинг"
  ]
}
```

**Delete a book (soft)**</br>
**Url:** /api/books/soft-delete/{id}; **Type - POST**</br>

**Delete a book (hard)**</br>
**Url:** /api/books/hard-delete/{id}; **Type - POST**</br>

# Searching specific or getting all data
**Get all of existing books:**</br>
**Url:** /api/books/all; **Type - GET**</br>