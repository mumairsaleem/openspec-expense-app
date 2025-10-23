# Project Context

## Purpose

**Home Expense & Investment Tracker (HEIT)** is a spec-driven, instrumented web application designed to help young users living in **joint family systems** track shared and personal expenses, budgets, and investments with granular role-based access control.

### Primary Goals
- Track shared and personal expenses, recurring bills, budgets, and investment holdings at **family** and **individual** levels
- Provide **Role-Based Access Control (RBAC)** with one SuperAdmin per tenant (family) and granular permissions for Admins, Members, Guests, and Auditors
- Offer **transparent reporting**: month-over-month (MoM), year-to-date (YTD), category breakdowns, wallets, and investment performance
- Enable **spec-driven development**: stable API contracts, acceptance criteria, and fixtures for repeatable benchmarks across toolchains

### Success Criteria
- Clear feature parity across implementations
- Measured performance, reliability, and DX metrics (API < 200ms p95, page load < 2s LCP)
- Household adoption with secure RBAC and least-privilege defaults
- Support 10,000 families with 100 concurrent users per family

### Non-Goals (V1)
- Automated bank aggregation (Plaid, OpenBanking) — use CSV/Excel/OFX import first
- Tax filing automation
- Complex portfolio analytics (options/derivatives) beyond basic SIP/ETF/stocks

## Tech Stack

### Frontend
- **React** (TypeScript)
  - Mobile-first responsive design
  - Progressive Web App (PWA) capabilities
  - Dark mode support
  - WCAG AA compliant accessibility
  - Virtual scrolling for large transaction lists
  - State management (Context API or Redux/Zustand)
  - Chart library for analytics (e.g., Recharts, Chart.js)

### Backend
- **Node.js** with Express or Fastify
  - TypeScript for type safety
  - REST API (Base URL: `/v1`)
  - JWT-based authentication with refresh token rotation
  - Multi-tenant architecture (family-based tenancy)
  - Middleware for RBAC enforcement
  - File upload handling with virus scanning

### Database
- **PostgreSQL**
  - Row-level security (RLS) for multi-tenancy
  - UUID primary keys
  - JSONB for flexible metadata (e.g., notification_preferences, splits)
  - Indexes on family_id, user_id, date fields
  - Encrypted columns for private_notes (pgcrypto)
  - Soft delete with deleted_at timestamps
  - Audit log table (append-only)
  - Daily backups with point-in-time recovery
  - Support for 10,000+ families, 1M+ transactions per family per year

### Development Tools
- **TypeScript** for both frontend and backend
- **ESLint** and **Prettier** for code formatting
- **Jest** for unit testing
- **Playwright** or **Cypress** for E2E testing
- **Docker** for local development and deployment
- **GitHub Actions** for CI/CD

### Security
- **bcrypt** or **argon2** for password hashing
- **JWT** with short TTL (15 min) + refresh tokens
- **Helmet.js** for security headers
- **CORS** configuration for API
- **Rate limiting** to prevent abuse
- **pg-promise** or **Prisma** ORM with parameterized queries

## Project Conventions

### Code Style
- **No magic strings**: Always use constants to render any text instead of hardcoding it (enforced by project convention)
- **TypeScript strict mode** enabled
- **Naming conventions**:
  - Database: snake_case (e.g., `family_id`, `super_admin_id`, `fiscal_year_start`)
  - API endpoints: kebab-case (e.g., `/api/v1/transaction-splits`)
  - React components: PascalCase (e.g., `TransactionForm.tsx`)
  - Functions/variables: camelCase (e.g., `getUserFamilies()`)
  - Constants: UPPER_SNAKE_CASE (e.g., `MAX_FILE_SIZE`, `API_BASE_URL`)
- **File organization**:
  - Frontend: `components/`, `pages/`, `hooks/`, `services/`, `utils/`, `constants/`, `types/`
  - Backend: `routes/`, `controllers/`, `services/`, `middleware/`, `models/`, `utils/`, `config/`
- **Error handling**: Use centralized error handling middleware; return consistent error format with error codes

### Architecture Patterns

#### Multi-Tenant Architecture
- **Tenant Model**: Each Family is a separate tenant
- Users can belong to multiple families with independent roles
- All database queries filtered by `family_id`
- Row-level security enforced at PostgreSQL level
- Middleware extracts `family_id` from JWT and validates membership

#### RBAC Pattern
- **Roles**: SuperAdmin, Admin, Member, Guest, Auditor
- Per-request membership authorization via middleware
- Permission matrix enforced in both backend (authoritative) and frontend (UX)
- Time-boxed access for Auditor role (expires_at field)
- Role hierarchy: SuperAdmin > Admin > Member > Guest/Auditor

#### API Design
- **RESTful conventions**: Resource-oriented endpoints
- **Versioning**: `/v1` prefix for API stability
- **Pagination**: Cursor-based or offset/limit with defaults
- **Filtering**: Query params for date ranges, categories, members
- **Error responses**: Consistent format with domain-specific codes
  ```json
  {
    "error": {
      "code": "TXN_SPLIT_MISMATCH",
      "message": "Split amounts do not sum to transaction total",
      "details": {}
    }
  }
  ```

#### Data Scoping
- **Family Scope**: Visible to all family members based on permissions
- **Personal Scope**: Visible only to owner and Admin
- Proposal-based workflow for Member edits to shared family transactions
  - Member proposes change → stored in `change_requests` table
  - Admin reviews → accepts/rejects with audit log entry

#### Audit Pattern
- Write-only append audit log in separate table
- Soft delete with 30-day recovery window (`deleted_at` timestamp)
- Before/after snapshots for all CRUD operations (JSONB columns)
- Background job for permanent deletion after retention period

#### Service Layer Pattern
- Controllers handle HTTP concerns (request/response)
- Services contain business logic
- Repositories/Models handle data access
- Clear separation of concerns

### Testing Strategy

#### Unit Testing
- **Jest** for backend services, utilities, and middleware
- **React Testing Library** for frontend components
- **Coverage target**: 80%+ for critical business logic
- Mock external dependencies (database, email service)

#### Integration Testing
- Test API endpoints with actual database (test DB)
- Test RBAC enforcement across different roles
- Test transaction splitting calculations
- Test budget threshold alerts
- Test CSV import with various formats

#### E2E Testing
- **Playwright** or **Cypress** for critical user flows
- Key scenarios:
  - Family setup flow
  - Monthly expense entry with splits
  - Investment update
  - Month-end report generation
- Run against staging environment

#### Acceptance Testing
- Gherkin-based acceptance criteria (see requirement.md section 13)
- Feature-driven scenarios covering:
  - Budget threshold alerts
  - Import deduplication
  - RBAC enforcement
  - Transaction splitting
  - Audit logging

#### Performance Testing
- Load testing with realistic data volumes
- API responses: < 200ms (p95)
- Page load: < 2s (LCP)
- Database queries: < 100ms (p95)
- Support 100 concurrent users per family

### Git Workflow
- **Branching strategy**: Feature branch workflow
  - `main`: Production-ready code
  - `develop`: Integration branch
  - `feature/`: Feature branches (e.g., `feature/transaction-splits`)
  - `bugfix/`: Bug fix branches
- **Commit conventions**: Conventional commits
  - `feat:` New features
  - `fix:` Bug fixes
  - `refactor:` Code refactoring
  - `test:` Test additions/changes
  - `docs:` Documentation changes
  - `chore:` Maintenance tasks
- **Pull requests**: Required for all changes to `main` and `develop`
- **Code review**: At least one approval required
- **CI/CD**: Automated testing and deployment via GitHub Actions

## Domain Context

### Joint Family System
The application is specifically designed for **joint family systems** common in South Asian households where:
- Multiple generations live together
- Household expenses are shared among contributing members
- Different family members have varying levels of financial oversight
- Privacy is important for personal expenses while maintaining family transparency
- One person (SuperAdmin) has ultimate control, but others help manage

### Key Personas
1. **Ayesha (Family Admin)**: 27, manages shared bills and groceries; needs category budgets and reminders
2. **Hamza (Investor Member)**: 30, tracks SIPs and ETFs; wants portfolio return and allocation view
3. **Nani (Elder Guest)**: 68, occasional viewer; prefers large typography and read-only dashboards
4. **Uncle Imran (Auditor)**: 55, oversees spending; wants exportable reports without editing rights

### Core Domain Entities

#### Family (Tenant)
- Tenant unit with currency, timezone, fiscal year settings
- One SuperAdmin per family
- Settings: name, currency (PKR default), timezone (Asia/Karachi), fiscal_year_start

#### Membership
- Links users to families with specific roles
- One user can belong to multiple families
- Role-based permissions independent per family
- Time-boxed access for Auditor role

#### Transaction
- Expense or income with optional splits among members
- Scope: Family (shared) or Personal (private)
- Attributes: date, amount, wallet, category, merchant, notes, private_notes (encrypted), attachments
- Splits: Divide expense by percentage or absolute amount among members

#### Wallet/Account
- Types: Cash, Bank, Credit Card, Digital Wallet, Loan
- Owner: Personal or Family
- Balance tracking with reconciliation support
- Status: Active or Archived

#### Category
- Default categories: Housing, Food, Transport, Utilities, Healthcare, Education, Entertainment, Others
- Custom categories with icons and colors
- Admin-managed

#### Budget
- Category-based spending limits
- Periods: Monthly, Quarterly, Yearly
- Alert thresholds: 50%, 80%, 100%
- Rollover option: Carry forward unused budget or reset

#### Investment
- Holdings: Stocks, ETFs, Mutual Funds, SIPs, Fixed Deposits, Gold
- Attributes: symbol, quantity, purchase_price, current_price, purchase_date
- Performance metrics: Absolute returns, XIRR for SIPs, allocation percentages
- Scope: Family or Personal portfolios

### Financial Concepts
- **Transaction Splitting**: Divide expenses among family members by percentage or absolute amount; must sum to transaction total
- **Budget Rollover**: Carry forward unused budget or reset monthly/quarterly/yearly
- **Investment Performance**: Absolute returns = (current_value - invested_value) / invested_value; XIRR for irregular cash flows (SIPs)
- **Reconciliation**: Manual balance adjustments with reason tracking in audit log

### RBAC Model

#### Roles
- **SuperAdmin** (one per Family): Full control over tenant, billing, role assignment, policy, data retention
- **Family Admin**: Manage categories, budgets, members (invite/remove except SuperAdmin), wallets, reports
- **Member**: Create/edit own transactions, attach receipts, view family dashboards, propose edits to shared entries
- **Guest (Read-only)**: View assigned wallets/reports only
- **Auditor (Time-boxed Read-only)**: View all reports/ledgers during audit window; cannot see encrypted private notes

#### Permission Matrix (Key Resources)
| Resource | SuperAdmin | Admin | Member | Guest | Auditor |
|----------|------------|-------|--------|-------|---------|
| **Families** | CRUD | Update, View | View | View (limited) | View |
| **Users & Roles** | Assign/Revoke all | Assign/Revoke (non-SuperAdmin) | - | - | - |
| **Wallets** | CRUD | CRUD | Create personal, View | View assigned | View all |
| **Transactions (Family)** | CRUD | CRUD | Create, Update own*, View | View | View |
| **Transactions (Personal)** | - | View | CRUD | - | - |
| **Categories/Budgets** | CRUD | CRUD | View | View | View |
| **Investments** | CRUD | CRUD | CRUD own | View | View |
| **Reports/Exports** | Generate, Export | Generate, Export | Generate own | View assigned | Generate, Export |
| **Policy/Retention** | Configure | - | - | - | - |

*Member edits to shared family transactions trigger "Propose Change" → Admin reviews → Accept/Reject with audit log

## Important Constraints

### Regulatory & Compliance
- **GDPR compliance** for EU users
  - Data export capability (right to portability)
  - Right to deletion with audit trail
  - Privacy policy and consent management
- **Data retention**: Configurable by SuperAdmin (1-7 years)
- **Audit requirements**: All CRUD operations logged with timestamp, actor, before/after values

### Performance Requirements
- **99.9% uptime SLA**
- **API response times**: < 200ms (p95)
- **Page load (LCP)**: < 2s
- **Database queries**: < 100ms (p95)
- **Scalability**: Support 10,000 families initially with 100 concurrent users per family
- **Data volume**: 1 million transactions per family per year
- **Automatic failover** and load balancing

### Security Constraints
- **Private notes must be encrypted at rest** (PostgreSQL pgcrypto)
- **Attachments must be virus-scanned** before storage
- **Audit log is immutable** (write-only append, no updates/deletes)
- **Hard delete requires SuperAdmin approval** with 30-day delay window
- **Session timeout**: 30 minutes (configurable)
- **2FA required** for SuperAdmin accounts
- **JWT short TTL**: 15 minutes with refresh token rotation
- **Row-level security**: All queries filtered by family_id
- **Rate limiting**: Prevent brute force and API abuse

### Accessibility Requirements
- **WCAG AA compliance** mandatory
- Scalable font sizes for elderly users (Nani persona)
- Keyboard navigation support
- Screen reader compatibility
- High contrast mode support
- Touch-friendly mobile interface (min 44px touch targets)

### Technical Limitations (V1)
- **No automated bank aggregation** (Plaid, OpenBanking) — use CSV/Excel/OFX import
- **No tax filing automation**
- **No complex portfolio analytics** beyond basic SIP/ETF/stocks
- **Manual price updates** for investments (third-party API planned for V1.1)
- **OCR receipt scanning**: Stub only in V1.1
- **Multi-currency**: Single currency per family in V1 (revaluation out of scope)

### Localization
- **English**: Primary language
- **Urdu**: Planned for V1.2
- Currency formatting: Respect locale (PKR default)
- Date/time: Respect timezone (Asia/Karachi default)

## External Dependencies

### Authentication & Security
- **Email service** (SendGrid, AWS SES, or similar)
  - Invite flows with role pre-selection
  - Password reset
  - Budget alerts and notifications
  - Approval request notifications
- **2FA provider** (OTP via email or authenticator app)

### File Storage
- **Object storage** (AWS S3, Google Cloud Storage, or similar)
  - Receipt attachments (images)
  - CSV/Excel/OFX import files
  - Export files (PDF, Excel, CSV)
- **Presigned upload URLs** for secure client-side uploads
- **Virus scanning service** (ClamAV, AWS S3 with Lambda, or similar)

### Notifications
- **Email delivery service** (SendGrid, Mailgun, or similar)
- **In-app notification system** (WebSockets or polling)
- **Push notification capability** for mobile PWA (Firebase Cloud Messaging or similar)

### Import/Export
- **Import Formats**: CSV, Excel (.xlsx), OFX (Quicken format)
- **Export Formats**: PDF (via library like PDFKit or Puppeteer), Excel, CSV
- **CSV parsing**: Support for common bank statement formats
- **Date format support**: `YYYY-MM-DD`, `DD/MM/YYYY`, `MM/DD/YYYY`
- **Deduplication logic**: Smart matching based on date, amount, merchant

### Reporting & Analytics
- **Chart generation**: Client-side using Recharts or Chart.js
- **PDF generation**: Server-side using Puppeteer or PDFKit
- **Excel generation**: Server-side using ExcelJS or similar

### Future Integrations (Out of Scope for V1)
- **Third-party price feeds** for investment tracking (Alpha Vantage, Yahoo Finance API)
- **Automated bank sync** via Plaid or OpenBanking
- **OCR service** for receipt scanning (Google Cloud Vision, AWS Textract)
- **Credit score integration**
- **FX rate feeds** for multi-currency support
- **Tax calculation services**

### Development & Monitoring
- **Error tracking**: Sentry or similar
- **Logging**: Winston or Pino for structured logging
- **Monitoring**: Prometheus + Grafana or similar for metrics
- **APM**: New Relic, Datadog, or similar for performance monitoring

## Release Phases

### MVP (8-10 weeks)
- Families, Memberships (RBAC)
- Wallets, Transactions with splits
- Categories and Budgets with alerts
- Basic Reports (monthly, category breakdown)
- CSV Imports with deduplication
- Audit log
- Exports (PDF, Excel, CSV)

### V1.1
- Investments (Stocks, ETFs, SIPs, Mutual Funds)
- Simple returns calculation (absolute and XIRR)
- Goal tracker
- OCR stub for receipt scanning
- Auditor role with time-boxed access

### V1.2
- Mobile PWA polish (offline support, app install)
- Urdu labels and localization
- Push notifications
- Advanced budgeting rules (custom alert thresholds, category groups)
- Investment price feeds via third-party API
