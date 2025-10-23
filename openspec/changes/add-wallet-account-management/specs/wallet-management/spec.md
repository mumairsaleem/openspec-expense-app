# Wallet Management Specification

## ADDED Requirements

### Requirement: Wallet Creation

The system SHALL allow authorized users to create wallets with specified type, name, and ownership.

#### Scenario: Create personal wallet as Member
- **GIVEN** a user with Member role
- **WHEN** creating a wallet with owner_type=PERSONAL and owner_id=self
- **THEN** the wallet is created
- **AND** initial balance is set to 0 or specified amount
- **AND** the wallet is active by default
- **AND** an audit log entry is created

#### Scenario: Create family wallet as Admin
- **GIVEN** a user with Admin or SuperAdmin role
- **WHEN** creating a wallet with owner_type=FAMILY
- **THEN** the wallet is created
- **AND** all family members can view the wallet
- **AND** permissions are based on user roles

#### Scenario: Member cannot create family wallet
- **GIVEN** a user with Member role
- **WHEN** attempting to create a wallet with owner_type=FAMILY
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`
- **AND** no wallet is created

#### Scenario: Wallet with all supported types
- **GIVEN** a user creates wallets of each type
- **WHEN** creating Cash, Bank, Credit Card, Digital Wallet, and Loan types
- **THEN** all wallet types are created successfully
- **AND** type-specific attributes are validated

#### Scenario: Invalid wallet type
- **GIVEN** a user provides an unsupported wallet type
- **WHEN** the creation request is submitted
- **THEN** the request is rejected with error code `INVALID_WALLET_TYPE`
- **AND** the response lists valid wallet types

#### Scenario: Missing required fields
- **GIVEN** a user creates a wallet without required name
- **WHEN** the creation request is submitted
- **THEN** the request is rejected with error code `MISSING_REQUIRED_FIELDS`

#### Scenario: Currency defaults to family currency
- **GIVEN** a user creates a wallet without specifying currency
- **WHEN** the wallet is created
- **THEN** the currency is set to the family's default currency
- **AND** the currency can be overridden if specified

### Requirement: Wallet Update

The system SHALL allow wallet owners and authorized users to update wallet details.

#### Scenario: Owner updates personal wallet
- **GIVEN** a user updates their personal wallet
- **WHEN** changing name, type, or status
- **THEN** the wallet is updated
- **AND** balance remains unchanged
- **AND** an audit log entry is created

#### Scenario: Admin updates family wallet
- **GIVEN** a user with Admin role updates a family wallet
- **WHEN** changing wallet details
- **THEN** the wallet is updated
- **AND** all family members see the changes

#### Scenario: Member cannot update others' personal wallets
- **GIVEN** a user with Member role
- **WHEN** attempting to update another member's personal wallet
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

#### Scenario: Admin can view but not edit others' personal wallets
- **GIVEN** a user with Admin role views another member's personal wallet
- **WHEN** viewing wallet details
- **THEN** the wallet is visible with all details
- **AND** editing is not allowed per permission matrix

#### Scenario: Cannot change wallet ownership
- **GIVEN** a user attempts to change owner_type or owner_id
- **WHEN** the update request is submitted
- **THEN** the request is rejected with error code `WALLET_OWNERSHIP_IMMUTABLE`
- **AND** ownership fields remain unchanged

### Requirement: Wallet Balance Tracking

The system SHALL maintain accurate wallet balances and prevent direct balance manipulation.

#### Scenario: Initial balance on creation
- **GIVEN** a user creates a wallet with initial_balance=1000
- **WHEN** the wallet is created
- **THEN** the balance is set to 1000
- **AND** a wallet_adjustment record is created with reason "Initial balance"

#### Scenario: Balance updates only via reconciliation
- **GIVEN** a user attempts to update wallet balance directly via PUT
- **WHEN** the update request includes balance field
- **THEN** the balance field is ignored
- **AND** only reconciliation endpoint can modify balance

#### Scenario: Balance reflects transaction impact
- **GIVEN** a wallet has a balance of 1000
- **WHEN** transactions are created against the wallet
- **THEN** the balance is automatically updated by transaction service
- **AND** balance = initial_balance + income - expenses

### Requirement: Manual Reconciliation

The system SHALL allow authorized users to reconcile wallet balances with actual amounts.

#### Scenario: Reconcile personal wallet
- **GIVEN** a wallet owner's actual balance differs from recorded balance
- **WHEN** submitting a reconciliation with new_balance and reason
- **THEN** the wallet balance is updated to new_balance
- **AND** a wallet_adjustment record is created with previous_balance, new_balance, reason, and timestamp
- **AND** an audit log entry is created

#### Scenario: Reconcile family wallet as Admin
- **GIVEN** a user with Admin role reconciles a family wallet
- **WHEN** providing new balance and reason
- **THEN** the reconciliation is processed
- **AND** all family members see the updated balance

#### Scenario: Member cannot reconcile family wallet
- **GIVEN** a user with Member role
- **WHEN** attempting to reconcile a family wallet
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

#### Scenario: Reconciliation requires reason
- **GIVEN** a user submits reconciliation without a reason
- **WHEN** the reconciliation request is processed
- **THEN** the request is rejected with error code `RECONCILIATION_REASON_REQUIRED`
- **AND** no balance change occurs

#### Scenario: View reconciliation history
- **GIVEN** a wallet with multiple reconciliations
- **WHEN** requesting reconciliation history
- **THEN** all wallet_adjustments are returned in chronological order
- **AND** each entry shows user, timestamp, previous/new balance, and reason

### Requirement: Wallet Archival

The system SHALL allow users to archive wallets instead of deleting them.

#### Scenario: Archive personal wallet
- **GIVEN** a wallet owner archives their wallet
- **WHEN** the archive request is submitted
- **THEN** the wallet status is set to ARCHIVED
- **AND** the wallet is hidden from default wallet lists
- **AND** historical transactions remain accessible
- **AND** an audit log entry is created

#### Scenario: Cannot create transactions in archived wallet
- **GIVEN** a wallet with ARCHIVED status
- **WHEN** attempting to create a transaction
- **THEN** the request is rejected with error code `WALLET_ARCHIVED`
- **AND** the transaction is not created

#### Scenario: Reactivate archived wallet
- **GIVEN** a wallet owner reactivates an archived wallet
- **WHEN** setting status back to ACTIVE
- **THEN** the wallet becomes active
- **AND** transactions can be created again
- **AND** the wallet appears in default lists

#### Scenario: Admin can archive family wallets
- **GIVEN** a user with Admin role
- **WHEN** archiving a family wallet
- **THEN** the wallet is archived
- **AND** all family members see it as archived

### Requirement: Wallet Deletion

The system SHALL support soft deletion of wallets with a recovery window.

#### Scenario: Soft delete personal wallet
- **GIVEN** a wallet owner deletes their wallet
- **WHEN** the deletion is confirmed
- **THEN** the wallet is soft-deleted with deleted_at timestamp
- **AND** the wallet is hidden from all lists
- **AND** transactions are preserved but marked as from deleted wallet
- **AND** recovery is possible within 30 days

#### Scenario: Cannot delete wallet with pending transactions
- **GIVEN** a wallet has recent transactions in the last 30 days
- **WHEN** attempting to delete the wallet
- **THEN** the request is rejected with error code `WALLET_HAS_RECENT_ACTIVITY`
- **AND** the user is prompted to archive instead or wait 30 days

#### Scenario: Admin can delete family wallets
- **GIVEN** a user with Admin or SuperAdmin role
- **WHEN** deleting a family wallet without recent activity
- **THEN** the wallet is soft-deleted
- **AND** an audit log entry is created

#### Scenario: Permanent deletion after 30 days
- **GIVEN** a wallet has been soft-deleted for 30 days
- **WHEN** the background job runs
- **THEN** the wallet is permanently deleted
- **AND** associated adjustments are also deleted

### Requirement: Wallet Listing

The system SHALL allow users to list wallets based on their permissions.

#### Scenario: Member lists own wallets
- **GIVEN** a user with Member role
- **WHEN** requesting their wallet list
- **THEN** all personal wallets they own are returned
- **AND** all family wallets are returned
- **AND** archived wallets are excluded by default

#### Scenario: Include archived wallets
- **GIVEN** a user requests wallets with include_archived=true
- **WHEN** the list is retrieved
- **THEN** both active and archived wallets are returned
- **AND** each wallet includes status field

#### Scenario: Filter by wallet type
- **GIVEN** a user filters wallets by type=BANK
- **WHEN** the list is retrieved
- **THEN** only wallets with type BANK are returned

#### Scenario: Filter by owner
- **GIVEN** an Admin filters wallets by owner_type=PERSONAL
- **WHEN** the list is retrieved
- **THEN** only personal wallets are returned
- **AND** wallets from all family members are visible to Admin

#### Scenario: Guest sees assigned wallets only
- **GIVEN** a user with Guest role
- **WHEN** requesting wallet list
- **THEN** only wallets explicitly assigned to them are visible
- **AND** they cannot see other family wallets

#### Scenario: Auditor sees all wallets
- **GIVEN** a user with Auditor role within validity period
- **WHEN** requesting wallet list
- **THEN** all family and personal wallets are visible
- **AND** full details are shown except private notes

### Requirement: Wallet Details

The system SHALL provide detailed wallet information based on permissions.

#### Scenario: View personal wallet details
- **GIVEN** a wallet owner views their wallet
- **WHEN** requesting wallet details
- **THEN** all fields are visible (name, type, balance, currency, status, owner)
- **AND** recent transactions are included
- **AND** reconciliation history is accessible

#### Scenario: View family wallet details as Member
- **GIVEN** a user with Member role views a family wallet
- **WHEN** requesting wallet details
- **THEN** name, type, balance, and currency are visible
- **AND** recent transactions are included
- **AND** reconciliation history is visible but not editable

#### Scenario: Admin views any personal wallet
- **GIVEN** a user with Admin role views another member's personal wallet
- **WHEN** requesting wallet details
- **THEN** all details are visible
- **AND** the Admin can view but not reconcile personal wallets per permission matrix

### Requirement: Wallet Ownership Model

The system SHALL enforce ownership rules for wallets.

#### Scenario: Personal wallet owned by user
- **GIVEN** a wallet with owner_type=PERSONAL
- **WHEN** checking ownership
- **THEN** owner_id references the user who created it
- **AND** only that user can reconcile the wallet
- **AND** only that user and Admins can view it

#### Scenario: Family wallet owned by family
- **GIVEN** a wallet with owner_type=FAMILY
- **WHEN** checking ownership
- **THEN** owner_id references the family
- **AND** all family members can view it based on role
- **AND** only Admin/SuperAdmin can reconcile it

#### Scenario: Wallet must belong to a family
- **GIVEN** any wallet creation
- **WHEN** the wallet is created
- **THEN** family_id must be set
- **AND** family_id matches the user's active family context
- **AND** multi-tenancy isolation is enforced

### Requirement: Wallet Currency Support

The system SHALL support wallet-specific currency with validation.

#### Scenario: Create wallet with family currency
- **GIVEN** a family with currency=PKR
- **WHEN** creating a wallet without specifying currency
- **THEN** the wallet currency is set to PKR

#### Scenario: Create wallet with different currency
- **GIVEN** a family with currency=PKR
- **WHEN** creating a wallet with currency=USD
- **THEN** the wallet is created with USD currency
- **AND** the wallet currency is stored
- **AND** future transactions must use USD for this wallet

#### Scenario: Invalid currency code
- **GIVEN** a user creates a wallet with currency=INVALID
- **WHEN** the creation request is submitted
- **THEN** the request is rejected with error code `INVALID_CURRENCY`
- **AND** the response includes valid ISO currency codes

#### Scenario: Currency immutable after creation
- **GIVEN** a wallet with currency=USD
- **WHEN** attempting to change currency to EUR
- **THEN** the request is rejected with error code `WALLET_CURRENCY_IMMUTABLE`
- **AND** the currency remains USD

### Requirement: Wallet Type-Specific Validation

The system SHALL apply type-specific validation rules for wallets.

#### Scenario: Credit Card balance can be negative
- **GIVEN** a wallet with type=CREDIT_CARD
- **WHEN** balance becomes negative due to transactions
- **THEN** negative balance is allowed
- **AND** represents outstanding credit card debt

#### Scenario: Loan balance tracking
- **GIVEN** a wallet with type=LOAN
- **WHEN** tracking loan balance
- **THEN** negative balance represents money owed
- **AND** positive payments reduce the loan balance

#### Scenario: Cash wallet balance cannot be negative
- **GIVEN** a wallet with type=CASH
- **WHEN** a transaction would make balance negative
- **THEN** a warning is shown but transaction is allowed
- **AND** reconciliation can correct the balance

#### Scenario: Digital Wallet balance validation
- **GIVEN** a wallet with type=DIGITAL_WALLET
- **WHEN** reconciling balance
- **THEN** standard validation applies
- **AND** balance can be zero or positive typically
