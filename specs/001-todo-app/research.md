# Phase 0: Research & Technical Decisions

**Feature**: Todo Application  
**Date**: 2026-01-05  
**Status**: Complete

## Overview

This document consolidates all research findings and technical decisions made to resolve unknowns from the Technical Context. Each decision is documented with rationale and alternatives considered.

---

## 1. State Management Strategy (Frontend)

### Decision: Zustand for Global State

**Rationale:**
- Lightweight (~1KB) vs Redux (~10KB)
- Simpler API without boilerplate (actions, reducers, providers)
- Built-in TypeScript support
- Supports optimistic updates via middleware
- React Context sufficient for auth state only

**Alternatives Considered:**
- **Redux Toolkit**: Rejected due to boilerplate overhead for simple task state management
- **React Context only**: Rejected because task list re-renders would trigger entire component tree updates
- **Recoil**: Rejected due to atom/selector complexity unnecessary for linear task list

**Best Practices:**
- Separate stores for auth vs tasks
- Normalize task data by ID for efficient updates
- Implement optimistic update middleware for instant UI feedback

---

## 2. Database Schema Design

### Decision: MongoDB with Embedded Task Documents

**Rationale:**
- Task entity maps naturally to document structure
- No complex relationships (tasks don't reference other tasks)
- Flexible schema supports future additions (tags, due dates) without migrations
- Atomic update operations prevent race conditions
- Query performance: index on `userId` + `createdAt` for default sort

**Alternatives Considered:**
- **PostgreSQL with relational schema**: Rejected because:
  - Task-User relationship is simple one-to-many (no joins needed)
  - Schema migrations add deployment complexity
  - ACID guarantees overkill for single-document updates
- **SQLite**: Rejected for production (dev-only database, not cloud-native)

**Schema Strategy:**
```json
// User Document
{
  "_id": "ObjectId",
  "email": "string (unique)",
  "passwordHash": "string",
  "createdAt": "ISODate",
  "updatedAt": "ISODate"
}

// Task Document
{
  "_id": "ObjectId",
  "userId": "ObjectId (indexed)",
  "description": "string (1-500 chars)",
  "isComplete": "boolean",
  "order": "number (user-defined position)",
  "createdAt": "ISODate",
  "completedAt": "ISODate | null",
  "updatedAt": "ISODate"
}
```

**Indexes:**
- `userId + createdAt (desc)` - default task retrieval
- `email (unique)` - user lookup during auth

---

## 3. Authentication Implementation

### Decision: JWT with HttpOnly Cookies

**Rationale:**
- Stateless auth (no session storage required)
- Token contains userId claim for authorization
- HttpOnly cookies prevent XSS attacks (localStorage vulnerable)
- Refresh token pattern enables long-lived sessions

**Alternatives Considered:**
- **Session-based auth**: Rejected due to server-side session storage requirement (violates stateless architecture)
- **JWT in localStorage**: Rejected due to XSS vulnerability
- **OAuth2**: Rejected as out-of-scope (spec assumes auth handled separately)

**Best Practices:**
- Access token: 15-minute expiry
- Refresh token: 7-day expiry in separate HttpOnly cookie
- Token rotation on refresh
- Middleware validates token on every protected route

---

## 4. API Design Patterns

### Decision: RESTful Resources with JSON

**Rationale:**
- Standard HTTP semantics (GET, POST, PUT, DELETE)
- Predictable URL structure (`/api/tasks/:id`)
- Stateless (each request self-contained)
- Easy to test with curl/Postman
- Browser-native support (Fetch API)

**Endpoints:**
```
POST   /api/auth/register      - Create user account
POST   /api/auth/login         - Authenticate user
POST   /api/auth/refresh       - Refresh access token
POST   /api/auth/logout        - Invalidate tokens

GET    /api/tasks              - List user tasks (supports ?completed=true)
POST   /api/tasks              - Create new task
GET    /api/tasks/:id          - Get single task (unused in MVP)
PUT    /api/tasks/:id          - Update task (description, completion, order)
DELETE /api/tasks/:id          - Delete task
```

**Alternatives Considered:**
- **GraphQL**: Rejected due to over-engineering for simple CRUD
- **gRPC**: Rejected due to browser compatibility issues
- **Custom RPC**: Rejected in favor of REST standard

**Error Response Format:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Task description is required",
    "details": [
      { "field": "description", "issue": "must not be empty" }
    ]
  }
}
```

---

## 5. Concurrent Session Handling

### Decision: Last-Write-Wins with Optimistic Locking

**Rationale:**
- Spec explicitly requires last-write-wins (FR-009)
- Optimistic locking via `updatedAt` timestamp comparison
- If client timestamp < server timestamp, warn user of conflict
- Simple implementation: MongoDB `findOneAndUpdate` is atomic

**Alternatives Considered:**
- **Pessimistic locking**: Rejected (too complex for single-user context)
- **Operational Transform (OT)**: Rejected (overkill for task descriptions)
- **No conflict detection**: Rejected (silent data loss violates UX expectations)

**Implementation:**
```javascript
// Client sends updatedAt timestamp with update
PUT /api/tasks/:id
{ "description": "...", "updatedAt": "2026-01-05T10:00:00Z" }

// Server compares timestamps
if (req.body.updatedAt < task.updatedAt) {
  return 409 Conflict { message: "Task was modified in another session" }
}
```

---

## 6. Performance Optimization Strategy

### Decision: Pagination + Virtual Scrolling

**Rationale:**
- Spec requires 1000+ tasks without lag (FR-015)
- Loading 1000 DOM nodes causes browser slowdown
- Virtual scrolling renders only visible items (20-30 nodes)
- Backend pagination reduces initial load time

**Alternatives Considered:**
- **Load all tasks**: Rejected (violates performance requirement)
- **Infinite scroll without virtualization**: Rejected (DOM grows unbounded)
- **Server-side rendering**: Rejected (unnecessary complexity for SPA)

**Best Practices:**
- Frontend: `react-window` or `react-virtualized` library
- Backend: Cursor-based pagination (vs offset/limit)
- Default page size: 50 tasks
- Debounce reorder operations (300ms) to reduce API calls

---

## 7. Testing Strategy Details

### Decision: Test Pyramid with E2E Focus

**Rationale:**
- Unit tests: Fast feedback on business logic (services, validators)
- Integration tests: API contract verification
- E2E tests: Cover critical user journeys (P1 stories)
- Target: 80% coverage per constitution

**Test Distribution:**
- **70% Unit Tests**:
  - Task validation logic (empty description, character limits)
  - Auth token generation/verification
  - Date formatting utilities
  
- **20% Integration Tests**:
  - API endpoint responses (status codes, JSON structure)
  - Database operations (CRUD, indexing)
  - Auth middleware (protected routes)
  
- **10% E2E Tests**:
  - User Story 1: Create task → persists → reload
  - User Story 2: Complete task → visual change → persists
  - User Story 3: View tasks → empty state → populated list

**Tools:**
- Jest: Backend unit/integration
- Vitest: Frontend unit (Vite compatibility)
- React Testing Library: Component behavior
- Playwright: E2E user flows

---

## 8. Build & Deployment Pipeline

### Decision: Separate Frontend/Backend Deployments

**Rationale:**
- Frontend (static files): CDN/edge hosting (Vercel, Netlify)
- Backend (Node.js): Container-based hosting (Render, Railway)
- Independent scaling (frontend serves cached assets)
- Align with library-first architecture (separate deployments)

**Alternatives Considered:**
- **Monolithic deployment**: Rejected (frontend doesn't need Node.js runtime)
- **Serverless backend**: Rejected (cold start latency conflicts with <200ms requirement)

**Environment Variables:**
```
# Backend (.env)
MONGODB_URI=mongodb://...
JWT_SECRET=...
JWT_EXPIRY=15m
REFRESH_SECRET=...
CORS_ORIGIN=https://frontend.domain.com

# Frontend (.env)
VITE_API_URL=https://api.domain.com
```

---

## 9. Error Handling & Observability

### Decision: Centralized Error Boundary + Structured Logging

**Rationale:**
- React Error Boundary catches UI crashes (prevents white screen)
- Backend middleware standardizes error responses
- Structured JSON logs for debugging (timestamp, userId, error stack)
- Align with Constitution V (User-Centric Design - no silent failures)

**Frontend Error Boundary:**
```jsx
<ErrorBoundary fallback={<ErrorPage />}>
  <TaskList />
</ErrorBoundary>
```

**Backend Error Middleware:**
```javascript
app.use((err, req, res, next) => {
  logger.error({ err, userId: req.user?.id, path: req.path })
  res.status(err.status || 500).json({
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message: err.message || 'An error occurred'
    }
  })
})
```

---

## 10. Security Measures

### Decision: Defense-in-Depth Strategy

**Rationale:**
- Input validation prevents injection attacks
- JWT auth prevents unauthorized access (FR-010)
- CORS restricts API access to known frontend
- Rate limiting prevents abuse

**Security Layers:**

1. **Input Validation**:
   - `express-validator` on all endpoints
   - Sanitize HTML/script tags in task descriptions
   - Validate string lengths (1-500 chars)

2. **Authentication**:
   - bcrypt for password hashing (10 rounds)
   - JWT signature verification
   - Token expiry enforcement

3. **Authorization**:
   - Middleware checks `userId` matches token claim
   - Database queries filter by `userId` (prevents cross-user access)

4. **Transport**:
   - HTTPS only in production
   - Secure + HttpOnly cookies
   - CORS whitelist

---

## Summary of Research Findings

| Topic | Decision | Risk Mitigation |
|-------|----------|-----------------|
| State Management | Zustand | Lightweight, simple API |
| Database | MongoDB | Flexible schema, atomic ops |
| Auth | JWT + HttpOnly | XSS protection via cookies |
| API | REST + JSON | Standard, debuggable |
| Concurrency | Last-write-wins | Conflict detection via timestamps |
| Performance | Pagination + Virtual Scroll | Handles 1000+ tasks smoothly |
| Testing | Test Pyramid (70/20/10) | 80% coverage target |
| Deployment | Separate Frontend/Backend | Independent scaling |
| Errors | Centralized Boundaries | No silent failures |
| Security | Defense-in-Depth | Input validation, JWT, CORS |

---

## Next Steps

All NEEDS CLARIFICATION items from Technical Context are resolved. Ready to proceed to:
- **Phase 1**: Data model design (`data-model.md`)
- **Phase 1**: API contract documentation (`contracts/`)
- **Phase 1**: Developer quickstart guide (`quickstart.md`)
