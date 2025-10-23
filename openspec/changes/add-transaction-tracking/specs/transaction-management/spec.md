# Transaction Management Specification

## ADDED Requirements

### Requirement: Transaction Creation

The system SHALL allow authorized users to create transactions with type, amount, wallet, category, and metadata.

#### Scenario: Create expense transaction as Member
- **GIVEN** a user with Member role
- **WHEN** creating an expense transaction for their personal wallet
- **THEN** the transaction is created
- **AND** the wallet balance is decreased by the amount
- **AND** an audit log entry is created

#### Scenario: Create income transaction
- **GIVEN** a user creates an income transaction
- **WHEN** specifying type=INCOME
- **THEN** the transaction is created
- **AND** the wallet balance is increased by the amount

#### Scenario: Create family transaction as Admin
- **GIVEN** a user with Admin role
- **WHEN** creating a transaction with scope=FAMILY
- **THEN** the transaction is created
- **AND** all family members can view it
- **AND** the transaction appears in family reports

#### Scenario: Member creates personal transaction
- **GIVEN** a user with Member role
- **WHEN** creating a transaction with scope=PERSONAL
- **THEN** the transaction is created
- **AND** only the creator and Admins can view it
- **AND** other members cannot see it

#### Scenario: Required fields validation
- **GIVEN** a user creates a transaction without required fields
- **WHEN** the request is missing amount, wallet_id, type, or date
- **THEN** the request is rejected with error code `MISSING_REQUIRED_FIELDS`
- **AND** the response indicates which fields are missing

#### Scenario: Wallet archived check
- **GIVEN** a user attempts to create a transaction
- **WHEN** the selected wallet has status=ARCHIVED
- **THEN** the request is rejected with error code `WALLET_ARCHIVED`
- **AND** no transaction is created

#### Scenario: Date defaults to today
- **GIVEN** a user creates a transaction without specifying date
- **WHEN** the transaction is created
- **THEN** the date is set to today's date in the family timezone

#### Scenario: Merchant auto-complete suggestion
- **GIVEN** a user enters merchant name
- **WHEN** typing the merchant field
- **THEN** previously used merchant names are suggested
- **AND** suggestions are based on family transaction history

### Requirement: Transaction Attachments

The system SHALL support attaching receipt images to transactions.

#### Scenario: Attach receipt on creation
- **GIVEN** a user creates a transaction with receipt image
- **WHEN** uploading the attachment
- **THEN** the file is virus-scanned
- **AND** the file is stored with presigned URL
- **AND** the attachment URL is saved in transaction attachments array

#### Scenario: Multiple attachments per transaction
- **GIVEN** a transaction supports multiple receipts
- **WHEN** uploading up to 5 attachments
- **THEN** all attachments are stored
- **AND** attachments array contains all URLs

#### Scenario: File size limit
- **GIVEN** a user uploads a file larger than 10MB
- **WHEN** the upload is processed
- **THEN** the request is rejected with error code `FILE_TOO_LARGE`
- **AND** the transaction can still be created without attachment

#### Scenario: Supported file types
- **GIVEN** a user uploads an attachment
- **WHEN** the file type is image/jpeg, image/png, or application/pdf
- **THEN** the upload is accepted
- **AND** other file types are rejected with error code `INVALID_FILE_TYPE`

### Requirement: Transaction Update

The system SHALL allow authorized users to update transactions based on scope and permissions.

#### Scenario: Creator updates own transaction
- **GIVEN** a user updates their own transaction
- **WHEN** changing amount, category, merchant, or notes
- **THEN** the transaction is updated
- **AND** wallet balance is recalculated
- **AND** an audit log entry is created

#### Scenario: Admin updates any family transaction
- **GIVEN** a user with Admin role
- **WHEN** updating any family transaction
- **THEN** the transaction is updated
- **AND** an audit log entry is created

#### Scenario: Member proposes change to shared family transaction
- **GIVEN** a user with Member role updates a family transaction not created by them
- **WHEN** submitting the update
- **THEN** a change_request is created with status=PENDING
- **AND** the transaction is not immediately updated
- **AND** Admins are notified to review the proposal

#### Scenario: Admin reviews change request
- **GIVEN** a change_request with status=PENDING
- **WHEN** an Admin accepts the request
- **THEN** the transaction is updated with proposed changes
- **AND** the change_request status is set to APPROVED
- **AND** the requesting member is notified

#### Scenario: Admin rejects change request
- **GIVEN** a change_request with status=PENDING
- **WHEN** an Admin rejects the request
- **THEN** the transaction remains unchanged
- **AND** the change_request status is set to REJECTED
- **AND** the requesting member is notified with rejection reason

#### Scenario: Cannot change wallet or date after creation
- **GIVEN** a user attempts to update a transaction
- **WHEN** trying to change wallet_id or date
- **THEN** the request is rejected with error code `IMMUTABLE_FIELD`
- **AND** the user is instructed to delete and recreate if needed

### Requirement: Transaction Deletion

The system SHALL support soft deletion of transactions with recovery window.

#### Scenario: Soft delete own transaction
- **GIVEN** a user deletes their own transaction
- **WHEN** the deletion is confirmed
- **THEN** the transaction is soft-deleted with deleted_at timestamp
- **AND** the wallet balance is recalculated
- **AND** the transaction is hidden from default lists
- **AND** recovery is possible within 30 days

#### Scenario: Admin deletes any transaction
- **GIVEN** a user with Admin or SuperAdmin role
- **WHEN** deleting any transaction
- **THEN** the transaction is soft-deleted
- **AND** an audit log entry is created

#### Scenario: Member cannot delete others' transactions
- **GIVEN** a user with Member role
- **WHEN** attempting to delete another member's transaction
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

#### Scenario: Permanent deletion after 30 days
- **GIVEN** a transaction has been soft-deleted for 30 days
- **WHEN** the background job runs
- **THEN** the transaction is permanently deleted
- **AND** associated splits and attachments are also deleted

### Requirement: Transaction Listing

The system SHALL allow users to list transactions with filtering and pagination.

#### Scenario: List transactions for active family
- **GIVEN** a user requests transaction list
- **WHEN** filtering by active family context
- **THEN** all family transactions and user's personal transactions are returned
- **AND** transactions are sorted by date DESC by default

#### Scenario: Filter by date range
- **GIVEN** a user filters transactions by date range
- **WHEN** specifying start_date and end_date
- **THEN** only transactions within the range are returned

#### Scenario: Filter by category
- **GIVEN** a user filters by category_id
- **WHEN** the request includes category filter
- **THEN** only transactions in that category are returned

#### Scenario: Filter by wallet
- **GIVEN** a user filters by wallet_id
- **WHEN** the request includes wallet filter
- **THEN** only transactions from that wallet are returned

#### Scenario: Filter by type
- **GIVEN** a user filters by type=EXPENSE or type=INCOME
- **WHEN** the request includes type filter
- **THEN** only transactions of that type are returned

#### Scenario: Filter by merchant
- **GIVEN** a user searches by merchant name
- **WHEN** the request includes merchant search term
- **THEN** transactions with matching merchant names are returned
- **AND** search is case-insensitive and supports partial matches

#### Scenario: Pagination support
- **GIVEN** a family has 1000+ transactions
- **WHEN** requesting transaction list with limit=50 and offset=0
- **THEN** the first 50 transactions are returned
- **AND** total count is included in response metadata

#### Scenario: Guest sees limited transactions
- **GIVEN** a user with Guest role
- **WHEN** requesting transaction list
- **THEN** only transactions from assigned wallets are visible
- **AND** personal transactions of other members are hidden

#### Scenario: Auditor sees all transactions except private notes
- **GIVEN** a user with Auditor role within validity period
- **WHEN** requesting transaction list
- **THEN** all family and personal transactions are visible
- **AND** private_notes field is excluded from response

### Requirement: Transaction Details

The system SHALL provide detailed transaction information based on permissions.

#### Scenario: View own transaction details
- **GIVEN** a user views their own transaction
- **WHEN** requesting transaction details
- **THEN** all fields are visible including private_notes
- **AND** attachments are accessible
- **AND** split details are included if applicable

#### Scenario: View family transaction as Member
- **GIVEN** a user with Member role views a family transaction
- **WHEN** requesting transaction details
- **THEN** all fields except private_notes are visible
- **AND** attachments are accessible
- **AND** creator information is shown

#### Scenario: Admin views any transaction
- **GIVEN** a user with Admin role views any transaction
- **WHEN** requesting transaction details
- **THEN** all fields including private_notes are visible
- **AND** full audit trail is accessible

### Requirement: Transaction Scope Management

The system SHALL enforce visibility and permissions based on transaction scope.

#### Scenario: Personal transaction visibility
- **GIVEN** a transaction with scope=PERSONAL
- **WHEN** other members request to view it
- **THEN** only the creator and Admins can access it
- **AND** other members receive error code `INSUFFICIENT_PERMISSIONS`

#### Scenario: Family transaction visibility
- **GIVEN** a transaction with scope=FAMILY
- **WHEN** any family member requests to view it
- **THEN** the transaction is visible based on their role
- **AND** Members can view but not edit others' family transactions

#### Scenario: Scope cannot be changed after creation
- **GIVEN** a transaction with scope=PERSONAL
- **WHEN** attempting to change scope to FAMILY
- **THEN** the request is rejected with error code `SCOPE_IMMUTABLE`
- **AND** the user is instructed to create a new transaction if needed

### Requirement: Private Notes Encryption

The system SHALL encrypt private notes at rest for sensitive information.

#### Scenario: Private notes encrypted on storage
- **GIVEN** a user creates a transaction with private_notes
- **WHEN** the transaction is saved to database
- **THEN** private_notes field is encrypted using pgcrypto
- **AND** encrypted value is stored

#### Scenario: Private notes decrypted on retrieval
- **GIVEN** a transaction has encrypted private_notes
- **WHEN** the owner or Admin retrieves the transaction
- **THEN** private_notes are decrypted and returned
- **AND** decryption is transparent to the application layer

#### Scenario: Auditor cannot see private notes
- **GIVEN** a transaction has private_notes
- **WHEN** a user with Auditor role views the transaction
- **THEN** private_notes field is excluded from response
- **AND** other fields are visible

### Requirement: Wallet Balance Updates

The system SHALL automatically update wallet balances when transactions are created, updated, or deleted.

#### Scenario: Balance decreases on expense creation
- **GIVEN** a wallet with balance=1000
- **WHEN** an expense transaction of 200 is created
- **THEN** the wallet balance becomes 800

#### Scenario: Balance increases on income creation
- **GIVEN** a wallet with balance=1000
- **WHEN** an income transaction of 500 is created
- **THEN** the wallet balance becomes 1500

#### Scenario: Balance recalculated on update
- **GIVEN** an existing expense transaction of 200
- **WHEN** the amount is updated to 300
- **THEN** the wallet balance is decreased by additional 100

#### Scenario: Balance restored on deletion
- **GIVEN** an expense transaction of 200
- **WHEN** the transaction is deleted
- **THEN** the wallet balance is increased by 200

#### Scenario: Transaction currency must match wallet
- **GIVEN** a wallet with currency=USD
- **WHEN** creating a transaction with different currency
- **THEN** the request is rejected with error code `CURRENCY_MISMATCH`
- **AND** the transaction must use the wallet's currency

### Requirement: Transaction Search

The system SHALL provide full-text search across transaction fields.

#### Scenario: Search by merchant
- **GIVEN** transactions with various merchants
- **WHEN** searching for "grocery"
- **THEN** transactions with merchant names containing "grocery" are returned
- **AND** search is case-insensitive

#### Scenario: Search by notes
- **GIVEN** transactions with various notes
- **WHEN** searching for keywords in notes
- **THEN** transactions with matching notes are returned
- **AND** private_notes are excluded from search for non-owners

#### Scenario: Search by amount
- **GIVEN** transactions with various amounts
- **WHEN** filtering by amount range (min, max)
- **THEN** transactions within the amount range are returned

#### Scenario: Combined filters
- **GIVEN** multiple filter criteria
- **WHEN** applying category, date range, and amount filters together
- **THEN** transactions matching all criteria are returned
- **AND** filters use AND logic
