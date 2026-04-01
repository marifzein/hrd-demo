# Implementation Plan: HRD Web Application

## Overview

Build a full HRD Web Application based on the [FRD](file:///C:/Users/User/.gemini/antigravity/brain/a6a82fff-d02f-4135-9161-5e77ccd17ef2/frd_hrd_web_app.md) from the previous conversation. The app is a **Go single-binary** with **HTMX** for interactivity, **Tailwind CSS** for styling, and **SQLite** for data storage.

> [!IMPORTANT]
> Given the massive scope of the FRD (10 modules, ~24 weeks estimated), I propose we build this **incrementally** following the FRD's milestone phases. This plan covers **Phase 1 (Foundation & Core)** which includes:
> - Project scaffolding & architecture
> - Module toggle system (MOD)
> - Authentication & RBAC (AUTH)
> - Employee Management (EMP)
> - Dashboard shell (RPT - basic)
>
> Subsequent phases (ATT, LVE, PAY, REC, PER, TRN) will be built iteratively.

---

## User Review Required

> [!IMPORTANT]
> **Technology Choices:**
> - **Router**: Using Go's `net/http` with a lightweight custom router (Chi-style) to keep dependencies minimal
> - **SQLite driver**: `modernc.org/sqlite` (pure Go, no CGO needed — simplifies cross-compilation)
> - **Tailwind CSS**: CDN version (simplest for local deployment)
> - **Session**: Cookie-based with server-side session store in SQLite

> [!WARNING]
> **Scope Decision:** The full FRD has 10 modules. Building everything at once is impractical. I will build Phase 1 (MOD + AUTH + EMP + basic Dashboard) fully functional with the modular toggle system ready for future modules. Approve to proceed with Phase 1 first?

---

## Project Structure

```
C:\Users\User\.gemini\antigravity\scratch\
├── main.go                     # Entry point, server startup
├── go.mod / go.sum
├── config/
│   └── config.go               # App configuration
├── database/
│   ├── database.go             # SQLite connection & migrations
│   ├── migrations.go           # Schema definitions
│   └── seed.go                 # Default seed data (admin user, modules)
├── middleware/
│   ├── auth.go                 # Authentication middleware
│   ├── module.go               # Module toggle middleware
│   ├── csrf.go                 # CSRF protection
│   └── logging.go              # Request/audit logging
├── models/
│   ├── user.go                 # User model & queries
│   ├── employee.go             # Employee model & queries
│   ├── module.go               # Module model & queries
│   ├── session.go              # Session model & queries
│   ├── notification.go         # Notification model & queries
│   └── audit.go                # Audit log model & queries
├── handlers/
│   ├── auth.go                 # Login, logout, password reset
│   ├── dashboard.go            # Dashboard handler
│   ├── employee.go             # Employee CRUD
│   ├── module.go               # Module toggle settings
│   ├── notification.go         # Notification endpoints
│   └── profile.go              # User profile (self-service)
├── templates/
│   ├── layouts/
│   │   ├── base.html           # Base layout (head, scripts, sidebar, footer)
│   │   └── auth.html           # Login/auth layout (no sidebar)
│   ├── partials/
│   │   ├── sidebar.html        # Dynamic sidebar (module-aware)
│   │   ├── header.html         # Top navbar with notifications
│   │   ├── pagination.html     # Reusable pagination
│   │   └── notifications.html  # Notification dropdown
│   ├── auth/
│   │   ├── login.html
│   │   └── reset_password.html
│   ├── dashboard/
│   │   ├── index.html          # Main dashboard
│   │   ├── admin.html          # Admin dashboard widgets
│   │   ├── manager.html        # Manager dashboard widgets
│   │   └── employee.html       # Employee dashboard widgets
│   ├── employees/
│   │   ├── list.html           # Employee list with search/filter
│   │   ├── detail.html         # Employee detail view
│   │   ├── form.html           # Add/Edit employee form
│   │   └── org_chart.html      # Organization chart
│   └── settings/
│       └── modules.html        # Module toggle page
├── static/
│   ├── css/
│   │   └── app.css             # Custom CSS (beyond Tailwind)
│   ├── js/
│   │   └── app.js              # Custom JS (minimal, mostly HTMX)
│   └── img/
│       └── logo.svg            # App logo
└── uploads/                    # File uploads directory
```

---

## Database Schema (Phase 1)

### Core Tables

```sql
-- Module registry
CREATE TABLE modules (
    code TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    icon TEXT,
    enabled INTEGER NOT NULL DEFAULT 1,
    is_core INTEGER NOT NULL DEFAULT 0,
    sort_order INTEGER NOT NULL DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Users (login accounts)
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    role TEXT NOT NULL CHECK(role IN ('super_admin','hr_admin','manager','employee')),
    employee_id INTEGER REFERENCES employees(id),
    is_active INTEGER NOT NULL DEFAULT 1,
    is_locked INTEGER NOT NULL DEFAULT 0,
    locked_until DATETIME,
    failed_login_count INTEGER DEFAULT 0,
    must_change_password INTEGER DEFAULT 0,
    security_question TEXT,
    security_answer_hash TEXT,
    last_login DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Sessions
CREATE TABLE sessions (
    id TEXT PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    data TEXT,
    expires_at DATETIME NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Employees
CREATE TABLE employees (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nik TEXT UNIQUE NOT NULL,           -- Nomor Induk Karyawan
    full_name TEXT NOT NULL,
    ktp_number TEXT UNIQUE,
    npwp TEXT,
    birth_place TEXT,
    birth_date DATE,
    gender TEXT CHECK(gender IN ('M','F')),
    religion TEXT,
    marital_status TEXT,
    address_ktp TEXT,
    address_domicile TEXT,
    phone TEXT,
    email_personal TEXT,
    email_work TEXT,
    photo_path TEXT,
    
    -- Employment data
    department_id INTEGER REFERENCES departments(id),
    position TEXT,
    level TEXT,
    employment_status TEXT CHECK(employment_status IN ('tetap','kontrak','probation','magang')),
    join_date DATE,
    contract_end_date DATE,
    manager_id INTEGER REFERENCES employees(id),
    work_location TEXT,
    
    -- Bank & financial
    bank_name TEXT,
    bank_account TEXT,
    bank_account_name TEXT,
    bpjs_tk TEXT,
    bpjs_kes TEXT,
    
    -- Status
    status TEXT NOT NULL DEFAULT 'active' CHECK(status IN ('active','inactive')),
    inactive_reason TEXT,
    inactive_date DATE,
    
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Departments
CREATE TABLE departments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL,
    parent_id INTEGER REFERENCES departments(id),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Emergency contacts
CREATE TABLE emergency_contacts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    employee_id INTEGER NOT NULL REFERENCES employees(id),
    name TEXT NOT NULL,
    relationship TEXT,
    phone TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Employee documents
CREATE TABLE employee_documents (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    employee_id INTEGER NOT NULL REFERENCES employees(id),
    doc_type TEXT NOT NULL,
    file_name TEXT NOT NULL,
    file_path TEXT NOT NULL,
    uploaded_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Audit log
CREATE TABLE audit_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER REFERENCES users(id),
    action TEXT NOT NULL,
    module TEXT,
    entity_type TEXT,
    entity_id TEXT,
    old_value TEXT,
    new_value TEXT,
    ip_address TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Notifications
CREATE TABLE notifications (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL REFERENCES users(id),
    title TEXT NOT NULL,
    message TEXT NOT NULL,
    module TEXT,
    link TEXT,
    is_read INTEGER NOT NULL DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## Proposed Changes

### Foundation & Configuration

#### [NEW] main.go
- HTTP server setup on `:8080`
- Route registration
- Static file serving
- Graceful shutdown

#### [NEW] go.mod
- Dependencies: `modernc.org/sqlite`, `golang.org/x/crypto` (bcrypt)

#### [NEW] config/config.go
- App configuration struct (port, DB path, upload path, session settings)

---

### Database Layer

#### [NEW] database/database.go
- SQLite connection management
- WAL mode for better concurrent reads

#### [NEW] database/migrations.go
- All table creation DDL
- Auto-migration on startup

#### [NEW] database/seed.go
- Seed default modules (MOD, AUTH, EMP, ATT, LVE, PAY, REC, PER, TRN, RPT)
- Seed Super Admin account (admin/admin123 — forced password change on first login)
- Seed sample departments

---

### Middleware

#### [NEW] middleware/auth.go
- Session validation
- Role-based access control
- Login redirect for unauthenticated requests

#### [NEW] middleware/module.go
- Check if target module is enabled
- Return 404 or redirect if disabled
- In-memory cache with invalidation

#### [NEW] middleware/csrf.go
- CSRF token generation & validation per form

#### [NEW] middleware/logging.go
- Request logging
- Audit trail for CRUD operations

---

### Models (Data Access Layer)

#### [NEW] models/user.go
- CRUD operations for users
- Password verification (bcrypt)
- Login attempt tracking & account locking

#### [NEW] models/employee.go
- Full CRUD for employee data
- Search & filter with pagination
- NIK auto-generation (format: EMP-YYYY-XXXX)

#### [NEW] models/module.go
- Module status queries
- Toggle enable/disable
- Cached module registry

#### [NEW] models/session.go
- Session CRUD
- Cleanup expired sessions

#### [NEW] models/notification.go
- Create/read/mark-as-read notifications
- Unread count

#### [NEW] models/audit.go
- Insert audit log entries

---

### Handlers (HTTP Handlers)

#### [NEW] handlers/auth.go
- `GET /login` — Login page
- `POST /login` — Process login
- `POST /logout` — Logout
- `GET /reset-password` — Reset password page
- `POST /reset-password` — Process reset

#### [NEW] handlers/dashboard.go
- `GET /dashboard` — Role-based dashboard
- Widget data aggregation

#### [NEW] handlers/employee.go
- `GET /employees` — List with HTMX search/filter
- `GET /employees/new` — Add form
- `POST /employees` — Create
- `GET /employees/:id` — Detail view
- `GET /employees/:id/edit` — Edit form
- `PUT /employees/:id` — Update
- `POST /employees/:id/deactivate` — Deactivate/offboard

#### [NEW] handlers/module.go
- `GET /settings/modules` — Module toggle page
- `PUT /settings/modules/:code` — Toggle module on/off

#### [NEW] handlers/notification.go
- `GET /notifications` — Notification list (HTMX partial)
- `PUT /notifications/:id/read` — Mark as read

#### [NEW] handlers/profile.go
- `GET /profile` — Current user profile
- `PUT /profile` — Update profile (self-service fields)

---

### Templates (UI)

#### [NEW] templates/layouts/base.html
- Full page layout with sidebar, header, main content area
- Tailwind CSS CDN
- HTMX CDN
- Dynamic sidebar based on enabled modules & user role

#### [NEW] templates/layouts/auth.html
- Clean login layout without sidebar

#### All template partials and page templates
- HTMX-powered partial swap for SPA-like experience
- Responsive design with Tailwind
- Modern, clean UI following FRD UI/UX guidelines

---

## Open Questions

> [!IMPORTANT]
> 1. **Bahasa UI**: Apakah UI menggunakan **Bahasa Indonesia** atau **English**? FRD ditulis dalam Bahasa Indonesia, saya asumsikan UI juga Bahasa Indonesia.
> 2. **Phase 1 Scope**: Apakah setuju kita mulai dengan **Phase 1 (MOD + AUTH + EMP + Dashboard shell)** dulu, lalu lanjut phase berikutnya setelah Phase 1 selesai?
> 3. **Sample Data**: Apakah perlu saya buatkan sample data (dummy employees, departments) untuk demo?

---

## Verification Plan

### Automated Tests
- Build & run: `go build && ./hrd-app.exe`
- Access `http://localhost:8080` in browser
- Test login with default admin credentials
- Test CSRF protection
- Test module toggle functionality
- Test employee CRUD operations

### Browser Testing
- Login flow (valid & invalid credentials)
- Dashboard rendering per-role
- Employee list with search/sort/paginate
- Module toggle ON/OFF with sidebar update
- Responsive layout check
- HTMX partial page swaps

### Manual Verification
- Database migration runs successfully
- Session management (login, idle timeout, logout)
- Audit log entries created for CRUD operations
- File upload for employee documents & photos
