# Data Model Design

**Feature**: Todo Application  
**Date**: 2026-01-05  
**Source**: Derived from [spec.md](spec.md) Key Entities and [research.md](research.md) decisions

## Overview

This document defines the data model for the Todo application, including entities, relationships, validation rules, and state transitions. The model is designed to be implementation-agnostic while providing sufficient detail for database schema creation.

---

## Entities

### User

Represents an individual using the application for personal task management.

**Purpose**: Establish user identity and ownership of tasks.

**Attributes**:

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| `id` | Unique Identifier | Required, Unique, Immutable | System-generated unique identifier |
| `email` | String | Required, Unique, Format: valid email | User's email address for authentication |
| `passwordHash` | String | Required, Min length: 60 chars | Bcrypt hash of user's password (never store plaintext) |
| `createdAt` | Timestamp | Required, Auto-generated | Account creation time |
| `updatedAt` | Timestamp | Required, Auto-updated | Last account modification time |

**Validation Rules**:
- Email must match pattern: `^[^\s@]+@[^\s@]+\.[^\s@]+$`
- Password (before hashing) must be 8-64 characters
- Email is case-insensitive (normalize to lowercase)

**Relationships**:
- One User has many Tasks (one-to-many)
- User cannot be deleted if Tasks exist (referential integrity)

**Indexes**:
- Primary: `id` (unique)
- Unique: `email` (for login lookup)

---

### Task

Represents a unit of work to be completed by a user.

**Purpose**: Capture, track, and organize user tasks.

**Attributes**:

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| `id` | Unique Identifier | Required, Unique, Immutable | System-generated unique identifier |
| `userId` | User Identifier (FK) | Required, Immutable | Owner of this task (foreign key to User) |
| `description` | String | Required, Length: 1-500 chars | Text describing the task |
| `isComplete` | Boolean | Required, Default: false | Completion status |
| `order` | Number | Required, Default: 0 | User-defined display order (lower = higher priority) |
| `createdAt` | Timestamp | Required, Auto-generated | Task creation time |
| `completedAt` | Timestamp | Optional, Nullable | Time when task was marked complete (null if incomplete) |
| `updatedAt` | Timestamp | Required, Auto-updated | Last modification time (for conflict detection) |

**Validation Rules**:
- Description must not be empty or whitespace-only (trim before validation)
- Description length: 1-500 characters after trimming
- `order` must be a non-negative integer
- `completedAt` must be null when `isComplete` is false
- `completedAt` must be set when `isComplete` transitions to true
- `updatedAt` must be later than or equal to `createdAt`

**Relationships**:
- Many Tasks belong to one User (many-to-one)
- Task cannot exist without a User (cascade delete)

**Indexes**:
- Primary: `id` (unique)
- Composite: `userId + createdAt DESC` (default task list retrieval)
- Index: `userId + order` (manual ordering support)

---

## Relationships Diagram

```
User (1) ─────< (many) Task
  │                      │
  └─ id                  └─ userId (FK)
```

**Cardinality**:
- One User can have zero or many Tasks
- One Task belongs to exactly one User
- Tasks are not shared between Users

**Constraints**:
- `Task.userId` must reference a valid `User.id`
- Deleting a User cascades delete to all owned Tasks
- Tasks cannot be orphaned (no null `userId`)

---

## State Transitions

### Task Completion States

Tasks have two completion states: **Incomplete** and **Complete**.

**State Diagram**:
```
    ┌─────────────┐
    │ Incomplete  │ (default)
    │ isComplete: │
    │   false     │
    └─────────────┘
         │   ▲
         │   │
 toggle  │   │ toggle
complete │   │ incomplete
         │   │
         ▼   │
    ┌─────────────┐
    │  Complete   │
    │ isComplete: │
    │   true      │
    └─────────────┘
```

**Transition Rules**:

1. **Incomplete → Complete**:
   - Set `isComplete = true`
   - Set `completedAt = current timestamp`
   - Update `updatedAt`
   - Visual feedback: strikethrough or checkmark

2. **Complete → Incomplete**:
   - Set `isComplete = false`
   - Set `completedAt = null`
   - Update `updatedAt`
   - Visual feedback: remove strikethrough

**Invariants**:
- If `isComplete = true`, then `completedAt` must not be null
- If `isComplete = false`, then `completedAt` must be null
- `completedAt` must never precede `createdAt`

---

## Data Validation Rules

### User Entity

| Rule ID | Field | Validation | Error Message |
|---------|-------|------------|---------------|
| U-001 | email | Not empty | "Email is required" |
| U-002 | email | Valid email format | "Invalid email address" |
| U-003 | email | Unique in system | "Email already registered" |
| U-004 | password | 8-64 characters | "Password must be 8-64 characters" |
| U-005 | password | Not empty | "Password is required" |

### Task Entity

| Rule ID | Field | Validation | Error Message |
|---------|-------|------------|---------------|
| T-001 | description | Not empty/whitespace | "Task description is required" |
| T-002 | description | 1-500 characters | "Description must be 1-500 characters" |
| T-003 | userId | References valid User | "Invalid user" |
| T-004 | order | Non-negative integer | "Order must be >= 0" |
| T-005 | completedAt | Null when isComplete=false | "Completed timestamp inconsistent" |
| T-006 | completedAt | Not null when isComplete=true | "Completed timestamp missing" |

---

## Consistency Rules

### Single-User Isolation

**Rule**: Users can only access tasks where `Task.userId = authenticatedUser.id`

**Enforcement**:
- All task queries MUST filter by authenticated user's ID
- Authorization layer validates ownership before updates/deletes
- API responses MUST NOT leak tasks belonging to other users

**Example Query Pattern**:
```
// Correct: filtered by userId
SELECT * FROM tasks WHERE userId = :authenticatedUserId

// Incorrect: no user filter
SELECT * FROM tasks
```

### Concurrent Modification Handling

**Rule**: Last-write-wins with optimistic locking

**Enforcement**:
- Client sends `updatedAt` timestamp with update requests
- Server compares client timestamp with database timestamp
- If `clientTimestamp < dbTimestamp`, reject with 409 Conflict
- If `clientTimestamp >= dbTimestamp`, accept update and refresh `updatedAt`

**Example Flow**:
```
1. Client fetches task: { id: "123", description: "Buy milk", updatedAt: "2026-01-05T10:00:00Z" }
2. User edits description to "Buy milk and eggs"
3. Client sends: PUT /api/tasks/123 { description: "...", updatedAt: "2026-01-05T10:00:00Z" }
4. Server checks: if task.updatedAt > "2026-01-05T10:00:00Z" → 409 Conflict
5. Otherwise: update task, set updatedAt = now()
```

---

## Ordering Behavior

### Default Order

Tasks are displayed in **reverse chronological order** by creation time (newest first).

**Default Sort**: `createdAt DESC`

### Manual Reordering

Users can manually reorder tasks by dragging or similar interaction.

**Implementation**:
- Each task has an `order` field (integer)
- Lower `order` value = higher display position
- When user reorders, update `order` values for affected tasks
- Sort by `order ASC` when manual ordering is active

**Reorder Algorithm** (simplified):
```
1. User moves Task B (order: 2) between Task A (order: 1) and Task C (order: 3)
2. Set Task B order = 1.5 (midpoint)
3. Optional: periodically renormalize orders to integers (1, 2, 3, ...)
```

---

## Performance Considerations

### Indexing Strategy

**Critical Indexes**:
1. `User.email` (unique) - Login queries (frequent)
2. `Task.userId + Task.createdAt DESC` (composite) - Default task list retrieval (most common query)
3. `Task.userId + Task.order` (composite) - Manual ordering support

**Query Patterns**:
- List all user tasks: `WHERE userId = ? ORDER BY createdAt DESC`
- Filter completed tasks: `WHERE userId = ? AND isComplete = true`
- Search tasks (future): `WHERE userId = ? AND description LIKE ?`

### Pagination

For task lists exceeding 100 items, implement cursor-based pagination:
- Cursor: `createdAt` timestamp of last task in page
- Next page: `WHERE userId = ? AND createdAt < :cursor ORDER BY createdAt DESC LIMIT 50`

---

## Security Constraints

### Authorization

**Ownership Verification**:
- All task operations (read, update, delete) MUST verify `Task.userId = authenticatedUser.id`
- Unauthorized access returns `403 Forbidden` (not 404 to avoid leaking task existence)

### Data Sanitization

**Input Sanitization**:
- Strip HTML tags from task descriptions (prevent XSS)
- Trim whitespace from description before validation
- Normalize email to lowercase before storage

**Example**:
```
Input:  "  <script>alert('xss')</script>Buy groceries  "
Output: "Buy groceries" (trimmed, tags stripped)
```

---

## Migration Strategy

### Schema Versioning

**Version 1.0** (Initial):
- User: id, email, passwordHash, createdAt, updatedAt
- Task: id, userId, description, isComplete, order, createdAt, completedAt, updatedAt

**Future Extensions** (Out of Scope for MVP):
- Task.tags: Array of strings (requires index on array field)
- Task.dueDate: Timestamp (requires new index on userId + dueDate)
- Task.category: String (requires category lookup table)

---

## Summary

This data model supports all functional requirements from the specification:
- **FR-001-005**: CRUD operations on Task entity
- **FR-006**: Persistence via database storage
- **FR-007-008**: Ordering and display via `order` and `createdAt` fields
- **FR-009**: Concurrent updates via `updatedAt` timestamp
- **FR-010**: User isolation via `userId` foreign key
- **FR-011**: Timestamps for creation and completion

**Validation Coverage**:
- All required fields enforced
- All constraints documented
- All state transitions defined

**Next Steps**:
- Implement database schema (MongoDB collections or SQL tables)
- Create API contracts referencing this data model
- Write validation logic in backend services
