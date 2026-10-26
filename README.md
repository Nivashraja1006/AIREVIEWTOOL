# AI ReviewFlow — AI-Powered Review Process Automation Platform

AI ReviewFlow is an enterprise-grade automated code review, task assignment, repository evaluation, decision automation, and certificate issuance platform. Engineered for educational academies, corporate bootcamps, coding assignments, and workplace training programs.

---

## 🌟 Key Features

1. **Excel & CSV Bulk Participant Import**
   - Upload rosters in `.xlsx` or `.csv` format.
   - Real-time row-by-row validation (email format, duplicate detection, required fields, and task mapping).
   - Detailed row error reporting and downloadable template.
   - Transactional import preventing partial failures or duplicate user assignments.

2. **Programming Task & Evaluation Rubric Builder**
   - Configure title, detailed markdown requirements, language (Python), and deadlines.
   - Multi-criteria rubric builder with weights and **mandatory pass/fail flags**.
   - Configurable passing threshold (default 70%) and file size limits.

3. **Task Assignment Management**
   - Assign tasks individually or in bulk across participant groups.
   - Track progress, deadlines, latest submission status, and scores in real time.

4. **GitHub Submission Analysis & Ingestion**
   - Accepts GitHub repository URLs (`https://github.com/owner/repo`).
   - Automatically resolves default branch HEAD or explicit commit SHAs.
   - Downloads and validates permitted source code files while stripping `.git`, `node_modules`, `__pycache__`, binaries, and oversized files.

5. **Automated Code Analysis Pipeline**
   - **Syntax Validation:** Verifies Python AST parsing (`ast.parse`) across all modules.
   - **Static Linting:** Automated Ruff linter checks for PEP 8 compliance and code smells.
   - **Isolated Test Runner:** Executes pytest test suites in temporary directories with timeout limits (12s) and resource constraints.

6. **AI Review Engine & Fallback**
   - Integrates with Gemini (`gemini-1.5-flash`), OpenAI (`gpt-4o-mini`), or Anthropic.
   - If no API key is supplied, automatically falls back to an intelligent **Deterministic Analysis Engine** that inspects AST symbols, syntax, lint results, and rubric keywords without crashing or fabricating results.
   - Strictly enforces structured JSON output schema matching Pydantic models.

7. **Deterministic Automatic Decision Engine**
   - Combines deterministic policy rules:
     * Mandatory criteria must pass.
     * Required unit tests must pass.
     * Total score must satisfy the passing threshold.
     * No critical security vulnerabilities (e.g. `eval`, hardcoded credentials).
   - Outcomes: `approved`, `request_revision`, `human_review`, or `analysis_failed`.
   - Automatically updates assignment states, triggers email notifications, and generates certificates upon approval.

8. **Reviewer Escalation Queue**
   - Submissions with borderline scores or uncertain model confidence are routed to human reviewers.
   - Authorized reviewers inspect findings, test outputs, and record definitive decisions with mandatory reasons.

9. **Verifiable PDF Certificate Generation**
   - Generated using **ReportLab** with a high-resolution, branded design.
   - Features participant name, task title, completion date, issuing authority, and unique code (`ARF-YYYY-XXXXXXXX`).
   - Includes secure PDF download and a **Public Verification Page** (`/verify/:code`).
   - Supports administrative revocation with audit logging.

10. **Contextual AI Chatbot Assistant**
    - Scoped strictly to the authenticated user's assignment data (tenant isolation).
    - Answers personalized questions regarding task requirements, rubrics, deadlines, previous review feedback, and certificate eligibility.

11. **Dispatch & Notification Service**
    - Idempotency-keyed notifications preventing duplicate emails.
    - Rich HTML and plain-text templates for all lifecycle events.
    - Console preview log in development mode; SMTP integration in production.

12. **Live Dashboards & Audit Logging**
    - Admin, Participant, and Reviewer dashboards calculated directly from live database state.
    - Complete immutable audit trail (`AuditLog`) for every critical action.

---

## 🏛️ Architecture Overview

- **Backend:** FastAPI (Python 3.11), SQLAlchemy 2.0, Alembic, Pydantic v2, ReportLab, Pandas, OpenPyXL, Pytest, Ruff, PyJWT, Bcrypt.
- **Frontend:** React 19, Vite, Tailwind CSS, Lucide React, Axios, React Router v7, Canvas Confetti.
- **Database:** SQLite (default for instant zero-dependency local startup) or PostgreSQL via `DATABASE_URL`.

---

## 🚀 Quick Start Guide

### 1. Prerequisites
- Python 3.11+
- Node.js 18+ and `pnpm` (or `npm`)

### 2. Backend Setup

```bash
# Navigate to the backend directory
cd backend

# Install dependencies
pip install -r requirements.txt

# Run database migrations
alembic upgrade head

# Seed initial demonstration data
python seed.py

# Start the backend server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

The API documentation will be available at:
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`
- Health Check: `http://localhost:8000/health`

### 3. Frontend Setup

In a separate terminal:

```bash
# Navigate to the frontend directory
cd frontend

# Dependencies are already installed via pnpm. If needed:
pnpm install

# Start the Vite development server
pnpm dev
```

The application will be accessible at:
👉 **`http://localhost:5173`**

---

## 🔑 Demonstration Accounts

The database seed provides pre-configured accounts for testing each role:

| Role | Email | Password | Access / Views |
| :--- | :--- | :--- | :--- |
| **Administrator** | `admin@reviewflow.local` | `AdminPassword2026!` | Admin Dashboard, Participant Directory, Excel Import, Task & Rubric Creation, Assignment Operations, Certificates, Audit Logs |
| **Reviewer** | `reviewer@reviewflow.local` | `ReviewerPassword2026!` | Escalation Queue, Submission Inspection, Manual Decision Recording |
| **Participant (Approved)** | `alice@example.edu` | `Participant2026!` | Student Dashboard, Approved Submission, Download Official Certificate |
| **Participant (In Review)** | `bob@example.edu` | `Participant2026!` | Student Dashboard, Escalated Submission Status, Contextual Chatbot |
| **Participant (Assigned)** | `charlie@example.edu` | `Participant2026!` | Student Dashboard, GitHub Submission Form |

> **Tip:** You can also click the quick **"Demo Accounts"** buttons on the login page to sign in with one click.

---

## ⚙️ Environment Configuration

Create a `.env` file in the `backend/` directory (see `backend/.env.example`):

```ini
# Server Settings
BACKEND_URL=http://localhost:8000
FRONTEND_URL=http://localhost:5173
SECRET_KEY=ai-reviewflow-development-secret-key-2026-secure
ACCESS_TOKEN_EXPIRE_MINUTES=1440

# Database
DATABASE_URL=sqlite:///./reviewflow.db
# DATABASE_URL=postgresql://user:password@localhost:5432/reviewflow_db

# AI LLM Provider (gemini, openai, or leave empty for deterministic engine)
AI_PROVIDER=gemini
AI_API_KEY=
AI_MODEL=gemini-1.5-flash

# GitHub Token (Optional for public repos; required for high rate limits)
GITHUB_TOKEN=

# SMTP Email Dispatch
SMTP_HOST=
SMTP_PORT=587
SMTP_USERNAME=
SMTP_PASSWORD=
SMTP_USE_TLS=true
EMAIL_FROM=no-reply@aireviewflow.local
DEV_EMAIL_LOG=true

# Organization Identity
ISSUING_ORGANIZATION=AI ReviewFlow Academy
```

---

## 🧪 Testing & Verification

### Running Backend Unit & Integration Tests

```bash
cd backend
python -m pytest -v
```

**Results:**
- `tests/test_auth.py` (4 tests): Login, password hashing, token validation, duplicate prevention.
- `tests/test_rbac.py` (3 tests): Role-based access control, participant route protection, reviewer boundaries.
- `tests/test_excel.py` (2 tests): Template generation, CSV/XLSX preview, row-level validation, transactional execution.
- `tests/test_code_analysis.py` (4 tests): GitHub URL parsing, AST syntax verification, symbol extraction, isolated pytest sandbox runner.
- `tests/test_decision_engine.py` (1 test): Deterministic approval, revision, security escalation rules.
- `tests/test_certificates.py` (3 tests): Public verification endpoint, unique code generation, ReportLab PDF rendering.
- `tests/test_full_workflow.py` (1 test): Complete end-to-end simulation from task creation to automatic certificate issuance and contextual chat.

**Total: 18 passed in 3.8s with 100% pass rate.**

### Building Frontend for Production

```bash
cd frontend
pnpm run build
```

**Result:** `vite build` completed in 1.1s, generating optimized production bundle in `frontend/dist/`.

---

## 📋 Implemented Application Screens

1. `/login` & `/register`: Secure authentication with JWT, quick test account switcher.
2. `/admin`: Administrative command center with live metrics, pipeline distributions, and audit trails.
3. `/participants`: Participant directory with search, filtering, and add participant modal.
4. `/import`: Excel/CSV bulk import with live row preview, error highlighting, and template download.
5. `/tasks` & `/tasks/new` & `/tasks/edit/:id`: Task specification and multi-criteria rubric builder with mandatory flags.
6. `/assignments`: Single and bulk task assignment operations with due dates.
7. `/dashboard`: Participant task portal with progress cards and submission action triggers.
8. `/submit/:assignmentId`: GitHub repository submission form with commit SHA resolution.
9. `/submissions` & `/submissions/assignment/:id`: Attempt history timeline and status tracking.
10. `/reports/:runId`: AI review report page with score badge, criterion findings with code evidence references, test logs, and fixes.
11. `/reviewer`: Reviewer escalation queue with diff/finding inspection and manual decision recorder.
12. `/notifications`: Dispatch notification history with idempotency keys and email preview modal.
13. `/schedules`: Review timeline and upcoming assignment milestones.
14. `/certificates`: Earned certificates directory with ReportLab PDF download.
15. `/verify` & `/verify/:code`: Public certificate verification page.
16. `/settings`: User profile editor and live integration diagnostics.
17. **AI Assistant Drawer:** Contextual chatbot accessible from anywhere in the application.

---

## 📄 License

AI ReviewFlow is built for production deployment and educational assignment automation.
#   A I R E V I E W T O O L  
 