# Recurring Transactions Specification

## ADDED Requirements

### Requirement: Recurring Transaction Template Creation

The system SHALL allow authorized users to create recurring transaction templates.

#### Scenario: Create monthly recurring bill
- **GIVEN** a user with Member or Admin role
- **WHEN** creating a recurring template for monthly electricity bill
- **THEN** the template is created with frequency=MONTHLY
- **AND** next_date is set to the first occurrence date
- **AND** is_active is true by default

#### Scenario: Supported frequencies
- **GIVEN** a user creates recurring templates
- **WHEN** specifying frequency as DAILY, WEEKLY, BIWEEKLY, MONTHLY, QUARTERLY, or YEARLY
- **THEN** all frequencies are supported
- **AND** next_date is calculated based on frequency

#### Scenario: Template requires wallet and category
- **GIVEN** a user creates a recurring template without wallet_id
- **WHEN** the creation request is submitted
- **THEN** the request is rejected with error code `MISSING_REQUIRED_FIELDS`
- **AND** wallet_id and category_id are required

#### Scenario: Personal recurring transaction
- **GIVEN** a Member creates a recurring template
- **WHEN** the template is for personal scope
- **THEN** only that member receives reminders
- **AND** created transactions are scoped as PERSONAL

#### Scenario: Family recurring transaction
- **GIVEN** an Admin creates a recurring template
- **WHEN** the template is for family scope
- **THEN** all Admins receive reminders
- **AND** created transactions are scoped as FAMILY

### Requirement: Auto-Creation of Transactions

The system SHALL automatically create transactions from recurring templates.

#### Scenario: Daily job processes due templates
- **GIVEN** a recurring template with next_date=today
- **WHEN** the background job runs
- **THEN** a new transaction is created from the template
- **AND** next_date is updated based on frequency
- **AND** the transaction is linked to the template

#### Scenario: Monthly recurrence calculation
- **GIVEN** a template with frequency=MONTHLY and next_date=2025-01-15
- **WHEN** the transaction is created
- **THEN** next_date becomes 2025-02-15
- **AND** monthly recurrence handles month-end dates correctly

#### Scenario: Inactive template skipped
- **GIVEN** a recurring template with is_active=false
- **WHEN** the background job runs
- **THEN** no transaction is created
- **AND** next_date is not updated

#### Scenario: Archived wallet prevents creation
- **GIVEN** a recurring template with archived wallet
- **WHEN** the background job attempts to create transaction
- **THEN** the transaction is not created
- **AND** the template is marked as failed with reason
- **AND** Admins are notified

#### Scenario: Transaction creation failure handling
- **GIVEN** a recurring template fails to create transaction
- **WHEN** an error occurs (e.g., wallet deleted)
- **THEN** the template is marked as failed
- **AND** next_date is not updated
- **AND** Admins are notified to review

### Requirement: Recurring Template Update

The system SHALL allow authorized users to update recurring templates.

#### Scenario: Update template amount
- **GIVEN** a recurring template owner
- **WHEN** updating the amount or merchant
- **THEN** the template is updated
- **AND** future transactions use new values
- **AND** already created transactions are not affected

#### Scenario: Change frequency
- **GIVEN** a recurring template with frequency=MONTHLY
- **WHEN** changing to frequency=WEEKLY
- **THEN** the template is updated
- **AND** next_date is recalculated based on new frequency

#### Scenario: Admin updates any family template
- **GIVEN** a user with Admin role
- **WHEN** updating any family recurring template
- **THEN** the template is updated
- **AND** an audit log entry is created

#### Scenario: Member can only update own templates
- **GIVEN** a user with Member role
- **WHEN** attempting to update another member's template
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

### Requirement: Recurring Template Activation

The system SHALL allow pausing and resuming recurring templates.

#### Scenario: Pause recurring template
- **GIVEN** an active recurring template
- **WHEN** setting is_active=false
- **THEN** no future transactions are created
- **AND** next_date is preserved
- **AND** the template can be resumed later

#### Scenario: Resume recurring template
- **GIVEN** a paused recurring template
- **WHEN** setting is_active=true
- **THEN** the template resumes creating transactions
- **AND** next_date is recalculated if needed

#### Scenario: Update next_date manually
- **GIVEN** a recurring template
- **WHEN** manually updating next_date to a future date
- **THEN** the next occurrence is scheduled for that date
- **AND** frequency-based calculation continues from that date

### Requirement: Recurring Template Deletion

The system SHALL support soft deletion of recurring templates.

#### Scenario: Delete recurring template
- **GIVEN** a recurring template owner
- **WHEN** deleting the template
- **THEN** the template is soft-deleted with deleted_at timestamp
- **AND** no future transactions are created
- **AND** previously created transactions remain unchanged

#### Scenario: Admin can delete any family template
- **GIVEN** a user with Admin role
- **WHEN** deleting any family recurring template
- **THEN** the template is deleted
- **AND** an audit log entry is created

### Requirement: Recurring Transaction Reminders

The system SHALL send reminders before recurring transactions are created.

#### Scenario: Reminder sent 1 day before
- **GIVEN** a recurring template with next_date=tomorrow
- **WHEN** the reminder job runs
- **THEN** a reminder email is sent to the creator
- **AND** Admins receive reminders for family templates
- **AND** reminder includes amount, merchant, and due date

#### Scenario: Reminder preferences
- **GIVEN** a user has notification preferences
- **WHEN** a recurring transaction reminder is sent
- **THEN** the user's reminder preferences are respected
- **AND** reminders can be sent via email or in-app notification

#### Scenario: No reminder for inactive templates
- **GIVEN** a recurring template with is_active=false
- **WHEN** the reminder job runs
- **THEN** no reminder is sent

### Requirement: Recurring Template Listing

The system SHALL allow users to list recurring templates.

#### Scenario: List active recurring templates
- **GIVEN** a user requests recurring template list
- **WHEN** filtering by is_active=true
- **THEN** all active templates are returned
- **AND** templates are sorted by next_date ASC

#### Scenario: Filter by frequency
- **GIVEN** multiple recurring templates with different frequencies
- **WHEN** filtering by frequency=MONTHLY
- **THEN** only monthly templates are returned

#### Scenario: Member sees own and family templates
- **GIVEN** a user with Member role
- **WHEN** requesting recurring template list
- **THEN** their personal templates and family templates are returned

#### Scenario: Guest cannot see recurring templates
- **GIVEN** a user with Guest role
- **WHEN** requesting recurring template list
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`
- **AND** Guest role does not have access to recurring features

### Requirement: Recurring Template Details

The system SHALL provide detailed recurring template information.

#### Scenario: View template details
- **GIVEN** a recurring template owner views details
- **WHEN** requesting template details
- **THEN** all fields are visible including creation history
- **AND** list of recent transactions created from template is shown

#### Scenario: View generated transactions
- **GIVEN** a recurring template
- **WHEN** requesting transaction history
- **THEN** all transactions created from this template are listed
- **AND** transactions are linked to the template_id

### Requirement: Recurring Transaction Modification

The system SHALL allow modifying individual auto-created transactions.

#### Scenario: Edit auto-created transaction
- **GIVEN** a transaction created from recurring template
- **WHEN** editing the transaction amount or category
- **THEN** only that transaction is updated
- **AND** the template remains unchanged
- **AND** future transactions use template values

#### Scenario: Transaction retains template link
- **GIVEN** an auto-created transaction is edited
- **WHEN** the transaction is saved
- **THEN** the recurring_template_id field is preserved
- **AND** the link to template remains for audit trail

### Requirement: Skip Occurrence

The system SHALL allow skipping a single occurrence without affecting the template.

#### Scenario: Skip next occurrence
- **GIVEN** a recurring template with next_date=2025-06-15
- **WHEN** user chooses to skip next occurrence
- **THEN** next_date is updated to 2025-07-15 (for monthly)
- **AND** no transaction is created for June
- **AND** the template continues normally after skip

#### Scenario: Skip logging
- **GIVEN** a user skips an occurrence
- **WHEN** the skip is processed
- **THEN** an audit log entry is created
- **AND** the reason for skip can be recorded
