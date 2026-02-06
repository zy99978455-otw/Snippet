# Snippet

Snippet is a text-sharing web application built with Go.

This project aims to demonstrate core concepts in Go web development, including routing, middleware usage, database integration, HTML template rendering, form processing, and session management.

## ✨ Features

* **Publish Snippet**: Users can create text snippets with a title, content, and expiration time.
* **View Snippet**: View published content via its ID.
* **Data Persistence**: Uses MySQL for all data storage.
* **Middleware Architecture**: Uses `alice` to manage middleware chains (Panic recovery, request logging, security headers).
* **Form Processing**:
    * Automatic decoding of HTML form data into Go structs (`go-playground/form`).
    * Custom form validation logic (`validator` package).
* **Session Management**:
    * MySQL-backed server-side session storage (`alexedwards/scs`).
    * Support for Flash Messages (one-time notifications).
* **Dynamic HTML**: Rendered using the Go standard library `html/template`, featuring common layouts and partial templates.
* **Static File Serving**: Handling CSS, JS, and image resources.

## 🛠️ Tech Stack

* **Language**: Go (Golang)
* **Database**: MySQL
* **Key Dependencies**:
    * [`github.com/go-sql-driver/mysql`](https://github.com/go-sql-driver/mysql): MySQL driver
    * [`github.com/justinas/alice`](https://github.com/justinas/alice): Middleware chaining
    * [`github.com/go-playground/form`](https://github.com/go-playground/form): Form decoding
    * [`github.com/alexedwards/scs/v2`](https://github.com/alexedwards/scs): Session management

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone [https://github.com/zy99978455-otw/Snippet.git](https://github.com/zy99978455-otw/Snippet.git)
cd Snippet
```

### 2. Database Setup
You need to install MySQL and create a database named snippetbox. Then run the following SQL script to create the necessary tables (snippets and sessions):

```sql
-- Switch database
USE snippetbox;

-- 1. Create snippets table
CREATE TABLE snippets (
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    content TEXT NOT NULL,
    created DATETIME NOT NULL,
    expires DATETIME NOT NULL
);

CREATE INDEX idx_snippets_created ON snippets(created);

-- 2. Add some initial dummy data
INSERT INTO snippets (title, content, created, expires) VALUES (
    'An old silent pond',
    'An old silent pond...\nA frog jumps into the pond,\nsplash! Silence again.',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 365 DAY)
);

INSERT INTO snippets (title, content, created, expires) VALUES (
    'Over the wintry forest',
    'Over the wintry forest,\nwinds howl in rage\nwith no leaves to blow.',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 365 DAY)
);

INSERT INTO snippets (title, content, created, expires) VALUES (
    'First autumn morning',
    'First autumn morning\nthe mirror I stare into\nshows my father''s face.',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 7 DAY)
);

-- 3. Create sessions table (for scs library)
CREATE TABLE sessions (
    token CHAR(43) PRIMARY KEY,
    data BLOB NOT NULL,
    expiry TIMESTAMP(6) NOT NULL
);

CREATE INDEX sessions_expiry_idx ON sessions (expiry);
```

### 3. Create Database User
For security reasons, it is recommended to create a dedicated web user:
```sql
CREATE USER 'web'@'localhost' IDENTIFIED BY 'pass';
GRANT SELECT, INSERT, UPDATE, DELETE ON snippetbox.* TO 'web'@'localhost';
```

### 4. Run the Application
Ensure dependencies are downloaded:

```Bash
go mod tidy
```

Run the web server:
```Bash
go run ./cmd/web
```
By default, the server will start on port `:4000`. You can also change the configuration via command-line flags:

```Bash
go run ./cmd/web -addr=":8080" -dsn="web:pass@/snippetbox?parseTime=true"
Visit in browser: http://localhost:4000
```

📂 Project Structure
```Plaintext
snippetbox/
├── cmd/
│   └── web/
│       ├── main.go        # App entry point, dependency injection
│       ├── handlers.go    # HTTP handlers
│       ├── routes.go      # Router definition (ServeMux)
│       ├── middleware.go  # Middleware logic
│       ├── helpers.go     # Helper functions (e.g., render, serverError)
│       └── templates.go   # Template caching and data structures
├── internal/
│   ├── models/            # Database models and operations
│   └── validator/         # Form validation logic
├── ui/
│   ├── html/              # HTML template files
│   └── static/            # Static assets (CSS, JS, Images)
├── go.mod
└── README.md
```

## 👤 Author
This project is developed and maintained by LeoZhao.

If you find this project helpful, please give it a Star ⭐️!

## 🙏 Acknowledgements
This project is a practice application based on Alex Edwards' book 《Let's Go》.

Special thanks to the original author for providing an excellent Go Web development tutorial, which helped in building a clear project architecture and establishing best practices.

## 📄 License
This project is licensed under the **MIT License**. See the LICENSE file for details.