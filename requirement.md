# Product Requirements Document (PRD)

**Product**: Home Expense & Investment Tracker (HEIT)

**Purpose**: Build a spec-driven, instrumented web app to compare development tools/frameworks (e.g., different backends, ORMs, test frameworks, CI/CD stacks) by implementing the same set of features under a single, unambiguous specification.

**Primary Audience**: Young users living in **joint family systems** where multiple household members contribute to expenses and investments, with varying levels of access and oversight.

**Success Criteria**: Clear feature parity across implementations; measured performance, reliability, and DX metrics; household adoption; secure RBAC with least-privilege defaults.

---

## 1. Goals & Non-Goals

### Goals

- Track shared and personal expenses, recurring bills, budgets, and investment holdings at **family** and **individual** levels.
- Provide **RBAC** with one **SuperAdmin** per tenant (family) and granular permissions for Admins, Members, Guests, and Auditors.
- Offer **transparent reporting**: month-over-month (MoM), year-to-date (YTD), category breakdowns, wallets, and investment performance.
- Enable **spec-driven development**: stable API contracts, acceptance criteria, and fixtures for repeatable benchmarks across toolchains.

### Non-Goals (V1)

- Automated bank aggregation (Plaid, OpenBanking) — use CSV/Excel/OFX import first.
- Tax filing automation.
- Complex portfolio analytics (options/derivatives) beyond basic SIP/ETF/stocks.

---

## 2. Personas (Joint-Family Focus)

- **Ayesha (Family Admin)**: 27, manages shared bills and groceries; needs category budgets and reminders.
- **Hamza (Investor Member)**: 30, tracks SIPs and ETFs; wants portfolio return and allocation view.
- **Nani (Elder Guest)**: 68, occasional viewer; prefers large typography and read-only dashboards.
- **Uncle Imran (Auditor)**: 55, oversees spending; wants exportable reports without editing rights.

**Accessibility**: WCAG AA; scalable font sizes; bilingual labels (English + Urdu stub in roadmap).

---

## 3. Roles & Permissions (RBAC)

### Tenancy Model
Each **Family** is a tenant. Users can belong to multiple families with independent roles.

### Roles

- **SuperAdmin** (one per Family): Full control over tenant, billing, role assignment, policy, and data retention.
- **Family Admin**: Manage categories, budgets, members (invite/remove except SuperAdmin), wallets, and reports.
- **Member**: Create/edit own transactions, attach receipts, view family dashboards, propose edits to shared entries.
- **Guest (Read-only)**: View assigned wallets/reports only.
- **Auditor (Time-boxed Read-only)**: Can view all reports/ledgers during audit window; cannot see hidden private notes.

### Permission Matrix (excerpt)

| Resource | SuperAdmin | Admin | Member | Guest | Auditor |
|----------|------------|-------|--------|-------|---------|
| **Families** | Create, Update, Delete, View | Update, View | View | View (limited) | View |
| **Users & Roles** | Assign/Revoke all | Assign/Revoke (non-SuperAdmin) | - | - | - |
| **Wallets/Accounts** | CRUD | CRUD | Create personal, View | View assigned | View all |
| **Transactions (Family)** | CRUD | CRUD | Create, Update own*, View | View | View |
| **Transactions (Personal)** | - | View | CRUD | - | - |
| **Categories/Budgets** | CRUD | CRUD | View | View | View |
| **Investments** | CRUD | CRUD | CRUD own | View | View |
| **Reports/Exports** | Generate, Export | Generate, Export | Generate own | View assigned | Generate, Export |
| **Policy/Retention** | Configure | - | - | - | - |

*Member edits to shared family transactions trigger "Propose Change" → Admin reviews → Accept/Reject with audit log.

---

## 4. Feature List (Scope)

### 4.1 Onboarding & Tenancy
- Family creation wizard (name, currency, timezone)
- SuperAdmin account setup with 2FA
- Invite flow: email with role pre-selection
- Member join: accept invite, profile setup
- Multi-family support with role context switching

### 4.2 Expense Tracking
- **Transaction Entry**: Date, amount, wallet/account, category, merchant, notes, attachments (receipts)
- **Splits**: Divide transactions among members; percentage or absolute amounts
- **Recurring Transactions**: Monthly bills with auto-entry and reminder notifications
- **Scope**: Personal (visible to owner + Admin) or Family (visible to all members)
- **Quick Entry**: OCR receipt scanning (V1.1 stub), voice input consideration

### 4.3 Wallets & Accounts
- **Types**: Cash, Bank, Credit Card, Digital Wallet, Loan
- **Attributes**: Name, balance, currency, owner (personal/family), status (active/archived)
- **Reconciliation**: Manual balance adjustments with reason tracking

### 4.4 Categories & Budgets
- **Default Categories**: Housing, Food, Transport, Utilities, Healthcare, Education, Entertainment, Others
- **Custom Categories**: Admin-created with icons and colors
- **Budgets**: Monthly/quarterly/yearly limits per category
- **Alerts**: 50%, 80%, 100% threshold notifications
- **Rollover Options**: Carry forward unused budget or reset

### 4.5 Investment Tracking
- **Holdings**: Stocks, ETFs, Mutual Funds, SIPs, Fixed Deposits, Gold
- **Attributes**: Symbol, quantity, purchase price, current price, purchase date
- **Performance**: Absolute returns, XIRR for SIPs, allocation pie charts
- **Family vs Personal Portfolios**: Separate tracking with consolidated view for Admins

### 4.6 Reporting & Analytics
- **Dashboards**: MoM spending trends, category breakdowns, budget vs actual
- **Custom Date Ranges**: Filter by month/quarter/year or custom
- **Member Contribution Reports**: Who paid for what in shared expenses
- **Investment Summary**: Portfolio value, gains/losses, asset allocation
- **Export Formats**: PDF, Excel, CSV with customizable columns

### 4.7 Imports & Integrations
- **CSV/Excel Import**: Bank statements with mapping wizard
- **OFX Support**: Quicken format for investment accounts
- **Deduplication**: Smart matching to prevent duplicate entries
- **Import History**: Log of all imports with rollback capability

### 4.8 Notifications & Reminders
- **Types**: Budget alerts, bill due dates, approval requests, audit notifications
- **Channels**: In-app, email, push (mobile PWA)
- **Preferences**: Per-user notification settings by type

### 4.9 Audit & Compliance
- **Audit Log**: All CRUD operations with timestamp, actor, before/after values
- **Time-boxed Access**: Auditor role with expiry date
- **Data Retention**: Configurable by SuperAdmin (1-7 years)
- **Soft Delete**: 30-day recovery window before permanent deletion

---

## 5. User Flows

### Flow 1: Family Setup
1. SuperAdmin creates family account
2. Sets currency, timezone, fiscal year
3. Creates initial categories and budgets
4. Invites family members with roles
5. Members accept and create profiles

### Flow 2: Monthly Expense Entry
1. Member selects wallet
2. Enters transaction details
3. Optionally splits with other members
4. Attaches receipt photo
5. System updates budget tracking
6. Notifications sent if thresholds crossed

### Flow 3: Investment Update
1. Member navigates to Investments
2. Adds new purchase or sells holding
3. System fetches latest prices (manual in V1)
4. Dashboard updates returns and allocation
5. Family view shows consolidated portfolio

### Flow 4: Month-End Review
1. Admin generates monthly report
2. Reviews member contributions
3. Checks budget adherence
4. Exports report for record keeping
5. Optionally adjusts next month's budgets

---

## 6. API Specification (REST)

### Base URL
```
https://api.heit.app/v1
```

### Authentication
- Bearer JWT tokens
- Refresh token rotation
- Session timeout: 30 minutes (configurable)

### Key Endpoints

#### Families
```
POST   /families                 # Create family (SuperAdmin)
GET    /families                 # List user's families
GET    /families/{id}            # Get family details
PUT    /families/{id}            # Update family settings
DELETE /families/{id}            # Delete family (SuperAdmin only)
```

#### Transactions
```
POST   /transactions             # Create transaction
GET    /transactions             # List transactions (filtered)
GET    /transactions/{id}        # Get transaction details
PUT    /transactions/{id}        # Update transaction
DELETE /transactions/{id}        # Delete transaction
POST   /transactions/{id}/split  # Create/update split
```

#### Budgets
```
POST   /budgets                  # Create budget
GET    /budgets                  # List budgets
GET    /budgets/{id}/usage       # Get current usage
PUT    /budgets/{id}             # Update budget
```

#### Reports
```
GET    /reports/monthly          # Monthly summary
GET    /reports/category         # Category breakdown
GET    /reports/member           # Member contributions
POST   /reports/export           # Generate export
```

---

## 7. Data Model (Core Entities)

### Family
```json
{
  "id": "uuid",
  "name": "string",
  "currency": "PKR",
  "timezone": "Asia/Karachi",
  "fiscal_year_start": "04-01",
  "created_at": "timestamp",
  "super_admin_id": "user_uuid"
}
```

### User
```json
{
  "id": "uuid",
  "email": "string",
  "name": "string",
  "phone": "string",
  "profile_picture": "url",
  "notification_preferences": {},
  "created_at": "timestamp"
}
```

### Membership
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "family_id": "uuid",
  "role": "MEMBER|ADMIN|GUEST|AUDITOR|SUPERADMIN",
  "joined_at": "timestamp",
  "expires_at": "timestamp"  // For Auditor role
}
```

### Transaction
```json
{
  "id": "uuid",
  "family_id": "uuid",
  "wallet_id": "uuid",
  "amount": "decimal",
  "type": "EXPENSE|INCOME",
  "category_id": "uuid",
  "date": "date",
  "merchant": "string",
  "notes": "string",
  "private_notes": "encrypted_string",
  "scope": "FAMILY|PERSONAL",
  "created_by": "user_uuid",
  "attachments": ["urls"],
  "splits": [
    {
      "member_id": "uuid",
      "amount": "decimal",
      "percentage": "decimal"
    }
  ]
}
```

### Budget
```json
{
  "id": "uuid",
  "family_id": "uuid",
  "category_id": "uuid",
  "amount": "decimal",
  "period": "MONTHLY|QUARTERLY|YEARLY",
  "start_date": "date",
  "end_date": "date",
  "rollover_enabled": "boolean",
  "alert_thresholds": [50, 80, 100]
}
```

---

## 8. UI/UX Requirements

### Design Principles
- **Mobile-first** responsive design
- **Accessibility**: WCAG AA compliance
- **Localization**: English primary, Urdu support planned
- **Dark mode** support
- **Progressive Web App** capabilities

### Key Screens

#### Dashboard
- Summary cards: This month's spend, budget status, recent transactions
- Quick action buttons: Add expense, pay bill, view reports
- Family/Personal toggle
- Notification bell with badge

#### Transaction List
- Infinite scroll with virtual scrolling for performance
- Filter bar: Date range, category, member, wallet
- Search by merchant/notes
- Bulk actions: Export, delete (Admin)

#### Add/Edit Transaction
- Smart date picker (defaults to today)
- Auto-complete for merchants
- Category suggestions based on merchant
- Receipt capture with crop/rotate
- Split calculator with member checkboxes

#### Reports
- Interactive charts (responsive, touch-enabled)
- Drill-down from summary to details
- Print-friendly layout
- Export options clearly visible

---

## 9. Performance Requirements

### Response Times
- API responses: < 200ms (p95)
- Page load: < 2s (LCP)
- Database queries: < 100ms (p95)

### Scalability
- Support 10,000 families initially
- 100 concurrent users per family
- 1 million transactions per family per year

### Reliability
- 99.9% uptime SLA
- Automatic failover
- Daily backups with point-in-time recovery

---

## 10. Security & Privacy

### Security Measures
- JWT with short TTL + refresh; per-request membership authorization
- Row-level access checks; private notes field encrypted at rest
- Attachments virus-scanned; presigned upload URLs
- Audit log is write-only append; time synchronization
- Data retention configurable; hard-delete requires SuperAdmin + delay window

### Privacy
- GDPR compliance for EU users
- Data export capability
- Right to deletion (with audit trail)
- Separate personal and family data scopes

---

## 11. Analytics & KPIs

### Adoption Metrics
- Number of families created
- Monthly Active Users (MAU)
- Active members per family

### Financial Metrics
- Transactions per month
- Import success rate
- Budget adherence rate

### Performance Metrics
- p95 latency for key endpoints
- Import throughput

### Reliability Metrics
- Error rate
- Rollback/undo frequency
- Approval queue latency

---

## 12. Release Plan

### MVP (8-10 weeks as spec baseline)
- Families, Memberships (RBAC)
- Wallets, Transactions with splits
- Budgets, Basic Reports
- Imports (CSV)
- Audit log, Exports

### V1.1
- Investments, SIP tracking
- Simple returns calculation
- Goal tracker
- OCR stub
- Auditor role link

### V1.2
- Mobile PWA polish
- Urdu labels
- Push notifications
- Advanced budgeting rules

---

## 13. Acceptance Test Scenarios (Gherkin Samples)

```gherkin
Feature: Family budgets
  Scenario: Budget threshold alert at 80%
    Given a family budget of 10,000 for Groceries in Oct 2025
    And existing spend of 7,500
    When a member records a 1,500 expense in Groceries
    Then the budget utilization becomes 90%
    And an 80%+ alert is sent to Admins

Feature: Import deduplication
  Scenario: Duplicate line item ignored
    Given a CSV with two identical lines for the same wallet and day
    When I run the import
    Then only one transaction is created
```

---

## 14. Open Questions

- Should family admins view members' **personal** wallets by default? (Currently: Admin can view but cannot edit.)
- Price feeds for investments: manual import vs third-party API? (Planned manual V1.)
- Budget carryover rules per category or global?

---

## 15. Out of Scope (Now)

- Automated bank sync, credit score, loans
- Multi-currency revaluation with FX gain/loss accounting
- Tax lot selection (FIFO/LIFO) — use average cost initially

---

## Appendices

### Appendix A: Role Policy Examples (Pseudo-Rego)

```rego
allow: post /transactions if role in [ADMIN, MEMBER] and (scope == personal || scope == family)
allow: patch /memberships if actor.role == SUPERADMIN
deny if actor.role == AUDITOR and method != GET
```

### Appendix B: CSV Import Mapping Template

- **Columns**: `Date, Description, Amount, Type(Dr/Cr), Wallet, Category(optional)`
- **Date formats supported**: `YYYY-MM-DD`, `DD/MM/YYYY`

### Appendix C: Error Codes

- `TXN_SPLIT_MISMATCH`
- `BUDGET_EXCEEDED`
- `DUPLICATE_IMPORT_ROW`
- `ROLE_INSUFFICIENT`
- `AUDIT_LOCKED_PERIOD`
