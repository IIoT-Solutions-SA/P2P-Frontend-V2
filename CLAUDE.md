# P2P Sandbox Project Context

## Project Overview

P2P Sandbox is a peer-driven collaboration platform for Saudi Arabia's industrial SMEs, featuring forums, use case submissions, and knowledge sharing capabilities.

**Tech Stack:**
- Frontend: React + TypeScript + Vite + TailwindCSS + shadcn/ui
- Backend: FastAPI (Python) with Async/Await
- Databases: PostgreSQL (async with AsyncPG) + MongoDB (async with Motor)
- Authentication: SuperTokens
- File Storage: AWS S3 / Azure Blob Storage

## Project Structure

```
P2P-V2/
├── docs/                      # Project documentation
│   ├── aadil_docs/           # Backend development guides
│   ├── architecture/         # System architecture docs
│   ├── epics/               # Project epics
│   ├── prd/                 # Product requirement docs
│   └── stories/             # User stories by epic
├── p2p-frontend-app/         # React frontend application
│   ├── src/
│   │   ├── components/      # Reusable UI components
│   │   ├── contexts/        # React contexts (Auth)
│   │   ├── pages/           # Page components
│   │   └── types/           # TypeScript types
│   └── package.json
└── p2p-backend-app/          # FastAPI backend (to be implemented)
    ├── requirements.txt
    └── venv/
```

## Current Development Status

- **Frontend**: Fully implemented with mock data
- **Backend**: Planning phase - async FastAPI implementation pending
- **Branch**: `aadil-backend` (current working branch)

## Backend Development Guidelines

### Async Programming Strategy

All backend code should use async/await patterns for:
- Database operations (AsyncPG for PostgreSQL, Motor for MongoDB)
- File uploads to S3/Azure
- External API calls
- WebSocket connections for real-time features

### API Development Standards

1. **Endpoint Structure**: `/api/v1/{resource}`
2. **Authentication**: All endpoints except auth require SuperTokens session
3. **Response Format**: Consistent JSON structure with proper error handling
4. **Testing**: Write async tests for all endpoints

### Database Schema

**PostgreSQL** (Relational data):
- Users, Organizations, Forum Topics, Messages, Use Case Submissions

**MongoDB** (Document data):
- Forum Posts (with nested replies)
- Use Cases (detailed content with media)
- Activity Logs

### Key Implementation Tasks

1. **Authentication System**
   - SuperTokens integration
   - Organization-based signup
   - Session management

2. **Core Features**
   - Forum system with WebSocket support
   - Use case submission with file uploads
   - User management within organizations
   - Dashboard statistics API

3. **Testing Requirements**
   - Unit tests for all services
   - Integration tests for API endpoints
   - Frontend-backend integration tests
   - Performance tests for async operations

## Application Startup Requirements

### Docker-First Development Environment

The entire application stack runs in Docker containers for consistent development across all environments. Claude MUST:

- **Always use Docker for startup/shutdown**: Reference `/DOCKER.md` for comprehensive Docker setup guide and data persistence solution
- **CRITICAL: Follow the data persistence workflow**: Use backup/restore scripts to prevent data loss
- **Use the unified docker-control.sh script**: Located at project root for all container management  
- **Never use legacy startup methods**: All non-Docker startup methods have been deprecated
- **Verify services are healthy after startup**: Check all containers are running and accessible

### Critical Data Persistence Workflow

**🚨 IMPORTANT**: Docker Desktop on Windows/WSL can lose all data on restart. Follow this workflow:

#### When Starting Work:
```bash
# 1. Start containers
./docker-control.sh start

# 2. Restore data from backup (if exists)
./restore-databases.sh

# 3. CRITICAL: Create SuperTokens users for seeded accounts
docker exec p2p-backend python scripts/seed_supertokens_users.py

# 4. Verify services and user authentication
./docker-control.sh status
```

#### When Stopping Work:
```bash
# 1. ALWAYS backup data first (NEVER SKIP!)
./backup-databases.sh

# 2. Then stop containers
./docker-control.sh stop
```

#### First Time Setup:
```bash
# 1. Start containers
./docker-control.sh start

# 2. Initialize and seed databases
./init-databases.sh

# 3. CRITICAL: Create SuperTokens users for all seeded accounts
docker exec p2p-backend python scripts/seed_supertokens_users.py

# 4. Create initial backup
./backup-databases.sh
```

**Complete Documentation**: `/DOCKER.md` - Contains comprehensive Docker setup, data persistence solution, troubleshooting, and all commands

## Code Writing Requirements

### SuperTokens Documentation

When working with SuperTokens authentication, Claude MUST:
- **Always refer to the official documentation**: https://supertokens.com/docs
- Check the latest SuperTokens best practices and API usage
- Verify compatibility between Core and SDK versions
- Use the recommended patterns for session management and auth flows

### Ref MCP Tool Usage

When writing any code for this project, Claude MUST:

1. **Always Use Ref MCP Tool**: Before writing any code, consult the Ref MCP tool to ensure using the latest and most up-to-date:
   - Language features and syntax
   - Framework versions and best practices
   - Security patterns and recommendations
   - Performance optimizations

2. **Verify Current Practices**: Check for:
   - Latest React 19 patterns and hooks
   - Current FastAPI async patterns
   - Modern TypeScript features
   - Updated library APIs

3. **Implementation Process**:
   - First, query Ref for current best practices
   - Verify version compatibility with project dependencies
   - Implement using the most recent recommended patterns
   - Document any significant updates or changes

### Example Workflow:
1. Before implementing a React component → Check latest React patterns
2. Before writing FastAPI endpoints → Verify current async best practices
3. Before database operations → Confirm latest SQLModel usage patterns

### Semgrep Security Scanning

When writing any code for this project, Claude MUST:

1. **Always Run Security Scans**: After writing any code, use Semgrep via the Bash tool to scan for:
   - Security vulnerabilities (OWASP Top 10)
   - Authentication and authorization issues
   - Injection vulnerabilities (SQL, NoSQL, Command)
   - Cross-site scripting (XSS) risks
   - Insecure data exposure
   - Configuration security issues

2. **Fix Identified Issues**: When Semgrep identifies security issues:
   - Address all high and critical severity findings immediately
   - Document why medium/low severity issues are acceptable if not fixed
   - Re-scan after fixes to ensure issues are resolved

3. **Security Scanning Process**:
   - Write the initial code implementation
   - Run Semgrep scan using: `~/.local/bin/semgrep --config=auto <file_or_directory>`
   - Review and fix any security findings
   - Document any security considerations in comments

### Security Workflow Example:
1. After implementing API endpoints → `~/.local/bin/semgrep --config=auto app/api/`
2. After authentication code → `~/.local/bin/semgrep --config=auto app/auth/`
3. After file handling code → `~/.local/bin/semgrep --config=auto --pattern 'path.join'`
4. After database queries → `~/.local/bin/semgrep --config=auto --pattern 'execute|query'`

## Documentation Guidelines

When working in the `/docs` folder:
- Follow existing markdown structure
- Update architecture docs when making system changes
- Keep user stories aligned with implementation

## Frontend Integration Points

Key areas requiring backend integration:
- `AuthContext.tsx`: Replace mock auth with API calls
- `Dashboard.tsx`: Connect to real statistics API
- `UseCases.tsx`: Load from backend instead of JSON file
- `Forum.tsx`: Implement real-time updates via WebSocket


## Testing Commands

```bash
# Backend tests
./docker-control.sh exec backend python -m pytest

# Frontend tests
./docker-control.sh exec frontend npm test

# Backend tests with coverage
./docker-control.sh exec backend python -m pytest --cov=app

# All services health check
curl http://localhost:8000/health && curl http://localhost:5173
```

## Development Workflow

1. **Start Development Environment**: 
   - `./docker-control.sh start`
   - `./restore-databases.sh` (restore previous data if exists)
2. **Always work on the `aadil-backend` branch**
3. **Test API endpoints with frontend before committing**
4. **Run tests**: Use Docker commands from Testing section above
5. **Update documentation when adding major features**
6. **Follow async best practices for all I/O operations**
7. **Stop cleanly**: 
   - `./backup-databases.sh` (CRITICAL: backup data first)
   - `./docker-control.sh stop`

**See `/DOCKER.md` for complete development workflows and data persistence solution.**

## Version Control Requirements for Claude

When making any changes to the project, Claude MUST follow these version control practices:

### Commit Workflow
1. **Every Change Must Be Committed**: No matter how small, every file modification must be committed
2. **Exhaustive Commit Messages**: Each commit must have a detailed message that explains:
   - **WHAT**: What changes were made
   - **WHY**: Why the changes were necessary
   - **HOW**: How the changes were implemented
3. **Message Format**: Use conventional commit format with detailed description:
   ```
   type: brief summary
   
   WHAT: Detailed description of changes made
   WHY: Reason for making these changes
   HOW: Implementation approach and any important details
   ```

### Commit Message Examples:
```
docs: add comprehensive frontend-backend integration guide

WHAT: Created a detailed guide covering frontend-backend integration including dependencies, 
environment setup, API client configuration, authentication flow, error handling, and testing strategies
WHY: Frontend currently uses mock data and needs proper integration with the FastAPI backend. 
Developers need clear guidance on how to connect the two systems
HOW: Documented required npm packages, environment variables, CORS configuration, SuperTokens setup, 
and provided testing checklists for verifying integration
```

```
feat: implement type alignment strategy between frontend and backend

WHAT: Added comprehensive documentation for handling type differences between TypeScript and Python
WHY: Type mismatches between frontend and backend can cause runtime errors. UUID, date, and enum 
handling needs standardization
HOW: Created mapping tables, transformation strategies, and validation approaches for consistent 
type handling across the stack
```

### Required Git Commands:
After making changes, always execute:
```bash
# Add all changes
git add .

# Commit with clear message
git commit -m "type: descriptive message explaining the change"

# Push to current branch (aadil-backend)
git push origin aadil-backend
```

### Important Rules:
- Never make changes without committing
- Always push to the remote branch after committing
- If making multiple related changes, commit them separately with clear messages
- Each commit should represent one logical change

## Important Notes

- Never store credentials in code
- Use environment variables for configuration
- Follow REST API conventions
- Implement proper CORS for frontend integration
- Use structured logging for debugging

## Resources

### Primary Documentation
- **Docker Setup**: `/DOCKER.md` - Complete containerization guide and startup instructions
- **Main README**: `/README.md` - Developer onboarding and quick start
- **Contributing Guide**: `/CONTRIBUTING.md` - Development workflow and standards

### Planning Documents
- Backend Plan: `/docs/aadil_docs/unified-backend-development-plan.md`
- Implementation Tasks: `/docs/aadil_docs/backend-implementation-tasks.md`
- AWS Deployment Plan: `/docs/aadil_docs/aws-deployment-cicd-plan.md`

### Progress Tracking
- Development Checklist: `/docs/aadil_docs/development-checklist.md`
- Implementation Progress: `/docs/aadil_docs/implementation-progress.md`
- Development Log: `/docs/aadil_docs/development-log.md`

### Documentation Update Requirements
When completing tasks, Claude MUST update the following documents:
1. **Implementation Progress** (`/docs/aadil_docs/implementation-progress.md`): Update completion percentages and mark tasks complete
2. **Development Log** (`/docs/aadil_docs/development-log.md`): Add detailed log entries for work completed
3. **Backend Implementation Tasks** (`/docs/aadil_docs/backend-implementation-tasks.md`): Mark tasks as complete with ✅ and update progress summary

### Development Guides
- Frontend-Backend Integration: `/docs/aadil_docs/frontend-backend-integration-guide.md`
- Type Alignment Guide: `/docs/aadil_docs/type-alignment-guide.md`
- Authentication Flow: `/docs/aadil_docs/authentication-flow-guide.md`
- API Communication: `/docs/aadil_docs/api-communication-guide.md`
- Development Workflow: `/docs/aadil_docs/development-workflow-guide.md`

### Container Management
- **Docker Setup Guide**: `/DOCKER.md` - Primary method for all application management and data persistence solution
- **Complete Shutdown & Restart Guide**: `/docs/aadil_docs/complete-shutdown-restart-guide.md` - Comprehensive workflow for safely shutting down entire development environment (Docker Desktop, WSL, PC restart) while preserving ALL data including dynamic content created during development sessions
- **Data Persistence Documentation**: `/DOCKER_DATA_PERSISTENCE.md` - Detailed explanation of backup/restore solution

### Architecture & Testing
- Architecture: `/docs/architecture/`
- Async Guide: `/docs/aadil_docs/async-implementation-guide.md`
- Testing Plan: `/docs/aadil_docs/comprehensive-testing-plan.md`

## Task Tracking Requirements

### Subtask Documentation Rule
When working on any main task (e.g., P2.AUTH.02), Claude MUST:

1. **Break down main tasks into subtasks**: Every main task should be decomposed into specific, actionable subtasks
2. **Track all subtasks in TodoWrite tool**: Use the TodoWrite tool to track each subtask as it's completed
3. **Document all subtasks in implementation-progress.md**: Under each main task, list ALL completed subtasks with checkmarks and dates
4. **Update progress files**: Always update both implementation-progress.md and development-log.md with comprehensive subtask details

### Required Format in implementation-progress.md:
```markdown
- [x] P2.AUTH.02 - Login Endpoint ✅ 2025-08-05
  - [x] Research SuperTokens signin API and session handling ✅ 2025-08-05
  - [x] Implement login API override with user/org data retrieval ✅ 2025-08-05
  - [x] Add session enhancement with organization context ✅ 2025-08-05
  - [x] Test login flow and session validation ✅ 2025-08-05
  - [x] Create session info endpoint for testing ✅ 2025-08-05
  - [x] Run security scanning on signin implementation ✅ 2025-08-05
```

### Why This Matters:
- Provides complete visibility into implementation approach
- Shows thorough testing and validation
- Demonstrates security considerations
- Helps with future maintenance and debugging
- Creates detailed audit trail of work completed

**This rule applies to ALL future tasks - no exceptions.**

## Developer-Specific Context

For developer-specific guidelines and personal notes, reference:
- Aadil's Backend Context: `/docs/aadil_docs/aadil-backend-context.md`