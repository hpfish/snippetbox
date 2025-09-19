# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go web application called "Snippetbox" - a snippet management system built with Go 1.24. It follows a clean architecture pattern with separate concerns for handlers, models, and middleware. The application uses MySQL for data persistence and includes user authentication, session management, and CSRF protection.

## Development Commands

### Running the Application
```bash
# Run the web server (default port :4000)
go run cmd/web/main.go

# Run with custom address and DSN
go run cmd/web/main.go -addr=:9999 -dsn="user:pass@tcp(localhost:3306)/snippetbox?parseTime=true"
```

### Testing
```bash
# Run all tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Run tests for specific package
go test ./cmd/web/
go test ./internal/models/
```

### Build
```bash
# Build the application
go build ./cmd/web

# Build with custom output
go build -o snippetbox ./cmd/web
```

### Linting and Code Quality
```bash
# Format code
go fmt ./...

# Run go vet for static analysis
go vet ./...

# Install and run golangci-lint (if available)
# golangci-lint run
```

### Database Setup
The application expects a MySQL database with the following structure:
- `snippets` table for storing code snippets
- `users` table for user authentication
- Default DSN: `web:pass@tcp(localhost:3306)/snippetbox?parseTime=true`

## Architecture

### Directory Structure
- `cmd/web/` - Main application entry point and HTTP handlers
- `internal/models/` - Data models and database operations
- `internal/validator/` - Form validation helpers
- `internal/assert/` - Testing utilities
- `ui/` - Embedded static files and templates
- `tls/` - SSL/TLS certificates
- `config/` - Application configuration

### Key Components

#### Application Structure
The main application is defined in `cmd/web/main.go:20-27` with these core dependencies:
- Structured logging with `slog`
- Database models for snippets and users
- Template cache for HTML rendering
- Form decoder for POST data
- Session management with MySQL storage

#### Models Layer
- **SnippetModel** (`internal/models/snippets.go`): Handles CRUD operations for code snippets
- **UserModel** (`internal/models/users.go`): Manages user authentication and registration
- Both models implement interfaces for easy testing and mocking

#### Handler Layer
- HTTP handlers in `cmd/web/handlers.go` handle web requests
- Routes defined in `cmd/web/routes.go` using Alice middleware chaining
- Template rendering with context data in `cmd/web/templates.go`

#### Middleware Chain
The application uses a layered middleware approach:
1. Standard middleware (recovery, logging, headers)
2. Dynamic middleware (session management, CSRF protection, authentication)
3. Protected middleware (authentication required)

### Key Dependencies
- `github.com/alexedwards/scs/v2` - Session management
- `github.com/go-playground/form/v4` - Form decoding
- `github.com/go-sql-driver/mysql` - MySQL database driver
- `golang.org/x/crypto` - Cryptographic functions including bcrypt
- `github.com/justinas/alice` - Middleware chaining

### Security Features
- bcrypt password hashing
- Session-based authentication
- CSRF protection (noSurf middleware)
- TLS configuration with modern cipher suites
- SQL injection prevention through parameterized queries

### Testing
- Mock implementations for both SnippetModel and UserModel in `internal/models/mocks/`
- Test utilities in `internal/assert/assert.go`
- Comprehensive test coverage for handlers and models

### Template System
- Embedded UI files using Go's embed package
- Template cache for performance
- Context data passing for dynamic content
- Static file serving through embedded filesystem

## Database Schema
The application expects two main database tables:
- `snippets`: id, title, content, created, expires
- `users`: id, name, email, hashed_password, created

## Configuration
The server uses TLS certificates located in `./tls/cert.pem` and `./tls/key.pem` for HTTPS. Default configuration can be overridden via command-line flags for address and database DSN.