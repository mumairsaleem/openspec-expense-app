# Add Wallet and Account Management

## Why

Users need to track where their money is stored across different accounts (cash, bank accounts, credit cards, digital wallets, loans). Wallets are the foundation for transaction tracking - every transaction must be associated with a wallet. Without wallet management, users cannot record expenses or income.

## What Changes

- Add wallet/account creation with support for 5 types: Cash, Bank, Credit Card, Digital Wallet, Loan
- Add wallet ownership model (personal vs family-owned)
- Add wallet balance tracking with manual reconciliation
- Add wallet status management (active vs archived)
- Add wallet listing and filtering by type, owner, and status
- Add balance adjustment history for audit trail
- Add RBAC enforcement for wallet operations based on ownership and user role

## Impact

**New Capabilities:**
- `wallet-management`: Wallet CRUD, balance tracking, reconciliation, ownership management

**Affected Code:**
- New: Backend wallet routes (`/v1/wallets/*`)
- New: Database schema (wallets table, wallet_adjustments table)
- New: Frontend wallet pages (list, create, edit, reconcile)
- New: Frontend wallet selection components (for transaction entry)

**Affected Specs:**
- Builds on: `membership-rbac` (permission enforcement)
- Builds on: `family-tenancy` (family context)

**Database Changes:**
- New table: wallets (id, family_id, name, type, balance, currency, owner_type, owner_id, status, created_at, updated_at, deleted_at)
- New table: wallet_adjustments (id, wallet_id, user_id, previous_balance, new_balance, reason, created_at)

**External Dependencies:**
- None (uses existing family and membership infrastructure)
