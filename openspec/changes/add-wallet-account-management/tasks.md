# Implementation Tasks

## Prerequisites

This change depends on the completion of:
- `add-auth-multitenancy-foundation` (user authentication, families, memberships, RBAC)

## 1. Database Schema

- [ ] 1.1 Create wallets table
  - Columns: id (UUID), family_id (UUID FK), name (VARCHAR), type (ENUM), balance (DECIMAL), currency (VARCHAR), owner_type (ENUM: PERSONAL, FAMILY), owner_id (UUID), status (ENUM: ACTIVE, ARCHIVED), created_at, updated_at, deleted_at
  - Indexes: family_id, owner_id, status, type
  - Foreign keys: family_id → families(id)
- [ ] 1.2 Create wallet_adjustments table
  - Columns: id (UUID), wallet_id (UUID FK), user_id (UUID FK), previous_balance (DECIMAL), new_balance (DECIMAL), reason (TEXT), created_at
  - Indexes: wallet_id, user_id, created_at
  - Foreign keys: wallet_id → wallets(id), user_id → users(id)
- [ ] 1.3 Add row-level security policies for multi-tenancy on wallets table
- [ ] 1.4 Create database migration scripts
- [ ] 1.5 Add wallet type ENUM (CASH, BANK, CREDIT_CARD, DIGITAL_WALLET, LOAN)
- [ ] 1.6 Add owner type ENUM (PERSONAL, FAMILY)
- [ ] 1.7 Add status ENUM (ACTIVE, ARCHIVED)
- [ ] 1.8 Create check constraint: balance precision (2 decimal places)

## 2. Backend - Constants and Types

- [ ] 2.1 Define WALLET_TYPES constant array
- [ ] 2.2 Define OWNER_TYPES constant array
- [ ] 2.3 Define WALLET_STATUS constant array
- [ ] 2.4 Create TypeScript types for Wallet entity
- [ ] 2.5 Create TypeScript types for WalletAdjustment entity
- [ ] 2.6 Define error codes (INVALID_WALLET_TYPE, WALLET_ARCHIVED, WALLET_HAS_RECENT_ACTIVITY, WALLET_OWNERSHIP_IMMUTABLE, WALLET_CURRENCY_IMMUTABLE, RECONCILIATION_REASON_REQUIRED)
- [ ] 2.7 Create wallet validation schemas (create, update, reconcile)
- [ ] 2.8 Define user-facing messages for wallet operations

## 3. Backend - Wallet Service

- [ ] 3.1 Create wallet service with CRUD methods
- [ ] 3.2 Implement createWallet method
  - Validate wallet type, currency, owner_type
  - Set default currency to family currency if not specified
  - Create initial balance adjustment if initial_balance > 0
  - Enforce ownership rules (Member can only create personal wallets)
- [ ] 3.3 Implement updateWallet method
  - Prevent modification of owner_type, owner_id, currency, balance
  - Allow updates to name, type, status
  - Validate permissions based on ownership and role
- [ ] 3.4 Implement deleteWallet method (soft delete)
  - Check for recent transactions (last 30 days)
  - Reject if recent activity exists
  - Set deleted_at timestamp
- [ ] 3.5 Implement archiveWallet method
  - Set status to ARCHIVED
  - Create audit log entry
- [ ] 3.6 Implement reactivateWallet method
  - Set status to ACTIVE
  - Create audit log entry
- [ ] 3.7 Implement getWallets method with filtering
  - Filter by type, owner_type, status, include_archived
  - Apply RBAC: Members see own + family; Admin sees all; Guest sees assigned only
  - Return wallets sorted by name
- [ ] 3.8 Implement getWalletById method
  - Validate permissions based on ownership and role
  - Include recent transactions (last 10)
  - Include reconciliation history
- [ ] 3.9 Implement reconcileWallet method
  - Validate permissions (owner can reconcile personal; Admin can reconcile family)
  - Require reason parameter
  - Create wallet_adjustment record
  - Update wallet balance
  - Create audit log entry
- [ ] 3.10 Implement getReconciliationHistory method
  - Return all wallet_adjustments for a wallet
  - Include user who made adjustment
  - Sort by created_at DESC
- [ ] 3.11 Add audit logging for all wallet operations
- [ ] 3.12 Write unit tests for wallet service (Jest)

## 4. Backend - Wallet Routes

- [ ] 4.1 Create wallet router (GET, POST, PUT, DELETE /v1/wallets)
- [ ] 4.2 Create POST /v1/wallets endpoint
  - Validate request body with schema
  - Check authentication and family context
  - Enforce RBAC (Member can create personal; Admin can create family)
  - Call createWallet service
  - Return created wallet with 201 status
- [ ] 4.3 Create GET /v1/wallets endpoint
  - Support query params: type, owner_type, status, include_archived
  - Check authentication and family context
  - Call getWallets service with filters
  - Return wallet list with 200 status
- [ ] 4.4 Create GET /v1/wallets/:id endpoint
  - Validate wallet_id parameter
  - Check authentication and family context
  - Verify user has permission to view wallet
  - Call getWalletById service
  - Return wallet details with 200 status
- [ ] 4.5 Create PUT /v1/wallets/:id endpoint
  - Validate request body with schema
  - Check authentication and family context
  - Verify user has permission to update wallet
  - Call updateWallet service
  - Return updated wallet with 200 status
- [ ] 4.6 Create DELETE /v1/wallets/:id endpoint
  - Check authentication and family context
  - Verify user has permission to delete wallet
  - Call deleteWallet service
  - Return 204 status on success
- [ ] 4.7 Create POST /v1/wallets/:id/reconcile endpoint
  - Validate request body (new_balance, reason)
  - Check authentication and family context
  - Verify user has permission to reconcile
  - Call reconcileWallet service
  - Return updated wallet with 200 status
- [ ] 4.8 Create POST /v1/wallets/:id/archive endpoint
  - Check authentication and family context
  - Verify permissions
  - Call archiveWallet service
  - Return updated wallet with 200 status
- [ ] 4.9 Create POST /v1/wallets/:id/reactivate endpoint
  - Check authentication and family context
  - Verify permissions
  - Call reactivateWallet service
  - Return updated wallet with 200 status
- [ ] 4.10 Create GET /v1/wallets/:id/reconciliation-history endpoint
  - Check authentication and family context
  - Verify permissions
  - Call getReconciliationHistory service
  - Return adjustment list with 200 status
- [ ] 4.11 Add error handling middleware to wallet routes
- [ ] 4.12 Write integration tests for wallet routes (Jest with test database)

## 5. Backend - RBAC Integration

- [ ] 5.1 Create wallet permission checker middleware
- [ ] 5.2 Implement canCreateWallet check (Member for personal; Admin for family)
- [ ] 5.3 Implement canViewWallet check (owner, Admin, Auditor based on ownership)
- [ ] 5.4 Implement canUpdateWallet check (owner for personal; Admin for family)
- [ ] 5.5 Implement canDeleteWallet check (owner for personal; Admin for family)
- [ ] 5.6 Implement canReconcileWallet check (owner for personal; Admin for family)
- [ ] 5.7 Integrate permission checks into wallet routes
- [ ] 5.8 Write tests for permission enforcement

## 6. Frontend - Constants and Types

- [ ] 6.1 Create WALLET_TYPES constant with labels and icons
- [ ] 6.2 Create WALLET_STATUS constant with labels
- [ ] 6.3 Create TypeScript types for Wallet
- [ ] 6.4 Create TypeScript types for WalletAdjustment
- [ ] 6.5 Define wallet-related UI messages and labels

## 7. Frontend - Wallet Service

- [ ] 7.1 Create wallet API service with methods
- [ ] 7.2 Implement createWallet API call
- [ ] 7.3 Implement getWallets API call with filters
- [ ] 7.4 Implement getWalletById API call
- [ ] 7.5 Implement updateWallet API call
- [ ] 7.6 Implement deleteWallet API call
- [ ] 7.7 Implement reconcileWallet API call
- [ ] 7.8 Implement archiveWallet API call
- [ ] 7.9 Implement reactivateWallet API call
- [ ] 7.10 Implement getReconciliationHistory API call
- [ ] 7.11 Add error handling and error message mapping

## 8. Frontend - Wallet State Management

- [ ] 8.1 Create wallet store/context (using Zustand/Redux/Context)
- [ ] 8.2 Add state for wallet list
- [ ] 8.3 Add state for active wallet filters
- [ ] 8.4 Add state for selected wallet
- [ ] 8.5 Add actions for CRUD operations
- [ ] 8.6 Add loading and error states

## 9. Frontend - Wallet List Page

- [ ] 9.1 Create WalletList component
- [ ] 9.2 Display wallets in a responsive grid or list
- [ ] 9.3 Show wallet name, type icon, balance, and currency
- [ ] 9.4 Add filter controls (type, owner, status, include archived)
- [ ] 9.5 Add "Create Wallet" button (visible based on permissions)
- [ ] 9.6 Add click handler to view wallet details
- [ ] 9.7 Show empty state when no wallets exist
- [ ] 9.8 Add loading skeleton while fetching
- [ ] 9.9 Display archived wallets with visual indicator
- [ ] 9.10 Implement responsive design (mobile-first)
- [ ] 9.11 Write tests for WalletList component (React Testing Library)

## 10. Frontend - Wallet Create/Edit Form

- [ ] 10.1 Create WalletForm component (used for both create and edit)
- [ ] 10.2 Add form fields: name, type, currency, owner_type, initial_balance
- [ ] 10.3 Implement form validation (required fields, valid currency)
- [ ] 10.4 Default currency to family currency
- [ ] 10.5 Hide owner_type field for Members (default to PERSONAL)
- [ ] 10.6 Show owner_type selection for Admin/SuperAdmin
- [ ] 10.7 Add wallet type selection with icons
- [ ] 10.8 Add form submission handler
- [ ] 10.9 Show success/error messages
- [ ] 10.10 Redirect to wallet list after successful creation
- [ ] 10.11 Disable immutable fields in edit mode (currency, owner_type)
- [ ] 10.12 Write tests for WalletForm component

## 11. Frontend - Wallet Details Page

- [ ] 11.1 Create WalletDetails component
- [ ] 11.2 Display wallet information (name, type, balance, currency, owner, status)
- [ ] 11.3 Show recent transactions (placeholder for now, will integrate with transaction feature)
- [ ] 11.4 Add "Edit Wallet" button (visible based on permissions)
- [ ] 11.5 Add "Archive/Reactivate" button (visible based on permissions and status)
- [ ] 11.6 Add "Delete Wallet" button (visible based on permissions)
- [ ] 11.7 Add "Reconcile Balance" button (visible based on permissions)
- [ ] 11.8 Show reconciliation history in expandable section
- [ ] 11.9 Add confirmation dialogs for delete and archive actions
- [ ] 11.10 Implement responsive layout
- [ ] 11.11 Write tests for WalletDetails component

## 12. Frontend - Reconciliation Modal

- [ ] 12.1 Create ReconciliationModal component
- [ ] 12.2 Show current balance prominently
- [ ] 12.3 Add input for new balance
- [ ] 12.4 Add textarea for reason (required)
- [ ] 12.5 Calculate and display adjustment amount
- [ ] 12.6 Add form validation (new_balance required, reason required)
- [ ] 12.7 Add submit and cancel buttons
- [ ] 12.8 Handle reconciliation submission
- [ ] 12.9 Show success/error messages
- [ ] 12.10 Close modal and refresh wallet on success
- [ ] 12.11 Write tests for ReconciliationModal component

## 13. Frontend - Wallet Selection Component

- [ ] 13.1 Create WalletSelector component (reusable dropdown for transaction entry)
- [ ] 13.2 Fetch and display active wallets
- [ ] 13.3 Group wallets by type or owner
- [ ] 13.4 Show wallet name, type icon, and current balance
- [ ] 13.5 Add search/filter functionality
- [ ] 13.6 Handle wallet selection
- [ ] 13.7 Support disabled state for archived wallets
- [ ] 13.8 Write tests for WalletSelector component

## 14. Frontend - Accessibility

- [ ] 14.1 Ensure all wallet forms have proper labels and ARIA attributes
- [ ] 14.2 Support keyboard navigation for wallet list and forms
- [ ] 14.3 Add screen reader announcements for wallet operations
- [ ] 14.4 Ensure color contrast meets WCAG AA standards
- [ ] 14.5 Test with screen reader

## 15. Integration Testing

- [ ] 15.1 Write E2E test for creating personal wallet as Member
- [ ] 15.2 Write E2E test for creating family wallet as Admin
- [ ] 15.3 Write E2E test for permission denial (Member creating family wallet)
- [ ] 15.4 Write E2E test for wallet reconciliation flow
- [ ] 15.5 Write E2E test for archive and reactivate wallet
- [ ] 15.6 Write E2E test for soft delete with recent activity check
- [ ] 15.7 Write E2E test for wallet filtering
- [ ] 15.8 Write E2E test for viewing wallet details and reconciliation history

## 16. Documentation

- [ ] 16.1 Document wallet API endpoints with examples
- [ ] 16.2 Document wallet permission matrix
- [ ] 16.3 Document wallet types and their use cases
- [ ] 16.4 Document reconciliation process
- [ ] 16.5 Add wallet management to user guide

## Dependencies and Parallelization Notes

**Sequential dependencies:**
- Task 1 (Database) must complete first
- Task 2-5 (Backend) must complete before Task 12 (Integration tests)
- Task 6-13 (Frontend) can start after Task 7 (Wallet API service)
- Task 15 (E2E tests) requires both backend and frontend complete

**Can be done in parallel:**
- Tasks 2-5 (Backend work) can proceed together after database is ready
- Tasks 6-13 (Frontend components) can be built in parallel once wallet API service exists
- Tasks 9, 10, 11, 12, 13 (Different frontend components) are independent

**Critical path:**
Database → Backend Service → Backend Routes → Frontend API Service → Frontend Components → E2E Tests
