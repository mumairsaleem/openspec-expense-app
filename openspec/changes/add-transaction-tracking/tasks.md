# Implementation Tasks

## Prerequisites

This change depends on the completion of:
- `add-auth-multitenancy-foundation` (authentication, families, memberships, RBAC)
- `add-wallet-account-management` (wallets for transaction association)

## 1. Database Schema

- [ ] 1.1 Create transactions table
  - Columns: id (UUID), family_id (UUID FK), wallet_id (UUID FK), amount (DECIMAL), type (ENUM: EXPENSE, INCOME), category_id (UUID FK), date (DATE), merchant (VARCHAR), notes (TEXT), private_notes (ENCRYPTED TEXT), scope (ENUM: FAMILY, PERSONAL), created_by (UUID FK), attachments (JSONB array of URLs), recurring_template_id (UUID FK nullable), created_at, updated_at, deleted_at
  - Indexes: family_id, wallet_id, category_id, date, created_by, type, scope, merchant
  - Foreign keys: family_id → families(id), wallet_id → wallets(id), category_id → categories(id), created_by → users(id)
- [ ] 1.2 Create transaction_splits table
  - Columns: id (UUID), transaction_id (UUID FK), member_id (UUID FK), amount (DECIMAL), percentage (DECIMAL), created_at
  - Indexes: transaction_id, member_id
  - Foreign keys: transaction_id → transactions(id) ON DELETE CASCADE, member_id → users(id)
  - Constraint: CHECK (percentage >= 0 AND percentage <= 100)
- [ ] 1.3 Create recurring_transactions table
  - Columns: id (UUID), family_id (UUID FK), wallet_id (UUID FK), amount (DECIMAL), type (ENUM), category_id (UUID FK), merchant (VARCHAR), notes (TEXT), scope (ENUM), frequency (ENUM: DAILY, WEEKLY, BIWEEKLY, MONTHLY, QUARTERLY, YEARLY), next_date (DATE), is_active (BOOLEAN), created_by (UUID FK), created_at, updated_at, deleted_at
  - Indexes: family_id, next_date, is_active, frequency
  - Foreign keys: family_id → families(id), wallet_id → wallets(id), category_id → categories(id), created_by → users(id)
- [ ] 1.4 Create change_requests table
  - Columns: id (UUID), transaction_id (UUID FK), requested_by (UUID FK), requested_at (TIMESTAMP), changes (JSONB), status (ENUM: PENDING, APPROVED, REJECTED), reviewed_by (UUID FK nullable), reviewed_at (TIMESTAMP nullable), rejection_reason (TEXT nullable)
  - Indexes: transaction_id, requested_by, status
  - Foreign keys: transaction_id → transactions(id), requested_by → users(id), reviewed_by → users(id)
- [ ] 1.5 Add row-level security policies for transactions, splits, recurring templates
- [ ] 1.6 Create database migration scripts
- [ ] 1.7 Add check constraint: split amounts sum equals transaction amount (application-level enforcement)
- [ ] 1.8 Create trigger to update wallet balance on transaction insert/update/delete

## 2. Backend - Constants and Types

- [ ] 2.1 Define TRANSACTION_TYPES constant (EXPENSE, INCOME)
- [ ] 2.2 Define TRANSACTION_SCOPE constant (FAMILY, PERSONAL)
- [ ] 2.3 Define RECURRENCE_FREQUENCY constant (DAILY, WEEKLY, BIWEEKLY, MONTHLY, QUARTERLY, YEARLY)
- [ ] 2.4 Define CHANGE_REQUEST_STATUS constant (PENDING, APPROVED, REJECTED)
- [ ] 2.5 Create TypeScript types for Transaction entity
- [ ] 2.6 Create TypeScript types for TransactionSplit entity
- [ ] 2.7 Create TypeScript types for RecurringTransaction entity
- [ ] 2.8 Create TypeScript types for ChangeRequest entity
- [ ] 2.9 Define error codes (TXN_SPLIT_MISMATCH, CANNOT_SPLIT_PERSONAL_TRANSACTION, CURRENCY_MISMATCH, SCOPE_IMMUTABLE, IMMUTABLE_FIELD, DUPLICATE_SPLIT_MEMBER, MUST_DELETE_ALL_SPLITS, MEMBER_NOT_IN_FAMILY)
- [ ] 2.10 Create validation schemas for create/update transaction
- [ ] 2.11 Create validation schemas for splits
- [ ] 2.12 Create validation schemas for recurring templates

## 3. Backend - File Upload Service

- [ ] 3.1 Set up file storage integration (AWS S3 or similar)
- [ ] 3.2 Implement presigned URL generation for uploads
- [ ] 3.3 Implement virus scanning for uploaded files
- [ ] 3.4 Add file type validation (image/jpeg, image/png, application/pdf)
- [ ] 3.5 Add file size limit enforcement (10MB max)
- [ ] 3.6 Create file deletion utility for cleanup
- [ ] 3.7 Write tests for file upload service

## 4. Backend - Transaction Service

- [ ] 4.1 Create transaction service with CRUD methods
- [ ] 4.2 Implement createTransaction method
  - Validate wallet is active and belongs to family
  - Validate currency matches wallet currency
  - Set default date to today if not provided
  - Process file attachments
  - Update wallet balance
  - Create audit log entry
- [ ] 4.3 Implement updateTransaction method
  - Check permissions (owner or Admin for direct update; Member triggers change request)
  - Prevent modification of wallet_id, date, recurring_template_id
  - Recalculate wallet balance if amount changes
  - Create audit log entry
- [ ] 4.4 Implement deleteTransaction method (soft delete)
  - Check permissions
  - Reverse wallet balance update
  - Soft delete splits if any
  - Create audit log entry
- [ ] 4.5 Implement getTransactions method with filtering
  - Filter by date range, category, wallet, type, merchant, scope
  - Apply RBAC: show family + personal for owner; hide others' personal for Member
  - Support pagination (limit, offset)
  - Include split information
  - Sort by date DESC by default
- [ ] 4.6 Implement getTransactionById method
  - Validate permissions based on scope and ownership
  - Include splits, attachments, creator info
  - Exclude private_notes for Auditor
- [ ] 4.7 Implement searchTransactions method
  - Full-text search on merchant and notes
  - Case-insensitive search
  - Exclude private_notes from search for non-owners
- [ ] 4.8 Implement merchant autocomplete suggestion
  - Return previously used merchant names
  - Filter by family context
  - Sort by frequency of use
- [ ] 4.9 Add wallet balance update utility
  - Calculate balance delta based on transaction type and amount
  - Handle both creation and updates
  - Ensure atomic balance updates
- [ ] 4.10 Write unit tests for transaction service

## 5. Backend - Transaction Split Service

- [ ] 5.1 Create transaction split service
- [ ] 5.2 Implement createSplits method
  - Validate transaction is family-scoped
  - Validate all members belong to family
  - Check for duplicate members
  - Validate amounts/percentages sum to transaction amount
  - Create all splits atomically
- [ ] 5.3 Implement updateSplits method
  - Delete existing splits
  - Create new splits
  - Validate total matches transaction amount
- [ ] 5.4 Implement deleteSplits method
  - Delete all splits for a transaction
  - Create audit log entry
- [ ] 5.5 Implement validateSplits utility
  - Check split total equals transaction amount (tolerance of 0.01 for rounding)
  - Validate member IDs
  - Calculate percentages from amounts or vice versa
- [ ] 5.6 Implement split calculator utilities
  - Equal split calculation with rounding
  - Percentage to amount conversion
  - Amount to percentage conversion
- [ ] 5.7 Implement getMemberContributionReport
  - Calculate total contributions per member for date range
  - Show who paid for what in shared expenses
- [ ] 5.8 Write unit tests for split service

## 6. Backend - Recurring Transaction Service

- [ ] 6.1 Create recurring transaction service
- [ ] 6.2 Implement createRecurringTemplate method
  - Validate wallet, category, frequency
  - Calculate initial next_date
  - Set is_active=true by default
- [ ] 6.3 Implement updateRecurringTemplate method
  - Allow updates to amount, merchant, notes, frequency, is_active, next_date
  - Recalculate next_date if frequency changes
  - Check permissions
- [ ] 6.4 Implement deleteRecurringTemplate method (soft delete)
  - Soft delete template
  - Do not affect already created transactions
- [ ] 6.5 Implement getRecurringTemplates method
  - Filter by is_active, frequency
  - Apply RBAC
  - Sort by next_date ASC
- [ ] 6.6 Implement getRecurringTemplateById method
  - Include recent generated transactions
  - Show creation history
- [ ] 6.7 Implement pauseTemplate and resumeTemplate methods
  - Toggle is_active flag
  - Preserve next_date when pausing
- [ ] 6.8 Implement skipNextOccurrence method
  - Update next_date to next cycle
  - Log skip in audit trail
- [ ] 6.9 Write unit tests for recurring service

## 7. Backend - Recurring Transaction Background Job

- [ ] 7.1 Create background job scheduler (using node-cron or similar)
- [ ] 7.2 Implement daily job to process due recurring templates
  - Query templates where next_date <= today AND is_active=true
  - Create transaction from template
  - Update next_date based on frequency
  - Handle errors (wallet archived, deleted, etc.)
  - Mark template as failed if errors occur
- [ ] 7.3 Implement frequency calculation utility
  - DAILY: next_date + 1 day
  - WEEKLY: next_date + 7 days
  - BIWEEKLY: next_date + 14 days
  - MONTHLY: next_date + 1 month (same day)
  - QUARTERLY: next_date + 3 months
  - YEARLY: next_date + 1 year
  - Handle month-end dates correctly
- [ ] 7.4 Implement reminder job (runs daily)
  - Query templates where next_date = tomorrow
  - Send reminder emails to creator and Admins
  - Respect user notification preferences
- [ ] 7.5 Add error notification for failed template processing
  - Email Admins when template fails
  - Include failure reason and template details
- [ ] 7.6 Write tests for background jobs

## 8. Backend - Change Request Service

- [ ] 8.1 Create change request service
- [ ] 8.2 Implement createChangeRequest method
  - Store proposed changes in JSONB
  - Set status=PENDING
  - Notify Admins
- [ ] 8.3 Implement approveChangeRequest method
  - Apply changes to transaction
  - Update change request status to APPROVED
  - Notify requester
- [ ] 8.4 Implement rejectChangeRequest method
  - Update status to REJECTED
  - Store rejection reason
  - Notify requester
- [ ] 8.5 Implement getPendingChangeRequests method
  - List all pending requests for family
  - Show requester, transaction, and proposed changes
- [ ] 8.6 Write unit tests for change request service

## 9. Backend - Transaction Routes

- [ ] 9.1 Create transaction router
- [ ] 9.2 Create POST /v1/transactions endpoint
- [ ] 9.3 Create GET /v1/transactions endpoint (with filters)
- [ ] 9.4 Create GET /v1/transactions/search endpoint
- [ ] 9.5 Create GET /v1/transactions/merchants/autocomplete endpoint
- [ ] 9.6 Create GET /v1/transactions/:id endpoint
- [ ] 9.7 Create PUT /v1/transactions/:id endpoint
- [ ] 9.8 Create DELETE /v1/transactions/:id endpoint
- [ ] 9.9 Create POST /v1/transactions/:id/splits endpoint
- [ ] 9.10 Create PUT /v1/transactions/:id/splits endpoint
- [ ] 9.11 Create DELETE /v1/transactions/:id/splits endpoint
- [ ] 9.12 Create GET /v1/reports/member-contributions endpoint
- [ ] 9.13 Add file upload endpoint for attachments (POST /v1/transactions/attachments/upload)
- [ ] 9.14 Write integration tests for transaction routes

## 10. Backend - Recurring Transaction Routes

- [ ] 10.1 Create recurring transaction router
- [ ] 10.2 Create POST /v1/recurring-transactions endpoint
- [ ] 10.3 Create GET /v1/recurring-transactions endpoint
- [ ] 10.4 Create GET /v1/recurring-transactions/:id endpoint
- [ ] 10.5 Create PUT /v1/recurring-transactions/:id endpoint
- [ ] 10.6 Create DELETE /v1/recurring-transactions/:id endpoint
- [ ] 10.7 Create POST /v1/recurring-transactions/:id/pause endpoint
- [ ] 10.8 Create POST /v1/recurring-transactions/:id/resume endpoint
- [ ] 10.9 Create POST /v1/recurring-transactions/:id/skip-next endpoint
- [ ] 10.10 Write integration tests for recurring routes

## 11. Backend - Change Request Routes

- [ ] 11.1 Create change request router
- [ ] 11.2 Create GET /v1/change-requests endpoint (pending requests)
- [ ] 11.3 Create POST /v1/change-requests/:id/approve endpoint
- [ ] 11.4 Create POST /v1/change-requests/:id/reject endpoint
- [ ] 11.5 Write integration tests for change request routes

## 12. Frontend - Constants and Types

- [ ] 12.1 Create TRANSACTION_TYPES constant with labels
- [ ] 12.2 Create RECURRENCE_FREQUENCY constant with labels
- [ ] 12.3 Create TypeScript types for Transaction
- [ ] 12.4 Create TypeScript types for TransactionSplit
- [ ] 12.5 Create TypeScript types for RecurringTransaction
- [ ] 12.6 Define transaction-related UI messages

## 13. Frontend - Transaction API Service

- [ ] 13.1 Create transaction API service
- [ ] 13.2 Implement createTransaction API call
- [ ] 13.3 Implement getTransactions API call with filters
- [ ] 13.4 Implement searchTransactions API call
- [ ] 13.5 Implement getTransactionById API call
- [ ] 13.6 Implement updateTransaction API call
- [ ] 13.7 Implement deleteTransaction API call
- [ ] 13.8 Implement uploadAttachment API call
- [ ] 13.9 Implement createSplits API call
- [ ] 13.10 Implement updateSplits API call
- [ ] 13.11 Implement deleteSplits API call
- [ ] 13.12 Implement getMerchantSuggestions API call
- [ ] 13.13 Implement recurring transaction API calls (CRUD)
- [ ] 13.14 Implement change request API calls

## 14. Frontend - Transaction State Management

- [ ] 14.1 Create transaction store/context
- [ ] 14.2 Add state for transaction list
- [ ] 14.3 Add state for active filters (date range, category, wallet, type)
- [ ] 14.4 Add state for selected transaction
- [ ] 14.5 Add actions for CRUD operations
- [ ] 14.6 Add loading and error states

## 15. Frontend - Transaction List Page

- [ ] 15.1 Create TransactionList component
- [ ] 15.2 Display transactions in chronological order with virtual scrolling
- [ ] 15.3 Show transaction date, merchant, amount, category icon, wallet
- [ ] 15.4 Add filter controls (date range picker, category selector, wallet selector, type selector)
- [ ] 15.5 Add search bar for merchant/notes
- [ ] 15.6 Add "Add Transaction" button
- [ ] 15.7 Show empty state when no transactions exist
- [ ] 15.8 Add loading skeleton while fetching
- [ ] 15.9 Implement infinite scroll pagination
- [ ] 15.10 Color code expenses (red) vs income (green)
- [ ] 15.11 Show split indicator icon for split transactions
- [ ] 15.12 Implement responsive design (mobile-first)
- [ ] 15.13 Write tests for TransactionList component

## 16. Frontend - Transaction Create/Edit Form

- [ ] 16.1 Create TransactionForm component
- [ ] 16.2 Add form fields: type, amount, wallet, category, date, merchant, notes, private_notes
- [ ] 16.3 Implement form validation
- [ ] 16.4 Add wallet selector (reuse WalletSelector component)
- [ ] 16.5 Add category selector
- [ ] 16.6 Add date picker (default to today)
- [ ] 16.7 Add merchant field with autocomplete
- [ ] 16.8 Add scope selector (personal/family) based on permissions
- [ ] 16.9 Add file upload for receipt attachments (drag-drop + click)
- [ ] 16.10 Show attachment preview with remove option
- [ ] 16.11 Add "Add Split" button (only for family transactions)
- [ ] 16.12 Integrate split calculator component
- [ ] 16.13 Show running balance preview
- [ ] 16.14 Handle form submission
- [ ] 16.15 Show success/error messages
- [ ] 16.16 Redirect to transaction list after creation
- [ ] 16.17 Write tests for TransactionForm component

## 17. Frontend - Split Calculator Component

- [ ] 17.1 Create SplitCalculator component
- [ ] 17.2 Show transaction amount prominently
- [ ] 17.3 Add member selection (multi-select from family members)
- [ ] 17.4 Add "Equal Split" button
- [ ] 17.5 Allow custom amount input for each member
- [ ] 17.6 Allow percentage input for each member
- [ ] 17.7 Calculate and display running total
- [ ] 17.8 Highlight when total doesn't match transaction amount
- [ ] 17.9 Add validation error messages
- [ ] 17.10 Handle rounding for equal splits
- [ ] 17.11 Write tests for SplitCalculator component

## 18. Frontend - Transaction Details Page

- [ ] 18.1 Create TransactionDetails component
- [ ] 18.2 Display all transaction information
- [ ] 18.3 Show attachments with lightbox view
- [ ] 18.4 Show split breakdown if applicable
- [ ] 18.5 Add "Edit Transaction" button (based on permissions)
- [ ] 18.6 Add "Delete Transaction" button (based on permissions)
- [ ] 18.7 Show change request indicator if pending changes exist
- [ ] 18.8 Add confirmation dialog for delete
- [ ] 18.9 Show linked recurring template if applicable
- [ ] 18.10 Write tests for TransactionDetails component

## 19. Frontend - Recurring Transaction Management

- [ ] 19.1 Create RecurringTemplateList component
- [ ] 19.2 Display templates with next due date, amount, frequency
- [ ] 19.3 Add "Create Template" button
- [ ] 19.4 Show active/paused indicator
- [ ] 19.5 Add quick pause/resume toggle
- [ ] 19.6 Create RecurringTemplateForm component
- [ ] 19.7 Add frequency selector
- [ ] 19.8 Add next_date picker
- [ ] 19.9 Add "Skip Next Occurrence" button
- [ ] 19.10 Show history of generated transactions
- [ ] 19.11 Write tests for recurring components

## 20. Frontend - Change Request Review

- [ ] 20.1 Create ChangeRequestList component (Admin only)
- [ ] 20.2 Show pending change requests with diff view
- [ ] 20.3 Add "Approve" and "Reject" buttons
- [ ] 20.4 Add rejection reason textarea
- [ ] 20.5 Show before/after comparison
- [ ] 20.6 Implement approve/reject actions
- [ ] 20.7 Show notification badge for pending requests
- [ ] 20.8 Write tests for change request components

## 21. Frontend - Accessibility

- [ ] 21.1 Ensure all forms have proper labels and ARIA attributes
- [ ] 21.2 Support keyboard navigation for transaction list and forms
- [ ] 21.3 Add screen reader announcements for transaction operations
- [ ] 21.4 Ensure color contrast meets WCAG AA
- [ ] 21.5 Test with screen reader

## 22. Integration Testing

- [ ] 22.1 Write E2E test for creating expense transaction
- [ ] 22.2 Write E2E test for creating income transaction
- [ ] 22.3 Write E2E test for transaction with split
- [ ] 22.4 Write E2E test for member change request workflow
- [ ] 22.5 Write E2E test for Admin approving change request
- [ ] 22.6 Write E2E test for creating recurring template
- [ ] 22.7 Write E2E test for auto-creation from recurring template
- [ ] 22.8 Write E2E test for transaction search and filtering
- [ ] 22.9 Write E2E test for file attachment upload

## 23. Documentation

- [ ] 23.1 Document transaction API endpoints
- [ ] 23.2 Document split calculation logic
- [ ] 23.3 Document recurring transaction frequency calculations
- [ ] 23.4 Document change request workflow
- [ ] 23.5 Add transaction management to user guide

## Dependencies and Parallelization Notes

**Sequential dependencies:**
- Task 1 (Database) must complete first
- Tasks 2-11 (Backend) must complete before Task 22 (E2E tests)
- Task 13 (Frontend API service) must complete before tasks 14-20
- Background jobs (Task 7) require transaction service (Task 4)

**Can be done in parallel:**
- Tasks 4, 5, 6, 8 (Different backend services) can proceed together
- Tasks 9, 10, 11 (Different route groups) independent
- Tasks 15-20 (Different frontend components) can be built in parallel
- Task 3 (File upload) can be done independently

**Critical path:**
Database → Transaction Service → Transaction Routes → Frontend API → Frontend Components → E2E Tests
