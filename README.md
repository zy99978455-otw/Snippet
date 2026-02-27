# Snippet

Snippet is a text-sharing web application built with Go.

This project aims to demonstrate core concepts in Go web development, including routing, middleware usage, database integration, HTML template rendering, form processing, and session management.

## ✨ Features

* **Core Functionality**:
  * **Publish & View**: Users can create text snippets with a title, content, and expiration time, and view published content via a unique ID.
* **Authentication & Authorization**:
  * User signup, login, and secure logout functionalities.
  * Session-based route protection (unauthenticated users cannot publish snippets).
* **Security Enhancements**:
  * **HTTPS Support**: TLS encryption enabled by default, enforcing high-performance elliptic curves (X25519, P256) for TLS handshakes.
  * **CSRF Protection**: Token-based Cross-Site Request Forgery defense using `justinas/nosurf`.
  * **Session Fixation Defense**: Automatic rotation of Session IDs (`RenewToken`) during privilege level changes (login/logout).
  * **Password Security**: Uses the `bcrypt` hashing algorithm for secure password storage.
  * **Server Hardening**: Configured `IdleTimeout`, `ReadTimeout`, and `WriteTimeout` to defend against slow-client attacks (e.g., Slowloris).
* **Single Binary Deployment**:
  * Utilizes Go 1.16+ `//go:embed` feature to package HTML templates and static assets (CSS, JS) directly into the compiled binary for effortless deployment.
* **Quality Assurance (Testing)**:
  * Comprehensive unit testing using the `net/http/httptest` package to mock HTTP requests and responses.
  * Implementation of elegant **Table-driven tests**.
  * Custom generic test assertion helpers (`internal/assert`) for high code reusability.
* **Data Persistence**: Uses MySQL for all data storage (Snippets and Users).
* **Form & Session Management**:
  * Automatic decoding of HTML form data and custom validation logic (`validator` package).
  * MySQL-backed server-side session storage supporting Flash Messages.

## 🛠️ Tech Stack

* **Language**: Go (Golang)
* **Database**: MySQL
* **Key Dependencies**:
    * [`github.com/go-sql-driver/mysql`](https://github.com/go-sql-driver/mysql): MySQL driver
    * [`github.com/justinas/alice`](https://github.com/justinas/alice): Middleware chaining
    * [`github.com/go-playground/form`](https://github.com/go-playground/form): Form decoding
    * [`github.com/alexedwards/scs/v2`](https://github.com/alexedwards/scs): Session management
    * [`golang.org/x/crypto/bcrypt`](https://pkg.go.dev/golang.org/x/crypto/bcrypt): Password hashing
    * [`github.com/justinas/nosurf`](https://github.com/justinas/nosurf): CSRF protection middleware

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

### 4. Generate TLS Certificates (Required for HTTPS)
Before running the server, you need to generate self-signed certificates for development:

```Bash
# Create directory for certificates
mkdir tls
cd tls

# Use Go standard library tool to generate certs (Linux/macOS/WSL)
go run $(go env GOROOT)/src/crypto/tls/generate_cert.go --rsa-bits=2048 --host=localhost

# Return to project root
cd ..
```

### 5. Run the Application
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

### 6. Access the Application
Visit in browser: https://localhost:4000

⚠️ Note: Since we are using a self-signed certificate, your browser will show a "Not Secure" warning. This is expected. Please click "Advanced" -> "Proceed" to access the site.

📂 Project Structure
```Plaintext
snippetbox/
├── cmd/
│   └── web/
│       ├── context.go     # Context keys management
│       ├── handlers.go    # HTTP handlers (core business logic)
│       ├── handlers_test.go # Automated tests for handlers
│       ├── helpers.go     # Helper functions (rendering, error handling)
│       ├── main.go        # App entry point (Dependency injection, server config)
│       ├── middleware.go  # Middleware (Security, logging, session, auth)
│       ├── middleware_test.go # Automated tests for middleware
│       ├── routes.go      # Router definition and dispatch
│       ├── templates.go   # Template caching and custom functions
│       └── templates_test.go # Automated tests for template formatting
├── internal/
│   ├── assert/            # Generic test assertion library
│   │   └── assert.go
│   ├── models/            # Database interaction models
│   │   ├── errors.go
│   │   ├── snippets.go
│   │   └── users.go
│   └── validator/         # Independent form validation logic
├── ui/
│   ├── html/              # HTML template files
│   ├── static/            # Static assets (CSS, JS, Images)
│   └── efs.go             # Embedded file system directive (go:embed)
├── tls/                   # TLS certificates (Ignored by Git)
├── go.mod
├── README.md              # English Documentation
└── README_ZH.md           # Chinese Documentation
```

## 👤 Author
This project is developed and maintained by LeoZhao.

If you find this project helpful, please give it a Star ⭐️!

## 🙏 Acknowledgements
This project is a practice application based on Alex Edwards' book 《Let's Go》.

Special thanks to the original author for providing an excellent Go Web development tutorial, which helped in building a clear project architecture and establishing best practices.

## 📄 License
This project is licensed under the **MIT License**. See the LICENSE file for details.