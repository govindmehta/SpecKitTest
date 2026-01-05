# Tasks: Todo Application

**Input**: Design documents from `/specs/001-todo-app/`
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [data-model.md](data-model.md), [contracts/api.md](contracts/api.md)

**Organization**: Tasks are grouped by user story (from spec.md) to enable independent implementation and testing. Each user story phase represents a complete, independently deliverable increment.

## Format: `- [ ] [ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: User story label (US1, US2, etc.) - only for story-specific tasks
- Include exact file paths in descriptions

---

## Phase 1: Setup (Project Initialization)

**Purpose**: Initialize project structure, install dependencies, configure environments

**Goal**: Backend and frontend applications run locally with health checks passing

### Backend Setup

- [ ] T001 [P] Initialize Node.js project with package.json in backend/
- [ ] T002 [P] Install backend dependencies (Express, Mongoose, jsonwebtoken, bcrypt, express-validator, cors, helmet, dotenv) in backend/
- [ ] T003 [P] Install backend dev dependencies (Jest, Supertest, nodemon, eslint) in backend/
- [ ] T004 Create backend/.env.example with all required environment variables (PORT, MONGODB_URI, JWT_SECRET, REFRESH_SECRET, CORS_ORIGIN)
- [ ] T005 Create backend/src/server.js with Express app initialization, middleware setup, and MongoDB connection
- [ ] T006 [P] Configure backend/package.json scripts (start, dev, test, test:watch, test:coverage)
- [ ] T007 [P] Configure Jest in backend/jest.config.js for unit and integration tests

### Frontend Setup

- [ ] T008 [P] Initialize Vite React project in frontend/
- [ ] T009 [P] Install frontend dependencies (zustand, axios, react-router-dom) in frontend/
- [ ] T010 [P] Install frontend dev dependencies (vitest, @testing-library/react, @testing-library/jest-dom, @playwright/test) in frontend/
- [ ] T011 Create frontend/.env.example with VITE_API_URL variable
- [ ] T012 [P] Configure Vite in frontend/vite.config.js with test globals and jsdom environment
- [ ] T013 [P] Configure frontend/package.json scripts (dev, build, preview, test, test:e2e)

### Verification

- [ ] T014 Verify backend starts successfully with `npm run dev` and connects to MongoDB
- [ ] T015 Verify frontend starts successfully with `npm run dev` on port 5173

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that all user stories depend on

**Goal**: Authentication system working, data models defined, middleware in place

### Data Models

- [ ] T016 [P] Create User model in backend/src/models/User.js with email, passwordHash, timestamps per data-model.md
- [ ] T017 [P] Create Task model in backend/src/models/Task.js with userId, description, isComplete, order, timestamps per data-model.md
- [ ] T018 [P] Add indexes to User model (email unique index)
- [ ] T019 [P] Add indexes to Task model (userId + createdAt composite, userId + order composite)

### Authentication Infrastructure

- [ ] T020 Create AuthService in backend/src/services/AuthService.js with register, login, generateTokens methods
- [ ] T021 Create JWT auth middleware in backend/src/middleware/auth.js to verify access tokens and attach user to request
- [ ] T022 Create validation middleware in backend/src/middleware/validate.js using express-validator
- [ ] T023 Create error handling middleware in backend/src/middleware/errorHandler.js for consistent error responses
- [ ] T024 Create auth routes in backend/src/routes/auth.js (POST /register, POST /login, POST /refresh, POST /logout)
- [ ] T025 Wire auth routes into backend/src/server.js

### Frontend Infrastructure

- [ ] T026 [P] Create Axios client in frontend/src/services/api.js with baseURL, interceptors for tokens and error handling
- [ ] T027 [P] Create auth store in frontend/src/store/authStore.js using Zustand (user, login, logout, register methods)
- [ ] T028 [P] Create ErrorBoundary component in frontend/src/components/ErrorBoundary.jsx
- [ ] T029 [P] Create Auth page in frontend/src/pages/Auth.jsx with login/register forms

### Testing Foundation

- [ ] T030 [P] Write unit tests for User model validation in backend/tests/unit/User.test.js
- [ ] T031 [P] Write unit tests for Task model validation in backend/tests/unit/Task.test.js
- [ ] T032 Write integration tests for auth endpoints in backend/tests/integration/auth.test.js (register, login success/failure)

---

## Phase 3: User Story 1 - Task Capture (Priority P1)

**User Story**: A user needs to quickly record a task so they don't forget it. They open the application, type "Buy groceries", and submit. The task immediately appears in their list and will be there when they return later.

**Independent Test**: Create task → appears in list → reload → task persists

**Why P1**: Without task capture, the application has no value. This is the absolute core functionality.

### Backend Implementation

- [ ] T033 [US1] Create TaskService in backend/src/services/TaskService.js with createTask method (validates description, sets userId, saves to DB)
- [ ] T034 [US1] Create task routes in backend/src/routes/tasks.js with POST /tasks endpoint using auth middleware
- [ ] T035 [US1] Add GET /tasks endpoint in backend/src/routes/tasks.js to retrieve all tasks for authenticated user
- [ ] T036 [US1] Wire task routes into backend/src/server.js

### Frontend Implementation

- [ ] T037 [P] [US1] Create TaskInput component in frontend/src/components/TaskInput.jsx with form, validation, and submit handler
- [ ] T038 [P] [US1] Create TaskList component in frontend/src/components/TaskList.jsx to display array of tasks
- [ ] T039 [US1] Create task store in frontend/src/store/taskStore.js with tasks array, createTask, fetchTasks methods
- [ ] T040 [US1] Create TaskListPage in frontend/src/pages/TaskListPage.jsx integrating TaskInput and TaskList
- [ ] T041 [US1] Add task API methods in frontend/src/services/api.js (createTask, getTasks)

### Testing

- [ ] T042 [P] [US1] Write unit tests for TaskService.createTask in backend/tests/unit/TaskService.test.js (valid input, empty description, description too long)
- [ ] T043 [P] [US1] Write integration test for POST /tasks in backend/tests/integration/tasks.test.js (201 created, 400 validation error, 401 unauthorized)
- [ ] T044 [P] [US1] Write integration test for GET /tasks in backend/tests/integration/tasks.test.js (200 with tasks, empty array when no tasks)
- [ ] T045 [P] [US1] Write component test for TaskInput in frontend/tests/unit/TaskInput.test.jsx (submit calls createTask, validates empty input)
- [ ] T046 [US1] Write E2E test in frontend/tests/e2e/task-capture.spec.js (create task → appears in list → reload → task persists)

---

## Phase 4: User Story 2 - Task Completion (Priority P1)

**User Story**: A user has completed a task and wants to mark it as done. They see their task "Buy groceries" in the list, click to complete it, and it is visually marked as finished. They can still see it in the list but know it's done.

**Independent Test**: Create task → mark complete → visual change → reload → status persists

**Why P1**: Completing tasks is the primary goal of task management. Without this, users can't track progress.

### Backend Implementation

- [ ] T047 [US2] Add updateTask method to TaskService in backend/src/services/TaskService.js (validates ownership, updates isComplete/completedAt)
- [ ] T048 [US2] Add PUT /tasks/:id endpoint in backend/src/routes/tasks.js with optimistic locking via updatedAt comparison
- [ ] T049 [US2] Implement toggle completion logic in TaskService.updateTask (set/unset completedAt based on isComplete)

### Frontend Implementation

- [ ] T050 [P] [US2] Create TaskItem component in frontend/src/components/TaskItem.jsx with checkbox, description display, and toggle handler
- [ ] T051 [US2] Add updateTask method to task store in frontend/src/store/taskStore.js with optimistic update
- [ ] T052 [US2] Update TaskList component to use TaskItem and display completed tasks with visual distinction (strikethrough/checkmark)
- [ ] T053 [US2] Add updateTask API method in frontend/src/services/api.js

### Testing

- [ ] T054 [P] [US2] Write unit tests for TaskService.updateTask in backend/tests/unit/TaskService.test.js (toggle complete/incomplete, ownership check, conflict detection)
- [ ] T055 [P] [US2] Write integration test for PUT /tasks/:id in backend/tests/integration/tasks.test.js (200 updated, 403 forbidden, 409 conflict)
- [ ] T056 [P] [US2] Write component test for TaskItem in frontend/tests/unit/TaskItem.test.jsx (checkbox toggles, strikethrough appears)
- [ ] T057 [US2] Write E2E test in frontend/tests/e2e/task-completion.spec.js (mark complete → visual change → reload → status persists)

---

## Phase 5: User Story 3 - Task Viewing and Organization (Priority P2)

**User Story**: A user opens the application to review what they need to do. They see all their tasks in one view, with incomplete tasks clearly separated from completed ones. They can see what's most important and decide what to work on next.

**Independent Test**: Create multiple tasks → verify display → verify visual distinction → verify ordering

**Why P2**: Users need to understand their task landscape. View must be clear and scannable.

### Backend Implementation

- [ ] T058 [P] [US3] Add query parameter support to GET /tasks in backend/src/routes/tasks.js (?completed=true/false filter)
- [ ] T059 [P] [US3] Add pagination support to GET /tasks (limit, cursor parameters) per research.md
- [ ] T060 [P] [US3] Update TaskService to support filtering by completion status and pagination

### Frontend Implementation

- [ ] T061 [P] [US3] Create EmptyState component in frontend/src/components/EmptyState.jsx for when no tasks exist
- [ ] T062 [US3] Add empty state handling to TaskListPage when tasks array is empty
- [ ] T063 [P] [US3] Implement manual reordering in TaskList using drag-and-drop or up/down buttons
- [ ] T064 [US3] Add reorderTask method to task store in frontend/src/store/taskStore.js
- [ ] T065 [P] [US3] Implement virtual scrolling in TaskList using react-window for 1000+ tasks
- [ ] T066 [US3] Add completed task filter toggle to TaskListPage

### Testing

- [ ] T067 [P] [US3] Write integration test for GET /tasks with filters in backend/tests/integration/tasks.test.js (completed=true returns only completed)
- [ ] T068 [P] [US3] Write component test for EmptyState in frontend/tests/unit/EmptyState.test.jsx
- [ ] T069 [US3] Write E2E test in frontend/tests/e2e/task-viewing.spec.js (empty state → create tasks → verify order → verify filters)

---

## Phase 6: User Story 4 - Task Editing (Priority P2)

**User Story**: A user realizes they need to update a task's description. They select "Buy groceries", edit it to "Buy groceries and milk", and save. The change is immediately visible and persists.

**Independent Test**: Create task → edit description → verify change → reload → verify persistence

**Why P2**: Requirements change. Without editing, users must delete and recreate tasks.

### Backend Implementation

- [ ] T070 [US4] Ensure TaskService.updateTask supports description updates (already implemented in US2, verify coverage)
- [ ] T071 [US4] Add description validation to updateTask (1-500 chars, non-empty)

### Frontend Implementation

- [ ] T072 [P] [US4] Add edit mode to TaskItem component with inline text input and save/cancel buttons
- [ ] T073 [US4] Add updateTaskDescription method to task store
- [ ] T074 [US4] Implement optimistic update for description changes in task store

### Testing

- [ ] T075 [P] [US4] Write unit test for description validation in backend/tests/unit/TaskService.test.js
- [ ] T076 [P] [US4] Write component test for TaskItem edit mode in frontend/tests/unit/TaskItem.test.jsx (enter edit → change text → save)
- [ ] T077 [US4] Write E2E test in frontend/tests/e2e/task-editing.spec.js (edit description → immediate update → reload → persists)

---

## Phase 7: User Story 5 - Task Deletion (Priority P3)

**User Story**: A user wants to remove an obsolete task. They select "Buy groceries" (which is now irrelevant), confirm deletion, and the task disappears permanently from their list.

**Independent Test**: Create task → delete with confirmation → verify removal → reload → confirm deleted

**Why P3**: Task cleanup is useful but not critical. Users can tolerate some clutter initially.

### Backend Implementation

- [ ] T078 [US5] Add deleteTask method to TaskService in backend/src/services/TaskService.js (verify ownership, permanent delete)
- [ ] T079 [US5] Add DELETE /tasks/:id endpoint in backend/src/routes/tasks.js

### Frontend Implementation

- [ ] T080 [P] [US5] Add delete button to TaskItem component
- [ ] T081 [P] [US5] Create ConfirmDialog component in frontend/src/components/ConfirmDialog.jsx for delete confirmation
- [ ] T082 [US5] Add deleteTask method to task store with optimistic removal
- [ ] T083 [US5] Wire delete button in TaskItem to show ConfirmDialog and call deleteTask
- [ ] T084 [US5] Add deleteTask API method in frontend/src/services/api.js

### Testing

- [ ] T085 [P] [US5] Write unit test for TaskService.deleteTask in backend/tests/unit/TaskService.test.js (successful delete, ownership check)
- [ ] T086 [P] [US5] Write integration test for DELETE /tasks/:id in backend/tests/integration/tasks.test.js (204 no content, 403 forbidden)
- [ ] T087 [P] [US5] Write component test for ConfirmDialog in frontend/tests/unit/ConfirmDialog.test.jsx (confirm calls callback, cancel dismisses)
- [ ] T088 [US5] Write E2E test in frontend/tests/e2e/task-deletion.spec.js (delete with confirm → removed, cancel → remains)

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Error handling, loading states, security, edge cases

**Goal**: Production-ready application meeting all acceptance criteria

### Error Handling & UX

- [ ] T089 [P] Create LoadingSpinner component in frontend/src/components/LoadingSpinner.jsx
- [ ] T090 [P] Create ErrorMessage component in frontend/src/components/ErrorMessage.jsx
- [ ] T091 Add loading states to task store (isLoading flag)
- [ ] T092 Add error states to task store (error message)
- [ ] T093 Display LoadingSpinner in TaskListPage when isLoading is true
- [ ] T094 Display ErrorMessage in TaskListPage when error exists
- [ ] T095 Add retry logic to API client in frontend/src/services/api.js for network failures

### Security & Validation

- [ ] T096 [P] Add input sanitization to TaskService (strip HTML tags from description)
- [ ] T097 [P] Add rate limiting middleware to backend/src/middleware/rateLimit.js (5 req/min for auth, 100 req/min for tasks)
- [ ] T098 Add CORS configuration to backend/src/server.js per research.md
- [ ] T099 Add request validation to all task endpoints using express-validator
- [ ] T100 Add authorization checks to all task endpoints (verify userId matches token)

### Edge Cases

- [ ] T101 [P] Handle concurrent session conflicts in frontend (show conflict dialog when 409 returned)
- [ ] T102 [P] Handle session expiry in frontend (redirect to login on 401, preserve unsaved work)
- [ ] T103 Add character counter to TaskInput showing 500 char limit
- [ ] T104 Handle network errors gracefully (show retry button, queue offline changes)
- [ ] T105 Add warning when task count exceeds 10,000 per research.md

### Performance

- [ ] T106 [P] Implement debouncing on task reorder operations (300ms delay)
- [ ] T107 [P] Add cursor-based pagination to backend GET /tasks endpoint
- [ ] T108 Optimize React re-renders in TaskList (React.memo, useMemo)
- [ ] T109 Add indexes verification script for MongoDB collections

### Final Testing & Documentation

- [ ] T110 [P] Write E2E test for full user journey (register → login → create → complete → edit → delete)
- [ ] T111 [P] Verify test coverage >80% for backend services
- [ ] T112 [P] Verify test coverage >80% for frontend components
- [ ] T113 Run API contract tests from contracts/api-tests.http against running backend
- [ ] T114 Update README.md with setup instructions, deployment guide, and architecture overview
- [ ] T115 Create backend/.env.example and frontend/.env.example with all variables documented

---

## Dependencies & Execution Strategy

### Critical Path (Must Complete in Order)

1. **Phase 1** (Setup) → **Phase 2** (Foundational) → **Phase 3** (US1: Task Capture)
2. **Phase 3** (US1) is required before **Phase 4** (US2: Task Completion)
3. **Phase 3** (US1) is required before **Phase 5** (US3: Task Viewing)

### Parallel Opportunities

**After Phase 2 completes**, these can run in parallel:
- User Story 2 (Task Completion) - T047-T057
- User Story 3 (Task Viewing) - T058-T069
- User Story 4 (Task Editing) - T070-T077
- User Story 5 (Task Deletion) - T078-T088

**Within each phase**, tasks marked `[P]` can run in parallel.

### Independent User Story Delivery

Each User Story phase (3-7) can be delivered independently as a working increment:
- **US1 MVP**: Users can create and view tasks (29 tasks: T001-T046)
- **US1+US2**: Add task completion (11 tasks: T047-T057)
- **US1+US2+US3**: Add viewing/organization (12 tasks: T058-T069)
- **US1+US2+US3+US4**: Add editing (7 tasks: T070-T077)
- **Full Feature**: Add deletion and polish (37 tasks: T078-T115)

---

## Task Summary

**Total Tasks**: 115

**By Phase**:
- Phase 1 (Setup): 15 tasks
- Phase 2 (Foundational): 17 tasks
- Phase 3 (US1 - Task Capture): 14 tasks
- Phase 4 (US2 - Task Completion): 11 tasks
- Phase 5 (US3 - Task Viewing): 12 tasks
- Phase 6 (US4 - Task Editing): 7 tasks
- Phase 7 (US5 - Task Deletion): 11 tasks
- Phase 8 (Polish): 27 tasks

**Parallel Tasks**: 52 tasks marked [P] (45% of total)

**Suggested MVP Scope**: Phases 1-3 (46 tasks) delivers core task capture and viewing

---

## Implementation Strategy

### Week 1: Foundation
- Complete Phase 1 (Setup) and Phase 2 (Foundational)
- Deliverable: Working auth system, data models defined

### Week 2: Core Value
- Complete Phase 3 (US1: Task Capture)
- Deliverable: MVP - users can create and view tasks

### Week 3: Essential Features
- Complete Phase 4 (US2: Task Completion) and Phase 5 (US3: Task Viewing)
- Deliverable: Users can complete tasks and organize lists

### Week 4: Enhancement & Polish
- Complete Phase 6 (US4: Task Editing), Phase 7 (US5: Task Deletion), Phase 8 (Polish)
- Deliverable: Production-ready full feature set

**Total Estimated Effort**: 4 weeks for full feature (1 week for MVP)
