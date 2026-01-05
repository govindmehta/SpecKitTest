# ToDo App Constitution

## Core Principles

### I. Library-First Architecture
Every feature starts as a standalone, independently testable module. Services must be self-contained with clear responsibilities. No mixed concerns—business logic, data persistence, and UI are separate.

### II. Test-First Development
Tests are written before implementation (TDD mandatory). All critical paths require unit and integration tests. Coverage target: >80% for core business logic.

### III. Simple API Contract
Every service exposes functionality via REST API. Requests/responses use JSON. Errors include status codes and descriptive messages. No breaking changes without version increment.

### IV. Data Persistence Standards
Use a single source of truth for todos. Support atomic operations (create, read, update, delete). Maintain data consistency and prevent concurrent modification conflicts.

### V. User-Centric Design
UI responds immediately; debounce expensive operations. Loading states and error messages are always visible. No silent failures.

## Technology Stack Requirements

- **Backend**: Node.js with Express or equivalent
- **Frontend**: React
- **Database**: SQLite (development) or PostgreSQL (production)
- **Testing**: Jest or equivalent for unit tests; Cypress or Playwright for E2E
- **Version Control**: Git with branch protection on main

## Development Workflow

1. Create feature branch from main
2. Write tests first, get approval
3. Implement feature (tests must pass)
4. Code review by one other developer
5. Merge to main, tag release
6. Deploy to staging, then production

## Governance

This constitution is the single source of truth for development practices. All PRs must comply with these principles. Amendments require documented justification and team approval. Starting complexity must be justified against YAGNI principles.

**Version**: 1.0.0 | **Ratified**: 2026-01-05 | **Last Amended**: 2026-01-05
