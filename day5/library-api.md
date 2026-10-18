

# Library Books REST API

## Overview

The Library Books API allows clients to retrieve, create, update, and delete books. It also supports searching for books by author.

Base URL: `/api/books`

## Endpoints

### 1. List all books

- **Method:** GET
- **Path:** `/api/books`
- **Description:** Returns a list of all books in the library.
- **Example request body:** None required.
- **Success status:** `200 OK`

### 2. Get one book

- **Method:** GET
- **Path:** `/api/books/:id`
- **Description:** Returns the details of a book with the specified ID.
- **Example request body:** None required.
- **Success status:** `200 OK`

Example request:

`GET /api/books/12`

### 3. Create a book

- **Method:** POST
- **Path:** `/api/books`
- **Description:** Adds a new book to the library.
- **Example request body:**

```json
{
  "title": "Things Fall Apart",
  "author": "Chinua Achebe",
  "publishedYear": 1958
}
```

- **Success status:** `201 Created`

### 4. Update a book

- **Method:** PUT
- **Path:** `/api/books/:id`
- **Description:** Updates the details of an existing book.
- **Example request body:**

```json
{
  "title": "Things Fall Apart",
  "author": "Chinua Achebe",
  "publishedYear": 1958
}
```

- **Success status:** `200 OK`

### 5. Delete a book

- **Method:** DELETE
- **Path:** `/api/books/:id`
- **Description:** Deletes the book with the specified ID.
- **Example request body:** None required.
- **Success status:** `204 No Content`

### 6. List books by author

- **Method:** GET
- **Path:** `/api/books?author=Chinua%20Achebe`
- **Description:** Returns books written by the specified author using a query parameter.
- **Example request body:** None required.
- **Success status:** `200 OK`

## Error Responses

### 400 Bad Request

- **Description:** The request contains invalid data or parameters.
- **Example:** Creating a book without a required title or sending an invalid published year.
- **Example response:**

```json
{
  "error": "Invalid book data"
}
```

### 404 Not Found

- **Description:** The requested resource does not exist.
- **Example:** Requesting `GET /api/books/9999` when no book has that ID.
- **Example response:**

```json
{
  "error": "Book not found"
}
```
