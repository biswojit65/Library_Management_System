# Library Management System - Complete Technical Documentation

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [System Architecture](#2-system-architecture)
3. [Technology Stack](#3-technology-stack)
4. [Application Flow](#4-application-flow)
5. [Backend Design](#5-backend-design)
6. [Frontend Design](#6-frontend-design)
7. [Security](#7-security)
8. [DevOps & Deployment](#8-devops--deployment)
9. [Challenges & Solutions](#9-challenges--solutions)
10. [Future Improvements](#10-future-improvements)
11. [Interview Preparation](#11-interview-preparation)

---

## 1. Project Overview

### 1.1 Project Purpose
The Library Management System is a comprehensive, production-ready web application designed to digitize and automate library operations. It solves the real-world problem of managing library resources, user interactions, and administrative tasks in a scalable, efficient manner.

**Real-World Problems Solved:**
- **Manual Book Tracking**: Eliminates manual record-keeping and reduces human error in book inventory management
- **User Self-Service**: Enables users to browse, borrow, reserve, and manage their library activities independently
- **Automated Fine Calculation**: Automatically calculates and tracks fines for overdue books, reducing administrative overhead
- **Centralized Management**: Provides librarians and administrators with a unified dashboard for oversight and control
- **Real-Time Availability**: Ensures users always see accurate book availability, preventing booking conflicts

### 1.2 Target Users

#### Primary Users
- **Library Patrons (Regular Users)**: Browse books, borrow/reserve items, view borrowing history, pay fines
- **Librarians**: Manage book inventory, process borrows/returns, handle reservations, view reports
- **Administrators**: Full system access including user management, system analytics, and configuration

#### Use Cases
1. **User Registration & Authentication**: New users can create accounts with email verification
2. **Book Discovery**: Search and filter books by title, author, category, ISBN
3. **Borrowing Workflow**: Borrow books with automatic availability checks and due date calculation
4. **Reservation System**: Reserve unavailable books and receive notifications when available
5. **Fine Management**: Automatic fine calculation on overdue returns with payment processing
6. **Administrative Oversight**: Admin dashboard with analytics, user management, and system reports

### 1.3 Key Features

#### Core Features
- ✅ **User Authentication & Authorization**: JWT-based auth with role-based access control (RBAC)
- ✅ **Book Management**: Complete CRUD operations with search, filtering, and categorization
- ✅ **Borrowing System**: Check-out/return with automatic copy tracking and availability updates
- ✅ **Reservation Queue**: Smart reservation system with automatic expiration and notifications
- ✅ **Fine Calculation**: Automated fine calculation ($0.50/day, max $50) with payment processing
- ✅ **Smart Notifications**: Event-driven notification system for all library activities
- ✅ **Admin Dashboard**: Comprehensive analytics and system monitoring
- ✅ **Responsive UI**: Mobile-first design with modern, accessible components

#### Business Value
- **Operational Efficiency**: Reduces manual work by 70% through automation
- **User Satisfaction**: 24/7 self-service access improves user experience
- **Data Accuracy**: Real-time synchronization eliminates booking conflicts
- **Cost Reduction**: Automated fine management reduces administrative costs
- **Scalability**: Handles growth from small libraries to large institutions

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Layer                              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  React SPA (TypeScript, Vite, React Router, React Query) │   │
│  │  - Static assets served via Nginx                         │   │
│  │  - JWT tokens stored in localStorage                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS/REST API
                              │
┌─────────────────────────────────────────────────────────────────┐
│                      Application Layer                           │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Nginx Reverse Proxy                                     │   │
│  │  - SSL termination                                       │   │
│  │  - Load balancing (ready)                                │   │
│  │  - Static file serving                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTP
                              │
┌─────────────────────────────────────────────────────────────────┐
│                        Backend Layer                             │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Go API Server (Gin Framework)                           │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │   │
│  │  │ Routes   │→ │ Middleware│→ │Handlers  │              │   │
│  │  └──────────┘  └──────────┘  └──────────┘              │   │
│  │        │              │              │                   │   │
│  │        └──────────────┴──────────────┘                   │   │
│  │                     │                                     │   │
│  │              ┌──────────────┐                            │   │
│  │              │  Repository  │                            │   │
│  │              │    Layer     │                            │   │
│  │              └──────────────┘                            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
┌───────────▼────┐  ┌─────────▼─────┐  ┌───────▼────────┐
│   PostgreSQL   │  │     Redis     │  │   File System  │
│   Database     │  │   Cache/Sess  │  │  (Logs, etc.)  │
│                │  │               │  │                │
│ - Users        │  │ - Sessions    │  │ - Logs         │
│ - Books        │  │ - Cache       │  │ - Static files │
│ - Borrows      │  │ - Rate limit  │  │                │
│ - Reservations │  │               │  │                │
│ - Fines        │  │               │  │                │
│ - Notifications│  │               │  │                │
└────────────────┘  └───────────────┘  └────────────────┘
```

### 2.2 Component Interaction Flow

#### Authentication Flow
```
User → Frontend (Login Form)
    → POST /api/v1/auth/login
    → Backend (JWT Service)
    → Verify credentials (bcrypt)
    → Generate JWT tokens (access + refresh)
    → Return tokens + user data
    → Frontend stores token in localStorage
    → Subsequent requests include: Authorization: Bearer <token>
```

#### Book Borrowing Flow
```
User → Frontend (Book Detail Page)
    → Click "Borrow" button
    → Frontend validates: User authenticated, book available
    → POST /api/v1/borrows { bookId }
    → Backend Middleware: AuthMiddleware (validates JWT)
    → Handler: createBorrowHandler
    → Repository: Check book availability (atomic query)
    → Repository: Create borrow record (transaction)
    → Repository: Decrement availableCopies (transaction)
    → Repository: Generate notification (event-driven)
    → Return borrow record + updated book
    → Frontend: React Query invalidates cache
    → Frontend: Show success notification
    → Frontend: Update UI with new borrow
```

#### Database Transaction Flow
```
Borrow Request
    ↓
Begin Transaction
    ↓
Check Book Availability (SELECT FOR UPDATE)
    ↓
Validate User Can Borrow (check existing borrows)
    ↓
Create Borrow Record
    ↓
Decrement Available Copies (UPDATE books SET availableCopies = ...)
    ↓
Generate Notification
    ↓
Commit Transaction (or Rollback on error)
```

### 2.3 Authentication and Authorization Flow

#### JWT Token Structure
```json
{
  "user_id": 123,
  "email": "user@example.com",
  "role": "user",
  "iss": "library-management-system-backend",
  "exp": 1234567890,
  "iat": 1234567890
}
```

#### Authorization Levels
```
Public Routes (No Auth):
- POST /auth/register
- POST /auth/login
- GET /books (read-only)

User Routes (Auth Required):
- GET /users/profile
- POST /borrows
- GET /borrows
- POST /reservations

Librarian Routes (Auth + Librarian Role):
- GET /admin/borrows
- PUT /admin/borrows/:id/return
- POST /admin/books

Admin Routes (Auth + Admin Role):
- GET /admin/users
- PUT /admin/users/:id/activate
- GET /admin/reports/*
- DELETE /admin/books/:id
```

### 2.4 Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Docker Compose Stack                      │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Frontend  │  │   Backend   │  │   Nginx     │        │
│  │  Container  │  │  Container  │  │  Container  │        │
│  │  (React)    │  │   (Go API)  │  │ (Reverse    │        │
│  │  Port: 80   │  │  Port: 8080 │  │  Proxy)     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│         │                │                │                  │
│         └────────────────┴────────────────┘                  │
│                           │                                  │
│         ┌─────────────────┼─────────────────┐               │
│         │                 │                 │               │
│  ┌──────▼──────┐  ┌───────▼─────┐  ┌───────▼──────┐       │
│  │ PostgreSQL  │  │    Redis    │  │   Volumes    │       │
│  │  Container  │  │  Container  │  │  (Persistent │       │
│  │ Port: 5432  │  │ Port: 6379  │  │   Storage)   │       │
│  └─────────────┘  └─────────────┘  └──────────────┘       │
│                                                              │
│  Network: library_network (bridge)                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Technology Stack

### 3.1 Frontend Technologies

#### **React 19**
- **Why**: Industry standard, component-based architecture, excellent ecosystem
- **Usage**: Core UI framework for building reusable components and pages
- **Benefits**: Virtual DOM for performance, large community support, rich tooling

#### **TypeScript**
- **Why**: Type safety reduces bugs, improves developer experience, better IDE support
- **Usage**: All components, hooks, services, and utilities are strongly typed
- **Benefits**: Catch errors at compile time, better refactoring, self-documenting code

#### **Vite**
- **Why**: Fast development server, instant HMR, optimized production builds
- **Usage**: Build tool and dev server for the React application
- **Benefits**: Sub-second hot module replacement, fast builds, modern ES modules

#### **React Router DOM v7**
- **Why**: Declarative routing, code splitting support, nested routes
- **Usage**: Client-side routing with protected routes, role-based redirects
- **Benefits**: Clean URL management, lazy loading, browser history integration

#### **React Query (TanStack Query)**
- **Why**: Server state management, caching, background updates, optimistic updates
- **Usage**: All API calls are managed through React Query hooks
- **Benefits**: Automatic caching, background refetching, request deduplication

#### **Tailwind CSS**
- **Why**: Utility-first CSS, rapid UI development, consistent design system
- **Usage**: All styling uses Tailwind utility classes
- **Benefits**: No CSS conflicts, small bundle size with purging, responsive by default

#### **shadcn/ui**
- **Why**: Accessible, customizable component library built on Radix UI
- **Usage**: Button, Input, Dialog, Table, Select, and 15+ other components
- **Benefits**: Copy-paste components (not a dependency), full control, accessibility built-in

#### **Axios**
- **Why**: Promise-based HTTP client, interceptors, automatic JSON parsing
- **Usage**: Base HTTP client wrapped in ApiService class
- **Benefits**: Request/response interceptors for auth, automatic error handling

#### **React Hook Form + Formik + Yup**
- **Why**: React Hook Form for performance, Formik for complex forms, Yup for validation
- **Usage**: All forms use React Hook Form with Yup schema validation
- **Benefits**: Minimal re-renders, declarative validation, type-safe schemas

### 3.2 Backend Technologies

#### **Go (Golang) 1.21+**
- **Why**: High performance, excellent concurrency, simple deployment, fast compilation
- **Usage**: Entire backend API server
- **Benefits**: 
  - Compiles to single binary (easy deployment)
  - Goroutines for concurrent request handling
  - Strong typing and memory safety
  - Excellent standard library

#### **Gin Framework**
- **Why**: Fast HTTP router, middleware support, JSON validation, minimal boilerplate
- **Usage**: HTTP server, routing, middleware chain
- **Benefits**: 
  - Fastest Go web framework (benchmarks)
  - Clean API, excellent middleware ecosystem
  - Built-in JSON binding and validation

#### **GORM**
- **Why**: Developer-friendly ORM, migrations, relationships, query builder
- **Usage**: Database operations, migrations, model definitions
- **Benefits**: 
  - Reduces boilerplate SQL
  - Automatic migrations
  - Eager/lazy loading relationships
  - Transaction support

#### **PostgreSQL 15**
- **Why**: Robust relational database, ACID compliance, JSON support, excellent performance
- **Usage**: Primary data store for all entities
- **Benefits**: 
  - ACID transactions for data integrity
  - Complex queries with JOINs
  - Indexes for performance
  - Referential integrity with foreign keys

#### **Redis 7**
- **Why**: Fast in-memory store, pub/sub, caching, session storage
- **Usage**: Caching frequently accessed data, session storage (ready for implementation)
- **Benefits**: 
  - Sub-millisecond latency
  - Reduce database load
  - Future: Real-time notifications via pub/sub

#### **JWT (golang-jwt/jwt/v5)**
- **Why**: Stateless authentication, scalable, industry standard
- **Usage**: User authentication and authorization
- **Benefits**: 
  - No server-side session storage needed
  - Tokens contain user claims (role, ID)
  - Refresh token rotation for security

#### **bcrypt (golang.org/x/crypto)**
- **Why**: Industry-standard password hashing, adaptive cost factor
- **Usage**: Password hashing and verification
- **Benefits**: 
  - Resistant to rainbow table attacks
  - Adaptive cost prevents brute force
  - Built-in salt generation

### 3.3 DevOps & Deployment

#### **Docker**
- **Why**: Containerization for consistent environments, isolation, easy scaling
- **Usage**: All services run in containers
- **Benefits**: 
  - Same environment across dev/staging/prod
  - Easy horizontal scaling
  - Dependency isolation

#### **Docker Compose**
- **Why**: Multi-container orchestration, service dependencies, network management
- **Usage**: Define and run entire stack (frontend, backend, DB, Redis, Nginx)
- **Benefits**: 
  - One command to start entire system
  - Service health checks
  - Volume management

#### **Nginx**
- **Why**: Reverse proxy, load balancing, SSL termination, static file serving
- **Usage**: Routes requests to backend, serves frontend static files
- **Benefits**: 
  - SSL/TLS termination
  - Load balancing (ready for multiple backend instances)
  - Gzip compression
  - Static file caching

### 3.4 Authentication & Security

#### **JWT Authentication**
- Access tokens (24h expiry) + Refresh tokens (7 days)
- HS256 signing algorithm
- Token validation middleware on protected routes

#### **Role-Based Access Control (RBAC)**
- Three roles: `user`, `librarian`, `admin`
- Middleware-based authorization checks
- Route-level and endpoint-level protection

#### **Password Security**
- bcrypt hashing with cost factor 10
- Minimum 8 character requirement
- No plaintext storage

### 3.5 Monitoring & Logging

#### **Structured Logging** (Ready for Implementation)
- Go standard `log` package (can upgrade to `zap` or `logrus`)
- JSON log format for production
- Log levels: INFO, WARN, ERROR

#### **Health Checks**
- `/health` endpoint for container health checks
- Database connectivity checks
- Redis connectivity checks (ready)

---

## 4. Application Flow

### 4.1 End-to-End User Journey

#### User Registration Flow
```
1. User visits /register
2. Fills registration form (email, password, name, phone)
3. Frontend validates input (Yup schema)
4. POST /api/v1/auth/register
5. Backend validates email uniqueness, password strength
6. Hash password with bcrypt
7. Create user record in database
8. Generate JWT tokens
9. Return user data + tokens
10. Frontend stores token, redirects to /dashboard
11. React Query fetches user profile
12. Dashboard renders with user-specific data
```

#### Book Borrowing Journey
```
1. User browses /books
2. Searches/filters books
3. Clicks on book → navigates to /books/:id
4. Frontend fetches book details (React Query cache)
5. User clicks "Borrow" button
6. Frontend checks: Is user authenticated? Is book available?
7. POST /api/v1/borrows { bookId: 123 }
8. Backend validates JWT token
9. Repository: Begin transaction
10. Check book.availableCopies > 0 (SELECT FOR UPDATE)
11. Check user doesn't have too many active borrows
12. Create borrow record (dueDate = now + 14 days)
13. Decrement book.availableCopies
14. Generate notification: "Book borrowed successfully"
15. Commit transaction
16. Return borrow record
17. Frontend: Invalidate React Query cache for /books
18. Frontend: Show success toast notification
19. Frontend: Redirect to /borrows
```

### 4.2 Request Lifecycle

```
Client Request
    ↓
[Nginx] → SSL termination, routing
    ↓
[Go Server] → Parse request
    ↓
[Middleware Chain]
    ├─ CORS middleware
    ├─ Request logging
    ├─ AuthMiddleware (if protected route)
    │   └─ Validate JWT token
    │   └─ Extract user from token
    │   └─ Set user in context
    ├─ AdminMiddleware (if admin route)
    │   └─ Check user.role == "admin"
    ↓
[Route Handler] → Parse request body/params
    ↓
[Repository Layer] → Database operations
    ├─ Validate input
    ├─ Check business rules
    ├─ Execute queries (GORM)
    ├─ Handle relationships
    ↓
[Response] → Serialize to JSON
    ↓
[Client] → React Query caches response
```

### 4.3 Error Handling

#### Frontend Error Handling
```typescript
// Axios interceptor for global error handling
api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      // Token expired, redirect to login
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    
    // Show user-friendly error message
    toast({
      variant: "destructive",
      title: "Error",
      description: error.response?.data?.message || error.message
    });
    
    return Promise.reject(error);
  }
);

// React Error Boundary for component errors
<ErrorBoundary FallbackComponent={ErrorFallback}>
  <App />
</ErrorBoundary>
```

#### Backend Error Handling
```go
// Structured error responses
if err != nil {
    c.JSON(http.StatusBadRequest, gin.H{
        "error": "Invalid request",
        "message": err.Error(),
        "details": validationErrors,
    })
    return
}

// Database transaction rollback on error
err := db.Transaction(func(tx *gorm.DB) error {
    // Operations...
    if err != nil {
        return err // Auto-rollback
    }
    return nil // Auto-commit
})
```

### 4.4 Performance Considerations

#### Frontend Optimizations
- **Code Splitting**: Lazy load routes with `React.lazy()`
- **React Query Caching**: Automatic request deduplication and caching
- **Memoization**: `React.memo()` for expensive components
- **Virtual Scrolling**: Ready for large lists (react-window available)
- **Image Optimization**: Lazy loading, responsive images

#### Backend Optimizations
- **Database Indexing**: Indexes on frequently queried columns (email, userId, bookId, status)
- **Connection Pooling**: GORM connection pool (max 100 connections)
- **Pagination**: All list endpoints support pagination
- **Selective Loading**: Eager load only necessary relationships
- **Query Optimization**: Use SELECT specific columns, avoid N+1 queries

#### Scalability Strategies
- **Horizontal Scaling**: Stateless backend (JWT), can run multiple instances
- **Database Read Replicas**: Ready for read-heavy workloads
- **Redis Caching**: Cache frequently accessed data (book lists, user profiles)
- **CDN**: Static assets can be served via CDN
- **Load Balancer**: Nginx ready for load balancing multiple backend instances

---

## 5. Backend Design

### 5.1 API Design Principles

#### RESTful Conventions
- **Resource-Based URLs**: `/api/v1/books`, `/api/v1/borrows`
- **HTTP Methods**: GET (read), POST (create), PUT (update), DELETE (remove)
- **Status Codes**: 
  - 200 OK (success)
  - 201 Created (resource created)
  - 400 Bad Request (validation error)
  - 401 Unauthorized (authentication required)
  - 403 Forbidden (insufficient permissions)
  - 404 Not Found
  - 500 Internal Server Error

#### API Versioning
- Base URL: `/api/v1/`
- Allows future breaking changes in `/api/v2/`

#### Consistent Response Format
```json
{
  "message": "Operation successful",
  "data": { /* resource data */ },
  "pagination": { /* if list endpoint */ }
}
```

### 5.2 API Endpoints by Module

#### Authentication (`/api/v1/auth`)
```
POST   /auth/register          - Register new user
POST   /auth/login             - Login and get JWT tokens
POST   /auth/refresh           - Refresh access token
POST   /auth/logout            - Logout (client-side token removal)
GET    /auth/me                - Get current user (protected)
POST   /auth/forgot-password   - Request password reset
POST   /auth/reset-password    - Reset password with token
```

#### Books (`/api/v1/books`)
```
GET    /books                  - List books (paginated, searchable)
GET    /books/:id              - Get book details
GET    /books/categories       - Get all categories
POST   /admin/books            - Create book (admin only)
PUT    /admin/books/:id        - Update book (admin only)
DELETE /admin/books/:id        - Delete book (admin only)
```

#### Borrows (`/api/v1/borrows`)
```
GET    /borrows                - Get user's borrows
GET    /borrows/:id            - Get borrow details
POST   /borrows                - Borrow a book
PUT    /borrows/:id/return     - Return a book
GET    /borrows/overdue        - Get overdue borrows
GET    /admin/borrows          - Get all borrows (admin)
PUT    /admin/borrows/:id/return - Force return (admin)
```

#### Reservations (`/api/v1/reservations`)
```
GET    /reservations           - Get user's reservations
GET    /reservations/:id       - Get reservation details
POST   /reservations           - Create reservation
DELETE /reservations/:id       - Cancel reservation
GET    /admin/reservations     - Get all reservations (admin)
```

#### Fines (`/api/v1/fines`)
```
GET    /fines                  - Get user's fines
GET    /fines/:id              - Get fine details
PUT    /fines/:id/pay          - Pay a fine
GET    /fines/summary          - Get fine summary statistics
GET    /admin/fines            - Get all fines (admin)
```

#### Notifications (`/api/v1/notifications`)
```
GET    /notifications          - Get user notifications
GET    /notifications/unread-count - Get unread count
PUT    /notifications/:id/read - Mark as read
PUT    /notifications/mark-all-read - Mark all as read
DELETE /notifications/:id     - Delete notification
POST   /notifications/generate/book-return-reminder - Generate reminder
POST   /notifications/generate/overdue-notification - Generate overdue notice
```

#### Admin (`/api/v1/admin`)
```
GET    /admin/users            - List all users
PUT    /admin/users/:id/activate - Activate user
PUT    /admin/users/:id/deactivate - Deactivate user
GET    /admin/dashboard        - Admin dashboard stats
GET    /admin/reports/overview - System overview report
GET    /admin/reports/books    - Book analytics
GET    /admin/reports/users    - User analytics
```

### 5.3 Sample Request/Response Payloads

#### Create Book (Admin)
**Request:**
```http
POST /api/v1/admin/books
Authorization: Bearer <admin_jwt_token>
Content-Type: application/json

{
  "title": "The Great Gatsby",
  "author": "F. Scott Fitzgerald",
  "isbn": "978-0743273565",
  "category": "Fiction",
  "publishedDate": "1925-04-10",
  "description": "A story of the fabulously wealthy Jay Gatsby...",
  "copies": 5
}
```

**Response (201):**
```json
{
  "message": "Book created successfully",
  "book": {
    "id": 1,
    "title": "The Great Gatsby",
    "author": "F. Scott Fitzgerald",
    "isbn": "978-0743273565",
    "category": "Fiction",
    "publishedDate": "1925-04-10T00:00:00Z",
    "description": "A story of the fabulously wealthy Jay Gatsby...",
    "totalCopies": 5,
    "availableCopies": 5,
    "status": "available",
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:30:00Z"
  }
}
```

#### Borrow Book
**Request:**
```http
POST /api/v1/borrows
Authorization: Bearer <user_jwt_token>
Content-Type: application/json

{
  "bookId": 1
}
```

**Response (201):**
```json
{
  "message": "Book borrowed successfully",
  "borrow": {
    "id": 1,
    "userId": 123,
    "bookId": 1,
    "borrowedAt": "2024-01-15T10:30:00Z",
    "dueDate": "2024-01-29T10:30:00Z",
    "status": "borrowed",
    "fineAmount": 0,
    "book": {
      "id": 1,
      "title": "The Great Gatsby",
      "author": "F. Scott Fitzgerald",
      "availableCopies": 4
    }
  }
}
```

### 5.4 Database Schema Overview

#### Entity Relationship Diagram
```
┌──────────┐         ┌──────────┐         ┌──────────┐
│   User   │────┐    │   Book   │    ┌────│  Borrow  │
│          │    │    │          │    │    │          │
│ - id     │    │    │ - id     │    │    │ - id     │
│ - email  │    │    │ - title  │    │    │ - userId │
│ - role   │    │    │ - author │    │    │ - bookId │
└──────────┘    │    │ - isbn   │    │    │ - status │
                │    │ - copies │    │    └──────────┘
                │    └──────────┘    │         │
                │         │          │         │
                │         └──────────┘         │
                │                              │
                │         ┌──────────┐         │
                └─────────│Reservation│        │
                          │          │        │
                          │ - id     │        │
                          │ - userId │        │
                          │ - bookId │        │
                          │ - status │        │
                          └──────────┘        │
                                              │
                          ┌──────────┐        │
                          │   Fine   │────────┘
                          │          │
                          │ - id     │
                          │ - userId │
                          │ - borrowId│
                          │ - amount │
                          │ - isPaid │
                          └──────────┘
```

#### Table Definitions

**Users Table**
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    role VARCHAR(20) DEFAULT 'user' CHECK (role IN ('user', 'admin', 'librarian')),
    is_active BOOLEAN DEFAULT true,
    email_verified BOOLEAN DEFAULT false,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

**Books Table**
```sql
CREATE TABLE books (
    id BIGSERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) NOT NULL,
    isbn VARCHAR(20) UNIQUE NOT NULL,
    category VARCHAR(100) NOT NULL,
    published_date DATE,
    description TEXT,
    total_copies INTEGER NOT NULL DEFAULT 1,
    available_copies INTEGER NOT NULL DEFAULT 1,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE,
    CONSTRAINT available_copies_check CHECK (available_copies >= 0),
    CONSTRAINT total_copies_check CHECK (total_copies >= available_copies)
);

CREATE INDEX idx_books_title ON books(title);
CREATE INDEX idx_books_author ON books(author);
CREATE INDEX idx_books_category ON books(category);
CREATE INDEX idx_books_isbn ON books(isbn);
```

**Borrows Table**
```sql
CREATE TABLE borrows (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id BIGINT NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    borrowed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    due_date DATE NOT NULL,
    returned_at TIMESTAMP WITH TIME ZONE,
    fine_amount DECIMAL(10,2) DEFAULT 0,
    status VARCHAR(20) DEFAULT 'borrowed' CHECK (status IN ('borrowed', 'returned', 'overdue')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_borrows_user_id ON borrows(user_id);
CREATE INDEX idx_borrows_book_id ON borrows(book_id);
CREATE INDEX idx_borrows_status ON borrows(status);
CREATE INDEX idx_borrows_due_date ON borrows(due_date);
```

### 5.5 Indexing Strategy

#### Primary Indexes
- **Primary Keys**: All tables have `id` as primary key (auto-indexed)
- **Foreign Keys**: Indexed for JOIN performance

#### Search Indexes
- `users.email` - Unique index for login lookups
- `books.title`, `books.author`, `books.category` - For search queries
- `books.isbn` - Unique index for book identification
- `borrows.user_id`, `borrows.book_id` - For user/book history queries
- `borrows.status`, `borrows.due_date` - For filtering and overdue queries

#### Composite Indexes (Future Optimization)
- `(user_id, status)` on borrows - For user's active borrows query
- `(book_id, status)` on borrows - For book availability queries
- `(category, available_copies)` on books - For category filtering with availability

### 5.6 Transactions and Data Integrity

#### Critical Transactions

**Borrow Book Transaction:**
```go
err := db.Transaction(func(tx *gorm.DB) error {
    // Lock book row to prevent race conditions
    var book Book
    if err := tx.Set("gorm:query_option", "FOR UPDATE").
        Where("id = ? AND available_copies > 0", bookID).
        First(&book).Error; err != nil {
        return errors.New("book not available")
    }
    
    // Create borrow record
    borrow := Borrow{
        UserID: userID,
        BookID: bookID,
        DueDate: time.Now().AddDate(0, 0, 14),
        Status: "borrowed",
    }
    if err := tx.Create(&borrow).Error; err != nil {
        return err
    }
    
    // Decrement available copies
    book.AvailableCopies--
    if err := tx.Save(&book).Error; err != nil {
        return err
    }
    
    return nil
})
```

**Return Book Transaction:**
```go
err := db.Transaction(func(tx *gorm.DB) error {
    // Update borrow record
    borrow.ReturnedAt = &now
    borrow.Status = "returned"
    borrow.FineAmount = borrow.CalculateFine()
    tx.Save(&borrow)
    
    // Increment available copies
    book.AvailableCopies++
    tx.Save(&book)
    
    // Create fine if overdue
    if borrow.FineAmount > 0 {
        fine := Fine{
            UserID: borrow.UserID,
            BorrowID: borrow.ID,
            Amount: borrow.FineAmount,
            Reason: "overdue",
        }
        tx.Create(&fine)
    }
    
    return nil
})
```

---

## 6. Frontend Design

### 6.1 Component Architecture

#### Component Hierarchy
```
App
├── ErrorBoundary
├── NotificationProvider
├── Routes
│   ├── Public Routes (Login, Register)
│   ├── User Routes (Layout)
│   │   ├── Header
│   │   ├── Sidebar
│   │   └── Outlet (Dashboard, Books, Borrows, etc.)
│   └── Admin Routes (Layout)
│       ├── Header
│       ├── Sidebar (Admin)
│       └── Outlet (AdminDashboard, AdminUsers, etc.)
└── AlertContainer
```

#### Component Types

**Layout Components:**
- `Layout.tsx` - Main layout wrapper with header and sidebar
- `Header.tsx` - Top navigation bar with user menu
- `Sidebar.tsx` - Side navigation menu

**Page Components:**
- `Dashboard.tsx` - User dashboard with stats
- `Books.tsx` - Book catalog with search/filter
- `BookDetail.tsx` - Individual book details
- `Borrows.tsx` - User's borrowing history
- `AdminDashboard.tsx` - Admin overview

**UI Components (shadcn/ui):**
- `Button`, `Input`, `Card`, `Table`, `Dialog`, `Select`, `Toast`, etc.

**Auth Components:**
- `ProtectedRoute.tsx` - Route guard for authenticated users
- `AdminRoute.tsx` - Route guard for admin users
- `UserRoute.tsx` - Route guard for regular users

### 6.2 State Management Strategy

#### Server State (React Query)
```typescript
// All API data managed by React Query
const { data: books, isLoading } = useQuery({
  queryKey: ['books', { page, search, category }],
  queryFn: () => bookApi.getBooks({ page, search, category }),
  staleTime: 30000, // 30 seconds
  cacheTime: 300000, // 5 minutes
});

// Mutations for creating/updating
const { mutate: borrowBook } = useMutation({
  mutationFn: bookApi.borrowBook,
  onSuccess: () => {
    queryClient.invalidateQueries(['books']);
    queryClient.invalidateQueries(['borrows']);
  },
});
```

#### Client State (React Context + Hooks)
```typescript
// Auth Context
const { user, login, logout } = useAuth();

// Theme Context
const { theme, toggleTheme } = useTheme();

// Notification Context
const { notifications, markAsRead } = useNotifications();

// Local State (useState)
const [searchTerm, setSearchTerm] = useState('');
const [selectedCategory, setSelectedCategory] = useState('');
```

### 6.3 Routing and Guards

#### Route Structure
```typescript
<Routes>
  {/* Public routes */}
  <Route path="/login" element={<Login />} />
  <Route path="/register" element={<Register />} />
  
  {/* Protected user routes */}
  <Route path="/dashboard" element={
    <UserRoute>
      <Layout />
    </UserRoute>
  }>
    <Route index element={<Dashboard />} />
  </Route>
  
  {/* Protected admin routes */}
  <Route path="/admin" element={
    <AdminRoute>
      <Layout />
    </AdminRoute>
  }>
    <Route index element={<AdminDashboard />} />
  </Route>
</Routes>
```

#### Route Guards Implementation
```typescript
// UserRoute.tsx
const UserRoute = ({ children }: { children: React.ReactNode }) => {
  const { user, loading } = useAuth();
  
  if (loading) return <LoadingSpinner />;
  if (!user) return <Navigate to="/login" />;
  
  return <>{children}</>;
};

// AdminRoute.tsx
const AdminRoute = ({ children }: { children: React.ReactNode }) => {
  const { user, loading } = useAuth();
  
  if (loading) return <LoadingSpinner />;
  if (!user) return <Navigate to="/login" />;
  if (user.role !== 'admin') return <Navigate to="/dashboard" />;
  
  return <>{children}</>;
};
```

### 6.4 Form Handling and Validation

#### Form Implementation
```typescript
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

const schema = yup.object({
  email: yup.string().email().required(),
  password: yup.string().min(8).required(),
  firstName: yup.string().min(2).max(100).required(),
});

const { register, handleSubmit, formState: { errors } } = useForm({
  resolver: yupResolver(schema),
});

const onSubmit = async (data) => {
  await authApi.register(data);
};
```

### 6.5 API Integration Approach

#### Centralized API Service
```typescript
// services/api.ts
class ApiService {
  private api: AxiosInstance;
  
  constructor() {
    this.api = axios.create({
      baseURL: API_BASE_URL,
      headers: { 'Content-Type': 'application/json' },
    });
    
    // Request interceptor: Add JWT token
    this.api.interceptors.request.use((config) => {
      const token = localStorage.getItem('token');
      if (token) {
        config.headers.Authorization = `Bearer ${token}`;
      }
      return config;
    });
    
    // Response interceptor: Handle errors
    this.api.interceptors.response.use(
      response => response,
      error => {
        if (error.response?.status === 401) {
          localStorage.removeItem('token');
          window.location.href = '/login';
        }
        return Promise.reject(error);
      }
    );
  }
}

// Separate API classes for each resource
export const bookApi = new BookApi();
export const borrowApi = new BorrowApi();
// etc.
```

#### React Query Integration
```typescript
// hooks/useBooks.ts
export const useBooks = (params: BookSearchRequest) => {
  return useQuery({
    queryKey: ['books', params],
    queryFn: () => bookApi.getBooks(params),
  });
};

// Usage in component
const BooksPage = () => {
  const [search, setSearch] = useState('');
  const { data, isLoading } = useBooks({ search, page: 1 });
  
  return (
    // Render books
  );
};
```

### 6.6 UX and Accessibility Considerations

#### Accessibility Features
- **ARIA Labels**: All interactive elements have proper ARIA labels
- **Keyboard Navigation**: Full keyboard support for all interactions
- **Focus Management**: Proper focus handling in modals and forms
- **Screen Reader Support**: Semantic HTML, proper headings hierarchy
- **Color Contrast**: WCAG AA compliant color schemes

#### User Experience Enhancements
- **Loading States**: Skeleton loaders and spinners for async operations
- **Error Boundaries**: Graceful error handling with user-friendly messages
- **Optimistic Updates**: UI updates immediately, syncs with server
- **Toast Notifications**: Non-intrusive feedback for user actions
- **Responsive Design**: Mobile-first, works on all screen sizes
- **Dark Mode**: Theme switching for user preference

---

## 7. Security

### 7.1 Authentication Flow

#### JWT Token Generation
```go
// Generate access token (24h expiry)
claims := JWTClaims{
    UserID: user.ID,
    Email:  user.Email,
    Role:   user.Role,
    RegisteredClaims: jwt.RegisteredClaims{
        ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
        IssuedAt:  jwt.NewNumericDate(time.Now()),
    },
}
token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
accessToken := token.SignedString([]byte(secret))

// Generate refresh token (7 days expiry)
refreshToken := // Similar process with longer expiry
```

#### Token Refresh Flow
```
1. Frontend detects 401 Unauthorized
2. Attempts to refresh using refresh token
3. POST /api/v1/auth/refresh { refreshToken }
4. Backend validates refresh token
5. Issues new access + refresh token pair
6. Frontend updates stored tokens
7. Retry original request with new access token
```

### 7.2 Authorization and RBAC

#### Role Hierarchy
```
admin
  └─ Can access all endpoints
  └─ User management, system configuration

librarian
  └─ Can manage books, borrows, reservations
  └─ Cannot manage users or system settings

user
  └─ Can browse books, borrow, view own data
  └─ Cannot access admin endpoints
```

#### Middleware Implementation
```go
// AdminMiddleware
func AdminMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        user, exists := c.Get("user")
        if !exists || !user.(*models.User).IsAdmin() {
            c.JSON(403, gin.H{"error": "Admin access required"})
            c.Abort()
            return
        }
        c.Next()
    }
}

// Usage in routes
adminRoutes := api.Group("/admin", AdminMiddleware())
```

### 7.3 Input Validation and Sanitization

#### Backend Validation
```go
// GORM model validation
type BookCreateRequest struct {
    Title  string `json:"title" validate:"required,min=1,max=255"`
    Author string `json:"author" validate:"required,min=1,max=255"`
    ISBN   string `json:"isbn" validate:"required,min=10,max=20"`
}

// Handler validation
if err := c.ShouldBindJSON(&req); err != nil {
    c.JSON(400, gin.H{"error": "Invalid input", "details": err.Error()})
    return
}
```

#### Frontend Validation
```typescript
// Yup schema validation
const schema = yup.object({
  email: yup.string().email('Invalid email').required('Email required'),
  password: yup.string()
    .min(8, 'Password must be at least 8 characters')
    .matches(/[A-Z]/, 'Password must contain uppercase letter')
    .matches(/[a-z]/, 'Password must contain lowercase letter')
    .matches(/[0-9]/, 'Password must contain number')
    .required('Password required'),
});
```

### 7.4 Common Attack Prevention

#### SQL Injection Prevention
- **GORM Parameterized Queries**: All queries use parameter binding
```go
// Safe - parameterized
db.Where("email = ?", email).First(&user)

// NOT used - string concatenation
// db.Where("email = '" + email + "'")  // NEVER DO THIS
```

#### XSS Prevention
- **Content Security Policy**: HTTP headers prevent inline scripts
- **React Escaping**: React automatically escapes user input
- **Input Sanitization**: HTML sanitization for rich text (if needed)

#### CSRF Protection
- **SameSite Cookies**: Cookies set with SameSite=Strict (if using cookies)
- **JWT in Header**: Tokens in Authorization header (not cookies) reduces CSRF risk
- **Origin Validation**: Validate request origin header

#### Password Security
```go
// Hashing with bcrypt
hashedPassword, err := bcrypt.GenerateFromPassword(
    []byte(password), 
    bcrypt.DefaultCost, // Cost factor 10
)

// Verification
err := bcrypt.CompareHashAndPassword(hashedPassword, []byte(providedPassword))
```

### 7.5 Secrets Management

#### Environment Variables
```env
# .env (never commit to git)
JWT_SECRET=your_very_long_and_secure_secret_key_here_min_32_chars
DB_PASSWORD=secure_database_password
REDIS_PASSWORD=redis_password
```

#### Docker Secrets (Production)
```yaml
# docker-compose.yml
services:
  backend:
    environment:
      - JWT_SECRET_FILE=/run/secrets/jwt_secret
    secrets:
      - jwt_secret
secrets:
  jwt_secret:
    external: true
```

#### Best Practices
- Never commit secrets to version control
- Use secret management services (AWS Secrets Manager, HashiCorp Vault)
- Rotate secrets regularly
- Use different secrets for dev/staging/prod
- Limit secret access with proper IAM roles

---

## 8. DevOps & Deployment

### 8.1 Docker and Container Setup

#### Backend Dockerfile
```dockerfile
FROM golang:1.21-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main cmd/main.go

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]
```

#### Frontend Dockerfile
```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

#### Docker Compose Configuration
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: library_db
      POSTGRES_USER: library_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U library_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]

  backend:
    build: ./library-management-system-backend
    environment:
      DB_HOST: postgres
      DB_PASSWORD: ${DB_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  frontend:
    build: ./library-management-system-frontend
    depends_on:
      - backend

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - frontend
      - backend

volumes:
  postgres_data:
  redis_data:
```

### 8.2 Environment Configurations

#### Development
```env
ENVIRONMENT=development
DB_HOST=localhost
DB_PORT=5432
DB_NAME=library_db
JWT_SECRET=dev_secret_key_change_in_production
JWT_EXPIRY=24h
```

#### Production
```env
ENVIRONMENT=production
DB_HOST=postgres
DB_PORT=5432
DB_NAME=library_db
DB_SSL_MODE=require
JWT_SECRET=<secure_random_32_char_min>
JWT_EXPIRY=1h
REDIS_HOST=redis
REDIS_PORT=6379
```

### 8.3 CI/CD Pipeline Flow

#### GitHub Actions Example (Ready for Implementation)
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  backend-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      - run: go test ./...
      - run: go build ./cmd/main.go

  frontend-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test
      - run: npm run build

  deploy:
    needs: [backend-test, frontend-test]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: |
          docker-compose build
          docker-compose up -d
```

### 8.4 Scaling Strategy

#### Horizontal Scaling
```
Load Balancer (Nginx)
    ├─ Backend Instance 1
    ├─ Backend Instance 2
    └─ Backend Instance N
         │
         └─ Shared PostgreSQL (with connection pooling)
         └─ Shared Redis (cache + session store)
```

#### Database Scaling
- **Read Replicas**: Add read replicas for read-heavy workloads
- **Connection Pooling**: GORM connection pool (max 100 per instance)
- **Database Indexing**: Optimize slow queries with proper indexes
- **Query Optimization**: Use EXPLAIN ANALYZE to optimize queries

#### Caching Strategy
```go
// Redis caching example (ready for implementation)
func GetBook(id uint) (*Book, error) {
    // Check cache first
    cached, err := redis.Get(fmt.Sprintf("book:%d", id))
    if err == nil {
        return deserializeBook(cached), nil
    }
    
    // Cache miss - query database
    book := &Book{}
    db.First(book, id)
    
    // Store in cache (5 minute TTL)
    redis.Set(fmt.Sprintf("book:%d", id), serializeBook(book), 5*time.Minute)
    
    return book, nil
}
```

### 8.5 Reverse Proxy and Load Balancing

#### Nginx Configuration
```nginx
upstream backend {
    least_conn;
    server backend:8080 max_fails=3 fail_timeout=30s;
    # Add more backend instances for load balancing
    # server backend2:8080;
    # server backend3:8080;
}

server {
    listen 80;
    server_name library.example.com;

    # Frontend static files
    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }

    # Backend API
    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # SSL termination (when SSL certificates are added)
    # listen 443 ssl;
    # ssl_certificate /etc/nginx/ssl/cert.pem;
    # ssl_certificate_key /etc/nginx/ssl/key.pem;
}
```

---

## 9. Challenges & Solutions

### 9.1 Technical Challenges Faced

#### Challenge 1: Race Condition in Book Borrowing
**Problem**: Multiple users trying to borrow the last available copy simultaneously could result in over-borrowing.

**Solution**: 
- Used `SELECT FOR UPDATE` to lock the book row during transaction
- Atomic decrement of `availableCopies` within a database transaction
- Return appropriate error if book becomes unavailable during transaction

```go
// Solution implementation
err := db.Transaction(func(tx *gorm.DB) error {
    var book Book
    // Lock row for update
    if err := tx.Set("gorm:query_option", "FOR UPDATE").
        Where("id = ? AND available_copies > 0", bookID).
        First(&book).Error; err != nil {
        return errors.New("book not available")
    }
    // Continue with borrow creation...
})
```

#### Challenge 2: JWT Token Expiration Handling
**Problem**: Access tokens expire after 24h, requiring user to re-login frequently.

**Solution**:
- Implemented refresh token mechanism (7-day expiry)
- Frontend interceptor automatically refreshes token on 401 errors
- Seamless user experience without frequent logouts

#### Challenge 3: Large Book Catalog Performance
**Problem**: Loading thousands of books caused slow page loads and poor UX.

**Solution**:
- Implemented pagination (default 20 items per page)
- Added React Query caching to prevent redundant API calls
- Database indexes on searchable columns (title, author, category)
- Lazy loading of book details (load on-demand)
- Virtual scrolling ready for large lists

```typescript
// Pagination implementation
const { data } = useQuery({
  queryKey: ['books', { page, limit: 20, search }],
  queryFn: () => bookApi.getBooks({ page, limit: 20, search }),
  staleTime: 30000, // Cache for 30 seconds
});
```

#### Challenge 4: Real-Time Notification Updates
**Problem**: Users needed to refresh the page to see new notifications.

**Solution**:
- Implemented polling mechanism (every 30 seconds) for unread notifications
- React Query automatic refetching on window focus
- Optimistic updates for notification actions (mark as read)
- Notification count badge in header for real-time updates

### 9.2 Design Trade-offs

#### Trade-off 1: JWT vs Session-Based Authentication
**Chosen**: JWT (Stateless)
- **Pros**: Scalable, works across multiple servers, no session storage needed
- **Cons**: Cannot revoke tokens immediately (mitigated with short expiry times)
- **Decision**: Chose JWT for scalability and microservices-ready architecture

#### Trade-off 2: Client-Side vs Server-Side Rendering
**Chosen**: Client-Side Rendering (React SPA)
- **Pros**: Fast navigation, rich interactivity, better UX, lower server load
- **Cons**: Slower initial load, SEO concerns (mitigated with meta tags)
- **Decision**: CSR for better user experience, considering SSR for public pages in future

#### Trade-off 3: SQL vs NoSQL Database
**Chosen**: PostgreSQL (SQL)
- **Pros**: ACID transactions, complex queries, data integrity, relationships
- **Cons**: Less flexible schema, requires migrations
- **Decision**: SQL for data integrity and complex queries in library management

### 9.3 Performance Bottlenecks and Fixes

#### Bottleneck 1: N+1 Query Problem
**Problem**: Loading borrows with user and book data caused N+1 queries.

**Solution**:
```go
// Before: N+1 queries
var borrows []Borrow
db.Find(&borrows)
for _, borrow := range borrows {
    db.Model(&borrow).Association("User").Find(&borrow.User)
    db.Model(&borrow).Association("Book").Find(&borrow.Book)
}

// After: Eager loading with Preload
var borrows []Borrow
db.Preload("User").Preload("Book").Find(&borrows)
```

#### Bottleneck 2: Large Bundle Size
**Problem**: Initial bundle size was too large, causing slow initial load.

**Solution**:
- Code splitting with React.lazy() for routes
- Tree shaking to remove unused code
- Vite build optimization
- Lazy load heavy components (charts, admin panels)

#### Bottleneck 3: Database Connection Pool Exhaustion
**Problem**: Too many concurrent connections during peak usage.

**Solution**:
- Configured GORM connection pool limits (max 100 connections)
- Connection lifetime management (1 hour)
- Implemented connection retry logic
- Database read replicas (future scaling)

### 9.4 Lessons Learned

1. **Transaction Management**: Always use transactions for multi-step operations that must be atomic (borrowing, returning books)

2. **Error Handling**: Implement comprehensive error boundaries and user-friendly error messages at every layer

3. **Validation**: Validate on both frontend and backend - frontend for UX, backend for security

4. **Caching Strategy**: Implement caching early for frequently accessed, rarely changing data (book categories, user profiles)

5. **Database Indexing**: Index foreign keys and frequently queried columns from the start - easier than retrofitting

6. **API Design**: Consistent API structure and error responses make frontend development much easier

7. **Type Safety**: TypeScript caught many bugs at compile time - worth the initial setup overhead

8. **Testing**: Write tests alongside features, not after - easier to maintain and catch regressions

---

## 10. Future Improvements

### 10.1 Features to Add

#### Enhanced Search
- **Full-Text Search**: PostgreSQL full-text search for better relevance
- **Search Suggestions**: Autocomplete for book titles and authors
- **Advanced Filters**: Filter by publication date range, ratings, tags
- **Search History**: Track and suggest previous searches

#### Book Recommendations
- **Collaborative Filtering**: "Users who borrowed this also borrowed..."
- **Content-Based**: Recommendations based on book categories and metadata
- **Machine Learning**: ML-based recommendation engine
- **Personalized Dashboard**: Show personalized book suggestions

#### Social Features
- **User Reviews**: Allow users to rate and review books
- **Reading Lists**: Users can create and share reading lists
- **Book Clubs**: Groups of users discussing books
- **Wishlist**: Save books for future borrowing

#### Mobile Application
- **React Native App**: Native mobile experience
- **Push Notifications**: Real-time notifications for mobile users
- **Barcode Scanner**: Scan ISBN barcodes to quickly find books
- **Offline Mode**: Cache book catalog for offline browsing

#### Enhanced Admin Features
- **Bulk Operations**: Import/export books via CSV/Excel
- **Advanced Analytics**: Charts and graphs for trends
- **Audit Logging**: Track all admin actions for security
- **Automated Reports**: Scheduled email reports

#### Email Integration
- **Email Notifications**: Send emails for overdue books, reservations available
- **Email Verification**: Verify user email addresses
- **Newsletter**: Send library updates and new book announcements

### 10.2 Scalability Improvements

#### Microservices Architecture
- **Service Separation**: Split into user-service, book-service, borrow-service
- **API Gateway**: Centralized entry point for all services
- **Event-Driven Communication**: Use message queues (RabbitMQ, Kafka) for async operations
- **Service Discovery**: Implement service discovery for dynamic scaling

#### Database Optimization
- **Read Replicas**: Separate read and write operations
- **Database Sharding**: Partition data by user ID or region
- **Connection Pooling**: Use PgBouncer for better connection management
- **Query Optimization**: Analyze and optimize slow queries regularly

#### Caching Strategy
- **Redis Caching**: Cache frequently accessed data (book lists, user profiles)
- **CDN**: Serve static assets via CDN
- **Application-Level Caching**: Cache API responses
- **Cache Warming**: Pre-populate cache during off-peak hours

#### Load Balancing
- **Multiple Backend Instances**: Scale horizontally behind load balancer
- **Database Load Balancing**: Distribute read queries across replicas
- **Session Affinity**: Sticky sessions if needed (though JWT makes it stateless)

### 10.3 Security Enhancements

#### Enhanced Authentication
- **OAuth Integration**: Login with Google, GitHub, Microsoft
- **Multi-Factor Authentication (MFA)**: Add 2FA for admin accounts
- **Biometric Authentication**: Fingerprint/face ID for mobile app
- **Password Policies**: Enforce stronger password requirements

#### API Security
- **Rate Limiting**: Prevent API abuse with rate limiting
- **API Versioning**: Maintain backward compatibility
- **Request Signing**: Sign requests for additional security
- **CORS Refinement**: Stricter CORS policies

#### Data Protection
- **Data Encryption**: Encrypt sensitive data at rest
- **Backup Strategy**: Automated daily backups with point-in-time recovery
- **GDPR Compliance**: Add data export and deletion features
- **Audit Trail**: Comprehensive audit logging for compliance

### 10.4 Architecture Evolution

#### Event Sourcing (Future Consideration)
- **Event Store**: Store all events (book borrowed, returned, etc.)
- **CQRS**: Separate read and write models
- **Event Replay**: Rebuild state from events for debugging
- **Audit Logging**: Natural audit trail from events

#### Real-Time Features
- **WebSockets**: Real-time notifications without polling
- **Server-Sent Events (SSE)**: Push updates to clients
- **GraphQL Subscription**: Real-time data with GraphQL
- **Live Dashboard**: Real-time statistics for admins

#### Cloud Migration
- **Kubernetes**: Container orchestration for production
- **Cloud Managed Services**: Use managed PostgreSQL, Redis
- **Auto-Scaling**: Auto-scale based on load
- **Multi-Region Deployment**: Deploy to multiple regions for availability

---

## 11. Interview Preparation

This section contains 50 technical interview questions covering all aspects of the Library Management System project, with detailed, interview-ready answers.

### System Design Questions

#### Q1: Walk me through the high-level architecture of your Library Management System.

**Answer**: The system follows a three-tier architecture:
- **Client Layer**: React SPA (TypeScript, Vite) served via Nginx, communicates with backend via REST API
- **Application Layer**: Go backend (Gin framework) with clean architecture pattern:
  - Routes → Middleware → Handlers → Repository → Database
  - JWT authentication middleware for protected routes
  - RBAC middleware for role-based authorization
- **Data Layer**: PostgreSQL for persistent storage, Redis for caching
- **Infrastructure**: Docker containerization with Docker Compose for orchestration, Nginx as reverse proxy

The architecture is stateless (JWT-based), allowing horizontal scaling. All services communicate via REST API, making it microservices-ready.

#### Q2: Why did you choose Go for the backend instead of Node.js or Python?

**Answer**: I chose Go for several reasons:
- **Performance**: Go compiles to native binary, providing excellent performance (often faster than Node.js/Python)
- **Concurrency**: Goroutines make handling concurrent requests very efficient
- **Deployment**: Single binary deployment simplifies containerization and deployment
- **Type Safety**: Strong typing catches errors at compile time
- **Standard Library**: Excellent HTTP server and JSON support out of the box
- **Scalability**: Perfect for building scalable backend services

For a library system that may need to handle many concurrent borrow/return operations, Go's concurrency model is ideal.

#### Q3: Explain your database schema design. Why PostgreSQL over MongoDB?

**Answer**: I chose PostgreSQL because:
- **ACID Transactions**: Critical for book borrowing/returning operations - must be atomic
- **Data Integrity**: Foreign key constraints ensure referential integrity
- **Complex Queries**: Library systems need JOINs (user borrows → books, fines → borrows)
- **Mature Ecosystem**: Well-tested, reliable, excellent tooling

Schema design:
- **Users**: Core user data with role-based access
- **Books**: Book metadata with available/total copies tracking
- **Borrows**: Many-to-many relationship between users and books
- **Reservations**: Queue system for unavailable books
- **Fines**: Linked to borrows, tracks payment status
- **Notifications**: Event-driven notification system

Key relationships:
- User → Borrows (one-to-many)
- Book → Borrows (one-to-many)
- Borrow → Fine (one-to-many, optional)

This relational model ensures data consistency and supports complex queries efficiently.

#### Q4: How do you handle race conditions when multiple users try to borrow the last available copy?

**Answer**: I use database-level locking with transactions:
1. **SELECT FOR UPDATE**: Lock the book row during the transaction
2. **Atomic Operations**: Check availability and decrement in same transaction
3. **Transaction Isolation**: Ensures serializable isolation level

```go
err := db.Transaction(func(tx *gorm.DB) error {
    // Lock book row
    var book Book
    if err := tx.Set("gorm:query_option", "FOR UPDATE").
        Where("id = ? AND available_copies > 0", bookID).
        First(&book).Error; err != nil {
        return errors.New("book not available")
    }
    
    // Create borrow and decrement atomically
    // ...
})
```

If two users try simultaneously, the second transaction waits for the first to commit. If the first succeeds, the second fails gracefully with "book not available" error.

#### Q5: Describe your caching strategy. What do you cache and why?

**Answer**: Current implementation uses React Query for client-side caching:
- **Book Lists**: Cached for 30 seconds (balance between freshness and performance)
- **User Profile**: Cached until invalidated
- **Book Details**: Cached per book ID

Future Redis caching strategy:
- **Frequently Accessed, Rarely Changed**: Book categories, user roles
- **Expensive Queries**: Complex analytics queries, admin dashboard stats
- **Session Data**: If we move from JWT to sessions

Cache invalidation:
- **Time-based**: TTL for cached data (5 minutes for book lists)
- **Event-based**: Invalidate cache when book is updated/created
- **Manual**: Admin can clear cache for immediate updates

I avoided caching user-specific data like borrows (changes frequently) and used React Query's background refetching for freshness.

### Frontend Architecture Questions

#### Q6: Explain your state management strategy. Why React Query over Redux?

**Answer**: I use a hybrid approach:
- **React Query**: Server state (API data, caching, background updates)
- **React Context**: Client state (auth, theme, notifications)
- **useState**: Local component state (form inputs, UI toggles)

Why React Query over Redux:
- **Less Boilerplate**: No actions, reducers, or action creators
- **Built-in Caching**: Automatic caching and background refetching
- **Request Deduplication**: Multiple components requesting same data = single request
- **Optimistic Updates**: Easy to implement optimistic UI updates
- **Error Handling**: Built-in error and loading states

For a library system with lots of API calls, React Query reduces code complexity significantly while providing better UX.

#### Q7: How do you handle authentication and protected routes in React?

**Answer**: Multi-layered approach:
1. **Auth Context**: Global auth state with user data and login/logout functions
2. **Route Guards**: Wrapper components (UserRoute, AdminRoute) check auth before rendering
3. **Axios Interceptors**: Automatically add JWT token to requests, handle 401 errors

```typescript
// Route guard
const UserRoute = ({ children }) => {
  const { user, loading } = useAuth();
  if (loading) return <LoadingSpinner />;
  if (!user) return <Navigate to="/login" />;
  return <>{children}</>;
};

// Axios interceptor
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

On 401, interceptor redirects to login. Token stored in localStorage (with refresh token for seamless UX).

#### Q8: How do you handle form validation? Why Yup?

**Answer**: React Hook Form + Yup schema validation:
- **React Hook Form**: Performance-focused (uncontrolled components, minimal re-renders)
- **Yup**: Declarative schema validation, type-safe

```typescript
const schema = yup.object({
  email: yup.string().email().required(),
  password: yup.string().min(8).required(),
});

const { register, handleSubmit, formState: { errors } } = useForm({
  resolver: yupResolver(schema),
});
```

Why Yup:
- **Declarative**: Schema is readable and self-documenting
- **Type Safety**: Works well with TypeScript
- **Reusable**: Same schema can be used frontend and backend (JSON Schema)
- **Rich Validators**: Built-in validators for common cases

I also validate on backend for security - frontend validation is for UX only.

#### Q9: Explain your component architecture. How do you organize components?

**Answer**: Feature-based organization:
```
src/
├── components/
│   ├── Auth/          # Auth-related components
│   ├── Layout/        # Layout components (Header, Sidebar)
│   └── ui/            # Reusable UI components (shadcn/ui)
├── pages/
│   ├── Admin/         # Admin pages
│   ├── Books/         # Book-related pages
│   └── Auth/          # Login, Register
├── hooks/             # Custom React hooks
├── services/          # API service layer
└── contexts/          # React contexts
```

Principles:
- **Separation of Concerns**: Business logic in hooks, UI in components
- **Reusability**: Shared components in `components/ui`
- **Feature Modules**: Related pages/components grouped together
- **Atomic Design**: Small components (Button) compose into larger ones (BookCard)

shadcn/ui components are copied (not npm package) for full customization control.

#### Q10: How do you handle loading states and errors in your React app?

**Answer**: Multi-level error handling:
1. **React Query**: Built-in loading/error states
   ```typescript
   const { data, isLoading, error } = useQuery(...);
   if (isLoading) return <LoadingSpinner />;
   if (error) return <ErrorMessage error={error} />;
   ```

2. **Error Boundaries**: Catch component errors
   ```typescript
   <ErrorBoundary FallbackComponent={ErrorFallback}>
     <App />
   </ErrorBoundary>
   ```

3. **Axios Interceptors**: Global error handling
   ```typescript
   api.interceptors.response.use(
     response => response,
     error => {
       if (error.response?.status === 401) {
         // Redirect to login
       }
       toast.error(error.message);
       return Promise.reject(error);
     }
   );
   ```

4. **Loading States**: Skeleton loaders for better UX (shows layout immediately)

This ensures errors are caught at appropriate levels and users always see meaningful feedback.

### Backend API Questions

#### Q11: Walk me through what happens when a user borrows a book. Include the full request flow.

**Answer**: 
1. **Frontend**: User clicks "Borrow" → React Query mutation calls `POST /api/v1/borrows`
2. **Nginx**: Receives request, routes to backend
3. **Backend Middleware Chain**:
   - CORS middleware
   - AuthMiddleware: Validates JWT, extracts user from token, sets in context
4. **Route Handler**: `createBorrowHandler` receives request
5. **Validation**: Validates request body (bookId required)
6. **Repository Layer**:
   - Begin database transaction
   - SELECT FOR UPDATE on book (lock row)
   - Check `availableCopies > 0`
   - Check user doesn't have too many active borrows
   - Create Borrow record with dueDate = now + 14 days
   - Decrement `book.availableCopies`
   - Generate notification record
   - Commit transaction
7. **Response**: Return borrow record with status 201
8. **Frontend**: React Query invalidates book cache, shows success toast, redirects to borrows page

This entire flow is atomic - either all succeed or all roll back.

#### Q12: How do you implement pagination in your API? Show me the implementation.

**Answer**: Consistent pagination pattern across all list endpoints:

```go
type PaginationRequest struct {
    Page  int `form:"page" validate:"omitempty,min=1"`
    Limit int `form:"limit" validate:"omitempty,min=1,max=100"`
}

func getBooksHandler(db *database.Database) gin.HandlerFunc {
    return func(c *gin.Context) {
        var req PaginationRequest
        c.ShouldBindQuery(&req)
        
        // Defaults
        if req.Page < 1 {
            req.Page = 1
        }
        if req.Limit < 1 {
            req.Limit = 20
        }
        
        // Calculate offset
        offset := (req.Page - 1) * req.Limit
        
        // Query with pagination
        var books []Book
        var total int64
        db.Model(&Book{}).Count(&total)
        db.Offset(offset).Limit(req.Limit).Find(&books)
        
        // Response
        c.JSON(200, gin.H{
            "books": books,
            "pagination": gin.H{
                "page": req.Page,
                "limit": req.Limit,
                "total": total,
                "pages": int(math.Ceil(float64(total) / float64(req.Limit))),
            },
        })
    }
}
```

Benefits:
- Consistent API across all endpoints
- Prevents loading too much data
- Frontend can implement infinite scroll or page numbers

#### Q13: How do you handle file uploads? (if applicable, or how would you?)

**Answer**: For book cover images, I would implement:

**Backend (Go)**:
```go
// Handler
func uploadBookImage(c *gin.Context) {
    file, err := c.FormFile("image")
    if err != nil {
        c.JSON(400, gin.H{"error": "No file uploaded"})
        return
    }
    
    // Validate file type and size
    if !isValidImageType(file.Header.Get("Content-Type")) {
        c.JSON(400, gin.H{"error": "Invalid file type"})
        return
    }
    
    // Generate unique filename
    filename := generateUniqueFilename(file.Filename)
    
    // Save to storage (local filesystem or S3)
    path := filepath.Join("uploads", filename)
    c.SaveUploadedFile(file, path)
    
    // Store URL in database
    imageURL := fmt.Sprintf("/uploads/%s", filename)
    // Update book record with imageURL
}
```

**Frontend**:
```typescript
const uploadImage = async (file: File) => {
  const formData = new FormData();
  formData.append('image', file);
  
  await api.post('/books/:id/image', formData, {
    headers: { 'Content-Type': 'multipart/form-data' },
  });
};
```

For production, I'd use cloud storage (AWS S3, Cloudinary) instead of local filesystem.

#### Q14: Explain your error handling strategy in the backend.

**Answer**: Structured error handling at multiple levels:

**1. Validation Errors**:
```go
if err := c.ShouldBindJSON(&req); err != nil {
    c.JSON(400, gin.H{
        "error": "Validation failed",
        "details": err.Error(),
    })
    return
}
```

**2. Business Logic Errors**:
```go
if !book.IsAvailable() {
    c.JSON(400, gin.H{
        "error": "Book not available",
        "message": "This book is currently checked out",
    })
    return
}
```

**3. Database Errors**:
```go
if err := db.Create(&borrow).Error; err != nil {
    if errors.Is(err, gorm.ErrDuplicatedKey) {
        c.JSON(409, gin.H{"error": "Duplicate entry"})
        return
    }
    c.JSON(500, gin.H{"error": "Database error"})
    return
}
```

**4. Global Error Handler**:
```go
func errorHandler() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next()
        
        if len(c.Errors) > 0 {
            err := c.Errors.Last()
            c.JSON(500, gin.H{
                "error": "Internal server error",
                "message": err.Error(),
            })
        }
    }
}
```

Consistent error format helps frontend handle errors uniformly.

#### Q15: How would you implement rate limiting on your API?

**Answer**: I would use middleware with Redis for distributed rate limiting:

```go
func RateLimitMiddleware() gin.HandlerFunc {
    limiter := rate.NewLimiter(rate.Every(time.Second), 10) // 10 requests/second
    
    return func(c *gin.Context) {
        userID := getUserIDFromContext(c) // or IP address
        
        // Redis key: "rate_limit:user:123" or "rate_limit:ip:192.168.1.1"
        key := fmt.Sprintf("rate_limit:%s", userID)
        
        // Check and increment in Redis (atomic operation)
        count, err := redis.Incr(ctx, key)
        if err != nil {
            c.Next() // Allow request if Redis fails
            return
        }
        
        // Set expiry on first request
        if count == 1 {
            redis.Expire(ctx, key, time.Minute)
        }
        
        // Check limit (e.g., 60 requests per minute)
        if count > 60 {
            c.JSON(429, gin.H{
                "error": "Rate limit exceeded",
                "message": "Too many requests, please try again later",
            })
            c.Abort()
            return
        }
        
        c.Next()
    }
}
```

This prevents API abuse while allowing legitimate usage. Different limits for different endpoints (e.g., stricter for login attempts).

### Database Design Questions

#### Q16: Explain your database indexing strategy. What indexes do you have and why?

**Answer**: Strategic indexing for query performance:

**Primary Indexes** (automatic):
- All `id` columns (primary keys)

**Foreign Key Indexes**:
- `borrows.user_id`, `borrows.book_id` - For JOINs in user history queries
- `fines.borrow_id` - For fine lookup by borrow

**Search Indexes**:
- `users.email` (UNIQUE) - Login lookups must be fast
- `books.title`, `books.author`, `books.category` - For search queries
- `books.isbn` (UNIQUE) - Book identification

**Filtering Indexes**:
- `borrows.status` - Filter active/returned borrows
- `borrows.due_date` - Find overdue books efficiently

**Composite Indexes** (future optimization):
- `(user_id, status)` on borrows - For "user's active borrows" query
- `(book_id, status)` on borrows - For book availability checks

Trade-off: Indexes speed up reads but slow down writes. For library system (read-heavy), this is optimal.

#### Q17: How do you handle database migrations? Show me an example.

**Answer**: GORM AutoMigrate for development, SQL migration files for production:

**Development**:
```go
func (d *Database) AutoMigrate() error {
    return d.DB.AutoMigrate(
        &models.User{},
        &models.Book{},
        &models.Borrow{},
        // ...
    )
}
```

**Production** (manual SQL migrations):
```sql
-- migrations/001_create_users_table.sql
CREATE TABLE IF NOT EXISTS users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    -- ...
);

CREATE INDEX idx_users_email ON users(email);

-- migrations/002_add_phone_to_users.sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
```

For production, I'd use a migration tool like `golang-migrate`:
- Version control for schema changes
- Rollback capability
- Production-safe migrations

Best practice: Never modify existing migrations, always add new ones.

#### Q18: How do you prevent SQL injection in your application?

**Answer**: Multiple layers of protection:

**1. GORM Parameterized Queries** (Primary defense):
```go
// Safe - parameterized
db.Where("email = ?", email).First(&user)

// Safe - method chaining
db.Where("user_id = ?", userID).Find(&borrows)

// NEVER do this:
// db.Where("email = '" + email + "'")  // SQL Injection risk!
```

**2. Input Validation**:
```go
type LoginRequest struct {
    Email string `json:"email" validate:"required,email"` // Email format validation
}
```

**3. Prepared Statements**: GORM uses prepared statements automatically

**4. Least Privilege**: Database user has minimal required permissions (no DROP TABLE, etc.)

**5. Input Sanitization**: Validate and sanitize all user inputs before database queries

GORM's query builder ensures all values are parameterized, making SQL injection nearly impossible if used correctly.

#### Q19: Explain your transaction strategy. When do you use transactions?

**Answer**: Transactions for multi-step operations that must be atomic:

**Critical Transactions**:

1. **Borrowing a Book**:
   - Check availability
   - Create borrow record
   - Decrement available copies
   - Create notification
   → All must succeed or all fail

2. **Returning a Book**:
   - Update borrow status
   - Increment available copies
   - Create fine (if overdue)
   - Update notification
   → Atomic operation

3. **User Registration**:
   - Create user record
   - Send verification email (if email service fails, rollback optional)
   → Usually just user creation in transaction

**Implementation**:
```go
err := db.Transaction(func(tx *gorm.DB) error {
    // All operations use 'tx' instead of 'db'
    if err := tx.Create(&borrow).Error; err != nil {
        return err // Auto-rollback
    }
    if err := tx.Model(&book).Update("available_copies", gorm.Expr("available_copies - 1")).Error; err != nil {
        return err // Auto-rollback
    }
    return nil // Auto-commit
})
```

**Isolation Level**: Default (READ COMMITTED) is sufficient. For high concurrency, could use SERIALIZABLE for borrow operations.

#### Q20: How would you optimize a slow query that fetches a user's borrowing history with book details?

**Answer**: Several optimization strategies:

**Problem Query**:
```go
var borrows []Borrow
db.Where("user_id = ?", userID).Find(&borrows)
for _, borrow := range borrows {
    db.Model(&borrow).Association("Book").Find(&borrow.Book) // N+1 problem!
}
```

**Solution 1: Eager Loading**:
```go
var borrows []Borrow
db.Preload("Book").Where("user_id = ?", userID).Find(&borrows)
// Single query with JOIN
```

**Solution 2: Selective Loading**:
```go
db.Preload("Book", func(db *gorm.DB) *gorm.DB {
    return db.Select("id, title, author, isbn") // Only needed columns
}).Where("user_id = ?", userID).Find(&borrows)
```

**Solution 3: Pagination**:
```go
db.Preload("Book").
    Where("user_id = ?", userID).
    Order("created_at DESC").
    Limit(20).
    Offset(offset).
    Find(&borrows)
```

**Solution 4: Database Index**:
```sql
CREATE INDEX idx_borrows_user_id_created_at ON borrows(user_id, created_at DESC);
```

**Solution 5: Caching** (if data doesn't change often):
```go
// Cache for 5 minutes
cacheKey := fmt.Sprintf("user_borrows:%d", userID)
if cached := redis.Get(cacheKey); cached != nil {
    return deserialize(cached)
}
// ... query database and cache result
```

Combining these can reduce query time from seconds to milliseconds.

### Security Questions

#### Q21: Explain your JWT implementation. How do access and refresh tokens work?

**Answer**: Dual-token system for security and UX:

**Access Token**:
- Short-lived (24 hours)
- Contains: user_id, email, role
- Used for API authentication
- Stored in localStorage

**Refresh Token**:
- Long-lived (7 days)
- Used only to get new access tokens
- Stored in httpOnly cookie (more secure) or localStorage

**Flow**:
1. Login → Backend generates both tokens → Frontend stores both
2. API Request → Frontend includes access token in Authorization header
3. Token Expired (401) → Frontend calls `/auth/refresh` with refresh token
4. Backend validates refresh token → Issues new access + refresh token pair
5. Frontend updates tokens → Retries original request

**Security Features**:
- Tokens signed with HS256
- Refresh token rotation (new refresh token on each refresh)
- Token validation on every request
- Short access token expiry limits exposure if compromised

**Implementation**:
```go
func GenerateTokenPair(user *User) (*TokenPair, error) {
    accessToken := generateToken(user, 24*time.Hour, "access")
    refreshToken := generateToken(user, 7*24*time.Hour, "refresh")
    return &TokenPair{AccessToken: accessToken, RefreshToken: refreshToken}, nil
}
```

#### Q22: How do you implement role-based access control (RBAC)?

**Answer**: Multi-level authorization:

**1. Middleware-Based**:
```go
func AdminMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        user, exists := c.Get("user")
        if !exists || !user.(*User).Role != "admin" {
            c.JSON(403, gin.H{"error": "Admin access required"})
            c.Abort()
            return
        }
        c.Next()
    }
}

// Usage
adminRoutes := api.Group("/admin", AuthMiddleware(), AdminMiddleware())
```

**2. Route-Level Protection**:
- Public routes: No middleware
- User routes: `AuthMiddleware()`
- Admin routes: `AuthMiddleware() + AdminMiddleware()`
- Librarian routes: `AuthMiddleware() + LibrarianMiddleware()`

**3. Endpoint-Level Checks** (within handlers):
```go
func deleteBookHandler(c *gin.Context) {
    user := getUserFromContext(c)
    if !user.IsAdmin() {
        c.JSON(403, gin.H{"error": "Forbidden"})
        return
    }
    // Delete book...
}
```

**4. Database-Level**: 
- Role stored in user table
- Checked on every authenticated request
- Can be updated by admin (role promotion)

**Frontend**: Route guards prevent rendering, but backend validation is the source of truth.

#### Q23: How do you prevent XSS attacks?

**Answer**: Multiple layers of XSS prevention:

**1. React's Built-in Escaping**:
- React automatically escapes content in JSX
```typescript
// Safe - React escapes
<div>{userInput}</div>

// Dangerous - but we validate input
<div dangerouslySetInnerHTML={{__html: userInput}} /> // Only if sanitized
```

**2. Content Security Policy (CSP)**:
```nginx
# Nginx headers
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'";
```

**3. Input Validation**:
```go
// Backend validation
type BookCreateRequest struct {
    Title string `validate:"required,max=255,no_html"` // Custom validator to strip HTML
}
```

**4. Output Encoding**:
- All user input is HTML-escaped before rendering
- JSON responses are properly encoded

**5. Sanitization** (if HTML input needed):
- Use library like `bluemonday` (Go) or `DOMPurify` (JS)
- Whitelist allowed HTML tags/attributes only

**Best Practice**: Never trust user input. Validate, sanitize, and escape at every layer.

#### Q24: How do you secure your API against CSRF attacks?

**Answer**: JWT in Authorization header (not cookies) reduces CSRF risk, but additional measures:

**1. SameSite Cookies** (if using cookies):
```go
cookie := http.Cookie{
    Name:     "refresh_token",
    Value:    token,
    SameSite: http.SameSiteStrictMode,
    HttpOnly: true,
    Secure:   true, // HTTPS only
}
```

**2. Origin/Referer Validation**:
```go
func CSRFMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        origin := c.GetHeader("Origin")
        referer := c.GetHeader("Referer")
        
        allowedOrigins := []string{"https://library.example.com"}
        if !contains(allowedOrigins, origin) {
            c.JSON(403, gin.H{"error": "Invalid origin"})
            c.Abort()
            return
        }
        c.Next()
    }
}
```

**3. Custom Headers**:
- Require custom header (e.g., `X-Requested-With`)
- Browsers don't allow custom headers in cross-origin requests

**4. CORS Configuration**:
```go
config := cors.DefaultConfig()
config.AllowOrigins = []string{"https://library.example.com"}
config.AllowCredentials = true
```

**Note**: Since we use JWT in Authorization header (not cookies), CSRF is less of a concern, but these measures add defense-in-depth.

#### Q25: How do you handle password security? Explain bcrypt.

**Answer**: Industry-standard password hashing with bcrypt:

**Hashing on Registration**:
```go
func HashPassword(password string) (string, error) {
    hashedBytes, err := bcrypt.GenerateFromPassword(
        []byte(password),
        bcrypt.DefaultCost, // Cost factor 10
    )
    return string(hashedBytes), err
}
```

**Verification on Login**:
```go
func VerifyPassword(hashedPassword, password string) bool {
    err := bcrypt.CompareHashAndPassword(
        []byte(hashedPassword),
        []byte(password),
    )
    return err == nil
}
```

**Why bcrypt**:
- **Adaptive Cost**: Cost factor 10 means 2^10 = 1024 iterations (adjustable for hardware)
- **Salt**: Automatically generates unique salt per password (prevents rainbow table attacks)
- **Slow by Design**: Intentionally slow to prevent brute force (100ms+ per hash)
- **Industry Standard**: Widely used, well-tested

**Security Practices**:
- Never store plaintext passwords
- Minimum 8 characters (enforced in validation)
- Consider password strength requirements (uppercase, numbers, symbols)
- Future: Implement password complexity rules

**Cost Factor Trade-off**: Higher cost = more secure but slower. Cost 10 is good balance for web apps.

### Performance & Scalability Questions

#### Q26: How would you scale this application to handle 1 million users?

**Answer**: Multi-dimensional scaling strategy:

**1. Horizontal Scaling (Backend)**:
- Run multiple backend instances behind load balancer
- Stateless design (JWT) makes this easy
- Use Kubernetes for orchestration
- Auto-scale based on CPU/memory metrics

**2. Database Scaling**:
- **Read Replicas**: Separate read and write operations
  - Write to master, read from replicas
  - Use connection pool routing
- **Database Sharding**: Partition by user_id (e.g., shard by user_id % 10)
- **Connection Pooling**: PgBouncer for efficient connection management

**3. Caching Layer**:
- **Redis Clustering**: Distributed cache for frequently accessed data
- **CDN**: Serve static assets globally
- **Application Cache**: Cache book lists, categories, user profiles

**4. Message Queue**:
- **Async Operations**: Use RabbitMQ/Kafka for notification sending, email, reports
- **Event-Driven**: Decouple heavy operations from request path

**5. Monitoring & Observability**:
- **APM**: Application Performance Monitoring (New Relic, Datadog)
- **Logging**: Centralized logging (ELK stack)
- **Metrics**: Prometheus + Grafana for metrics

**6. Database Optimization**:
- Proper indexing (already done)
- Query optimization
- Archive old data (move old borrows to archive table)

**Estimated Capacity**: 
- 10 backend instances (1000 req/s each) = 10,000 req/s
- Read replicas handle read-heavy workload
- Can handle 1M+ users with proper caching

#### Q27: How do you optimize frontend performance?

**Answer**: Multiple optimization strategies:

**1. Code Splitting**:
```typescript
// Lazy load routes
const AdminDashboard = React.lazy(() => import('./pages/Admin/AdminDashboard'));

// Lazy load heavy components
const Chart = React.lazy(() => import('./components/Chart'));
```

**2. React Query Caching**:
- Automatic request deduplication
- Background refetching
- Stale-while-revalidate pattern

**3. Memoization**:
```typescript
// Memoize expensive computations
const filteredBooks = useMemo(() => {
  return books.filter(b => b.category === selectedCategory);
}, [books, selectedCategory]);

// Memoize components
const BookCard = React.memo(({ book }) => { ... });
```

**4. Image Optimization**:
- Lazy loading images
- Responsive images (srcset)
- WebP format with fallback
- Image CDN

**5. Bundle Optimization**:
- Tree shaking (Vite does this automatically)
- Minification
- Gzip compression (Nginx)
- Code splitting reduces initial bundle size

**6. Virtual Scrolling** (for large lists):
```typescript
import { FixedSizeList } from 'react-window';

<FixedSizeList
  height={600}
  itemCount={books.length}
  itemSize={100}
>
  {BookRow}
</FixedSizeList>
```

**Results**: Initial load < 2s, navigation < 100ms, smooth 60fps interactions.

#### Q28: Explain your approach to database query optimization.

**Answer**: Systematic query optimization:

**1. Indexing Strategy** (covered in Q16)
- Strategic indexes on foreign keys and frequently queried columns

**2. Query Analysis**:
```sql
-- Use EXPLAIN ANALYZE to identify slow queries
EXPLAIN ANALYZE
SELECT * FROM borrows 
WHERE user_id = 123 
ORDER BY created_at DESC 
LIMIT 20;
```

**3. Avoid N+1 Queries**:
```go
// Bad: N+1 queries
var borrows []Borrow
db.Find(&borrows)
for _, b := range borrows {
    db.Model(&b).Association("Book").Find(&b.Book)
}

// Good: Eager loading
db.Preload("Book").Find(&borrows)
```

**4. Selective Column Loading**:
```go
// Only select needed columns
db.Select("id, title, author").Find(&books)

// Instead of
db.Find(&books) // Selects all columns
```

**5. Pagination**:
- Always paginate list queries
- Use cursor-based pagination for large datasets

**6. Query Caching**:
- Cache expensive queries (analytics, reports)
- Redis cache with appropriate TTL

**7. Database Connection Pooling**:
```go
sqlDB.SetMaxIdleConns(10)
sqlDB.SetMaxOpenConns(100)
sqlDB.SetConnMaxLifetime(time.Hour)
```

**Monitoring**: Track slow query log, set up alerts for queries > 1s.

#### Q29: How would you implement real-time notifications without polling?

**Answer**: Replace polling with WebSockets:

**Backend (Go with Gorilla WebSocket)**:
```go
func handleWebSocket(c *gin.Context) {
    conn, err := upgrader.Upgrade(c.Writer, c.Request, nil)
    if err != nil {
        return
    }
    defer conn.Close()
    
    userID := getUserIDFromContext(c)
    
    // Register connection
    hub.register <- &Client{
        userID: userID,
        conn:   conn,
        send:   make(chan []byte, 256),
    }
    
    // Listen for messages and send notifications
    go client.writePump()
    client.readPump()
}

// Broadcast notification
func (h *Hub) broadcastToUser(userID uint, message []byte) {
    for client := range h.clients {
        if client.userID == userID {
            select {
            case client.send <- message:
            default:
                close(client.send)
                delete(h.clients, client)
            }
        }
    }
}
```

**Frontend**:
```typescript
const useWebSocket = (userID: number) => {
  const [notifications, setNotifications] = useState([]);
  
  useEffect(() => {
    const ws = new WebSocket(`ws://localhost:8080/ws?userID=${userID}`);
    
    ws.onmessage = (event) => {
      const notification = JSON.parse(event.data);
      setNotifications(prev => [notification, ...prev]);
    };
    
    return () => ws.close();
  }, [userID]);
  
  return notifications;
};
```

**Alternative: Server-Sent Events (SSE)**:
- Simpler than WebSockets (one-way, HTTP-based)
- Good for notification-only use case
- Automatic reconnection

**Trade-off**: WebSockets require connection management and can be complex. Polling is simpler but less efficient.

#### Q30: How do you handle database connection pool exhaustion under high load?

**Answer**: Proactive connection pool management:

**1. Pool Configuration**:
```go
sqlDB.SetMaxIdleConns(10)      // Keep 10 idle connections ready
sqlDB.SetMaxOpenConns(100)     // Maximum 100 concurrent connections
sqlDB.SetConnMaxLifetime(time.Hour) // Recycle connections after 1 hour
sqlDB.SetConnMaxIdleTime(10 * time.Minute) // Close idle connections
```

**2. Connection Pool Monitoring**:
```go
stats := sqlDB.Stats()
log.Printf("Open connections: %d, Idle: %d, InUse: %d", 
    stats.OpenConnections, stats.Idle, stats.InUse)
```

**3. Circuit Breaker Pattern**:
- If pool exhausted, return error immediately instead of waiting
- Prevents cascade failures

**4. Database-Level Solutions**:
- **PgBouncer**: Connection pooler for PostgreSQL
  - Reduces actual DB connections (100 app connections → 20 DB connections)
  - Transaction pooling mode for efficiency

**5. Read Replicas**:
- Distribute read queries across replicas
- Reduces load on primary database

**6. Query Optimization**:
- Faster queries = connections released sooner
- Avoid long-running queries

**7. Timeout Configuration**:
```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
db.WithContext(ctx).Find(&books) // Query times out after 5s
```

**8. Load Balancing**:
- Multiple backend instances = more connection pools
- Each instance has its own pool

**Alerting**: Set up alerts when connection pool usage > 80%.

### Debugging & Optimization Questions

#### Q31: How do you debug a production issue where users can't borrow books?

**Answer**: Systematic debugging approach:

**1. Check Logs**:
```bash
# Backend logs
docker logs library_backend --tail 100

# Look for errors
grep "ERROR" logs/application.log
```

**2. Check Database**:
```sql
-- Check if books are actually available
SELECT id, title, available_copies, total_copies 
FROM books 
WHERE id = <book_id>;

-- Check for locks
SELECT * FROM pg_locks WHERE relation = 'books'::regclass;
```

**3. Check API Response**:
- Monitor API responses (status codes, error messages)
- Check if requests reach backend (Nginx logs)

**4. Common Issues**:
- **Race condition**: Two users borrow last copy → one fails (expected)
- **Transaction timeout**: Long-running transaction blocks others
- **Connection pool exhausted**: Too many concurrent requests
- **Index missing**: Slow query causes timeout

**5. Add Debug Logging**:
```go
func createBorrowHandler(c *gin.Context) {
    log.Printf("Borrow request: userID=%d, bookID=%d", userID, bookID)
    
    var book Book
    if err := db.First(&book, bookID).Error; err != nil {
        log.Printf("ERROR: Book not found: %v", err)
        // ...
    }
    
    log.Printf("Book availability: %d/%d", book.AvailableCopies, book.TotalCopies)
    // ...
}
```

**6. Reproduce Locally**:
- Try to reproduce with same conditions
- Use database dump from production (sanitized)

**7. Monitor Metrics**:
- Request rate, error rate, response time
- Database connection pool usage
- Slow query log

**8. Rollback Strategy**:
- If issue is from recent deploy, rollback
- Feature flag to disable feature temporarily

#### Q32: How would you optimize a slow admin dashboard that shows statistics?

**Answer**: Multiple optimization strategies:

**1. Caching**:
```go
func getAdminStats() (*AdminStats, error) {
    cacheKey := "admin_stats"
    
    // Check cache first (5 minute TTL)
    if cached := redis.Get(cacheKey); cached != nil {
        return deserialize(cached), nil
    }
    
    // Compute stats (expensive queries)
    stats := &AdminStats{
        TotalUsers: db.Model(&User{}).Count(&count),
        TotalBooks: db.Model(&Book{}).Count(&count),
        // ... more expensive queries
    }
    
    // Cache result
    redis.Set(cacheKey, serialize(stats), 5*time.Minute)
    return stats, nil
}
```

**2. Database Indexes**:
- Ensure all COUNT queries use indexes
- Covering indexes for aggregate queries

**3. Materialized Views** (PostgreSQL):
```sql
CREATE MATERIALIZED VIEW admin_stats_view AS
SELECT 
    (SELECT COUNT(*) FROM users) as total_users,
    (SELECT COUNT(*) FROM books) as total_books,
    -- ...
;

-- Refresh periodically
REFRESH MATERIALIZED VIEW admin_stats_view;
```

**4. Background Jobs**:
- Compute stats in background (every 5 minutes)
- Store in Redis or cache table
- Dashboard reads from cache (instant)

**5. Incremental Updates**:
- Instead of recalculating everything, update counters on events
```go
// On user registration
redis.Incr("stats:total_users")

// On book creation
redis.Incr("stats:total_books")
```

**6. Pagination for Lists**:
- Don't load all users/books for dashboard
- Show summary + "View All" link
- Load top 10 recent items only

**7. Lazy Loading**:
- Load stats progressively (core stats first, then detailed metrics)
- Use React Query's `staleTime` to prevent unnecessary refetches

---

#### Q33: How would you implement search across multiple fields (title, author, ISBN, category) efficiently?

**Answer**: Multiple strategies for efficient multi-field search:

**1. Full-Text Search (PostgreSQL)**:
```sql
-- Create full-text search index
CREATE INDEX idx_books_search ON books 
USING gin(to_tsvector('english', 
    coalesce(title, '') || ' ' || 
    coalesce(author, '') || ' ' || 
    coalesce(isbn, '') || ' ' || 
    coalesce(category, '')
));

-- Query with full-text search
SELECT * FROM books 
WHERE to_tsvector('english', title || ' ' || author || ' ' || isbn || ' ' || category) 
      @@ to_tsquery('english', 'gatsby');
```

**2. Separate Indexes**:
```sql
CREATE INDEX idx_books_title ON books USING gin(to_tsvector('english', title));
CREATE INDEX idx_books_author ON books USING gin(to_tsvector('english', author));
```

**3. Backend Implementation**:
```go
func SearchBooks(db *gorm.DB, query string) ([]Book, error) {
    var books []Book
    
    // Use ILIKE for case-insensitive partial matching
    err := db.Where("title ILIKE ? OR author ILIKE ? OR isbn ILIKE ? OR category ILIKE ?",
        "%"+query+"%", "%"+query+"%", "%"+query+"%", "%"+query+"%").
        Limit(50).
        Find(&books).Error
    
    return books, err
}
```

**4. Ranking/Relevance**:
```sql
-- Order by relevance
SELECT *, 
    ts_rank(to_tsvector(title || ' ' || author), to_tsquery('gatsby')) as rank
FROM books
WHERE to_tsvector(title || ' ' || author) @@ to_tsquery('gatsby')
ORDER BY rank DESC;
```

**5. Frontend Debouncing**:
```typescript
const [searchTerm, setSearchTerm] = useState('');
const debouncedSearch = useDebounce(searchTerm, 300); // Wait 300ms

const { data } = useQuery({
  queryKey: ['books', { search: debouncedSearch }],
  queryFn: () => bookApi.getBooks({ search: debouncedSearch }),
});
```

---

#### Q34: How do you handle soft deletes? Why use them?

**Answer**: Soft deletes mark records as deleted without removing them from database.

**Implementation with GORM**:
```go
// Model includes DeletedAt
type Book struct {
    ID        uint           `gorm:"primaryKey"`
    Title     string
    DeletedAt gorm.DeletedAt `gorm:"index"` // Soft delete field
}

// Soft delete
db.Delete(&book) // Sets DeletedAt timestamp

// Hard delete (permanent)
db.Unscoped().Delete(&book)

// Query excludes soft-deleted by default
db.Find(&books) // Only active books

// Include soft-deleted
db.Unscoped().Find(&books) // All books including deleted

// Find only deleted
db.Unscoped().Where("deleted_at IS NOT NULL").Find(&books)
```

**Why Use Soft Deletes**:
- **Data Recovery**: Can restore accidentally deleted data
- **Audit Trail**: Maintain history of deletions
- **Referential Integrity**: Maintain foreign key relationships
- **Analytics**: Track deletion patterns
- **Compliance**: Regulatory requirements for data retention

**Considerations**:
- Need unique indexes to handle soft-deleted duplicates
- Queries must exclude deleted records (GORM does this automatically)
- Cleanup job needed for old soft-deleted records

---

#### Q35: How would you implement book recommendations based on user borrowing history?

**Answer**: Recommendation engine using collaborative filtering:

**1. Database Schema**:
```sql
CREATE TABLE user_book_preferences (
    user_id BIGINT REFERENCES users(id),
    book_id BIGINT REFERENCES books(id),
    rating INTEGER DEFAULT 0, -- -1, 0, 1 (dislike, neutral, like)
    borrowed_count INTEGER DEFAULT 0,
    last_borrowed_at TIMESTAMP,
    PRIMARY KEY (user_id, book_id)
);

CREATE TABLE book_similarity (
    book_id_1 BIGINT,
    book_id_2 BIGINT,
    similarity_score DECIMAL(5,4),
    PRIMARY KEY (book_id_1, book_id_2)
);
```

**2. Recommendation Algorithm**:
```go
func GetRecommendations(userID uint, limit int) ([]Book, error) {
    // Get user's borrowed books
    var userBooks []Borrow
    db.Where("user_id = ?", userID).
       Preload("Book").
       Find(&userBooks)
    
    // Find similar users (users who borrowed same books)
    var similarUsers []uint
    bookIDs := extractBookIDs(userBooks)
    
    db.Table("borrows").
       Select("user_id").
       Where("book_id IN ?", bookIDs).
       Where("user_id != ?", userID).
       Group("user_id").
       Having("COUNT(DISTINCT book_id) >= 2").
       Pluck("user_id", &similarUsers)
    
    // Get books borrowed by similar users (not yet borrowed by target user)
    var recommendations []Book
    db.Table("books").
       Joins("JOIN borrows ON borrows.book_id = books.id").
       Where("borrows.user_id IN ?", similarUsers).
       Where("books.id NOT IN ?", bookIDs).
       Group("books.id").
       Order("COUNT(borrows.id) DESC").
       Limit(limit).
       Find(&recommendations)
    
    return recommendations, nil
}
```

**3. Category-Based Fallback**:
```go
// If no similar users, recommend by category
func GetCategoryRecommendations(userID uint) []Book {
    // Get user's most borrowed categories
    var topCategories []string
    db.Table("borrows").
       Joins("JOIN books ON books.id = borrows.book_id").
       Where("borrows.user_id = ?", userID).
       Group("books.category").
       Order("COUNT(*) DESC").
       Limit(3).
       Pluck("books.category", &topCategories)
    
    // Recommend popular books in those categories
    var books []Book
    db.Where("category IN ?", topCategories).
       Where("available_copies > 0").
       Order("available_copies DESC").
       Limit(10).
       Find(&books)
    
    return books
}
```

**4. Caching Recommendations**:
```go
cacheKey := fmt.Sprintf("recommendations:%d", userID)
if cached := redis.Get(cacheKey); cached != nil {
    return deserialize(cached), nil
}

recommendations := computeRecommendations(userID)
redis.Set(cacheKey, serialize(recommendations), 1*time.Hour)
```

---

#### Q36: How do you handle timezone issues in your application?

**Answer**: Consistent timezone handling strategy:

**1. Database Storage**:
```sql
-- Use TIMESTAMP WITH TIME ZONE (stores UTC)
CREATE TABLE borrows (
    borrowed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    due_date DATE, -- DATE doesn't have timezone
    returned_at TIMESTAMP WITH TIME ZONE
);
```

**2. Backend (Always UTC)**:
```go
// Always work in UTC
type Borrow struct {
    BorrowedAt time.Time `gorm:"type:timestamptz"`
    DueDate    time.Time `gorm:"type:date"`
}

// Store in UTC
borrow.BorrowedAt = time.Now().UTC()

// Parse incoming dates in user's timezone, convert to UTC
loc, _ := time.LoadLocation("America/New_York")
userTime, _ := time.ParseInLocation("2006-01-02", dateString, loc)
utcTime := userTime.UTC()
```

**3. Frontend Display**:
```typescript
// Store user's timezone in user profile
const userTimezone = user.timezone || Intl.DateTimeFormat().resolvedOptions().timeZone;

// Convert UTC to user's timezone for display
const formatDate = (utcDate: string) => {
  return new Date(utcDate).toLocaleString('en-US', {
    timeZone: userTimezone,
    dateStyle: 'medium',
    timeStyle: 'short'
  });
};

// Send dates in ISO format (includes timezone)
const sendDate = new Date().toISOString(); // "2024-01-15T10:30:00.000Z"
```

**4. Date-Only Fields**:
```go
// For due_date (date only), normalize to start of day in UTC
func normalizeDate(d time.Time) time.Time {
    year, month, day := d.Year(), d.Month(), d.Day()
    return time.Date(year, month, day, 0, 0, 0, 0, time.UTC)
}
```

**5. Comparison Logic**:
```go
// Compare dates correctly
today := time.Now().UTC().Truncate(24 * time.Hour)
dueDate := borrow.DueDate.Truncate(24 * time.Hour)
isOverdue := dueDate.Before(today)
```

---

#### Q37: How would you implement an audit log for tracking all important actions?

**Answer**: Comprehensive audit logging system:

**1. Audit Log Table**:
```sql
CREATE TABLE audit_logs (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    action VARCHAR(50) NOT NULL, -- 'create', 'update', 'delete', 'borrow', 'return'
    entity_type VARCHAR(50) NOT NULL, -- 'book', 'borrow', 'user'
    entity_id BIGINT NOT NULL,
    old_values JSONB, -- Snapshot before change
    new_values JSONB, -- Snapshot after change
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_created ON audit_logs(created_at);
```

**2. Audit Logger Service**:
```go
type AuditLog struct {
    UserID     uint
    Action     string
    EntityType string
    EntityID   uint
    OldValues  map[string]interface{}
    NewValues  map[string]interface{}
    IPAddress  string
    UserAgent  string
}

func LogAudit(db *gorm.DB, log AuditLog) error {
    auditLog := models.AuditLog{
        UserID:     log.UserID,
        Action:     log.Action,
        EntityType: log.EntityType,
        EntityID:   log.EntityID,
        OldValues:  log.OldValues,
        NewValues:  log.NewValues,
        IPAddress:  log.IPAddress,
        UserAgent:  log.UserAgent,
    }
    return db.Create(&auditLog).Error
}
```

**3. Middleware for Automatic Logging**:
```go
func AuditMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Get user from context
        userID, exists := c.Get("userID")
        if !exists {
            c.Next()
            return
        }
        
        // Log request
        auditLog := AuditLog{
            UserID:    userID.(uint),
            Action:    c.Request.Method,
            IPAddress: c.ClientIP(),
            UserAgent: c.Request.UserAgent(),
        }
        
        // Store in context for handler to use
        c.Set("auditLog", auditLog)
        c.Next()
    }
}
```

**4. Handler Integration**:
```go
func createBookHandler(c *gin.Context) {
    var req BookCreateRequest
    c.ShouldBindJSON(&req)
    
    book := Book{Title: req.Title, ...}
    db.Create(&book)
    
    // Log audit
    auditLog, _ := c.Get("auditLog").(AuditLog)
    auditLog.Action = "create"
    auditLog.EntityType = "book"
    auditLog.EntityID = book.ID
    auditLog.NewValues = req
    LogAudit(db, auditLog)
}
```

**5. Query Audit Logs**:
```go
func GetAuditLogs(db *gorm.DB, filters AuditLogFilters) ([]AuditLog, error) {
    query := db.Model(&AuditLog{})
    
    if filters.UserID != 0 {
        query = query.Where("user_id = ?", filters.UserID)
    }
    if filters.EntityType != "" {
        query = query.Where("entity_type = ?", filters.EntityType)
    }
    if filters.EntityID != 0 {
        query = query.Where("entity_id = ?", filters.EntityID)
    }
    
    var logs []AuditLog
    query.Order("created_at DESC").
          Limit(filters.Limit).
          Offset(filters.Offset).
          Find(&logs)
    
    return logs, nil
}
```

---

#### Q38: How would you implement a reservation queue system where users can reserve unavailable books?

**Answer**: FIFO reservation queue with automatic notification:

**1. Reservation Model Enhancement**:
```go
type Reservation struct {
    ID            uint
    UserID        uint
    BookID        uint
    Priority      int       // Lower number = higher priority (FIFO order)
    Status        string    // 'active', 'fulfilled', 'expired', 'cancelled'
    ReservedAt    time.Time
    ExpiresAt     time.Time
    FulfilledAt   *time.Time
}
```

**2. Create Reservation**:
```go
func CreateReservation(db *gorm.DB, userID, bookID uint) (*Reservation, error) {
    // Check if book is available
    var book Book
    if err := db.First(&book, bookID).Error; err != nil {
        return nil, err
    }
    
    // If available, don't allow reservation
    if book.IsAvailable() {
        return nil, errors.New("book is available, please borrow instead")
    }
    
    // Get next priority (highest priority number + 1)
    var maxPriority int
    db.Model(&Reservation{}).
       Where("book_id = ? AND status = 'active'", bookID).
       Select("COALESCE(MAX(priority), 0)").
       Scan(&maxPriority)
    
    reservation := Reservation{
        UserID:     userID,
        BookID:     bookID,
        Priority:   maxPriority + 1,
        Status:     "active",
        ReservedAt: time.Now(),
        ExpiresAt:  time.Now().Add(7 * 24 * time.Hour), // 7 days
    }
    
    db.Create(&reservation)
    
    // Generate notification
    notification := Notification{
        UserID:   userID,
        Title:    "Reservation Created",
        Message:  fmt.Sprintf("You're #%d in queue for %s", reservation.Priority, book.Title),
        Category: "reservation",
    }
    db.Create(&notification)
    
    return &reservation, nil
}
```

**3. Process Reservation on Book Return**:
```go
func ProcessReservationsOnReturn(db *gorm.DB, bookID uint) error {
    // Get next reservation in queue (highest priority = lowest number)
    var reservation Reservation
    if err := db.Where("book_id = ? AND status = 'active'", bookID).
               Order("priority ASC").
               First(&reservation).Error; err != nil {
        return nil // No reservations
    }
    
    // Check if still valid (not expired)
    if time.Now().After(reservation.ExpiresAt) {
        // Mark as expired
        reservation.Status = "expired"
        db.Save(&reservation)
        
        // Process next reservation
        return ProcessReservationsOnReturn(db, bookID)
    }
    
    // Create borrow automatically (or notify user to borrow)
    borrow := Borrow{
        UserID:    reservation.UserID,
        BookID:    bookID,
        BorrowedAt: time.Now(),
        DueDate:   time.Now().AddDate(0, 0, 14),
        Status:    "borrowed",
    }
    db.Create(&borrow)
    
    // Mark reservation as fulfilled
    now := time.Now()
    reservation.Status = "fulfilled"
    reservation.FulfilledAt = &now
    db.Save(&reservation)
    
    // Update book availability
    var book Book
    db.First(&book, bookID)
    book.AvailableCopies-- // Already decremented by return, but double-check
    db.Save(&book)
    
    // Notify user
    notification := Notification{
        UserID:   reservation.UserID,
        Title:    "Book Available",
        Message:  fmt.Sprintf("%s is now available! You can pick it up.", book.Title),
        Category: "reservation",
    }
    db.Create(&notification)
    
    return nil
}
```

**4. Cleanup Expired Reservations (Cron Job)**:
```go
func CleanupExpiredReservations(db *gorm.DB) error {
    expiredReservations := []Reservation{}
    db.Where("status = 'active' AND expires_at < ?", time.Now()).
       Find(&expiredReservations)
    
    for _, reservation := range expiredReservations {
        reservation.Status = "expired"
        db.Save(&reservation)
        
        // Notify user
        notification := Notification{
            UserID:   reservation.UserID,
            Title:    "Reservation Expired",
            Message:  "Your reservation has expired",
            Category: "reservation",
        }
        db.Create(&notification)
        
        // Process next in queue
        ProcessReservationsOnReturn(db, reservation.BookID)
    }
    
    return nil
}
```

**5. Get User's Position in Queue**:
```go
func GetReservationPosition(db *gorm.DB, reservationID uint) (int, error) {
    var reservation Reservation
    if err := db.First(&reservation, reservationID).Error; err != nil {
        return 0, err
    }
    
    var count int
    db.Model(&Reservation{}).
       Where("book_id = ? AND status = 'active' AND priority < ?", 
             reservation.BookID, reservation.Priority).
       Count(&count)
    
    return count + 1, nil // Position (1-indexed)
}
```

---

#### Q39: How do you handle concurrent requests when updating the same resource?

**Answer**: Optimistic and pessimistic locking strategies:

**1. Optimistic Locking (Version Field)**:
```go
type Book struct {
    ID      uint
    Title   string
    Version int `gorm:"default:0"` // Version field for optimistic locking
}

// Update with version check
func UpdateBook(db *gorm.DB, book *Book) error {
    result := db.Model(book).
              Where("id = ? AND version = ?", book.ID, book.Version).
              Updates(map[string]interface{}{
                  "title": book.Title,
                  "version": book.Version + 1,
              })
    
    if result.RowsAffected == 0 {
        return errors.New("book was modified by another user, please refresh")
    }
    
    book.Version++
    return nil
}
```

**2. Pessimistic Locking (SELECT FOR UPDATE)**:
```go
func UpdateBookWithLock(db *gorm.DB, bookID uint, updates map[string]interface{}) error {
    return db.Transaction(func(tx *gorm.DB) error {
        var book Book
        
        // Lock row for update
        if err := tx.Set("gorm:query_option", "FOR UPDATE").
                   First(&book, bookID).Error; err != nil {
            return err
        }
        
        // Perform updates
        return tx.Model(&book).Updates(updates).Error
    })
}
```

**3. Frontend Optimistic Locking**:
```typescript
// Include version in update request
const updateBook = async (book: Book, updates: Partial<Book>) => {
  try {
    await bookApi.updateBook(book.id, {
      ...updates,
      version: book.version, // Include current version
    });
  } catch (error) {
    if (error.response?.status === 409) {
      // Conflict - refetch and show error
      toast.error("Book was modified by another user. Refreshing...");
      queryClient.invalidateQueries(['book', book.id]);
    }
  }
};
```

**4. Last-Write-Wins (Simple but can lose data)**:
```go
// Just update, overwrite any changes (not recommended for critical data)
db.Model(&book).Updates(updates)
```

**5. Conflict Resolution Strategy**:
```go
type ConflictResolution string

const (
    ConflictAbort     ConflictResolution = "abort"     // Fail on conflict
    ConflictOverwrite ConflictResolution = "overwrite" // Last write wins
    ConflictMerge     ConflictResolution = "merge"     // Merge changes
)

func UpdateBookWithConflictResolution(
    db *gorm.DB, 
    book *Book, 
    strategy ConflictResolution,
) error {
    switch strategy {
    case ConflictAbort:
        return UpdateBook(db, book) // Optimistic locking
    case ConflictOverwrite:
        return db.Save(book).Error
    case ConflictMerge:
        // Complex merge logic
        return mergeBookUpdates(db, book)
    }
    return nil
}
```

---

#### Q40: How would you implement fine calculation with different rates for different book types?

**Answer**: Flexible fine calculation system:

**1. Fine Configuration Table**:
```sql
CREATE TABLE fine_configurations (
    id BIGSERIAL PRIMARY KEY,
    book_category VARCHAR(100), -- NULL = default for all categories
    daily_rate DECIMAL(10,2) NOT NULL,
    max_fine DECIMAL(10,2),
    grace_period_days INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO fine_configurations (book_category, daily_rate, max_fine, grace_period_days) VALUES
(NULL, 0.50, 50.00, 0),                    -- Default
('Reference', 1.00, 100.00, 0),            -- Reference books cost more
('New Release', 0.75, 75.00, 1),           -- 1 day grace period
('Special Collection', 2.00, 200.00, 0);   -- Very expensive
```

**2. Fine Calculation Service**:
```go
type FineCalculator struct {
    db *gorm.DB
}

func (fc *FineCalculator) CalculateFine(borrow *Borrow) (float64, error) {
    // Get book category
    var book Book
    if err := fc.db.First(&book, borrow.BookID).Error; err != nil {
        return 0, err
    }
    
    // Get fine configuration for this category
    var config FineConfiguration
    query := fc.db.Where("book_category = ?", book.Category)
    
    if err := query.First(&config).Error; err != nil {
        // Fallback to default (NULL category)
        fc.db.Where("book_category IS NULL").First(&config)
    }
    
    // Check grace period
    daysOverdue := calculateDaysOverdue(borrow.DueDate)
    if daysOverdue <= config.GracePeriodDays {
        return 0, nil // No fine during grace period
    }
    
    // Calculate fine
    effectiveDays := daysOverdue - config.GracePeriodDays
    fine := float64(effectiveDays) * config.DailyRate
    
    // Apply maximum
    if config.MaxFine > 0 && fine > config.MaxFine {
        fine = config.MaxFine
    }
    
    return fine, nil
}

func calculateDaysOverdue(dueDate time.Time) int {
    now := time.Now()
    if now.Before(dueDate) {
        return 0
    }
    days := int(now.Sub(dueDate).Hours() / 24)
    return days
}
```

**3. Update Fine Configuration**:
```go
func UpdateFineConfiguration(
    db *gorm.DB, 
    category string, 
    config FineConfiguration,
) error {
    // Upsert configuration
    return db.Where("book_category = ?", category).
              Assign(FineConfiguration{
                  DailyRate:      config.DailyRate,
                  MaxFine:        config.MaxFine,
                  GracePeriodDays: config.GracePeriodDays,
              }).
              FirstOrCreate(&FineConfiguration{}, 
                  FineConfiguration{BookCategory: category}).
              Error
}
```

**4. Batch Fine Calculation**:
```go
func CalculateAllOverdueFines(db *gorm.DB) error {
    calculator := FineCalculator{db: db}
    
    var overdueBorrows []Borrow
    db.Where("status = 'borrowed' AND due_date < ?", time.Now()).
       Preload("Book").
       Find(&overdueBorrows)
    
    for _, borrow := range overdueBorrows {
        fineAmount, err := calculator.CalculateFine(&borrow)
        if err != nil {
            continue
        }
        
        if fineAmount > 0 {
            // Update borrow fine amount
            borrow.FineAmount = fineAmount
            borrow.Status = "overdue"
            db.Save(&borrow)
            
            // Create or update fine record
            fine := Fine{
                UserID:   borrow.UserID,
                BorrowID: borrow.ID,
                Amount:   fineAmount,
                Reason:   "overdue",
                IsPaid:   false,
            }
            
            db.Where("borrow_id = ? AND is_paid = false", borrow.ID).
               Assign(fine).
               FirstOrCreate(&Fine{})
        }
    }
    
    return nil
}
```

---

#### Q41: How do you ensure data consistency across microservices (if you were to split this into microservices)?

**Answer**: Distributed transaction patterns and eventual consistency:

**1. Saga Pattern (Event-Driven)**:
```go
// Book Service
func BorrowBook(bookID uint) error {
    // Decrease availability
    book.AvailableCopies--
    db.Save(&book)
    
    // Publish event
    eventBus.Publish("book.borrowed", BorrowEvent{
        BookID: bookID,
        Timestamp: time.Now(),
    })
    
    return nil
}

// Borrow Service (listens to events)
func HandleBookBorrowedEvent(event BorrowEvent) error {
    // Create borrow record
    borrow := Borrow{BookID: event.BookID, ...}
    db.Create(&borrow)
    
    return nil
}
```

**2. Two-Phase Commit (XA Transactions)**:
```go
// Coordinator service
func DistributedBorrowTransaction(bookID, userID uint) error {
    txID := generateTransactionID()
    
    // Phase 1: Prepare
    bookPrepared := bookService.PrepareBorrow(txID, bookID)
    borrowPrepared := borrowService.PrepareBorrow(txID, userID, bookID)
    
    if !bookPrepared || !borrowPrepared {
        // Abort both
        bookService.Rollback(txID)
        borrowService.Rollback(txID)
        return errors.New("transaction failed")
    }
    
    // Phase 2: Commit
    bookService.Commit(txID)
    borrowService.Commit(txID)
    
    return nil
}
```

**3. Compensating Transactions**:
```go
func BorrowWithCompensation(bookID, userID uint) error {
    // Step 1: Reserve book
    reservation := bookService.ReserveBook(bookID)
    
    // Step 2: Create borrow record
    borrow, err := borrowService.CreateBorrow(userID, bookID)
    if err != nil {
        // Compensate: Release reservation
        bookService.ReleaseReservation(reservation.ID)
        return err
    }
    
    // Step 3: Confirm reservation
    bookService.ConfirmReservation(reservation.ID)
    
    return nil
}
```

**4. Event Sourcing + CQRS**:
```go
// Event Store
type Event struct {
    ID        uint
    AggregateID uint
    EventType string
    Data      []byte
    Timestamp time.Time
}

// Append-only event log
func AppendEvent(aggregateID uint, eventType string, data interface{}) error {
    event := Event{
        AggregateID: aggregateID,
        EventType:   eventType,
        Data:        serialize(data),
        Timestamp:   time.Now(),
    }
    return db.Create(&event).Error
}

// Rebuild state from events
func RebuildAggregate(aggregateID uint) (*Aggregate, error) {
    var events []Event
    db.Where("aggregate_id = ?", aggregateID).
       Order("timestamp ASC").
       Find(&events)
    
    aggregate := &Aggregate{}
    for _, event := range events {
        aggregate.Apply(event)
    }
    
    return aggregate, nil
}
```

**5. Outbox Pattern (Reliable Event Publishing)**:
```sql
CREATE TABLE outbox (
    id BIGSERIAL PRIMARY KEY,
    event_type VARCHAR(50),
    payload JSONB,
    published BOOLEAN DEFAULT false,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

```go
func BorrowBook(bookID uint) error {
    return db.Transaction(func(tx *gorm.DB) error {
        // Update book
        book.AvailableCopies--
        tx.Save(&book)
        
        // Write to outbox (same transaction)
        outbox := Outbox{
            EventType: "book.borrowed",
            Payload:   serialize(BorrowEvent{BookID: bookID}),
        }
        tx.Create(&outbox)
        
        return nil
    })
}

// Background job publishes from outbox
func PublishOutboxEvents() {
    var events []Outbox
    db.Where("published = false").Find(&events)
    
    for _, event := range events {
        eventBus.Publish(event.EventType, event.Payload)
        event.Published = true
        db.Save(&event)
    }
}
```

---

#### Q42: How would you implement rate limiting to prevent API abuse?

**Answer**: Multiple rate limiting strategies:

**1. Token Bucket Algorithm (Redis)**:
```go
import "github.com/go-redis/redis/v8"

type RateLimiter struct {
    redis *redis.Client
}

func (rl *RateLimiter) IsAllowed(userID uint, limit int, window time.Duration) (bool, error) {
    key := fmt.Sprintf("rate_limit:%d", userID)
    
    // Get current count
    count, err := rl.redis.Get(ctx, key).Int()
    if err == redis.Nil {
        count = 0
    }
    
    if count >= limit {
        return false, nil // Rate limit exceeded
    }
    
    // Increment and set expiry
    pipe := rl.redis.Pipeline()
    pipe.Incr(ctx, key)
    pipe.Expire(ctx, key, window)
    _, err = pipe.Exec(ctx)
    
    return true, err
}
```

**2. Sliding Window Log (Redis Sorted Sets)**:
```go
func (rl *RateLimiter) SlidingWindowAllowed(
    userID uint, 
    limit int, 
    window time.Duration,
) (bool, int, error) {
    key := fmt.Sprintf("rate_limit:sliding:%d", userID)
    now := time.Now()
    windowStart := now.Add(-window)
    
    // Remove old entries
    rl.redis.ZRemRangeByScore(ctx, key, "0", fmt.Sprintf("%d", windowStart.Unix()))
    
    // Count current requests
    count, err := rl.redis.ZCard(ctx, key).Result()
    if err != nil {
        return false, 0, err
    }
    
    if int(count) >= limit {
        // Get oldest entry to calculate retry after
        oldest, _ := rl.redis.ZRange(ctx, key, 0, 0).Result()
        return false, 0, nil
    }
    
    // Add current request
    rl.redis.ZAdd(ctx, key, &redis.Z{
        Score:  float64(now.Unix()),
        Member: fmt.Sprintf("%d", now.UnixNano()),
    })
    rl.redis.Expire(ctx, key, window)
    
    return true, limit - int(count) - 1, nil
}
```

**3. Gin Middleware**:
```go
func RateLimitMiddleware(limiter *RateLimiter, limit int, window time.Duration) gin.HandlerFunc {
    return func(c *gin.Context) {
        userID := getUserIDFromContext(c) // or use IP address
        
        allowed, remaining, err := limiter.SlidingWindowAllowed(userID, limit, window)
        if err != nil {
            c.JSON(500, gin.H{"error": "Internal server error"})
            c.Abort()
            return
        }
        
        // Set headers
        c.Header("X-RateLimit-Limit", strconv.Itoa(limit))
        c.Header("X-RateLimit-Remaining", strconv.Itoa(remaining))
        
        if !allowed {
            c.Header("Retry-After", strconv.Itoa(window.Seconds()))
            c.JSON(429, gin.H{
                "error": "Rate limit exceeded",
                "message": fmt.Sprintf("Too many requests. Please try again in %d seconds.", window.Seconds()),
            })
            c.Abort()
            return
        }
        
        c.Next()
    }
}
```

**4. Different Limits for Different Endpoints**:
```go
// Route-specific rate limits
api.POST("/borrows", RateLimitMiddleware(limiter, 10, time.Minute), borrowHandler) // 10/min
api.POST("/books", RateLimitMiddleware(limiter, 5, time.Minute), createBookHandler) // 5/min
api.GET("/books", RateLimitMiddleware(limiter, 100, time.Minute), listBooksHandler) // 100/min
```

**5. IP-Based Rate Limiting**:
```go
func IPRateLimitMiddleware(limiter *RateLimiter) gin.HandlerFunc {
    return func(c *gin.Context) {
        ip := c.ClientIP()
        ipHash := hashIP(ip)
        
        allowed, _, _ := limiter.IsAllowed(ipHash, 60, time.Minute) // 60 req/min per IP
        if !allowed {
            c.JSON(429, gin.H{"error": "Rate limit exceeded"})
            c.Abort()
            return
        }
        
        c.Next()
    }
}
```

---

#### Q43: How do you handle file uploads (e.g., book cover images)?

**Answer**: Secure file upload implementation:

**1. File Upload Endpoint**:
```go
func UploadBookCover(c *gin.Context) {
    // Get book ID
    bookID := c.Param("id")
    
    // Get file from form
    file, header, err := c.Request.FormFile("cover")
    if err != nil {
        c.JSON(400, gin.H{"error": "No file provided"})
        return
    }
    defer file.Close()
    
    // Validate file type
    allowedTypes := map[string]bool{
        "image/jpeg": true,
        "image/png":  true,
        "image/webp": true,
    }
    
    contentType := header.Header.Get("Content-Type")
    if !allowedTypes[contentType] {
        c.JSON(400, gin.H{"error": "Invalid file type"})
        return
    }
    
    // Validate file size (max 5MB)
    if header.Size > 5*1024*1024 {
        c.JSON(400, gin.H{"error": "File too large (max 5MB)"})
        return
    }
    
    // Generate unique filename
    ext := filepath.Ext(header.Filename)
    filename := fmt.Sprintf("%s_%d%s", bookID, time.Now().Unix(), ext)
    filepath := filepath.Join("uploads", "covers", filename)
    
    // Save file
    out, err := os.Create(filepath)
    if err != nil {
        c.JSON(500, gin.H{"error": "Failed to save file"})
        return
    }
    defer out.Close()
    
    io.Copy(out, file)
    
    // Update book record with file path
    var book Book
    db.First(&book, bookID)
    book.ImageURL = "/uploads/covers/" + filename
    db.Save(&book)
    
    c.JSON(200, gin.H{
        "message": "File uploaded successfully",
        "url": book.ImageURL,
    })
}
```

**2. Cloud Storage (S3) Alternative**:
```go
import "github.com/aws/aws-sdk-go/service/s3"

func UploadToS3(file io.Reader, filename string) (string, error) {
    sess := session.Must(session.NewSession())
    svc := s3.New(sess)
    
    bucket := "library-book-covers"
    key := fmt.Sprintf("covers/%s", filename)
    
    _, err := svc.PutObject(&s3.PutObjectInput{
        Bucket:      aws.String(bucket),
        Key:         aws.String(key),
        Body:        file,
        ContentType: aws.String("image/jpeg"),
        ACL:         aws.String("public-read"),
    })
    
    if err != nil {
        return "", err
    }
    
    url := fmt.Sprintf("https://%s.s3.amazonaws.com/%s", bucket, key)
    return url, nil
}
```

**3. Image Processing**:
```go
import "github.com/disintegration/imaging"

func ProcessImage(file io.Reader) ([]byte, error) {
    // Decode image
    img, err := imaging.Decode(file)
    if err != nil {
        return nil, err
    }
    
    // Resize to max 800x800
    img = imaging.Fit(img, 800, 800, imaging.Lanczos)
    
    // Compress
    var buf bytes.Buffer
    err = imaging.Encode(&buf, img, imaging.JPEG, imaging.JPEGQuality(85))
    if err != nil {
        return nil, err
    }
    
    return buf.Bytes(), nil
}
```

**4. Frontend Upload**:
```typescript
const uploadCover = async (bookId: number, file: File) => {
  const formData = new FormData();
  formData.append('cover', file);
  
  const response = await fetch(`/api/v1/books/${bookId}/cover`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
    },
    body: formData,
  });
  
  return response.json();
};
```

---

#### Q44: How would you implement a bulk import feature for books?

**Answer**: Efficient bulk import with validation and error handling:

**1. CSV Import Endpoint**:
```go
func BulkImportBooks(c *gin.Context) {
    file, header, err := c.Request.FormFile("file")
    if err != nil {
        c.JSON(400, gin.H{"error": "No file provided"})
        return
    }
    defer file.Close()
    
    // Validate CSV
    if !strings.HasSuffix(header.Filename, ".csv") {
        c.JSON(400, gin.H{"error": "Invalid file type, CSV required"})
        return
    }
    
    // Parse CSV
    reader := csv.NewReader(file)
    records, err := reader.ReadAll()
    if err != nil {
        c.JSON(400, gin.H{"error": "Failed to parse CSV"})
        return
    }
    
    // Process in batches
    batchSize := 100
    var errors []ImportError
    var successCount int
    
    for i := 1; i < len(records); i++ { // Skip header
        if i%batchSize == 0 {
            // Process batch
            batch := records[i-batchSize : i]
            success, batchErrors := processBatch(batch)
            successCount += success
            errors = append(errors, batchErrors...)
        }
    }
    
    // Process remaining
    if len(records)-1 > (len(records)-1)/batchSize*batchSize {
        remaining := records[(len(records)-1)/batchSize*batchSize:]
        success, batchErrors := processBatch(remaining)
        successCount += success
        errors = append(errors, batchErrors...)
    }
    
    c.JSON(200, gin.H{
        "message":      "Import completed",
        "successCount": successCount,
        "errorCount":   len(errors),
        "errors":       errors,
    })
}
```

**2. Batch Processing**:
```go
func processBatch(records [][]string) (int, []ImportError) {
    var books []Book
    var errors []ImportError
    
    for rowNum, record := range records {
        if len(record) < 6 {
            errors = append(errors, ImportError{
                Row:    rowNum,
                Field:  "columns",
                Error:  "Insufficient columns",
            })
            continue
        }
        
        book := Book{
            Title:    strings.TrimSpace(record[0]),
            Author:   strings.TrimSpace(record[1]),
            ISBN:     strings.TrimSpace(record[2]),
            Category: strings.TrimSpace(record[3]),
            Copies:   parseInt(record[4]),
        }
        
        // Validate
        if err := validateBook(&book); err != nil {
            errors = append(errors, ImportError{
                Row:   rowNum,
                Field: err.Field,
                Error: err.Message,
            })
            continue
        }
        
        books = append(books, book)
    }
    
    // Bulk insert
    if len(books) > 0 {
        db.CreateInBatches(books, 100)
    }
    
    return len(books), errors
}
```

**3. Async Import with Job Queue**:
```go
type ImportJob struct {
    ID       uint
    UserID   uint
    FilePath string
    Status   string // 'pending', 'processing', 'completed', 'failed'
    Result   ImportResult
}

func CreateImportJob(userID uint, filePath string) (*ImportJob, error) {
    job := ImportJob{
        UserID:   userID,
        FilePath: filePath,
        Status:   "pending",
    }
    db.Create(&job)
    
    // Queue for processing
    jobQueue.Enqueue(job.ID)
    
    return &job, nil
}

// Background worker
func ImportWorker() {
    for {
        jobID := jobQueue.Dequeue()
        processImportJob(jobID)
    }
}
```

**4. Progress Tracking**:
```go
type ImportProgress struct {
    JobID       uint
    TotalRows   int
    Processed   int
    Success     int
    Errors      int
    Status      string
}

func (ip *ImportProgress) Update(processed, success, errors int) {
    ip.Processed = processed
    ip.Success = success
    ip.Errors = errors
    
    if ip.Processed >= ip.TotalRows {
        ip.Status = "completed"
    }
    
    // Store in Redis for real-time updates
    redis.Set(fmt.Sprintf("import:progress:%d", ip.JobID), serialize(ip), 1*time.Hour)
}
```

---

#### Q45: How do you handle database connection pooling and prevent connection leaks?

**Answer**: Proper connection pool configuration and monitoring:

**1. Connection Pool Configuration**:
```go
func NewDatabase(cfg *config.Config) (*Database, error) {
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        return nil, err
    }
    
    // Get underlying sql.DB for pool configuration
    sqlDB, err := db.DB()
    if err != nil {
        return nil, err
    }
    
    // Connection pool settings
    sqlDB.SetMaxIdleConns(10)              // Max idle connections
    sqlDB.SetMaxOpenConns(100)             // Max open connections
    sqlDB.SetConnMaxLifetime(time.Hour)    // Max connection lifetime
    sqlDB.SetConnMaxIdleTime(10 * time.Minute) // Max idle time
    
    // Test connection
    if err := sqlDB.Ping(); err != nil {
        return nil, err
    }
    
    return &Database{DB: db}, nil
}
```

**2. Connection Leak Prevention**:
```go
// Always close rows
func GetBooks() ([]Book, error) {
    rows, err := db.Raw("SELECT * FROM books").Rows()
    if err != nil {
        return nil, err
    }
    defer rows.Close() // CRITICAL: Always defer close
    
    var books []Book
    for rows.Next() {
        var book Book
        db.ScanRows(rows, &book)
        books = append(books, book)
    }
    
    return books, nil
}

// Use context with timeout
func GetBooksWithContext(ctx context.Context) ([]Book, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    
    var books []Book
    err := db.WithContext(ctx).Find(&books).Error
    return books, err
}
```

**3. Monitor Connection Pool**:
```go
func GetConnectionStats(db *gorm.DB) ConnectionStats {
    sqlDB, _ := db.DB()
    
    stats := sqlDB.Stats()
    return ConnectionStats{
        OpenConnections: stats.OpenConnections,
        InUse:          stats.InUse,
        Idle:           stats.Idle,
        WaitCount:      stats.WaitCount,
        WaitDuration:   stats.WaitDuration,
        MaxIdleClosed:  stats.MaxIdleClosed,
        MaxLifetimeClosed: stats.MaxLifetimeClosed,
    }
}

// Health check endpoint
func HealthCheck(c *gin.Context) {
    stats := GetConnectionStats(db)
    
    // Alert if pool is exhausted
    if stats.OpenConnections >= 95 { // 95% of max
        c.JSON(503, gin.H{
            "status": "degraded",
            "message": "Connection pool nearly exhausted",
            "stats": stats,
        })
        return
    }
    
    c.JSON(200, gin.H{
        "status": "healthy",
        "stats": stats,
    })
}
```

**4. Transaction Management**:
```go
// Always ensure transactions are committed or rolled back
func BorrowBook(bookID uint) error {
    tx := db.Begin()
    defer func() {
        if r := recover(); r != nil {
            tx.Rollback() // Rollback on panic
        }
    }()
    
    // Operations...
    if err := createBorrow(tx, bookID); err != nil {
        tx.Rollback()
        return err
    }
    
    return tx.Commit()
}
```

---

#### Q46: How would you implement a feature flag system for gradual rollouts?

**Answer**: Feature flag implementation for controlled releases:

**1. Feature Flag Table**:
```sql
CREATE TABLE feature_flags (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    enabled BOOLEAN DEFAULT false,
    rollout_percentage INTEGER DEFAULT 0, -- 0-100
    user_whitelist JSONB, -- List of user IDs
    user_blacklist JSONB, -- List of user IDs to exclude
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

**2. Feature Flag Service**:
```go
type FeatureFlagService struct {
    db    *gorm.DB
    cache *redis.Client
}

func (ffs *FeatureFlagService) IsEnabled(flagName string, userID uint) (bool, error) {
    // Check cache first
    cacheKey := fmt.Sprintf("feature_flag:%s:%d", flagName, userID)
    if cached := ffs.cache.Get(ctx, cacheKey); cached != nil {
        return cached.Val() == "1", nil
    }
    
    // Get flag from database
    var flag FeatureFlag
    if err := ffs.db.Where("name = ?", flagName).First(&flag).Error; err != nil {
        return false, err
    }
    
    // Check if globally disabled
    if !flag.Enabled {
        return false, nil
    }
    
    // Check whitelist
    if len(flag.UserWhitelist) > 0 {
        enabled := contains(flag.UserWhitelist, userID)
        ffs.cache.Set(ctx, cacheKey, enabled, 5*time.Minute)
        return enabled, nil
    }
    
    // Check blacklist
    if contains(flag.UserBlacklist, userID) {
        ffs.cache.Set(ctx, cacheKey, false, 5*time.Minute)
        return false, nil
    }
    
    // Percentage-based rollout
    enabled := userID%100 < uint(flag.RolloutPercentage)
    ffs.cache.Set(ctx, cacheKey, enabled, 5*time.Minute)
    
    return enabled, nil
}
```

**3. Middleware**:
```go
func FeatureFlagMiddleware(flagName string) gin.HandlerFunc {
    return func(c *gin.Context) {
        userID, _ := c.Get("userID").(uint)
        
        enabled, _ := featureFlagService.IsEnabled(flagName, userID)
        if !enabled {
            c.JSON(404, gin.H{"error": "Feature not available"})
            c.Abort()
            return
        }
        
        c.Next()
    }
}
```

**4. Frontend Feature Flag**:
```typescript
const useFeatureFlag = (flagName: string) => {
  const { user } = useAuth();
  
  return useQuery({
    queryKey: ['featureFlag', flagName, user?.id],
    queryFn: () => api.getFeatureFlag(flagName),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

// Usage
const { data: newUIEnabled } = useFeatureFlag('new_book_ui');
if (newUIEnabled) {
  return <NewBookUI />;
}
return <OldBookUI />;
```

---

#### Q47: How do you handle API versioning and backward compatibility?

**Answer**: API versioning strategy:

**1. URL-Based Versioning**:
```go
// Version 1
v1 := api.Group("/api/v1")
v1.GET("/books", listBooksV1)

// Version 2
v2 := api.Group("/api/v2")
v2.GET("/books", listBooksV2)
```

**2. Header-Based Versioning**:
```go
func VersionMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        version := c.GetHeader("API-Version")
        if version == "" {
            version = "v1" // Default
        }
        
        c.Set("api_version", version)
        c.Next()
    }
}

func listBooks(c *gin.Context) {
    version := c.GetString("api_version")
    
    switch version {
    case "v2":
        listBooksV2(c)
    default:
        listBooksV1(c)
    }
}
```

**3. Response Transformation**:
```go
type BookResponseV1 struct {
    ID    uint   `json:"id"`
    Title string `json:"title"`
    Author string `json:"author"`
}

type BookResponseV2 struct {
    ID     uint   `json:"id"`
    Title  string `json:"title"`
    Author struct {
        FirstName string `json:"firstName"`
        LastName  string `json:"lastName"`
    } `json:"author"`
}

func GetBooks(c *gin.Context) {
    books := getBooksFromDB()
    version := c.GetString("api_version")
    
    switch version {
    case "v2":
        c.JSON(200, transformToV2(books))
    default:
        c.JSON(200, transformToV1(books))
    }
}
```

**4. Deprecation Headers**:
```go
func DeprecatedEndpointMiddleware(newVersion string) gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("Deprecation", "true")
        c.Header("Sunset", "2025-12-31") // Date when endpoint will be removed
        c.Header("Link", fmt.Sprintf("<%s>; rel=\"successor-version\"", newVersion))
        c.Next()
    }
}
```

---

#### Q48: How would you implement a caching layer to reduce database load?

**Answer**: Multi-level caching strategy:

**1. Application-Level Cache (Redis)**:
```go
type CacheService struct {
    redis *redis.Client
}

func (cs *CacheService) Get(key string, dest interface{}) error {
    val, err := cs.redis.Get(ctx, key).Bytes()
    if err == redis.Nil {
        return ErrCacheMiss
    }
    if err != nil {
        return err
    }
    
    return json.Unmarshal(val, dest)
}

func (cs *CacheService) Set(key string, value interface{}, ttl time.Duration) error {
    data, err := json.Marshal(value)
    if err != nil {
        return err
    }
    
    return cs.redis.Set(ctx, key, data, ttl).Err()
}
```

**2. Cache-Aside Pattern**:
```go
func GetBook(id uint) (*Book, error) {
    cacheKey := fmt.Sprintf("book:%d", id)
    
    // Try cache first
    var book Book
    if err := cache.Get(cacheKey, &book); err == nil {
        return &book, nil
    }
    
    // Cache miss - query database
    if err := db.First(&book, id).Error; err != nil {
        return nil, err
    }
    
    // Store in cache
    cache.Set(cacheKey, book, 15*time.Minute)
    
    return &book, nil
}
```

**3. Write-Through Cache**:
```go
func CreateBook(book *Book) error {
    // Write to database
    if err := db.Create(book).Error; err != nil {
        return err
    }
    
    // Update cache immediately
    cacheKey := fmt.Sprintf("book:%d", book.ID)
    cache.Set(cacheKey, book, 15*time.Minute)
    
    // Invalidate list cache
    cache.Delete("books:*")
    
    return nil
}
```

**4. Cache Invalidation**:
```go
func UpdateBook(id uint, updates map[string]interface{}) error {
    // Update database
    db.Model(&Book{}).Where("id = ?", id).Updates(updates)
    
    // Invalidate cache
    cache.Delete(fmt.Sprintf("book:%d", id))
    cache.Delete("books:*") // Invalidate all book lists
    
    return nil
}
```

**5. Frontend Caching (React Query)**:
```typescript
const { data } = useQuery({
  queryKey: ['book', id],
  queryFn: () => bookApi.getBook(id),
  staleTime: 5 * 60 * 1000, // 5 minutes
  cacheTime: 30 * 60 * 1000, // 30 minutes
});
```

---

#### Q49: How do you ensure your application is production-ready?

**Answer**: Production readiness checklist:

**1. Error Handling & Logging**:
```go
// Structured logging
import "go.uber.org/zap"

logger, _ := zap.NewProduction()
logger.Info("Book borrowed",
    zap.Uint("userID", userID),
    zap.Uint("bookID", bookID),
    zap.String("status", "success"),
)
logger.Error("Failed to borrow book",
    zap.Error(err),
    zap.Uint("userID", userID),
)
```

**2. Health Checks**:
```go
// Health check endpoint
router.GET("/health", func(c *gin.Context) {
    if err := db.HealthCheck(); err != nil {
        c.JSON(503, gin.H{"status": "unhealthy", "database": "down"})
        return
    }
    c.JSON(200, gin.H{"status": "healthy"})
})
```

**3. Graceful Shutdown**:
```go
// Graceful shutdown on SIGTERM/SIGINT
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit

ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()

if err := srv.Shutdown(ctx); err != nil {
    log.Fatal("Server forced to shutdown:", err)
}
```

**4. Environment Configuration**:
- Separate configs for dev/staging/prod
- Secrets in environment variables (never in code)
- Feature flags for gradual rollouts

**5. Database Migrations**:
- Automated migrations on startup
- Rollback strategy in place
- Backup before migrations

**6. Monitoring & Observability**:
- Application metrics (Prometheus)
- Distributed tracing (Jaeger)
- Error tracking (Sentry)
- Uptime monitoring

**7. Security**:
- HTTPS/SSL enabled
- CORS properly configured
- Rate limiting implemented
- Input validation on all endpoints
- SQL injection prevention
- XSS protection

**8. Performance**:
- Database indexes optimized
- Connection pooling configured
- Caching layer implemented
- Query optimization done
- Load testing completed

**9. Documentation**:
- API documentation (OpenAPI/Swagger)
- Deployment guides
- Runbooks for common issues
- Architecture diagrams

**10. Testing**:
- Unit tests (>70% coverage)
- Integration tests
- E2E tests for critical flows
- Load testing results

**11. CI/CD**:
- Automated testing on PR
- Automated builds
- Staged deployments (dev → staging → prod)
- Rollback capability

**12. Disaster Recovery**:
- Database backups automated
- Backup restoration tested
- Disaster recovery plan documented

---

#### Q50: What are the biggest technical trade-offs you made in this project, and why?

**Answer**: Key trade-offs and rationale:

**1. JWT vs Session-Based Auth**
- **Chosen**: JWT (stateless tokens)
- **Trade-off**: Can't revoke tokens immediately (until expiry)
- **Why**: Better scalability, stateless backend, works across microservices
- **Mitigation**: Short access token expiry (24h), refresh token rotation

**2. REST vs GraphQL**
- **Chosen**: REST API
- **Trade-off**: Over-fetching/under-fetching possible
- **Why**: Simpler to implement, better caching, easier to understand
- **Mitigation**: Pagination, filtering, selective field loading

**3. Monolithic vs Microservices**
- **Chosen**: Monolithic architecture
- **Trade-off**: Less independent scaling per service
- **Why**: 
  - Faster development
  - Easier debugging
  - Lower operational complexity
  - Single database (consistency)
- **Future**: Can split into microservices when needed (user service, book service, etc.)

**4. PostgreSQL vs MongoDB**
- **Chosen**: PostgreSQL (relational)
- **Trade-off**: Less flexible schema, requires migrations
- **Why**: 
  - ACID transactions critical for borrow/return operations
  - Complex relationships (user-borrow-book-fine)
  - Strong consistency requirements
  - Rich querying capabilities

**5. React Query vs Redux**
- **Chosen**: React Query for server state
- **Trade-off**: Less control over exact state shape
- **Why**: 
  - Automatic caching and background updates
  - Less boilerplate
  - Built-in loading/error states
  - Better for server state management

**6. GORM vs Raw SQL**
- **Chosen**: GORM (ORM)
- **Trade-off**: Less control, potential N+1 queries if not careful
- **Why**: 
  - Faster development
  - Type safety
  - Automatic migrations
  - Cross-database compatibility
- **Mitigation**: Use raw SQL for complex queries when needed

**7. Redis Caching vs In-Memory**
- **Chosen**: Redis (external cache)
- **Trade-off**: Network latency, additional infrastructure
- **Why**: 
  - Shared across multiple backend instances
  - Persistence
  - Future: Pub/sub for real-time features

**8. Docker vs Kubernetes**
- **Chosen**: Docker Compose
- **Trade-off**: Less advanced orchestration
- **Why**: 
  - Simpler for single-server deployment
  - Easier local development
  - Sufficient for current scale
- **Future**: Can migrate to Kubernetes for multi-region

**9. Polling vs WebSockets for Notifications**
- **Chosen**: Polling (current)
- **Trade-off**: Higher server load, less real-time
- **Why**: 
  - Simpler implementation
  - Works behind firewalls
  - No persistent connections
- **Future**: WebSockets or Server-Sent Events for real-time

**10. Optimistic Updates vs Pessimistic**
- **Chosen**: Optimistic for most operations
- **Trade-off**: Need rollback on failure
- **Why**: 
  - Better UX (instant feedback)
  - Feels faster
- **Mitigation**: Proper error handling and rollback

**11. Database Transaction Scope**
- **Chosen**: Fine-grained transactions (per operation)
- **Trade-off**: More transaction overhead
- **Why**: 
  - Better concurrency
  - Shorter lock duration
  - Fewer deadlocks

**12. Code Splitting Strategy**
- **Chosen**: Route-based code splitting
- **Trade-off**: Initial load not minimal
- **Why**: 
  - Simpler to implement
  - Good balance of bundle size vs complexity
  - Users typically navigate by routes

---

## Conclusion

This Library Management System demonstrates a comprehensive understanding of full-stack development, system design, and production engineering practices. The architecture is scalable, maintainable, and follows industry best practices while making pragmatic trade-offs based on current requirements and constraints.

The system is production-ready and can be extended with additional features such as:
- Real-time notifications via WebSockets
- Advanced analytics and reporting
- Mobile applications
- Multi-library support
- Integration with external book APIs
- Machine learning recommendations

For questions or contributions, please refer to the project repository.

---

---

### Frontend-Specific Questions

#### Q51: How do you handle re-rendering performance issues in React? Explain memoization strategies.

**Answer**: Multiple strategies to prevent unnecessary re-renders:

**1. React.memo() for Component Memoization**:
```typescript
// Memoize expensive components
const BookCard = React.memo(({ book, onBorrow }: BookCardProps) => {
  return (
    <Card>
      <h3>{book.title}</h3>
      <Button onClick={() => onBorrow(book.id)}>Borrow</Button>
    </Card>
  );
}, (prevProps, nextProps) => {
  // Custom comparison function
  return prevProps.book.id === nextProps.book.id &&
         prevProps.book.availableCopies === nextProps.book.availableCopies;
});
```

**2. useMemo() for Expensive Calculations**:
```typescript
const BooksList = ({ books, searchTerm }: Props) => {
  // Memoize filtered results
  const filteredBooks = useMemo(() => {
    return books.filter(book => 
      book.title.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [books, searchTerm]);
  
  return filteredBooks.map(book => <BookCard key={book.id} book={book} />);
};
```

**3. useCallback() for Function Stability**:
```typescript
const BooksPage = () => {
  const [books, setBooks] = useState([]);
  
  // Memoize callback to prevent child re-renders
  const handleBorrow = useCallback((bookId: number) => {
    borrowApi.createBorrow(bookId).then(() => {
      // Update books list
    });
  }, []); // Empty deps - function never changes
  
  return books.map(book => (
    <BookCard 
      key={book.id} 
      book={book} 
      onBorrow={handleBorrow} // Stable reference
    />
  ));
};
```

**4. Context Optimization**:
```typescript
// Split contexts to prevent unnecessary re-renders
const UserContext = createContext(); // User data
const ThemeContext = createContext(); // Theme data

// Instead of one large context that causes all consumers to re-render
```

**5. Virtual Scrolling for Large Lists**:
```typescript
import { FixedSizeList } from 'react-window';

const BookList = ({ books }) => (
  <FixedSizeList
    height={600}
    itemCount={books.length}
    itemSize={150}
    width="100%"
  >
    {({ index, style }) => (
      <div style={style}>
        <BookCard book={books[index]} />
      </div>
    )}
  </FixedSizeList>
);
```

---

#### Q52: How do you implement infinite scrolling or pagination in your book list? Show both approaches.

**Answer**: Both approaches with React Query:

**1. Infinite Scrolling Implementation**:
```typescript
import { useInfiniteQuery } from '@tanstack/react-query';
import { useIntersectionObserver } from 'react-intersection-observer';

const BooksInfinite = () => {
  const { ref, inView } = useIntersectionObserver({
    threshold: 0.1,
  });

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ['books', 'infinite'],
    queryFn: ({ pageParam = 1 }) => 
      bookApi.getBooks({ page: pageParam, limit: 20 }),
    getNextPageParam: (lastPage) => 
      lastPage.pagination.page < lastPage.pagination.pages 
        ? lastPage.pagination.page + 1 
        : undefined,
  });

  // Auto-load when intersection observer detects bottom
  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  return (
    <div>
      {data?.pages.map((page) =>
        page.books.map((book) => (
          <BookCard key={book.id} book={book} />
        ))
      )}
      <div ref={ref}>
        {isFetchingNextPage && <LoadingSpinner />}
        {!hasNextPage && <p>No more books</p>}
      </div>
    </div>
  );
};
```

**2. Traditional Pagination Implementation**:
```typescript
const BooksPaginated = () => {
  const [page, setPage] = useState(1);
  const [limit] = useState(20);

  const { data, isLoading } = useQuery({
    queryKey: ['books', 'paginated', page],
    queryFn: () => bookApi.getBooks({ page, limit }),
    keepPreviousData: true, // Shows previous data while loading
  });

  return (
    <div>
      {data?.books.map(book => (
        <BookCard key={book.id} book={book} />
      ))}
      
      <Pagination>
        <Button 
          disabled={page === 1} 
          onClick={() => setPage(p => p - 1)}
        >
          Previous
        </Button>
        <span>Page {page} of {data?.pagination.pages}</span>
        <Button 
          disabled={page === data?.pagination.pages} 
          onClick={() => setPage(p => p + 1)}
        >
          Next
        </Button>
      </Pagination>
    </div>
  );
};
```

**Trade-offs**:
- **Infinite Scroll**: Better for mobile, continuous browsing, but harder to reach footer
- **Pagination**: Better for desktop, precise navigation, SEO-friendly URLs

---

#### Q53: How do you handle form state management with complex nested forms?

**Answer**: Using React Hook Form with nested structures:

```typescript
import { useForm, useFieldArray } from 'react-hook-form';

type BookFormData = {
  title: string;
  author: string;
  copies: number;
  categories: { name: string }[];
  metadata: {
    isbn: string;
    publishedDate: string;
    description: string;
  };
};

const BookForm = () => {
  const { register, control, handleSubmit, watch, formState: { errors } } = useForm<BookFormData>({
    defaultValues: {
      categories: [{ name: '' }],
      metadata: {
        isbn: '',
        publishedDate: '',
        description: '',
      },
    },
  });

  // Dynamic array fields for categories
  const { fields, append, remove } = useFieldArray({
    control,
    name: 'categories',
  });

  // Watch nested field for conditional logic
  const isbn = watch('metadata.isbn');

  const onSubmit = async (data: BookFormData) => {
    await bookApi.createBook({
      ...data,
      categories: data.categories.map(c => c.name),
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input 
        {...register('title', { required: 'Title is required' })} 
      />
      {errors.title && <span>{errors.title.message}</span>}

      <input {...register('author', { required: true })} />

      {/* Nested object */}
      <div>
        <label>ISBN</label>
        <input 
          {...register('metadata.isbn', { 
            required: true,
            pattern: /^\d{13}$/ 
          })} 
        />
        {errors.metadata?.isbn && <span>Invalid ISBN</span>}
      </div>

      {/* Dynamic array */}
      {fields.map((field, index) => (
        <div key={field.id}>
          <input 
            {...register(`categories.${index}.name` as const)} 
          />
          <button type="button" onClick={() => remove(index)}>
            Remove
          </button>
        </div>
      ))}
      <button type="button" onClick={() => append({ name: '' })}>
        Add Category
      </button>

      <button type="submit">Submit</button>
    </form>
  );
};
```

**Benefits**:
- Minimal re-renders (uncontrolled components)
- Built-in validation
- Easy nested object/array handling
- Type-safe with TypeScript

---

#### Q54: How do you implement drag-and-drop functionality for reordering items (e.g., book lists)?

**Answer**: Using react-beautiful-dnd or @dnd-kit:

```typescript
import { DragDropContext, Droppable, Draggable } from 'react-beautiful-dnd';

const ReorderableBookList = () => {
  const [books, setBooks] = useState<Book[]>([]);
  const { mutate: updateOrder } = useMutation({
    mutationFn: (newOrder: Book[]) => 
      bookApi.updateOrder(newOrder.map(b => b.id)),
  });

  const handleDragEnd = (result: DropResult) => {
    if (!result.destination) return;

    const items = Array.from(books);
    const [reorderedItem] = items.splice(result.source.index, 1);
    items.splice(result.destination.index, 0, reorderedItem);

    setBooks(items);
    updateOrder(items);
  };

  return (
    <DragDropContext onDragEnd={handleDragEnd}>
      <Droppable droppableId="books">
        {(provided) => (
          <div {...provided.droppableProps} ref={provided.innerRef}>
            {books.map((book, index) => (
              <Draggable 
                key={book.id} 
                draggableId={book.id.toString()} 
                index={index}
              >
                {(provided, snapshot) => (
                  <div
                    ref={provided.innerRef}
                    {...provided.draggableProps}
                    {...provided.dragHandleProps}
                    style={{
                      ...provided.draggableProps.style,
                      opacity: snapshot.isDragging ? 0.8 : 1,
                    }}
                  >
                    <BookCard book={book} />
                  </div>
                )}
              </Draggable>
            ))}
            {provided.placeholder}
          </div>
        )}
      </Droppable>
    </DragDropContext>
  );
};
```

**Alternative: @dnd-kit** (more modern, better accessibility):
```typescript
import { DndContext, closestCenter } from '@dnd-kit/core';
import { SortableContext, useSortable } from '@dnd-kit/sortable';

// Similar implementation with better keyboard support
```

---

#### Q55: How do you implement debouncing and throttling for search input?

**Answer**: Multiple approaches:

**1. Custom Hook for Debouncing**:
```typescript
import { useState, useEffect } from 'react';

function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

// Usage in component
const BookSearch = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);

  const { data: books } = useQuery({
    queryKey: ['books', debouncedSearchTerm],
    queryFn: () => bookApi.getBooks({ search: debouncedSearchTerm }),
    enabled: debouncedSearchTerm.length > 2, // Only search after 3 chars
  });

  return (
    <div>
      <Input
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search books..."
      />
      {books && <BookList books={books} />}
    </div>
  );
};
```

**2. Throttling for Scroll Events**:
```typescript
function useThrottle<T>(value: T, limit: number): T {
  const [throttledValue, setThrottledValue] = useState<T>(value);
  const lastRan = useRef(Date.now());

  useEffect(() => {
    const handler = setTimeout(() => {
      if (Date.now() - lastRan.current >= limit) {
        setThrottledValue(value);
        lastRan.current = Date.now();
      }
    }, limit - (Date.now() - lastRan.current));

    return () => clearTimeout(handler);
  }, [value, limit]);

  return throttledValue;
}
```

**3. Using lodash (if available)**:
```typescript
import { debounce } from 'lodash';

const debouncedSearch = useMemo(
  () => debounce((term: string) => {
    // API call
    bookApi.getBooks({ search: term });
  }, 500),
  []
);

useEffect(() => {
  return () => debouncedSearch.cancel();
}, [debouncedSearch]);
```

---

#### Q56: How do you handle optimistic updates in React Query mutations?

**Answer**: Optimistic updates for better UX:

```typescript
const BorrowButton = ({ bookId }: { bookId: number }) => {
  const queryClient = useQueryClient();

  const { mutate: borrowBook, isLoading } = useMutation({
    mutationFn: () => borrowApi.createBorrow(bookId),
    
    // Optimistic update
    onMutate: async () => {
      // Cancel outgoing queries
      await queryClient.cancelQueries(['books', bookId]);
      
      // Snapshot previous value
      const previousBook = queryClient.getQueryData(['books', bookId]);
      
      // Optimistically update
      queryClient.setQueryData(['books', bookId], (old: Book) => ({
        ...old,
        availableCopies: old.availableCopies - 1,
        status: 'borrowed',
      }));
      
      // Return context for rollback
      return { previousBook };
    },
    
    // On success
    onSuccess: () => {
      queryClient.invalidateQueries(['books']);
      queryClient.invalidateQueries(['borrows']);
      toast.success('Book borrowed successfully!');
    },
    
    // On error - rollback
    onError: (err, variables, context) => {
      if (context?.previousBook) {
        queryClient.setQueryData(['books', bookId], context.previousBook);
      }
      toast.error('Failed to borrow book');
    },
    
    // Always refetch to ensure consistency
    onSettled: () => {
      queryClient.invalidateQueries(['books', bookId]);
    },
  });

  return (
    <Button 
      onClick={() => borrowBook()} 
      disabled={isLoading}
    >
      Borrow Book
    </Button>
  );
};
```

**Benefits**:
- Instant UI feedback
- Better perceived performance
- Automatic rollback on error
- Eventual consistency

---

#### Q57: How do you implement feature flags in React to enable/disable features dynamically?

**Answer**: Feature flag system:

```typescript
// Context for feature flags
const FeatureFlagsContext = createContext<Record<string, boolean>>({});

export const FeatureFlagsProvider = ({ children }: { children: ReactNode }) => {
  const [flags, setFlags] = useState<Record<string, boolean>>({});

  useEffect(() => {
    // Fetch flags from API
    fetch('/api/feature-flags')
      .then(res => res.json())
      .then(setFlags);
  }, []);

  return (
    <FeatureFlagsContext.Provider value={flags}>
      {children}
    </FeatureFlagsContext.Provider>
  );
};

// Hook to check flags
export const useFeatureFlag = (flagName: string): boolean => {
  const flags = useContext(FeatureFlagsContext);
  return flags[flagName] ?? false;
};

// Usage in components
const BooksPage = () => {
  const enableAdvancedSearch = useFeatureFlag('advanced_search');
  const enableBookRecommendations = useFeatureFlag('book_recommendations');

  return (
    <div>
      {enableAdvancedSearch && <AdvancedSearchBar />}
      <BookList />
      {enableBookRecommendations && <Recommendations />}
    </div>
  );
};

// Higher-order component approach
const withFeatureFlag = (
  flagName: string,
  Component: React.ComponentType
) => {
  return (props: any) => {
    const enabled = useFeatureFlag(flagName);
    return enabled ? <Component {...props} /> : null;
  };
};
```

**Backend Integration**:
```typescript
// Feature flags based on user role/experiment
const flags = {
  advanced_search: user.role === 'premium',
  beta_features: user.betaUser,
  new_ui: experimentGroup === 'A',
};
```

---

#### Q58: How do you handle file uploads (e.g., book cover images) in React?

**Answer**: File upload with preview and progress:

```typescript
const BookCoverUpload = ({ onUpload }: { onUpload: (url: string) => void }) => {
  const [preview, setPreview] = useState<string | null>(null);
  const [uploading, setUploading] = useState(false);
  const [progress, setProgress] = useState(0);

  const handleFileChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    // Validate file
    if (!file.type.startsWith('image/')) {
      toast.error('Please select an image file');
      return;
    }
    if (file.size > 5 * 1024 * 1024) {
      toast.error('File size must be less than 5MB');
      return;
    }

    // Preview
    const reader = new FileReader();
    reader.onloadend = () => {
      setPreview(reader.result as string);
    };
    reader.readAsDataURL(file);

    // Upload
    const formData = new FormData();
    formData.append('file', file);
    formData.append('type', 'book_cover');

    setUploading(true);
    try {
      const response = await axios.post('/api/v1/upload', formData, {
        headers: { 'Content-Type': 'multipart/form-data' },
        onUploadProgress: (progressEvent) => {
          const percent = Math.round(
            (progressEvent.loaded * 100) / (progressEvent.total || 1)
          );
          setProgress(percent);
        },
      });
      onUpload(response.data.url);
      toast.success('Upload successful');
    } catch (error) {
      toast.error('Upload failed');
    } finally {
      setUploading(false);
    }
  };

  return (
    <div>
      <input
        type="file"
        accept="image/*"
        onChange={handleFileChange}
        disabled={uploading}
      />
      {preview && (
        <img 
          src={preview} 
          alt="Preview" 
          style={{ maxWidth: '200px' }} 
        />
      )}
      {uploading && (
        <Progress value={progress} />
      )}
    </div>
  );
};
```

**Drag & Drop Version**:
```typescript
const FileDropZone = ({ onDrop }: Props) => {
  const [isDragging, setIsDragging] = useState(false);

  const handleDragOver = (e: React.DragEvent) => {
    e.preventDefault();
    setIsDragging(true);
  };

  const handleDrop = (e: React.DragEvent) => {
    e.preventDefault();
    setIsDragging(false);
    const file = e.dataTransfer.files[0];
    if (file) onDrop(file);
  };

  return (
    <div
      onDragOver={handleDragOver}
      onDragLeave={() => setIsDragging(false)}
      onDrop={handleDrop}
      style={{
        border: isDragging ? '2px dashed blue' : '2px dashed gray',
        padding: '20px',
      }}
    >
      Drop file here
    </div>
  );
};
```

---

#### Q59: How do you implement internationalization (i18n) in React?

**Answer**: Using react-i18next:

```typescript
// i18n configuration
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';

i18n.use(initReactI18next).init({
  resources: {
    en: {
      translation: {
        welcome: 'Welcome to Library',
        borrow: 'Borrow Book',
        return: 'Return Book',
        search: 'Search books...',
      },
    },
    es: {
      translation: {
        welcome: 'Bienvenido a la Biblioteca',
        borrow: 'Prestar Libro',
        return: 'Devolver Libro',
        search: 'Buscar libros...',
      },
    },
  },
  lng: 'en',
  fallbackLng: 'en',
  interpolation: { escapeValue: false },
});

// Usage in components
import { useTranslation } from 'react-i18next';

const BooksPage = () => {
  const { t, i18n } = useTranslation();

  const changeLanguage = (lng: string) => {
    i18n.changeLanguage(lng);
    localStorage.setItem('language', lng);
  };

  return (
    <div>
      <h1>{t('welcome')}</h1>
      <Button onClick={() => changeLanguage('en')}>English</Button>
      <Button onClick={() => changeLanguage('es')}>Español</Button>
      
      <Input placeholder={t('search')} />
      <Button>{t('borrow')}</Button>
    </div>
  );
};

// With pluralization
const BookCount = ({ count }: { count: number }) => {
  const { t } = useTranslation();
  return (
    <p>
      {t('book_count', { count, defaultValue: '{{count}} book', defaultValue_plural: '{{count}} books' })}
    </p>
  );
};
```

**Dynamic Loading**:
```typescript
// Lazy load translations
i18n.use(Backend).init({
  backend: {
    loadPath: '/locales/{{lng}}/{{ns}}.json',
  },
});
```

---

#### Q60: How do you handle complex state synchronization between multiple components?

**Answer**: Multiple strategies:

**1. React Query for Server State**:
```typescript
// Shared server state - automatically synced
const BookDetail = ({ bookId }: { bookId: number }) => {
  const { data: book } = useQuery({
    queryKey: ['books', bookId],
    queryFn: () => bookApi.getBook(bookId),
  });
  // All components using this query get same data
};

const BookList = () => {
  const { data: books } = useQuery({
    queryKey: ['books'],
    queryFn: () => bookApi.getBooks(),
  });
  // When book is updated, invalidate to refetch
};
```

**2. Context for Client State**:
```typescript
// Theme context example
const ThemeContext = createContext<{
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}>({
  theme: 'light',
  toggleTheme: () => {},
});

export const ThemeProvider = ({ children }: { children: ReactNode }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
    localStorage.setItem('theme', theme === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Multiple components can access and update
const Header = () => {
  const { theme, toggleTheme } = useContext(ThemeContext);
  return <button onClick={toggleTheme}>Toggle {theme}</button>;
};
```

**3. Event Emitter Pattern**:
```typescript
// Custom event emitter
class EventEmitter {
  private events: Record<string, Function[]> = {};

  on(event: string, callback: Function) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(callback);
  }

  emit(event: string, data?: any) {
    if (this.events[event]) {
      this.events[event].forEach(cb => cb(data));
    }
  }

  off(event: string, callback: Function) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(cb => cb !== callback);
    }
  }
}

const eventEmitter = new EventEmitter();

// Component A
const ComponentA = () => {
  const handleClick = () => {
    eventEmitter.emit('bookBorrowed', { bookId: 123 });
  };
  return <Button onClick={handleClick}>Borrow</Button>;
};

// Component B
const ComponentB = () => {
  const [notifications, setNotifications] = useState([]);

  useEffect(() => {
    const handleBookBorrowed = (data: any) => {
      setNotifications(prev => [...prev, `Book ${data.bookId} borrowed`]);
    };
    eventEmitter.on('bookBorrowed', handleBookBorrowed);
    return () => eventEmitter.off('bookBorrowed', handleBookBorrowed);
  }, []);

  return <NotificationList items={notifications} />;
};
```

**4. Zustand for Complex State**:
```typescript
import create from 'zustand';

interface BookStore {
  selectedBook: Book | null;
  setSelectedBook: (book: Book | null) => void;
  favorites: number[];
  addFavorite: (bookId: number) => void;
}

const useBookStore = create<BookStore>((set) => ({
  selectedBook: null,
  setSelectedBook: (book) => set({ selectedBook: book }),
  favorites: [],
  addFavorite: (bookId) => set((state) => ({
    favorites: [...state.favorites, bookId],
  })),
}));

// Any component can use
const BookCard = ({ book }: { book: Book }) => {
  const { setSelectedBook, addFavorite, favorites } = useBookStore();
  const isFavorite = favorites.includes(book.id);
  
  return (
    <Card>
      <h3>{book.title}</h3>
      <Button onClick={() => setSelectedBook(book)}>View</Button>
      <Button onClick={() => addFavorite(book.id)}>
        {isFavorite ? '❤️' : '🤍'}
      </Button>
    </Card>
  );
};
```

---

### Backend-Specific Questions

#### Q61: How do you handle database connection pooling in Go with GORM?

**Answer**: Connection pool configuration:

```go
package database

import (
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "time"
)

func NewDatabase(cfg *config.Config) (*Database, error) {
    db, err := gorm.Open(postgres.Open(cfg.GetDSN()), &gorm.Config{})
    if err != nil {
        return nil, err
    }

    // Get underlying sql.DB to configure connection pool
    sqlDB, err := db.DB()
    if err != nil {
        return nil, err
    }

    // Connection pool settings
    sqlDB.SetMaxIdleConns(10)           // Max idle connections
    sqlDB.SetMaxOpenConns(100)          // Max open connections
    sqlDB.SetConnMaxLifetime(time.Hour) // Connection max lifetime
    sqlDB.SetConnMaxIdleTime(10 * time.Minute) // Max idle time

    // Health check
    if err := sqlDB.Ping(); err != nil {
        return nil, err
    }

    return &Database{DB: db}, nil
}
```

**Monitoring Pool Stats**:
```go
func (d *Database) GetPoolStats() sql.DBStats {
    sqlDB, _ := d.DB.DB()
    return sqlDB.Stats()
}

// Usage
stats := db.GetPoolStats()
log.Printf("Open connections: %d", stats.OpenConnections)
log.Printf("Idle connections: %d", stats.Idle)
log.Printf("In use: %d", stats.InUse)
log.Printf("Wait count: %d", stats.WaitCount)
```

**Best Practices**:
- `MaxOpenConns` should be less than database `max_connections`
- `MaxIdleConns` should be ~25% of `MaxOpenConns`
- `ConnMaxLifetime` prevents stale connections
- Monitor `WaitCount` - if high, increase pool size

---

#### Q62: How do you implement structured logging in Go? Show production-ready logging.

**Answer**: Using zap or logrus for structured logging:

```go
package logger

import (
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

var Logger *zap.Logger

func InitLogger(env string) error {
    var config zap.Config

    if env == "production" {
        config = zap.NewProductionConfig()
        config.EncoderConfig.TimeKey = "timestamp"
        config.EncoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
    } else {
        config = zap.NewDevelopmentConfig()
        config.EncoderConfig.EncodeLevel = zapcore.CapitalColorLevelEncoder
    }

    logger, err := config.Build()
    if err != nil {
        return err
    }

    Logger = logger
    return nil
}

// Usage in handlers
func createBorrowHandler(c *gin.Context) {
    userID, _ := c.Get("userID")
    bookID := c.Param("bookId")

    logger.Info("Creating borrow",
        zap.Uint("userID", userID.(uint)),
        zap.String("bookID", bookID),
        zap.String("method", "POST"),
        zap.String("path", c.Request.URL.Path),
    )

    // ... business logic

    if err != nil {
        logger.Error("Failed to create borrow",
            zap.Error(err),
            zap.Uint("userID", userID.(uint)),
            zap.String("bookID", bookID),
        )
        c.JSON(500, gin.H{"error": "Internal server error"})
        return
    }

    logger.Info("Borrow created successfully",
        zap.Uint("borrowID", borrow.ID),
    )
}
```

**Request Context Logger**:
```go
func LoggerMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()

        // Create request-scoped logger
        logger := zap.L().With(
            zap.String("requestID", uuid.New().String()),
            zap.String("method", c.Request.Method),
            zap.String("path", c.Request.URL.Path),
            zap.String("ip", c.ClientIP()),
        )
        
        c.Set("logger", logger)
        c.Next()

        // Log request completion
        logger.Info("Request completed",
            zap.Int("status", c.Writer.Status()),
            zap.Duration("duration", time.Since(start)),
        )
    }
}
```

---

#### Q63: How do you implement request validation and sanitization in Go handlers?

**Answer**: Comprehensive validation strategy:

```go
package handlers

import (
    "github.com/go-playground/validator/v10"
    "github.com/gin-gonic/gin"
)

type BookCreateRequest struct {
    Title         string    `json:"title" binding:"required,min=1,max=255"`
    Author        string    `json:"author" binding:"required,min=1,max=255"`
    ISBN          string    `json:"isbn" binding:"required,min=10,max=20,alphanum"`
    Category      string    `json:"category" binding:"required,oneof=Fiction Non-Fiction Science"`
    PublishedDate time.Time `json:"publishedDate" binding:"required"`
    Description   string    `json:"description" binding:"max=1000"`
    Copies        int       `json:"copies" binding:"required,min=1,max=100"`
}

// Custom validator
var validate *validator.Validate

func init() {
    validate = validator.New()
    validate.RegisterValidation("isbn_format", validateISBN)
}

func validateISBN(fl validator.FieldLevel) bool {
    isbn := fl.Field().String()
    // Remove hyphens
    isbn = strings.ReplaceAll(isbn, "-", "")
    // Check if 10 or 13 digits
    return len(isbn) == 10 || len(isbn) == 13
}

func createBookHandler(c *gin.Context) {
    var req BookCreateRequest

    // Bind and validate
    if err := c.ShouldBindJSON(&req); err != nil {
        // Extract validation errors
        var validationErrors []string
        if ve, ok := err.(validator.ValidationErrors); ok {
            for _, e := range ve {
                validationErrors = append(validationErrors, 
                    fmt.Sprintf("Field %s: %s", e.Field(), getValidationError(e)))
            }
        }
        
        c.JSON(400, gin.H{
            "error": "Validation failed",
            "details": validationErrors,
        })
        return
    }

    // Sanitize input
    req.Title = sanitizeString(req.Title)
    req.Author = sanitizeString(req.Author)
    req.Description = sanitizeString(req.Description)

    // Business logic...
}
```

**Sanitization Functions**:
```go
import (
    "html"
    "strings"
    "unicode"
)

func sanitizeString(s string) string {
    // Trim whitespace
    s = strings.TrimSpace(s)
    
    // Escape HTML
    s = html.EscapeString(s)
    
    // Remove control characters
    s = strings.Map(func(r rune) rune {
        if unicode.IsControl(r) && r != '\n' && r != '\r' {
            return -1
        }
        return r
    }, s)
    
    return s
}
```

**Custom Binding for Complex Validation**:
```go
type Email struct {
    Value string
}

func (e *Email) UnmarshalJSON(data []byte) error {
    var s string
    if err := json.Unmarshal(data, &s); err != nil {
        return err
    }
    
    // Validate email format
    if !isValidEmail(s) {
        return errors.New("invalid email format")
    }
    
    e.Value = strings.ToLower(strings.TrimSpace(s))
    return nil
}
```

---

#### Q64: How do you implement rate limiting in Go to prevent API abuse?

**Answer**: Multiple rate limiting strategies:

**1. Token Bucket Algorithm (golang.org/x/time/rate)**:
```go
package middleware

import (
    "golang.org/x/time/rate"
    "net/http"
)

var limiter = rate.NewLimiter(rate.Every(time.Second/10), 5) // 10 requests/sec, burst 5

func RateLimitMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        if !limiter.Allow() {
            c.JSON(429, gin.H{
                "error": "Rate limit exceeded",
                "message": "Too many requests, please try again later",
            })
            c.Abort()
            return
        }
        c.Next()
    }
}
```

**2. Per-IP Rate Limiting (using Redis)**:
```go
import (
    "github.com/go-redis/redis/v8"
    "context"
)

func PerIPRateLimitMiddleware(rdb *redis.Client) gin.HandlerFunc {
    return func(c *gin.Context) {
        ip := c.ClientIP()
        key := fmt.Sprintf("rate_limit:%s", ip)
        
        ctx := context.Background()
        
        // Increment counter
        count, err := rdb.Incr(ctx, key).Result()
        if err != nil {
            c.Next()
            return
        }
        
        // Set expiry on first request
        if count == 1 {
            rdb.Expire(ctx, key, time.Minute)
        }
        
        // Check limit (100 requests per minute per IP)
        if count > 100 {
            c.JSON(429, gin.H{
                "error": "Rate limit exceeded",
                "retry_after": rdb.TTL(ctx, key).Val().Seconds(),
            })
            c.Abort()
            return
        }
        
        // Set rate limit headers
        c.Header("X-RateLimit-Limit", "100")
        c.Header("X-RateLimit-Remaining", fmt.Sprintf("%d", 100-count))
        c.Header("X-RateLimit-Reset", fmt.Sprintf("%d", time.Now().Add(time.Minute).Unix()))
        
        c.Next()
    }
}
```

**3. Sliding Window Rate Limiting**:
```go
type SlidingWindowLimiter struct {
    requests map[string][]time.Time
    limit    int
    window   time.Duration
    mu       sync.Mutex
}

func NewSlidingWindowLimiter(limit int, window time.Duration) *SlidingWindowLimiter {
    return &SlidingWindowLimiter{
        requests: make(map[string][]time.Time),
        limit:    limit,
        window:   window,
    }
}

func (l *SlidingWindowLimiter) Allow(key string) bool {
    l.mu.Lock()
    defer l.mu.Unlock()

    now := time.Now()
    cutoff := now.Add(-l.window)

    // Clean old requests
    if times, exists := l.requests[key]; exists {
        filtered := []time.Time{}
        for _, t := range times {
            if t.After(cutoff) {
                filtered = append(filtered, t)
            }
        }
        l.requests[key] = filtered
    }

    // Check limit
    if len(l.requests[key]) >= l.limit {
        return false
    }

    // Add current request
    l.requests[key] = append(l.requests[key], now)
    return true
}
```

**4. Tiered Rate Limiting (Different limits per user role)**:
```go
func TieredRateLimitMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        user, exists := c.Get("user")
        if !exists {
            c.Next()
            return
        }

        var limit int
        role := user.(*models.User).Role
        
        switch role {
        case "admin":
            limit = 1000 // Higher limit for admins
        case "librarian":
            limit = 500
        default:
            limit = 100
        }

        // Apply limit based on role
        // ... implementation
    }
}
```

---

#### Q65: How do you implement graceful shutdown in Go to handle in-flight requests?

**Answer**: Complete graceful shutdown implementation:

```go
package main

import (
    "context"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    // Initialize server
    srv := &http.Server{
        Addr:    ":8080",
        Handler: setupRoutes(),
    }

    // Start server in goroutine
    go func() {
        log.Println("Starting server on :8080")
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("Server failed to start: %v", err)
        }
    }()

    // Wait for interrupt signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    log.Println("Shutting down server...")

    // Create shutdown context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    // Graceful shutdown
    if err := srv.Shutdown(ctx); err != nil {
        log.Fatalf("Server forced to shutdown: %v", err)
    }

    log.Println("Server exited gracefully")
}
```

**With Gin Framework**:
```go
func main() {
    router := gin.Default()
    // ... setup routes

    srv := &http.Server{
        Addr:    ":8080",
        Handler: router,
    }

    go func() {
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()

    // Graceful shutdown
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    log.Println("Shutting down...")

    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        log.Fatal("Server forced to shutdown:", err)
    }
}
```

**With Database Connection Cleanup**:
```go
func main() {
    db := initDatabase()
    defer db.Close() // Ensure DB connections are closed

    srv := setupServer(db)

    go func() {
        log.Println("Server starting...")
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()

    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    log.Println("Shutting down...")

    // Close database connections first
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    db.Close()

    // Then shutdown server
    ctx2, cancel2 := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel2()
    srv.Shutdown(ctx2)

    log.Println("Shutdown complete")
}
```

**Tracking In-Flight Requests**:
```go
type Server struct {
    inFlight int64
    mu       sync.Mutex
}

func (s *Server) trackRequest() func() {
    atomic.AddInt64(&s.inFlight, 1)
    return func() {
        atomic.AddInt64(&s.inFlight, -1)
    }
}

func (s *Server) waitForRequests(maxWait time.Duration) {
    deadline := time.Now().Add(maxWait)
    for time.Now().Before(deadline) {
        if atomic.LoadInt64(&s.inFlight) == 0 {
            return
        }
        time.Sleep(100 * time.Millisecond)
    }
    log.Printf("Warning: %d requests still in flight after shutdown timeout", atomic.LoadInt64(&s.inFlight))
}
```

---

#### Q66: How do you implement database query optimization and avoid N+1 queries in GORM?

**Answer**: Multiple optimization techniques:

**1. Eager Loading (Preloading)**:
```go
// N+1 Problem - BAD
func getUserBorrows(userID uint) ([]Borrow, error) {
    var borrows []Borrow
    db.Where("user_id = ?", userID).Find(&borrows)
    
    // This causes N+1 queries - one for each borrow's book
    for i := range borrows {
        db.Model(&borrows[i]).Association("Book").Find(&borrows[i].Book)
    }
    return borrows, nil
}

// Optimized - GOOD
func getUserBorrows(userID uint) ([]Borrow, error) {
    var borrows []Borrow
    db.Preload("Book").              // Eager load book
       Preload("User").              // Eager load user
       Where("user_id = ?", userID).
       Find(&borrows)
    return borrows, nil
}

// Conditional preloading
db.Preload("Book", func(db *gorm.DB) *gorm.DB {
    return db.Select("id, title, author") // Only load needed fields
}).Find(&borrows)
```

**2. Joins Instead of Preloads (When Appropriate)**:
```go
// Using Joins for better performance on large datasets
type BorrowWithBook struct {
    Borrow
    BookTitle  string
    BookAuthor string
}

func getUserBorrowsWithBooks(userID uint) ([]BorrowWithBook, error) {
    var results []BorrowWithBook
    db.Table("borrows").
       Select("borrows.*, books.title as book_title, books.author as book_author").
       Joins("LEFT JOIN books ON borrows.book_id = books.id").
       Where("borrows.user_id = ?", userID).
       Scan(&results)
    return results, nil
}
```

**3. Batch Loading**:
```go
// Load all related records in batches
var users []User
db.Find(&users)

var userIDs []uint
for _, user := range users {
    userIDs = append(userIDs, user.ID)
}

// Load all borrows in one query
var borrows []Borrow
db.Where("user_id IN ?", userIDs).Find(&borrows)

// Group by user
borrowsByUser := make(map[uint][]Borrow)
for _, borrow := range borrows {
    borrowsByUser[borrow.UserID] = append(borrowsByUser[borrow.UserID], borrow)
}
```

**4. Select Specific Fields**:
```go
// Instead of loading entire objects
db.Select("id", "title", "author").Find(&books)

// For nested preloads
db.Preload("Book", func(db *gorm.DB) *gorm.DB {
    return db.Select("id", "title", "author", "available_copies")
}).Find(&borrows)
```

**5. Using Raw SQL for Complex Queries**:
```go
type BorrowDetail struct {
    ID         uint
    UserName   string
    BookTitle  string
    BorrowedAt time.Time
    DueDate    time.Time
}

func getBorrowDetails() ([]BorrowDetail, error) {
    var results []BorrowDetail
    db.Raw(`
        SELECT 
            b.id,
            u.first_name || ' ' || u.last_name as user_name,
            bk.title as book_title,
            b.borrowed_at,
            b.due_date
        FROM borrows b
        JOIN users u ON b.user_id = u.id
        JOIN books bk ON b.book_id = bk.id
        WHERE b.status = 'borrowed'
        ORDER BY b.due_date ASC
    `).Scan(&results)
    return results, nil
}
```

**6. Query Result Caching**:
```go
func getPopularBooks(limit int) ([]Book, error) {
    cacheKey := fmt.Sprintf("popular_books:%d", limit)
    
    // Check cache
    if cached := redis.Get(cacheKey); cached != nil {
        var books []Book
        json.Unmarshal(cached, &books)
        return books, nil
    }
    
    // Query database
    var books []Book
    db.Order("total_copies - available_copies DESC").
       Limit(limit).
       Find(&books)
    
    // Cache result (5 minutes)
    data, _ := json.Marshal(books)
    redis.Set(cacheKey, data, 5*time.Minute)
    
    return books, nil
}
```

**7. Use Indexes for Joins**:
```go
// Ensure foreign keys are indexed
type Borrow struct {
    UserID uint `gorm:"index"`
    BookID uint `gorm:"index"`
}
```

---

#### Q67: How do you implement background job processing in Go (e.g., sending emails, generating reports)?

**Answer**: Multiple approaches for background jobs:

**1. Simple Goroutine Pool**:
```go
package worker

import (
    "context"
    "sync"
)

type Job func(context.Context) error

type WorkerPool struct {
    workers int
    jobs    chan Job
    wg      sync.WaitGroup
}

func NewWorkerPool(workers int, queueSize int) *WorkerPool {
    return &WorkerPool{
        workers: workers,
        jobs:    make(chan Job, queueSize),
    }
}

func (wp *WorkerPool) Start(ctx context.Context) {
    for i := 0; i < wp.workers; i++ {
        wp.wg.Add(1)
        go wp.worker(ctx)
    }
}

func (wp *WorkerPool) worker(ctx context.Context) {
    defer wp.wg.Done()
    for {
        select {
        case job, ok := <-wp.jobs:
            if !ok {
                return
            }
            if err := job(ctx); err != nil {
                log.Printf("Job failed: %v", err)
            }
        case <-ctx.Done():
            return
        }
    }
}

func (wp *WorkerPool) Submit(job Job) error {
    select {
    case wp.jobs <- job:
        return nil
    default:
        return errors.New("job queue full")
    }
}

func (wp *WorkerPool) Stop() {
    close(wp.jobs)
    wp.wg.Wait()
}

// Usage
func main() {
    pool := NewWorkerPool(10, 100)
    ctx := context.Background()
    pool.Start(ctx)

    // Submit jobs
    pool.Submit(func(ctx context.Context) error {
        return sendNotificationEmail(ctx, userID, message)
    })
}
```

**2. Cron Jobs**:
```go
import "github.com/robfig/cron/v3"

func setupCronJobs() {
    c := cron.New(cron.WithSeconds())

    // Daily job to check overdue books
    c.AddFunc("0 0 9 * * *", func() {
        checkOverdueBooks()
    })

    // Weekly report generation
    c.AddFunc("0 0 0 * * 0", func() {
        generateWeeklyReport()
    })

    // Hourly cleanup
    c.AddFunc("0 0 * * * *", func() {
        cleanupExpiredReservations()
    })

    c.Start()
}

func checkOverdueBooks() {
    var overdueBorrows []Borrow
    db.Where("due_date < ? AND status = ?", time.Now(), "borrowed").
       Find(&overdueBorrows)

    for _, borrow := range overdueBorrows {
        // Generate notification
        createOverdueNotification(borrow)
        
        // Calculate fine
        fine := borrow.CalculateFine()
        if fine > 0 {
            createFine(borrow.UserID, borrow.ID, fine)
        }
    }
}
```

**3. Using Message Queue (Redis + Worker)**:
```go
package worker

import (
    "github.com/go-redis/redis/v8"
    "context"
)

type EmailWorker struct {
    rdb    *redis.Client
    ctx    context.Context
}

func (w *EmailWorker) Start() {
    pubsub := w.rdb.Subscribe(w.ctx, "jobs:email")
    defer pubsub.Close()

    ch := pubsub.Channel()
    for msg := range ch {
        var job EmailJob
        json.Unmarshal([]byte(msg.Payload), &job)
        
        if err := w.processEmail(job); err != nil {
            log.Printf("Failed to process email: %v", err)
        }
    }
}

func (w *EmailWorker) processEmail(job EmailJob) error {
    // Send email logic
    return sendEmail(job.To, job.Subject, job.Body)
}

// Enqueue job
func EnqueueEmail(to, subject, body string) error {
    job := EmailJob{To: to, Subject: subject, Body: body}
    data, _ := json.Marshal(job)
    return rdb.Publish(ctx, "jobs:email", data).Err()
}
```

**4. Async Task with Retry**:
```go
type AsyncTask struct {
    ID        string
    TaskType  string
    Payload   []byte
    Retries   int
    MaxRetries int
}

func processTaskWithRetry(task AsyncTask) {
    for task.Retries < task.MaxRetries {
        err := executeTask(task)
        if err == nil {
            return
        }
        
        task.Retries++
        if task.Retries < task.MaxRetries {
            backoff := time.Duration(task.Retries) * time.Second
            time.Sleep(backoff)
        }
    }
    
    // Failed after max retries - log for manual intervention
    log.Printf("Task %s failed after %d retries", task.ID, task.MaxRetries)
}
```

---

#### Q68: How do you implement API versioning in Go to maintain backward compatibility?

**Answer**: Multiple versioning strategies:

**1. URL Path Versioning**:
```go
func setupRoutes() *gin.Engine {
    router := gin.Default()

    // Version 1
    v1 := router.Group("/api/v1")
    {
        v1.GET("/books", getBooksV1)
        v1.POST("/books", createBookV1)
    }

    // Version 2
    v2 := router.Group("/api/v2")
    {
        v2.GET("/books", getBooksV2) // New response format
        v2.POST("/books", createBookV2)
    }

    return router
}
```

**2. Header-Based Versioning**:
```go
func VersionMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        version := c.GetHeader("API-Version")
        if version == "" {
            version = "v1" // Default
        }
        c.Set("api_version", version)
        c.Next()
    }
}

func getBooks(c *gin.Context) {
    version := c.GetString("api_version")
    
    switch version {
    case "v2":
        getBooksV2(c)
    default:
        getBooksV1(c)
    }
}
```

**3. Response Wrapper for Compatibility**:
```go
type ResponseV1 struct {
    Books []BookV1 `json:"books"`
}

type ResponseV2 struct {
    Data struct {
        Books []BookV2 `json:"books"`
    } `json:"data"`
    Meta MetaInfo `json:"meta"`
}

func getBooks(c *gin.Context) {
    books := fetchBooks()
    version := c.GetString("api_version")

    switch version {
    case "v2":
        c.JSON(200, ResponseV2{
            Data: struct {
                Books []BookV2 `json:"books"`
            }{
                Books: convertToV2(books),
            },
            Meta: MetaInfo{Count: len(books)},
        })
    default:
        c.JSON(200, ResponseV1{
            Books: convertToV1(books),
        })
    }
}
```

**4. Deprecation Headers**:
```go
func DeprecationMiddleware(version string) gin.HandlerFunc {
    return func(c *gin.Context) {
        if version == "v1" {
            c.Header("Deprecation", "true")
            c.Header("Sunset", "2025-12-31") // Date when v1 will be removed
            c.Header("Link", "</api/v2/books>; rel=\"successor-version\"")
        }
        c.Next()
    }
}
```

**5. Version Negotiation**:
```go
func VersionNegotiation() gin.HandlerFunc {
    return func(c *gin.Context) {
        acceptHeader := c.GetHeader("Accept")
        
        // Parse: application/vnd.library.v2+json
        if strings.Contains(acceptHeader, "vnd.library.v2") {
            c.Set("api_version", "v2")
        } else if strings.Contains(acceptHeader, "vnd.library.v1") {
            c.Set("api_version", "v1")
        } else {
            c.Set("api_version", "v1") // Default
        }
        c.Next()
    }
}
```

**Document Version**: 1.0  
**Last Updated**: 2024  
**Maintained By**: Development Team