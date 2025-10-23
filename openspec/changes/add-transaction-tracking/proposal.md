# Add Transaction Tracking

## Why

Transaction tracking is the core feature of HEIT. Users need to record expenses and income, split costs among family members, and track recurring bills. This feature enables families to understand where their money is going and manage shared household expenses effectively.

## What Changes

- Add transaction creation with support for expenses and income
- Add transaction splitting with percentage or absolute amounts among family members
- Add transaction scope (personal vs family) with appropriate visibility
- Add recurring transaction templates with auto-creation and reminders
- Add attachment support for receipts
- Add transaction search, filtering, and pagination
- Add proposal workflow for member edits to shared family transactions
- Add RBAC enforcement based on transaction scope and ownership

## Impact

**New Capabilities:**
- `transaction-management`: Transaction CRUD, search, filtering, attachments, scope management
- `transaction-splits`: Split transactions among members, validation, calculation
- `recurring-transactions`: Template management, auto-creation, reminder notifications

**Affected Code:**
- New: Backend transaction routes (`/v1/transactions/*`)
- New: Backend split routes (`/v1/transactions/:id/splits`)
- New: Backend recurring transaction routes (`/v1/recurring-transactions/*`)
- New: Database schema (transactions, transaction_splits, recurring_transactions, change_requests tables)
- New: Frontend transaction pages (list, create, edit, detail)
- New: Frontend split calculator component
- New: Frontend recurring transaction management
- Modified: Wallet service (balance updates when transactions are created/deleted)

**Affected Specs:**
- Builds on: `wallet-management` (transactions affect wallet balances)
- Builds on: `membership-rbac` (permission enforcement, member proposal workflow)
- Builds on: `family-tenancy` (family context, categories)

**Database Changes:**
- New table: transactions (id, family_id, wallet_id, amount, type, category_id, date, merchant, notes, private_notes, scope, created_by, attachments, created_at, updated_at, deleted_at)
- New table: transaction_splits (id, transaction_id, member_id, amount, percentage)
- New table: recurring_transactions (id, family_id, wallet_id, amount, type, category_id, merchant, notes, frequency, next_date, is_active, created_by)
- New table: change_requests (id, transaction_id, requested_by, requested_at, changes, status, reviewed_by, reviewed_at)

**External Dependencies:**
- File storage for receipt attachments
- Background job scheduler for recurring transaction creation
- Email service for recurring transaction reminders

**Breaking Changes:**
- None (new feature)
