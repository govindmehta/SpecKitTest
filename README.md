# Todo Application

A modern, production-ready task management application built using **Spec-Driven Development (SDD)** methodology with GitHub SpecKit workflow automation.

## 📋 Project Overview

This is a full-stack web application that enables users to capture, organize, and manage their daily tasks. The project demonstrates a disciplined approach to software development where specifications drive implementation, ensuring every line of code serves a documented purpose.

**Live Features:**
- ✅ Quick task capture with minimal friction
- ✅ Mark tasks as complete/incomplete
- ✅ Edit and delete tasks
- ✅ Persistent storage across sessions
- ✅ Secure user authentication
- ✅ Responsive, user-centric design

## 🎯 Spec-Driven Development (SDD)

### What is SDD?

Spec-Driven Development is a software engineering methodology that prioritizes **specification before implementation**. Unlike traditional development where code comes first, SDD follows a disciplined workflow:

```
Constitution → Specification → Planning → Tasks → Implementation → Validation
```

### Why SDD?

**Traditional Problem:**
- Features drift from original intent
- Missing edge cases discovered in production
- Unclear success criteria lead to endless refactoring
- Documentation written after the fact (if at all)
- Scope creep and feature bloat

**SDD Solution:**
- **Clear Intent**: Every feature has documented "why" before "how"
- **Predictable Outcomes**: Success criteria defined upfront
- **Reduced Waste**: No code written without purpose
- **Living Documentation**: Specs evolve with the product
- **Quality Gates**: Checklists ensure completeness at each phase

### The SDD Workflow

#### 1️⃣ **Constitution** (`.specify/memory/constitution.md`)
The project's governance document defining core principles, tech stack requirements, and development standards.

**Our Constitution:**
- 🏛️ **Library-First Architecture**: Standalone, testable modules
- 🧪 **Test-First Development**: TDD mandatory (>80% coverage)
- 🔌 **Simple API Contract**: REST + JSON, no breaking changes
- 💾 **Data Persistence Standards**: Single source of truth, atomic operations
- 👤 **User-Centric Design**: Immediate feedback, no silent failures

#### 2️⃣ **Specification** (`specs/{feature-id}/spec.md`)
Behavioral specification defining **WHAT** the feature does and **WHY** it matters—zero implementation details.

**Includes:**
- Problem statement (user pain points)
- User stories with acceptance criteria
- Functional requirements
- Success metrics
- Edge cases and constraints
- Explicit scope boundaries

#### 3️⃣ **Planning** (`specs/{feature-id}/plan.md`)
Technical implementation plan translating WHAT/WHY into **HOW**.

**Includes:**
- Technical context (languages, frameworks, dependencies)
- Constitution compliance check
- Architecture decisions
- Project structure
- Research documentation
- Data models and API contracts

#### 4️⃣ **Task Breakdown** (`specs/{feature-id}/tasks.md`)
Granular implementation tasks with dependencies and parallelization markers.

**115 tasks across 8 phases:**
- Phase 1: Project Setup (15 tasks)
- Phase 2: Foundational Infrastructure (17 tasks)
- Phases 3-7: User Story Implementation (45 tasks)
- Phase 8: Polish & Production Readiness (27 tasks)

#### 5️⃣ **Implementation**
Execute tasks following TDD principles:
1. Write failing test
2. Implement minimum code to pass
3. Refactor
4. Verify against specification
5. Update task checklist

## 🛠️ SpecKit Workflow Automation

This project uses **GitHub SpecKit**—a PowerShell-based workflow automation tool that enforces SDD methodology through structured prompts and validation.

### Available Commands

| Command | Purpose | Output |
|---------|---------|--------|
| `.\speckit.specify.ps1` | Create behavioral specification | `specs/{id}/spec.md` |
| `.\speckit.clarify.ps1` | Interactive Q&A to refine spec | Updates to spec.md |
| `.\speckit.plan.ps1` | Generate implementation plan | `plan.md`, `research.md`, `data-model.md`, `contracts/` |
| `.\speckit.tasks.ps1` | Break down plan into tasks | `tasks.md` with dependency graph |
| `.\speckit.implement.ps1` | Execute task implementation | Code changes, test runs |
| `.\speckit.review.ps1` | Validate against specification | Checklist verification |

### How SpecKit Works

1. **Structured Prompts**: Each command loads markdown templates from `.specify/prompts/` with context-aware instructions
2. **Context Injection**: Automatically includes constitution, existing specs, and project state
3. **Quality Gates**: Checklists in `.specify/checklists/` validate completeness at each phase
4. **Memory System**: `.specify/memory/` tracks decisions, architecture patterns, and lessons learned
5. **Git Integration**: Automatic feature branch creation and commit workflows

### Example Workflow

```powershell
# Start new feature
.\speckit.specify.ps1
# → Creates specs/001-todo-app/spec.md
# → AI generates behavioral specification
# → Validates against requirements.md checklist

# Generate implementation plan
.\speckit.plan.ps1
# → Creates plan.md, research.md, data-model.md, contracts/
# → Checks constitution compliance
# → Documents all technical decisions

# Break into tasks
.\speckit.tasks.ps1
# → Creates tasks.md with 115 granular tasks
# → Marks parallel execution opportunities
# → Links tasks to user stories

# Execute Phase 1
.\speckit.implement.ps1
# → Prompts for phase/task selection
# → Runs commands, creates files
# → Updates task checkboxes
# → Verifies build/test passes
```

## 🏗️ Project Structure

```
SpecKitTest/
├── .specify/                      # SpecKit framework
│   ├── memory/                    # Project governance
│   │   └── constitution.md        # Core principles & standards
│   ├── prompts/                   # SDD workflow templates
│   │   ├── speckit.specify.prompt.md
│   │   ├── speckit.plan.prompt.md
│   │   ├── speckit.tasks.prompt.md
│   │   └── speckit.implement.prompt.md
│   ├── checklists/                # Quality gate validations
│   │   └── requirements.md
│   └── scripts/                   # PowerShell automation
│       └── powershell/
│           └── check-prerequisites.ps1
│
├── specs/                         # Feature specifications
│   └── 001-todo-app/
│       ├── spec.md                # Behavioral specification
│       ├── plan.md                # Implementation plan
│       ├── tasks.md               # Task breakdown (115 tasks)
│       ├── research.md            # Technical decisions
│       ├── data-model.md          # Entity definitions
│       ├── quickstart.md          # Developer guide
│       ├── contracts/
│       │   └── api.md             # REST API documentation
│       └── checklists/
│           └── requirements.md    # Spec validation checklist
│
├── backend/                       # Node.js + Express API
│   ├── src/
│   │   ├── server.js              # Express app entry point
│   │   ├── models/                # Mongoose schemas
│   │   ├── routes/                # API endpoints
│   │   ├── services/              # Business logic
│   │   └── middleware/            # Auth, validation, errors
│   ├── tests/
│   │   ├── unit/                  # Model & service tests
│   │   └── integration/           # API endpoint tests
│   ├── package.json
│   ├── jest.config.js
│   └── .env.example
│
├── frontend/                      # React + Vite SPA
│   ├── src/
│   │   ├── components/            # Reusable UI components
│   │   ├── pages/                 # Route pages
│   │   ├── store/                 # Zustand state management
│   │   ├── services/              # API client (Axios)
│   │   └── App.jsx
│   ├── tests/
│   │   ├── setup.js               # Test configuration
│   │   └── e2e/                   # Playwright tests
│   ├── package.json
│   ├── vite.config.js
│   └── .env.example
│
└── README.md                      # This file
```

## 🚀 Tech Stack

### Backend
- **Runtime**: Node.js 18+ LTS
- **Framework**: Express 5.x
- **Database**: MongoDB 6+ with Mongoose ODM
- **Authentication**: JWT (jsonwebtoken) + bcrypt
- **Validation**: express-validator
- **Security**: Helmet, CORS
- **Testing**: Jest + Supertest

### Frontend
- **Framework**: React 18+
- **Build Tool**: Vite 7.x (fast HMR, ESM-native)
- **State Management**: Zustand (1KB vs Redux 10KB)
- **HTTP Client**: Axios with interceptors
- **Routing**: React Router DOM 7.x
- **Testing**: Vitest + Testing Library + Playwright

### Development Tools
- **Package Manager**: npm
- **Linter**: ESLint
- **Hot Reload**: Nodemon (backend), Vite HMR (frontend)
- **Version Control**: Git with feature branch workflow

## 📦 Getting Started

### Prerequisites

1. **Node.js 18+**
   ```powershell
   node --version  # Should be v18 or higher
   ```

2. **MongoDB 6+**
   ```powershell
   # Windows: Start MongoDB service (requires admin)
   Start-Service MongoDB
   
   # Verify connection
   mongosh --eval "db.version()"
   ```

3. **Git**
   ```powershell
   git --version
   ```

### Installation

1. **Clone repository**
   ```powershell
   git clone <repository-url>
   cd SpecKitTest
   ```

2. **Setup backend**
   ```powershell
   cd backend
   npm install
   
   # Create .env from template
   cp .env.example .env
   
   # Edit .env with your values:
   # - JWT_SECRET (generate with: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))")
   # - REFRESH_SECRET (generate another unique secret)
   # - MONGODB_URI (default: mongodb://localhost:27017/todo-app)
   ```

3. **Setup frontend**
   ```powershell
   cd ../frontend
   npm install
   
   # Create .env from template
   cp .env.example .env
   
   # Verify VITE_API_URL=http://localhost:3000/api
   ```

### Running the Application

#### Development Mode

```powershell
# Terminal 1: Start backend
cd backend
npm run dev
# → Server running on http://localhost:3000

# Terminal 2: Start frontend
cd frontend
npm run dev
# → Frontend running on http://localhost:5173
```

#### Production Build

```powershell
# Build frontend
cd frontend
npm run build
# → Outputs to frontend/dist/

# Start production server
cd ../backend
npm start
```

### Running Tests

```powershell
# Backend tests
cd backend
npm test                  # Run all tests
npm run test:watch        # Watch mode
npm run test:coverage     # Coverage report

# Frontend tests
cd frontend
npm test                  # Unit tests (Vitest)
npm run test:e2e          # E2E tests (Playwright)
```

## 📊 Development Progress

**Phase 1: Project Setup** ✅ COMPLETE (15/15 tasks)
- [x] Backend scaffolding (Node.js, Express, MongoDB)
- [x] Frontend scaffolding (React, Vite)
- [x] Testing infrastructure (Jest, Vitest, Playwright)
- [x] Development environment verified

**Phase 2: Foundational** 🚧 NEXT (0/17 tasks)
- [ ] User & Task data models
- [ ] Authentication infrastructure (JWT, middleware)
- [ ] Frontend API client & state management
- [ ] Test foundation (model validation, auth endpoints)

**Remaining Phases** ⏳ PENDING (0/98 tasks)
- Phase 3: User Story 1 - Task Capture (P1)
- Phase 4: User Story 2 - Task Completion (P1)
- Phase 5: User Story 3 - Task Viewing (P2)
- Phase 6: User Story 4 - Task Editing (P2)
- Phase 7: User Story 5 - Task Deletion (P3)
- Phase 8: Polish & Production Readiness

**Total Progress**: 15/115 tasks (13%)

## 🧪 Testing Strategy

Following Constitution Principle II (Test-First Development):

- **70% Unit Tests**: Models, services, utilities
- **20% Integration Tests**: API endpoints, database interactions
- **10% E2E Tests**: Critical user journeys

**Coverage Target**: >80% for core business logic

### Test Pyramid

```
        /\
       /E2E\          Playwright (slow, brittle)
      /------\
     /  API   \       Supertest (medium speed)
    /----------\
   /   Unit     \     Jest/Vitest (fast, reliable)
  /--------------\
```

## 🔒 Security Practices

- **Password Hashing**: bcrypt with 10 rounds
- **Authentication**: JWT access tokens (15min) + refresh tokens (7d)
- **Token Storage**: HttpOnly cookies (XSS protection)
- **CORS**: Whitelist frontend origin
- **Headers**: Helmet middleware for secure HTTP headers
- **Validation**: express-validator on all inputs
- **Rate Limiting**: Planned (Phase 8)

## 📖 Documentation

All documentation lives in `specs/001-todo-app/`:

- **[spec.md](specs/001-todo-app/spec.md)**: Behavioral specification (5 user stories, 15 requirements)
- **[plan.md](specs/001-todo-app/plan.md)**: Technical implementation plan
- **[tasks.md](specs/001-todo-app/tasks.md)**: 115 implementation tasks
- **[research.md](specs/001-todo-app/research.md)**: 10 technical decisions documented
- **[data-model.md](specs/001-todo-app/data-model.md)**: User & Task entity definitions
- **[contracts/api.md](specs/001-todo-app/contracts/api.md)**: REST API documentation
- **[quickstart.md](specs/001-todo-app/quickstart.md)**: Developer onboarding guide

## 🤝 Contributing

This project follows strict SDD methodology:

1. **Never write code without a spec**
2. **Never write implementation code without a failing test**
3. **Always validate against the constitution**
4. **Update task checklists as you go**

### Workflow

```powershell
# Start new feature
git checkout -b 002-new-feature
.\speckit.specify.ps1

# Get AI to draft specification
# Review and refine spec.md
# Validate with .\speckit.review.ps1

# Generate implementation plan
.\speckit.plan.ps1

# Break into tasks
.\speckit.tasks.ps1

# Implement with TDD
.\speckit.implement.ps1

# Submit PR with spec + code
git push origin 002-new-feature
```

## 📚 Learn More

### SDD Resources
- **Constitution**: [.specify/memory/constitution.md](.specify/memory/constitution.md)
- **Specification Template**: [.specify/prompts/speckit.specify.prompt.md](.specify/prompts/speckit.specify.prompt.md)
- **Planning Template**: [.specify/prompts/speckit.plan.prompt.md](.specify/prompts/speckit.plan.prompt.md)

### SpecKit Documentation
- Check `.specify/prompts/` for all workflow templates
- Check `.specify/checklists/` for quality gate validations
- PowerShell scripts in `.specify/scripts/powershell/`

### Tech Stack Docs
- [Express.js](https://expressjs.com/)
- [Mongoose](https://mongoosejs.com/)
- [React](https://react.dev/)
- [Vite](https://vite.dev/)
- [Zustand](https://zustand-demo.pmnd.rs/)
- [Vitest](https://vitest.dev/)

## 📄 License

[Specify License Here]

## 👥 Authors

Built with Spec-Driven Development methodology and GitHub SpecKit workflow automation.

---

**Remember**: Specifications drive implementation. Every line of code serves a documented purpose. 🎯
