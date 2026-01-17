# Workflow Examples

Complete examples demonstrating the dev-workflow skill in action.

## Example 1: Backend API Development

### User Request
```
/dev-workflow Create a REST API for managing a book library with CRUD operations
```

### Stage 1: Requirements Understanding

**openspec output:**
```
Requirements Document:
- Functional Requirements:
  * Create, Read, Update, Delete operations for books
  * Book attributes: title, author, ISBN, publication year, genre
  * RESTful API endpoints
  * Data persistence (database)
  * Input validation

- Non-functional Requirements:
  * Response time < 200ms
  * Support 100 concurrent users
  * Data integrity and consistency

- Constraints:
  * Use existing tech stack (Node.js/Express)
  * PostgreSQL database

- Success Criteria:
  * All CRUD operations functional
  * API documented with OpenAPI spec
  * Unit tests with >80% coverage
```

### Stage 2: Proposal Creation

**openspec output:**
```
Technical Proposal:

Architecture:
- Three-tier architecture: API layer, Business logic, Data access layer

Technology Stack:
- Runtime: Node.js 18+
- Framework: Express.js
- Database: PostgreSQL 14+
- ORM: Prisma
- Testing: Jest + Supertest
- Validation: Joi

API Endpoints:
- POST   /api/books          - Create book
- GET    /api/books          - List all books
- GET    /api/books/:id      - Get book by ID
- PUT    /api/books/:id      - Update book
- DELETE /api/books/:id      - Delete book

Data Model:
Book {
  id: UUID (primary key)
  title: String (required)
  author: String (required)
  isbn: String (unique, required)
  publicationYear: Integer
  genre: String
  createdAt: Timestamp
  updatedAt: Timestamp
}

Implementation Plan:
1. Setup project structure
2. Configure database and Prisma
3. Implement data models
4. Create API routes and controllers
5. Add validation middleware
6. Write unit and integration tests
7. Generate API documentation

Frontend Required: NO
```

### Stage 3: Code Development

**codex execution:**
```
Prompt: "Implement the following technical proposal: [proposal content]"
fullAuto: true
sandbox: "workspace-write"

Actions taken by codex:
1. Created project structure:
   - src/routes/books.js
   - src/controllers/bookController.js
   - src/models/book.js
   - src/middleware/validation.js
   - prisma/schema.prisma

2. Installed dependencies:
   - express, prisma, joi, jest, supertest

3. Implemented all CRUD endpoints
4. Added input validation
5. Configured database connection
6. Created OpenAPI documentation

Session ID: abc123xyz
```

### Stage 4: Testing

**codex execution:**
```
Prompt: "Run all tests for the implemented code and fix any failures"
fullAuto: true
sandbox: "workspace-write"
sessionId: "abc123xyz"

Actions taken by codex:
1. Created test files:
   - tests/books.test.js
   - tests/validation.test.js

2. Ran tests: npm test
3. Fixed 2 validation bugs
4. Re-ran tests: All 24 tests passed
5. Coverage: 87%

Test Results: ✓ All tests passing
```

### Stage 5: Code Review

**codex review output:**
```
Code Review Report:

✓ Code Quality: Good
  - Clean separation of concerns
  - Consistent naming conventions
  - Proper error handling

✓ Security: No critical issues
  - SQL injection prevented (using Prisma ORM)
  - Input validation implemented
  - Recommendation: Add rate limiting

✓ Performance: Good
  - Efficient database queries
  - Proper indexing on ISBN field

✓ Maintainability: Good
  - Well-structured code
  - Adequate test coverage (87%)
  - API documented

Minor Recommendations:
1. Add rate limiting middleware
2. Implement pagination for GET /api/books
3. Add logging for debugging

Overall Assessment: Production-ready with minor improvements suggested
```

---

## Example 2: Frontend Page Development

### User Request
```
/dev-workflow Create a task management dashboard with drag-and-drop functionality
```

### Stage 1: Requirements Understanding

**openspec output:**
```
Requirements Document:
- Functional Requirements:
  * Display tasks in columns (To Do, In Progress, Done)
  * Drag-and-drop tasks between columns
  * Add new tasks
  * Edit task details
  * Delete tasks
  * Filter tasks by priority

- Non-functional Requirements:
  * Responsive design (mobile + desktop)
  * Smooth animations
  * Accessible (WCAG 2.1 AA)

- Success Criteria:
  * Drag-and-drop works smoothly
  * UI is intuitive and visually appealing
  * Works on Chrome, Firefox, Safari
```

### Stage 2: Proposal Creation

**openspec output:**
```
Technical Proposal:

Architecture:
- Single-page application (SPA)
- Component-based architecture

Technology Stack:
- Framework: React 18
- Drag-and-drop: react-beautiful-dnd
- State management: React Context API
- Styling: Tailwind CSS
- Icons: Lucide React

Component Structure:
- App (root)
  - Dashboard
    - Column (To Do, In Progress, Done)
      - TaskCard
    - AddTaskModal
    - FilterBar

Data Model:
Task {
  id: string
  title: string
  description: string
  priority: 'low' | 'medium' | 'high'
  status: 'todo' | 'inprogress' | 'done'
  createdAt: Date
}

Implementation Plan:
1. Setup React project with Vite
2. Install dependencies
3. Create component structure
4. Implement drag-and-drop logic
5. Add task CRUD operations
6. Implement filtering
7. Style with Tailwind CSS
8. Add animations and polish

Frontend Required: YES
```

### Stage 3: Code Development

**ui-ux-pro-max execution:**
```
Prompt: "Implement frontend based on proposal: [frontend specs]"

Actions taken by ui-ux-pro-max:
1. Created React project with Vite
2. Installed dependencies:
   - react-beautiful-dnd
   - tailwindcss
   - lucide-react

3. Implemented components:
   - Dashboard.jsx (main layout)
   - Column.jsx (task columns)
   - TaskCard.jsx (individual tasks)
   - AddTaskModal.jsx (task creation)
   - FilterBar.jsx (filtering UI)

4. Implemented drag-and-drop logic
5. Added state management with Context API
6. Styled with Tailwind CSS
7. Added smooth animations
8. Ensured responsive design

Output: Fully functional task dashboard
```

**codex execution (no backend needed):**
```
Skipped - no backend implementation required
```

### Stage 4: Testing

**codex execution:**
```
Prompt: "Run all tests for the implemented code and fix any failures"
fullAuto: true
sandbox: "workspace-write"

Actions taken by codex:
1. Created test files with React Testing Library:
   - Dashboard.test.jsx
   - TaskCard.test.jsx
   - AddTaskModal.test.jsx

2. Ran tests: npm test
3. All 18 tests passed
4. Coverage: 82%

Test Results: ✓ All tests passing
```

### Stage 5: Code Review

**codex review output:**
```
Code Review Report:

✓ Code Quality: Excellent
  - Clean component structure
  - Proper React hooks usage
  - Good separation of concerns

✓ Accessibility: Good
  - Semantic HTML
  - ARIA labels present
  - Keyboard navigation works
  - Recommendation: Add focus indicators

✓ Performance: Good
  - Proper use of React.memo
  - No unnecessary re-renders
  - Efficient drag-and-drop

✓ UX: Excellent
  - Smooth animations
  - Intuitive interface
  - Responsive design works well

Minor Recommendations:
1. Add loading states for async operations
2. Improve focus indicators for accessibility
3. Add error boundaries

Overall Assessment: Production-ready with excellent UX
```

---

## Example 3: Full-Stack Development

### User Request
```
/dev-workflow Build a blog system with user authentication and post management
```

### Stage 1: Requirements Understanding

**openspec output:**
```
Requirements Document:
- Functional Requirements:
  * User registration and login
  * Create, edit, delete blog posts
  * View all posts (public)
  * View own posts (authenticated)
  * Rich text editor for posts
  * Image upload for posts

- Non-functional Requirements:
  * Secure authentication (JWT)
  * Fast page loads (<2s)
  * SEO-friendly

- Success Criteria:
  * Users can register and login
  * Authenticated users can manage posts
  * Public can view all posts
  * System is secure
```

### Stage 2: Proposal Creation

**openspec output:**
```
Technical Proposal:

Architecture:
- Full-stack application
- RESTful API backend
- React SPA frontend
- JWT-based authentication

Technology Stack:
Backend:
- Runtime: Node.js 18+
- Framework: Express.js
- Database: PostgreSQL + Prisma
- Auth: JWT + bcrypt
- File storage: AWS S3 (or local for dev)

Frontend:
- Framework: React 18
- Routing: React Router
- State: React Query + Context
- Editor: TipTap (rich text)
- Styling: Tailwind CSS

API Endpoints:
Auth:
- POST /api/auth/register
- POST /api/auth/login
- POST /api/auth/logout

Posts:
- GET    /api/posts          (public)
- GET    /api/posts/:id      (public)
- POST   /api/posts          (authenticated)
- PUT    /api/posts/:id      (authenticated, owner only)
- DELETE /api/posts/:id      (authenticated, owner only)
- POST   /api/posts/:id/image (authenticated)

Data Models:
User {
  id: UUID
  email: String (unique)
  passwordHash: String
  name: String
  createdAt: Timestamp
}

Post {
  id: UUID
  title: String
  content: Text
  imageUrl: String (optional)
  authorId: UUID (foreign key)
  published: Boolean
  createdAt: Timestamp
  updatedAt: Timestamp
}

Frontend Required: YES
```

### Stage 3: Code Development

**ui-ux-pro-max execution:**
```
Prompt: "Implement frontend based on proposal: [frontend specs]"

Actions taken:
1. Created React app with Vite
2. Implemented pages:
   - Home (post list)
   - Login/Register
   - Post detail
   - Create/Edit post (with TipTap editor)
   - User dashboard

3. Implemented authentication flow
4. Added protected routes
5. Integrated with backend API
6. Styled with Tailwind CSS

Session ID: frontend-xyz
```

**codex execution:**
```
Prompt: "Implement backend API based on proposal: [backend specs]"
fullAuto: true
sandbox: "workspace-write"

Actions taken:
1. Setup Express server
2. Configured PostgreSQL + Prisma
3. Implemented auth endpoints with JWT
4. Implemented post CRUD endpoints
5. Added authentication middleware
6. Added authorization checks
7. Configured file upload
8. Added input validation

Session ID: backend-abc
```

**codex integration:**
```
Prompt: "Integrate frontend with backend API endpoints"
fullAuto: true
sandbox: "workspace-write"
sessionId: "backend-abc"

Actions taken:
1. Configured CORS
2. Updated API base URLs
3. Added API client in frontend
4. Tested all endpoints
5. Fixed authentication flow issues

Session ID: backend-abc (continued)
```

### Stage 4: Testing

**codex execution:**
```
Prompt: "Run all tests for the implemented code and fix any failures"
fullAuto: true
sandbox: "workspace-write"
sessionId: "backend-abc"

Actions taken:
1. Created backend tests:
   - Auth endpoints tests
   - Post CRUD tests
   - Authorization tests

2. Created frontend tests:
   - Component tests
   - Integration tests
   - Auth flow tests

3. Ran all tests
4. Fixed 3 bugs:
   - JWT expiration handling
   - Image upload validation
   - Post ownership check

5. Re-ran tests: All 42 tests passed
6. Coverage: Backend 89%, Frontend 78%

Test Results: ✓ All tests passing
```

### Stage 5: Code Review

**codex review output:**
```
Code Review Report:

✓ Code Quality: Good
  - Clean architecture
  - Consistent patterns
  - Good error handling

✓ Security: Good with recommendations
  - JWT implementation correct
  - Password hashing with bcrypt
  - SQL injection prevented
  - XSS protection in place
  - Recommendations:
    * Add rate limiting on auth endpoints
    * Implement refresh tokens
    * Add CSRF protection

✓ Performance: Good
  - Efficient queries
  - Proper indexing
  - Frontend code splitting
  - Recommendation: Add caching layer

✓ Maintainability: Good
  - Well-structured code
  - Good test coverage
  - API documented

Critical Recommendations:
1. Add rate limiting to prevent brute force attacks
2. Implement refresh token mechanism
3. Add CSRF protection for state-changing operations

Minor Recommendations:
1. Add Redis caching for frequently accessed posts
2. Implement pagination for post list
3. Add logging and monitoring

Overall Assessment: Production-ready after implementing critical security recommendations
```

---

## Key Patterns

### Pattern 1: Backend-Only Projects
- Stage 2 marks frontend=NO
- Stage 3 uses only codex
- Faster execution, simpler workflow

### Pattern 2: Frontend-Only Projects
- Stage 2 marks frontend=YES, backend=NO
- Stage 3 uses only ui-ux-pro-max
- Focus on UX and visual design

### Pattern 3: Full-Stack Projects
- Stage 2 marks frontend=YES
- Stage 3 uses both ui-ux-pro-max and codex
- Requires integration step
- More complex but complete solution

### Pattern 4: Microservices
- Break into multiple workflow executions
- Each service gets its own workflow
- Coordinate integration separately

## Tips for Success

1. **Be specific in initial request** - More detail = better requirements
2. **Trust the workflow** - Let each stage complete fully
3. **Review proposals carefully** - Catch issues early
4. **Let codex iterate on tests** - It will fix failures automatically
5. **Act on review findings** - Especially security issues
