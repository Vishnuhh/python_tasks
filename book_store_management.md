# Book Management API Documentation

This document provides comprehensive details for all API endpoints in the Book Management system, including example request payloads and expected responses.

## Table of Contents
- [Books API](#books-api)
- [Authors API](#authors-api)
- [Users API](#users-api)
- [Reviews API](#reviews-api)
- [Categories API](#categories-api)

## Books API

### Retrieve Books
```
GET /books/
```
Retrieve a list of all books.

**Query Parameters:**
- `limit` (optional): Maximum number of books to return (default: 20)
- `offset` (optional): Number of books to skip (default: 0)
- `author_id` (optional): Filter books by author ID
- `category_id` (optional): Filter books by category ID

**Example Request:**
```
GET /books/?limit=10&offset=0&author_id=123
```

**Example Response:**
```json
{
  "count": 42,
  "next": "/books/?limit=10&offset=10&author_id=123",
  "previous": null,
  "results": [
    {
      "id": 1,
      "title": "The Great Gatsby",
      "isbn": "9780743273565",
      "publication_date": "2004-09-30",
      "description": "A story of wealth, love, and the American Dream in the 1920s.",
      "page_count": 180,
      "language": "en",
      "author": {
        "id": 123,
        "name": "F. Scott Fitzgerald"
      },
      "categories": [
        {
          "id": 1,
          "name": "Fiction"
        },
        {
          "id": 2,
          "name": "Classics"
        }
      ],
      "average_rating": 4.2,
      "created_at": "2023-01-15T12:00:00Z",
      "updated_at": "2023-01-15T12:00:00Z"
    },
    // More books...
  ]
}
```

### Create Book
```
POST /books/
```
Create a new book.

**Request Body:**
```json
{
  "title": "1984",
  "isbn": "9780451524935",
  "publication_date": "1949-06-08",
  "description": "A dystopian novel about totalitarianism.",
  "page_count": 328,
  "language": "en",
  "author_id": 456,
  "category_ids": [1, 3]
}
```

**Example Response:**
```json
{
  "id": 2,
  "title": "1984",
  "isbn": "9780451524935",
  "publication_date": "1949-06-08",
  "description": "A dystopian novel about totalitarianism.",
  "page_count": 328,
  "language": "en",
  "author": {
    "id": 456,
    "name": "George Orwell"
  },
  "categories": [
    {
      "id": 1,
      "name": "Fiction"
    },
    {
      "id": 3,
      "name": "Science Fiction"
    }
  ],
  "average_rating": 0,
  "created_at": "2023-03-12T08:30:00Z",
  "updated_at": "2023-03-12T08:30:00Z"
}
```

**Error Response:**
```json
{
  "error": "Bad Request",
  "message": "Invalid input data",
  "details": {
    "isbn": ["ISBN format is invalid"],
    "author_id": ["Author with ID 456 does not exist"]
  }
}
```

### Retrieve Book
```
GET /books/{book_id}
```
Retrieve details of a specific book.

**Example Request:**
```
GET /books/1
```

**Example Response:**
```json
{
  "id": 1,
  "title": "The Great Gatsby",
  "isbn": "9780743273565",
  "publication_date": "2004-09-30",
  "description": "A story of wealth, love, and the American Dream in the 1920s.",
  "page_count": 180,
  "language": "en",
  "author": {
    "id": 123,
    "name": "F. Scott Fitzgerald"
  },
  "categories": [
    {
      "id": 1,
      "name": "Fiction"
    },
    {
      "id": 2,
      "name": "Classics"
    }
  ],
  "average_rating": 4.2,
  "created_at": "2023-01-15T12:00:00Z",
  "updated_at": "2023-01-15T12:00:00Z"
}
```

### Update Book
```
PUT /books/{book_id}
```
Update details of a specific book.

**Example Request:**
```
PUT /books/1
```

**Request Body:**
```json
{
  "title": "The Great Gatsby",
  "isbn": "9780743273565",
  "publication_date": "2004-09-30",
  "description": "Updated description about wealth and the American Dream.",
  "page_count": 180,
  "language": "en",
  "author_id": 123,
  "category_ids": [1, 2, 5]
}
```

**Example Response:**
```json
{
  "id": 1,
  "title": "The Great Gatsby",
  "isbn": "9780743273565",
  "publication_date": "2004-09-30",
  "description": "Updated description about wealth and the American Dream.",
  "page_count": 180,
  "language": "en",
  "author": {
    "id": 123,
    "name": "F. Scott Fitzgerald"
  },
  "categories": [
    {
      "id": 1,
      "name": "Fiction"
    },
    {
      "id": 2,
      "name": "Classics"
    },
    {
      "id": 5,
      "name": "Literary Fiction"
    }
  ],
  "average_rating": 4.2,
  "created_at": "2023-01-15T12:00:00Z",
  "updated_at": "2023-03-18T15:42:30Z"
}
```

## Authors API

### Retrieve Authors
```
GET /authors/
```
Retrieve a list of authors.

**Query Parameters:**
- `limit` (optional): Maximum number of authors to return (default: 20)
- `offset` (optional): Number of authors to skip (default: 0)
- `search` (optional): Search term to filter author names

**Example Request:**
```
GET /authors/?limit=10&search=tolkien
```

**Example Response:**
```json
{
  "count": 1,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 789,
      "name": "J.R.R. Tolkien",
      "biography": "John Ronald Reuel Tolkien was an English writer, poet, philologist, and academic.",
      "birth_date": "1892-01-03",
      "death_date": "1973-09-02",
      "country": "United Kingdom",
      "book_count": 12,
      "created_at": "2023-01-10T09:15:00Z",
      "updated_at": "2023-01-10T09:15:00Z"
    }
  ]
}
```

### Create Author
```
POST /authors/
```
Create a new author.

**Request Body:**
```json
{
  "name": "Stephen King",
  "biography": "Stephen Edwin King is an American author of horror, supernatural fiction, suspense, crime, science-fiction, and fantasy novels.",
  "birth_date": "1947-09-21",
  "country": "United States"
}
```

**Example Response:**
```json
{
  "id": 321,
  "name": "Stephen King",
  "biography": "Stephen Edwin King is an American author of horror, supernatural fiction, suspense, crime, science-fiction, and fantasy novels.",
  "birth_date": "1947-09-21",
  "death_date": null,
  "country": "United States",
  "book_count": 0,
  "created_at": "2023-03-18T16:20:00Z",
  "updated_at": "2023-03-18T16:20:00Z"
}
```

### Retrieve Author
```
GET /authors/{author_id}
```
Retrieve details of a specific author.

**Example Request:**
```
GET /authors/789
```

**Example Response:**
```json
{
  "id": 789,
  "name": "J.R.R. Tolkien",
  "biography": "John Ronald Reuel Tolkien was an English writer, poet, philologist, and academic.",
  "birth_date": "1892-01-03",
  "death_date": "1973-09-02",
  "country": "United Kingdom",
  "books": [
    {
      "id": 10,
      "title": "The Hobbit",
      "isbn": "9780547928227",
      "publication_date": "1937-09-21"
    },
    {
      "id": 11,
      "title": "The Fellowship of the Ring",
      "isbn": "9780547928210",
      "publication_date": "1954-07-29"
    }
    // More books...
  ],
  "created_at": "2023-01-10T09:15:00Z",
  "updated_at": "2023-01-10T09:15:00Z"
}
```

### Update Author
```
PUT /authors/{author_id}
```
Update details of a specific author.

**Example Request:**
```
PUT /authors/789
```

**Request Body:**
```json
{
  "name": "J.R.R. Tolkien",
  "biography": "Updated biography for J.R.R. Tolkien, creator of Middle-earth.",
  "birth_date": "1892-01-03",
  "death_date": "1973-09-02",
  "country": "United Kingdom"
}
```

**Example Response:**
```json
{
  "id": 789,
  "name": "J.R.R. Tolkien",
  "biography": "Updated biography for J.R.R. Tolkien, creator of Middle-earth.",
  "birth_date": "1892-01-03",
  "death_date": "1973-09-02",
  "country": "United Kingdom",
  "book_count": 12,
  "created_at": "2023-01-10T09:15:00Z",
  "updated_at": "2023-03-18T16:45:10Z"
}
```

## Users API

### Retrieve Users
```
GET /users/
```
Retrieve a list of users.

**Query Parameters:**
- `limit` (optional): Maximum number of users to return (default: 20)
- `offset` (optional): Number of users to skip (default: 0)
- `search` (optional): Search term to filter usernames or email addresses

**Example Request:**
```
GET /users/?limit=10
```

**Example Response:**
```json
{
  "count": 125,
  "next": "/users/?limit=10&offset=10",
  "previous": null,
  "results": [
    {
      "id": 1,
      "username": "bookworm42",
      "email": "user1@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "created_at": "2022-10-05T12:30:00Z",
      "updated_at": "2022-10-05T12:30:00Z",
      "review_count": 15
    },
    // More users...
  ]
}
```

### Create User
```
POST /users/
```
Create a new user.

**Request Body:**
```json
{
  "username": "booklover99",
  "email": "newuser@example.com",
  "password": "securePassword123",
  "first_name": "John",
  "last_name": "Doe"
}
```

**Example Response:**
```json
{
  "id": 126,
  "username": "booklover99",
  "email": "newuser@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "created_at": "2023-03-18T16:52:00Z",
  "updated_at": "2023-03-18T16:52:00Z",
  "review_count": 0
}
```

### Retrieve User
```
GET /users/{user_id}
```
Retrieve details of a specific user.

**Example Request:**
```
GET /users/1
```

**Example Response:**
```json
{
  "id": 1,
  "username": "bookworm42",
  "email": "user1@example.com",
  "first_name": "Jane",
  "last_name": "Smith",
  "created_at": "2022-10-05T12:30:00Z",
  "updated_at": "2022-10-05T12:30:00Z",
  "review_count": 15,
  "recent_reviews": [
    {
      "id": 201,
      "book": {
        "id": 1,
        "title": "The Great Gatsby"
      },
      "rating": 5,
      "created_at": "2023-02-15T10:20:00Z"
    },
    // More reviews...
  ]
}
```

### Update User
```
PUT /users/{user_id}
```
Update details of a specific user.

**Example Request:**
```
PUT /users/1
```

**Request Body:**
```json
{
  "username": "bookworm42",
  "email": "updated.email@example.com",
  "first_name": "Jane",
  "last_name": "Smith-Johnson"
}
```

**Example Response:**
```json
{
  "id": 1,
  "username": "bookworm42",
  "email": "updated.email@example.com",
  "first_name": "Jane",
  "last_name": "Smith-Johnson",
  "created_at": "2022-10-05T12:30:00Z",
  "updated_at": "2023-03-18T17:00:30Z",
  "review_count": 15
}
```

## Reviews API

### Retrieve Reviews
```
GET /books/{book_id}/reviews
```
Retrieve a list of reviews for a specific book.

**Query Parameters:**
- `limit` (optional): Maximum number of reviews to return (default: 20)
- `offset` (optional): Number of reviews to skip (default: 0)
- `rating` (optional): Filter by rating (1-5)
- `order_by` (optional): Sort by "newest" or "highest_rated" (default: "newest")

**Example Request:**
```
GET /books/1/reviews?limit=10&rating=5
```

**Example Response:**
```json
{
  "count": 8,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 201,
      "book_id": 1,
      "user": {
        "id": 1,
        "username": "bookworm42"
      },
      "rating": 5,
      "title": "A masterpiece!",
      "content": "This book perfectly captures the essence of the Roaring Twenties.",
      "created_at": "2023-02-15T10:20:00Z",
      "updated_at": "2023-02-15T10:20:00Z"
    },
    // More reviews...
  ]
}
```

### Create Review
```
POST /books/{book_id}/reviews
```
Create a new review for a specific book.

**Example Request:**
```
POST /books/2/reviews
```

**Request Body:**
```json
{
  "rating": 4,
  "title": "Thought-provoking classic",
  "content": "Orwell's predictions about surveillance society are eerily prescient."
}
```

**Example Response:**
```json
{
  "id": 305,
  "book_id": 2,
  "user": {
    "id": 126,
    "username": "booklover99"
  },
  "rating": 4,
  "title": "Thought-provoking classic",
  "content": "Orwell's predictions about surveillance society are eerily prescient.",
  "created_at": "2023-03-18T17:10:00Z",
  "updated_at": "2023-03-18T17:10:00Z"
}
```

### Retrieve Review
```
GET /books/{book_id}/reviews/{review_id}
```
Retrieve details of a specific review.

**Example Request:**
```
GET /books/1/reviews/201
```

**Example Response:**
```json
{
  "id": 201,
  "book_id": 1,
  "user": {
    "id": 1,
    "username": "bookworm42",
    "first_name": "Jane",
    "last_name": "Smith-Johnson"
  },
  "rating": 5,
  "title": "A masterpiece!",
  "content": "This book perfectly captures the essence of the Roaring Twenties.",
  "created_at": "2023-02-15T10:20:00Z",
  "updated_at": "2023-02-15T10:20:00Z"
}
```

### Update Review
```
PUT /books/{book_id}/reviews/{review_id}
```
Update a specific review. Only the user who created the review can update it.

**Example Request:**
```
PUT /books/1/reviews/201
```

**Request Body:**
```json
{
  "rating": 5,
  "title": "A timeless masterpiece!",
  "content": "Updated review: This book brilliantly captures the essence of the American Dream during the Roaring Twenties."
}
```

**Example Response:**
```json
{
  "id": 201,
  "book_id": 1,
  "user": {
    "id": 1,
    "username": "bookworm42"
  },
  "rating": 5,
  "title": "A timeless masterpiece!",
  "content": "Updated review: This book brilliantly captures the essence of the American Dream during the Roaring Twenties.",
  "created_at": "2023-02-15T10:20:00Z",
  "updated_at": "2023-03-18T17:25:45Z"
}
```

## Categories API

### Retrieve Categories
```
GET /categories/
```
Retrieve a list of book categories.

**Query Parameters:**
- `limit` (optional): Maximum number of categories to return (default: 50)
- `offset` (optional): Number of categories to skip (default: 0)

**Example Request:**
```
GET /categories/
```

**Example Response:**
```json
{
  "count": 10,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "Fiction",
      "description": "Literary works created from the imagination",
      "book_count": 156,
      "created_at": "2022-09-01T08:00:00Z",
      "updated_at": "2022-09-01T08:00:00Z"
    },
    {
      "id": 2,
      "name": "Classics",
      "description": "Books that have stood the test of time",
      "book_count": 78,
      "created_at": "2022-09-01T08:00:00Z",
      "updated_at": "2022-09-01T08:00:00Z"
    },
    // More categories...
  ]
}
```

### Create Category
```
POST /categories/
```
Create a new category. Restricted to admin users.

**Request Body:**
```json
{
  "name": "Historical Fiction",
  "description": "Fictional stories set in the past that often incorporate real historical events"
}
```

**Example Response:**
```json
{
  "id": 11,
  "name": "Historical Fiction",
  "description": "Fictional stories set in the past that often incorporate real historical events",
  "book_count": 0,
  "created_at": "2023-03-18T17:35:00Z",
  "updated_at": "2023-03-18T17:35:00Z"
}
```

### Retrieve Category
```
GET /categories/{category_id}
```
Retrieve details of a specific category.

**Example Request:**
```
GET /categories/1
```

**Example Response:**
```json
{
  "id": 1,
  "name": "Fiction",
  "description": "Literary works created from the imagination",
  "book_count": 156,
  "top_books": [
    {
      "id": 1,
      "title": "The Great Gatsby",
      "author": {
        "id": 123,
        "name": "F. Scott Fitzgerald"
      },
      "average_rating": 4.2
    },
    // More books...
  ],
  "created_at": "2022-09-01T08:00:00Z",
  "updated_at": "2022-09-01T08:00:00Z"
}
```

### Update Category
```
PUT /categories/{category_id}
```
Update a specific category. Restricted to admin users.

**Example Request:**
```
PUT /categories/1
```

**Request Body:**
```json
{
  "name": "Fiction",
  "description": "Updated description: Literary works created from the imagination, including novels, short stories, and plays."
}
```

**Example Response:**
```json
{
  "id": 1,
  "name": "Fiction",
  "description": "Updated description: Literary works created from the imagination, including novels, short stories, and plays.",
  "book_count": 156,
  "created_at": "2022-09-01T08:00:00Z",
  "updated_at": "2023-03-18T17:42:15Z"
}
```

## Error Handling

All endpoints follow a consistent error response format:

**400 Bad Request**
```json
{
  "error": "Bad Request",
  "message": "Invalid input data",
  "details": {
    "field_name": ["Error description"]
  }
}
```

**401 Unauthorized**
```json
{
  "error": "Unauthorized",
  "message": "Authentication credentials were not provided or are invalid"
}
```

**403 Forbidden**
```json
{
  "error": "Forbidden",
  "message": "You do not have permission to perform this action"
}
```

**404 Not Found**
```json
{
  "error": "Not Found",
  "message": "The requested resource was not found"
}
```

**500 Internal Server Error**
```json
{
  "error": "Internal Server Error",
  "message": "An unexpected error occurred"
}
```

## Authentication

The API uses JWT (JSON Web Token) authentication.

**Getting an access token:**
```
POST /auth/token/
```

**Request Body:**
```json
{
  "username": "booklover99",
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

Include the access token in the Authorization header for protected endpoints:
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```
