# PostgreSQL CRUD API with Code Examples

## Project Overview

This is a RESTful CRUD API built with Node.js, Express, and PostgreSQL for managing books. Below is a detailed explanation of each component with code examples.

## Code Structure & Examples

### 1. Database Connection (database.js)
```javascript
const { Pool } = require('pg')
 
const pool = new Pool({
  connectionString: process.env.POSTGRES_URL + "?sslmode=require",
})

pool.connect((err) => {
    if (err) throw err
    console.log("Connect to PostgreSQL successfully!")
})
```
This code establishes a connection to PostgreSQL using the connection string from environment variables.

### 2. Express Server Setup (index.js)
```javascript
const express = require("express")
const app = express()

require('dotenv').config()
app.use(express.json())

const bookRouter = require('./routes/book.router')
app.use("/api/books", bookRouter)

app.listen(process.env.PORT, () => console.log("Server is running on port 5000"))
```
This sets up the Express server with JSON parsing middleware and routes configuration.

### 3. API Routes (routes/book.router.js)
```javascript
const express = require("express")
const router = express.Router()

const bookController = require('../controllers/book.controller')

router.get("/", bookController.getAll)                // GET all books
router.get("/:id", bookController.getById)           // GET book by ID
router.get("/:id/notes", bookController.getByNotes)  // GET book notes
router.post("/", bookController.create)              // CREATE new book
router.put("/:id", bookController.updateById)        // UPDATE book
router.delete("/:id", bookController.deleteById)     // DELETE book
```

### 4. Controller Methods (controllers/book.controller.js)

#### Get All Books
```javascript
getAll: async (req, res) => {
    try {
        const result = await db.query('SELECT * FROM books ORDER BY created_at DESC');
        res.json(result.rows);
    } catch (error) {
        res.json({ msg: error.msg })
    }
}
```

#### Get Book by ID
```javascript
getById: async (req, res) => {
    const { id } = req.params;
    try {
        const result = await db.query('SELECT * FROM books WHERE id = $1', [id]);
        if (result.rows.length === 0) {
            return res.status(404).send("Book not found");
        }
        res.json(result.rows[0]);
    } catch (error) {
        res.json({ msg: error.msg })
    }
}
```

#### Create New Book
```javascript
create: async (req, res) => {
    const { title, author, cover_url, rating, notes, isbn, description } = req.body;
    try {
        const result = await db.query(
            'INSERT INTO books (title, author, cover_url, rating, notes, isbn, description) VALUES ($1, $2, $3, $4, $5, $6, $7) RETURNING *',
            [title, author, cover_url, rating, notes, isbn, description]
        );
        res.json(result.rows[0]);
    } catch (err) {
        console.error('Error', err.message);
        res.status(500).send("Server Error");
    }
}
```

## API Endpoints and Usage

### 1. Get All Books
```http
GET /api/books
```
Response:
```json
[
    {
        "id": 1,
        "title": "Book Title",
        "author": "Author Name",
        "cover_url": "http://example.com/cover.jpg",
        "rating": 5,
        "notes": "Great book",
        "isbn": "978-3-16-148410-0",
        "description": "Book description"
    }
]
```

### 2. Create New Book
```http
POST /api/books
Content-Type: application/json

{
    "title": "New Book",
    "author": "Author Name",
    "cover_url": "http://example.com/cover.jpg",
    "rating": 5,
    "notes": "Book notes",
    "isbn": "978-3-16-148410-0",
    "description": "Book description"
}
```

### 3. Update Book Notes
```http
PUT /api/books/1
Content-Type: application/json

{
    "notes": "Updated notes for the book"
}
```

### 4. Delete Book
```http
DELETE /api/books/1
```

## Environment Setup

Create a `.env` file with:
```
POSTGRES_URL=your_postgres_connection_string
PORT=5000
```

## Installation and Running

1. Install dependencies:
```bash
npm install
```

2. Start the server:
```bash
npm start
```

## Project Dependencies
```json
{
  "dependencies": {
    "dotenv": "^16.0.3",
    "express": "^4.18.2",
    "pg": "^8.10.0"
  }
}
```

## Deployment Configuration (vercel.json)
```json
{
    "version": 2,
    "builds": [
      {
        "src": "./index.js",
        "use": "@vercel/node"
      }
    ],
    "routes": [
      {
        "src": "/(.*)",
        "dest": "/"
      }
    ]
}
```