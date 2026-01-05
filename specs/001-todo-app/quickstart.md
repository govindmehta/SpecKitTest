# Developer Quickstart Guide

**Feature**: Todo Application  
**Date**: 2026-01-05  
**Target Audience**: Developers implementing this feature

## Overview

This guide provides step-by-step instructions for setting up the development environment and implementing the Todo application following the plan defined in [plan.md](plan.md).

---

## Prerequisites

Before starting, ensure you have:

- **Node.js**: Version 18+ LTS ([Download](https://nodejs.org/))
- **MongoDB**: Version 6+ (local or cloud instance)
  - Local: [MongoDB Community Edition](https://www.mongodb.com/try/download/community)
  - Cloud: [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) free tier
- **Git**: For version control
- **Code Editor**: VS Code recommended (with REST Client extension)

**Verify installations**:
```bash
node --version   # Should be v18+
npm --version    # Should be v9+
mongod --version # Should be v6+
git --version
```

---

## Project Setup

### 1. Clone Repository and Create Branch

```bash
git clone <repository-url>
cd <repository-name>
git checkout 001-todo-app  # Or create from main if not exists
```

### 2. Create Project Structure

```bash
# Create directories
mkdir -p backend/src/{models,services,routes,middleware}
mkdir -p backend/tests/{unit,integration,contract}
mkdir -p frontend/src/{components,pages,services,store}
mkdir -p frontend/tests/{unit,integration,e2e}
mkdir -p shared/contracts
```

---

## Backend Setup

### 1. Initialize Backend

```bash
cd backend
npm init -y
```

### 2. Install Dependencies

```bash
# Core dependencies
npm install express mongoose jsonwebtoken bcrypt dotenv cors

# Validation and security
npm install express-validator helmet

# Development dependencies
npm install --save-dev \
  jest \
  supertest \
  nodemon \
  eslint \
  @types/node
```

### 3. Configure Environment

Create `backend/.env`:
```env
# Server
PORT=3000
NODE_ENV=development

# Database
MONGODB_URI=mongodb://localhost:27017/todo-app

# Authentication
JWT_SECRET=your-secret-key-change-in-production
JWT_EXPIRY=15m
REFRESH_SECRET=your-refresh-secret-change-in-production
REFRESH_EXPIRY=7d

# CORS
CORS_ORIGIN=http://localhost:5173
```

### 4. Create Server Entry Point

Create `backend/src/server.js`:
```javascript
require('dotenv').config();
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const helmet = require('helmet');

const app = express();

// Middleware
app.use(helmet());
app.use(cors({ origin: process.env.CORS_ORIGIN, credentials: true }));
app.use(express.json());

// Routes (to be added)
// app.use('/api/auth', require('./routes/auth'));
// app.use('/api/tasks', require('./routes/tasks'));

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status || 500).json({
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message: err.message || 'An error occurred'
    }
  });
});

// Connect to MongoDB
mongoose.connect(process.env.MONGODB_URI)
  .then(() => console.log('✓ MongoDB connected'))
  .catch(err => console.error('MongoDB connection error:', err));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`✓ Server running on port ${PORT}`);
});
```

### 5. Configure Package Scripts

Update `backend/package.json`:
```json
{
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

### 6. Test Backend

```bash
npm run dev
```

Expected output:
```
✓ MongoDB connected
✓ Server running on port 3000
```

---

## Frontend Setup

### 1. Initialize Frontend with Vite

```bash
cd ../frontend
npm create vite@latest . -- --template react
```

### 2. Install Dependencies

```bash
# Core dependencies
npm install zustand axios react-router-dom

# Development dependencies
npm install --save-dev \
  vitest \
  @testing-library/react \
  @testing-library/jest-dom \
  @playwright/test
```

### 3. Configure Environment

Create `frontend/.env`:
```env
VITE_API_URL=http://localhost:3000/api
```

### 4. Configure Vite for Testing

Create `frontend/vite.config.js`:
```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './tests/setup.js'
  }
});
```

### 5. Configure Package Scripts

Update `frontend/package.json`:
```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:e2e": "playwright test"
  }
}
```

### 6. Test Frontend

```bash
npm run dev
```

Expected output:
```
  VITE v5.x.x  ready in 300 ms
  ➜  Local:   http://localhost:5173/
```

---

## Implementation Workflow

Follow **Test-Driven Development** (TDD) per constitution:

### Phase 1: Backend Models & Services

1. **Define Mongoose Schemas**:
   - Create `backend/src/models/User.js` (see [data-model.md](data-model.md))
   - Create `backend/src/models/Task.js`

2. **Write Service Tests First**:
   ```bash
   # Example test file
   backend/tests/unit/TaskService.test.js
   ```

3. **Implement Services**:
   - `backend/src/services/TaskService.js` (CRUD logic)
   - `backend/src/services/AuthService.js` (JWT, bcrypt)

4. **Run Tests**:
   ```bash
   cd backend
   npm test
   ```

### Phase 2: Backend API Routes

1. **Write API Integration Tests**:
   ```bash
   backend/tests/integration/tasks.test.js
   ```

2. **Implement Routes**:
   - `backend/src/routes/auth.js`
   - `backend/src/routes/tasks.js`
   - `backend/src/middleware/auth.js` (JWT verification)

3. **Test with REST Client**:
   Use [contracts/api-tests.http](contracts/api-tests.http)

### Phase 3: Frontend Components

1. **Create API Client**:
   - `frontend/src/services/api.js` (Axios instance with interceptors)

2. **Build Components** (test-first):
   - `frontend/src/components/TaskItem.jsx`
   - `frontend/src/components/TaskInput.jsx`
   - `frontend/src/components/ErrorBoundary.jsx`

3. **Create State Store**:
   - `frontend/src/store/taskStore.js` (Zustand)
   - `frontend/src/store/authStore.js`

4. **Build Pages**:
   - `frontend/src/pages/TaskList.jsx`
   - `frontend/src/pages/Auth.jsx`

### Phase 4: Integration Testing

1. **Write E2E Tests**:
   ```bash
   frontend/tests/e2e/task-workflow.spec.js
   ```

2. **Run E2E Tests**:
   ```bash
   npm run test:e2e
   ```

---

## Testing Checklist

Use this checklist to verify implementation against acceptance criteria:

### User Story 1: Task Capture (P1)
- [ ] User can create task with description
- [ ] Task appears in list immediately
- [ ] Empty description is rejected with error message
- [ ] Task persists after browser reload

### User Story 2: Task Completion (P1)
- [ ] User can toggle task complete/incomplete
- [ ] Visual change is immediate (checkmark/strikethrough)
- [ ] Completion timestamp is recorded
- [ ] Status persists after reload

### User Story 3: Task Viewing (P2)
- [ ] All tasks display in one view
- [ ] Completed tasks visually distinguished
- [ ] Empty state shows when no tasks
- [ ] Smooth scrolling with 100+ tasks

### User Story 4: Task Editing (P2)
- [ ] User can edit task description
- [ ] Changes appear immediately
- [ ] Empty description update is rejected
- [ ] Edits persist after reload

### User Story 5: Task Deletion (P3)
- [ ] User can delete task with confirmation
- [ ] Cancelled deletion leaves task unchanged
- [ ] Deleted task removed permanently

### Edge Cases
- [ ] Concurrent session updates show conflict warning
- [ ] 1000+ tasks load without lag
- [ ] Session expiry redirects to login
- [ ] Network errors show user-friendly messages

---

## Code Quality Gates

Before committing, verify:

1. **Tests Pass**:
   ```bash
   # Backend
   cd backend && npm test
   
   # Frontend
   cd frontend && npm test
   ```

2. **Coverage > 80%**:
   ```bash
   npm run test:coverage
   ```

3. **No Linting Errors**:
   ```bash
   npm run lint
   ```

4. **Constitution Compliance**:
   - [ ] Services are self-contained modules
   - [ ] All endpoints have JSON contracts
   - [ ] Error messages are descriptive
   - [ ] No mixed concerns (UI/business logic separated)

---

## Deployment Preview

### Local Testing

1. **Start MongoDB**:
   ```bash
   mongod --dbpath /path/to/data
   ```

2. **Start Backend**:
   ```bash
   cd backend
   npm run dev
   ```

3. **Start Frontend**:
   ```bash
   cd frontend
   npm run dev
   ```

4. **Access Application**:
   - Frontend: http://localhost:5173
   - Backend API: http://localhost:3000/api

### Production Deployment

See deployment plan (to be created in Phase 2):
- Backend: Docker container on Render/Railway
- Frontend: Static build on Vercel/Netlify
- Database: MongoDB Atlas

---

## Troubleshooting

### MongoDB Connection Fails

```bash
# Check if MongoDB is running
mongod --version

# Start MongoDB (macOS/Linux)
brew services start mongodb-community

# Start MongoDB (Windows)
net start MongoDB
```

### Port Already in Use

```bash
# Find process using port 3000
lsof -i :3000  # macOS/Linux
netstat -ano | findstr :3000  # Windows

# Kill process or change PORT in .env
```

### CORS Errors

Verify `CORS_ORIGIN` in `backend/.env` matches frontend URL:
```env
CORS_ORIGIN=http://localhost:5173
```

---

## Reference Documents

- [spec.md](spec.md) - Behavioral specification
- [plan.md](plan.md) - Implementation plan
- [research.md](research.md) - Technical decisions
- [data-model.md](data-model.md) - Entity schemas and validation
- [contracts/api.md](contracts/api.md) - API documentation

---

## Next Steps

1. **Review Constitution**: [.specify/memory/constitution.md](../.specify/memory/constitution.md)
2. **Read Data Model**: [data-model.md](data-model.md)
3. **Start Backend**: Follow "Backend Setup" above
4. **Run First Test**: Create User model test
5. **Iterate**: Red → Green → Refactor (TDD cycle)

**Questions?** Refer to [research.md](research.md) for technical decisions and rationale.
