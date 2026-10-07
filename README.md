# TaskFlow

TaskFlow is a full-stack task management platform for creating, organizing, tracking, searching, and managing work through a centralized web application.

The project is built around a **Go REST API**, **PostgreSQL database**, and **HTML/CSS/JavaScript frontend**, with automated testing and eventual AWS deployment.

TaskFlow is being developed as a complete software engineering portfolio project rather than as a collection of disconnected features. The project covers the lifecycle of designing a full-stack system: defining requirements, designing data models and APIs, implementing backend services, integrating a relational database, building a frontend client, testing application behavior, and deploying the completed system.

> **Project Status:** Active Development  
> **Current Phase:** Project Foundation

---

# Table of Contents

- [Project Overview](#project-overview)
- [Problem](#problem)
- [Project Scope](#project-scope)
- [MVP](#minimum-viable-product-mvp)
- [Functional Requirements](#functional-requirements)
- [Task Model](#task-model)
- [Task Lifecycle](#task-lifecycle)
- [System Architecture](#system-architecture)
- [Technology Stack](#technology-stack)
- [Backend Scope](#backend-scope)
- [REST API](#rest-api)
- [Database Scope](#database-scope)
- [Frontend Scope](#frontend-scope)
- [Search and Filtering](#search-and-filtering)
- [Validation and Error Handling](#validation-and-error-handling)
- [Testing](#testing)
- [Security and Configuration](#security-and-configuration)
- [Deployment](#deployment)
- [Project Structure](#project-structure)
- [Development Roadmap](#development-roadmap)
- [Definition of Done](#definition-of-done)
- [Future Scope](#future-scope)

---

# Project Overview

TaskFlow provides a web interface where a user can manage tasks throughout their lifecycle.

Instead of tasks existing only as temporary frontend data, TaskFlow uses a backend application and relational database.

A user will be able to create a task such as:

```text
Title: Finish database assignment

Description:
Complete PostgreSQL exercises and submit the assignment.

Status:
In Progress

Priority:
High
```

The task is sent from the browser to the TaskFlow API.

The Go backend validates the request and communicates with PostgreSQL to store the task.

When the user returns to TaskFlow later, the task can be retrieved from PostgreSQL and displayed again.

The complete flow is:

```text
User
  │
  ▼
TaskFlow Web Interface
  │
  │ HTTP Request
  ▼
Go REST API
  │
  ├── Validate Request
  ├── Execute Application Logic
  │
  ▼
PostgreSQL
  │
  ▼
API Response
  │
  ▼
JavaScript
  │
  ▼
Updated User Interface
```

TaskFlow therefore acts as one complete system rather than separate frontend and backend demonstrations.

---

# Problem

Managing work requires more than simply writing down a task.

Tasks frequently have different levels of importance and different stages of completion. As the number of tasks grows, users also need ways to locate and organize them.

TaskFlow addresses this by giving users a centralized system for:

- Recording tasks
- Describing work that needs to be completed
- Assigning priority
- Tracking progress
- Updating existing work
- Removing tasks that are no longer needed
- Filtering tasks
- Searching existing tasks
- Persisting information between sessions

The initial version intentionally focuses on these fundamental workflows instead of attempting to compete with large project-management systems.

---

# Project Scope

TaskFlow's primary scope is a **single-user task management application**.

The application will contain three major software layers:

### Frontend

Responsible for displaying tasks and accepting user interaction.

### Backend

Responsible for the API, validation, application behavior, and communication between the frontend and database.

### Database

Responsible for persistent storage and retrieval of application data.

The relationship is:

```text
Frontend
   ↓
REST API
   ↓
Backend
   ↓
Database
```

The first production-capable version of TaskFlow should allow someone to perform the complete lifecycle of a task without directly interacting with the database or backend.

---

# Minimum Viable Product (MVP)

The MVP represents the first complete version of TaskFlow.

A user must be able to:

- Create a task
- View all tasks
- View an individual task
- Edit a task
- Delete a task
- Assign a status
- Assign a priority
- Search tasks
- Filter tasks by status
- Filter tasks by priority
- Refresh/reopen the application without losing stored tasks

The application must also provide:

- PostgreSQL persistence
- REST API
- Input validation
- Error handling
- Automated backend testing
- Responsive web interface
- Environment-based configuration
- Production deployment

Anything beyond these requirements is secondary to completing the MVP.

---

# Functional Requirements

## FR-01 — Create Task

The user must be able to create a new task.

A task will contain at minimum:

- Title
- Description
- Status
- Priority

The backend must validate the submitted information before storing the task.

---

## FR-02 — View Tasks

The user must be able to retrieve and view existing tasks.

The application must support:

- Viewing all tasks
- Viewing an individual task

Task information must originate from persistent storage rather than only existing in browser memory.

---

## FR-03 — Edit Task

The user must be able to modify an existing task.

Editable information will include:

- Title
- Description
- Status
- Priority

Changes must persist in PostgreSQL.

---

## FR-04 — Delete Task

The user must be able to delete an existing task.

The backend must identify the correct task and remove it from persistent storage.

---

## FR-05 — Task Status

Tasks must contain a status representing their current stage.

Initial statuses:

```text
To Do
In Progress
Completed
```

The exact database representation will be decided during database design.

---

## FR-06 — Task Priority

Tasks must support priority levels.

Initial priorities:

```text
Low
Medium
High
```

Priority allows important work to be distinguished from less urgent work.

---

## FR-07 — Search

The user must be able to search existing tasks.

Initial search scope should include task titles.

Searching descriptions may be introduced if appropriate during implementation.

---

## FR-08 — Filtering

Users must be able to filter tasks using attributes such as:

```text
Status
Priority
```

Examples:

```text
Show all High priority tasks.

Show all Completed tasks.

Show all High priority tasks that are In Progress.
```

---

## FR-09 — Persistent Storage

Tasks must survive:

- Browser refreshes
- Backend restarts
- User sessions

PostgreSQL will serve as the source of truth for persisted task information.

---

# Task Model

The central domain object is the **Task**.

Conceptually:

```text
Task
│
├── ID
├── Title
├── Description
├── Status
├── Priority
├── CreatedAt
└── UpdatedAt
```

Possible representation:

| Field | Purpose |
|---|---|
| ID | Unique task identifier |
| Title | Short description of the task |
| Description | Detailed task information |
| Status | Current task state |
| Priority | Importance of task |
| CreatedAt | When the task was created |
| UpdatedAt | When the task was last modified |

This represents the domain requirements.

The exact Go types and PostgreSQL column types will be determined during implementation rather than prematurely locking the project into a schema.

---

# Task Lifecycle

A typical task moves through:

```text
        CREATE
          │
          ▼
        TO DO
          │
          ▼
     IN PROGRESS
          │
          ▼
      COMPLETED
```

A task may also be:

```text
Edited
Prioritized
Searched
Filtered
Deleted
```

TaskFlow does not initially require complex workflow rules.

For example, a task does not need to pass through every status before being completed.

---

# System Architecture

TaskFlow uses a client-server architecture.

```text
┌───────────────────────────────────────┐
│                CLIENT                 │
│                                       │
│        HTML + CSS + JavaScript        │
│                                       │
│  Forms • Task List • Search • Filter  │
└───────────────────┬───────────────────┘
                    │
                    │ HTTP
                    │ JSON
                    ▼
┌───────────────────────────────────────┐
│              GO BACKEND               │
│                                       │
│              REST API                 │
│                   │                   │
│                   ▼                   │
│              Validation               │
│                   │                   │
│                   ▼                   │
│          Application Logic            │
│                   │                   │
│                   ▼                   │
│          Database Operations          │
└───────────────────┬───────────────────┘
                    │
                    │ SQL
                    ▼
┌───────────────────────────────────────┐
│              POSTGRESQL               │
│                                       │
│          Persistent Task Data         │
└───────────────────────────────────────┘
```

A normal request might therefore follow:

```text
User clicks "Create Task"

        ↓

JavaScript reads form

        ↓

POST /api/tasks

        ↓

Go HTTP handler

        ↓

Validate task

        ↓

Application logic

        ↓

INSERT INTO PostgreSQL

        ↓

Database returns stored task

        ↓

Go returns JSON response

        ↓

JavaScript updates interface
```

---

# Technology Stack

## Backend

### Go

Go is the primary backend language.

Responsibilities include:

- Starting the HTTP server
- Routing requests
- Processing HTTP requests
- Parsing JSON
- Validating data
- Executing application logic
- Communicating with PostgreSQL
- Returning HTTP responses
- Handling errors
- Backend testing

The backend should initially favor Go's standard library where practical before introducing unnecessary frameworks.

---

## Frontend

### HTML

Provides the structure of the application.

### CSS

Provides layout, responsive behavior, and visual presentation.

### JavaScript

Provides browser-side application behavior.

JavaScript will be responsible for:

- Handling forms
- Sending HTTP requests
- Receiving JSON
- Rendering task data
- Updating task information
- Deleting tasks
- Search interaction
- Filtering interaction
- Displaying errors

The MVP does not require a frontend framework.

A framework can be evaluated later only if the application grows enough to justify it.

---

## Database

### PostgreSQL

PostgreSQL is responsible for persistent application data.

The project will use SQL directly to develop an understanding of relational database operations.

---

## Communication

### HTTP + REST + JSON

The frontend communicates with the Go backend through REST-style HTTP endpoints.

JSON serves as the primary data format between the client and server.

---

## Development

- Git
- GitHub

---

## Deployment

- AWS

AWS services will be selected when deployment requirements are known.

---

# Backend Scope

The backend will be responsible for four major areas.

```text
HTTP
 ↓
Handlers
 ↓
Application Logic
 ↓
Database
```

## HTTP Layer

Responsible for:

- Receiving requests
- Reading request parameters
- Decoding JSON
- Returning responses
- Returning appropriate HTTP status codes

## Validation

Responsible for ensuring incoming task data is acceptable.

Examples:

```text
Title cannot be empty.

Status must be supported.

Priority must be supported.

Task ID must be valid.
```

## Application Logic

Responsible for application behavior independent of the HTTP interface.

## Data Access

Responsible for communicating with PostgreSQL.

Keeping these responsibilities understandable and separated will become increasingly important as the project grows.

---

# REST API

The planned API is centered around `/api/tasks`.

## Create Task

```http
POST /api/tasks
```

Creates a new task.

---

## Get Tasks

```http
GET /api/tasks
```

Returns tasks.

This endpoint may eventually accept query parameters for searching and filtering.

Example concept:

```text
GET /api/tasks?status=completed

GET /api/tasks?priority=high

GET /api/tasks?search=database
```

---

## Get Task

```http
GET /api/tasks/{id}
```

Returns one task.

---

## Update Task

```http
PUT /api/tasks/{id}
```

Updates an existing task.

Whether `PATCH` is introduced for partial updates will be evaluated during API design.

---

## Delete Task

```http
DELETE /api/tasks/{id}
```

Deletes a task.

---

# HTTP Behavior

The API should use meaningful HTTP status codes.

Examples include:

```text
200 OK

201 Created

204 No Content

400 Bad Request

404 Not Found

405 Method Not Allowed

500 Internal Server Error
```

Exact response contracts will be documented as endpoints are implemented.

---

# Database Scope

PostgreSQL will eventually contain the application's task records.

A conceptual table might resemble:

```text
tasks
────────────────────
id
title
description
status
priority
created_at
updated_at
```

Database development will include:

- Table creation
- Primary keys
- Constraints
- SQL CRUD operations
- Parameterized queries
- Database connection management
- Error handling
- Search queries
- Filtering queries

SQL statements must not be constructed by directly concatenating untrusted user input.

---

# Frontend Scope

The frontend will eventually contain several major interface areas.

## Task Creation

Form for entering:

```text
Title
Description
Priority
Status
```

## Task List

Displays existing tasks and important information such as:

```text
Title
Status
Priority
```

## Task Actions

Each appropriate task should allow actions such as:

```text
Edit
Delete
Change Status
```

## Search

Users can enter search terms to locate tasks.

## Filters

Users can narrow the task list by attributes such as status and priority.

## Feedback

The interface should communicate application state.

Examples include:

```text
Task created successfully.

Unable to create task.

No tasks found.

Task deleted.

Server unavailable.
```

---

# Search and Filtering

Search and filtering are part of the MVP rather than decorative frontend features.

The architecture should eventually support queries such as:

```text
All tasks

Completed tasks

High-priority tasks

High-priority completed tasks

Tasks matching "PostgreSQL"
```

Where appropriate, filtering should be handled through the backend/database rather than requiring the browser to retrieve every record and perform all processing locally.

---

# Validation and Error Handling

TaskFlow must handle invalid input and application failures predictably.

Examples include:

```text
Missing title

Invalid status

Invalid priority

Invalid task ID

Task does not exist

Malformed JSON

Database unavailable

Unexpected server error
```

The backend should not expose sensitive internal implementation details to the client.

The frontend should convert API failures into understandable user feedback.

---

# Testing

Testing is part of the project scope.

Potential testing areas include:

### Unit Tests

Test isolated application behavior.

### Handler Tests

Verify HTTP endpoints.

Examples:

```text
POST valid task → expected success

POST invalid task → expected validation error

GET unknown task → 404

DELETE unknown task → expected error response
```

### Database Tests

Verify data-access behavior where appropriate.

### Integration Tests

Verify that multiple layers work together correctly.

Testing will be introduced alongside functionality instead of waiting until the entire application has been completed.

---

# Security and Configuration

Sensitive configuration must remain outside source code.

The repository must never contain real:

```text
Database passwords
AWS access keys
AWS secret keys
API secrets
Access tokens
Private keys
Production credentials
```

Environment variables will eventually be used for configuration such as:

```text
DATABASE_URL
PORT
APP_ENV
```

Example configuration files
