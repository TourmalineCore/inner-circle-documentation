# Book Contract ADR

## Status

Proposed

## Context

In books service, we are going to store information about books that are in the office. We have several copies of several books. We need to store information about the number of copies of books in general and about copies that are taken by employees or available in the office.

We need to discuss get book by id request contract: /api/books/{id}

## Alternatives 

### 1. We get the number of a book copies and their bookCopiesIds when open BookPage

```ts
// GET /api/books/{id}
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
  bookCoverUrl: ``,
  bookCopies: [ // bookCopiesIds
    {
      bookCopyId: 11,
    },
    {
      bookCopyId: 12,
    },
    {
      bookCopyId: 13,
    },
    {
      bookCopyId: 14,
    },
    {
      bookCopyId: 15,
    },
  ],
}
```

#### Pros

1. There will be ONE request for get data
2. Number of a book copies we count on frontend using bookCopies array

#### Cons

1. The bookCopiesIds array will not always be used when opening a page, but only when clicking a button that is unlikely to be clicked often, so is this data needed there specifically here?

### 3. We get bookCopiesIds only if it needs

```ts
// GET /api/books/{id}
{
  id: 1,
  title: `ChatGPT мастер подсказок или как создавать сильные промты  для нейросети`,
  annotation: `annotation`,
  count: 5, // number of a book copies
  language: `ru`,
  authors: [
    {
      fullName: `authors`, 
    },
  ],
  bookCoverUrl: ``,
}

// GET /api/books/{id}/copies (for ex)
{
  bookId: 1,
  bookCopies: [
    {
      bookCopyId: 11,
    },
    {
      bookCopyId: 12,
    },
    {
      bookCopyId: 13,
    },
    {
      bookCopyId: 14,
    },
    {
      bookCopyId: 15,
    },
  ],
}
```

#### Pros

1. The bookCopiesIds array will be request only when if will be need (only when we open modal qr form)

#### Cons

1. There will be TWO requests for get data
2. We need to go into the second table to get number of a book copies and we do it in two requests (get book and get copies)

## Decision

When we open BookPage in the first case we make only 1 request to the second table, and in the 2nd case 2 requests for each get request, so both options have the same performance when opening the page but the second option is inferior when opening a modal window with copies of books

Notes:

- qr has url https://innercircle.tourmalinecore.com/books/79?bookCopyId=8 or we can shorten because the shorter the url the more grainy the QR code is and in future we can make very short url and add redirects

- our BookPage has url https://innercircle.tourmalinecore.com/books/79

- our special book copy page has url https://innercircle.tourmalinecore.com/books/79?bookCopyId=8
