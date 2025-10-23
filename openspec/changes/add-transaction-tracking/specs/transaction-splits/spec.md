# Transaction Splits Specification

## ADDED Requirements

### Requirement: Split Transaction Creation

The system SHALL allow users to divide transaction amounts among family members.

#### Scenario: Split transaction equally
- **GIVEN** a family transaction of 1000
- **WHEN** splitting equally among 4 members
- **THEN** each split is created with amount=250 and percentage=25
- **AND** total split amounts equal transaction amount

#### Scenario: Split by custom amounts
- **GIVEN** a family transaction of 1000
- **WHEN** specifying splits as [300, 400, 300]
- **THEN** each member is assigned their specified amount
- **AND** percentages are calculated automatically

#### Scenario: Split by percentages
- **GIVEN** a family transaction of 1000
- **WHEN** specifying splits as [50%, 30%, 20%]
- **THEN** amounts are calculated as [500, 300, 200]
- **AND** both amount and percentage are stored

#### Scenario: Split validation - total must match
- **GIVEN** a transaction of 1000
- **WHEN** splits total to 900
- **THEN** the request is rejected with error code `TXN_SPLIT_MISMATCH`
- **AND** the error indicates the discrepancy

#### Scenario: Cannot split personal transaction
- **GIVEN** a transaction with scope=PERSONAL
- **WHEN** attempting to create splits
- **THEN** the request is rejected with error code `CANNOT_SPLIT_PERSONAL_TRANSACTION`
- **AND** only family transactions can be split

### Requirement: Split Update

The system SHALL allow authorized users to update transaction splits.

#### Scenario: Creator updates splits
- **GIVEN** a transaction creator updates splits
- **WHEN** changing split amounts or members
- **THEN** the splits are updated
- **AND** total still equals transaction amount
- **AND** an audit log entry is created

#### Scenario: Admin updates any splits
- **GIVEN** a user with Admin role
- **WHEN** updating splits on any family transaction
- **THEN** the splits are updated
- **AND** validation is performed

#### Scenario: Member cannot update splits on others' transactions
- **GIVEN** a user with Member role
- **WHEN** attempting to update splits on another member's transaction
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

### Requirement: Split Deletion

The system SHALL allow removal of splits to revert to unsplit transaction.

#### Scenario: Remove all splits
- **GIVEN** a transaction with splits
- **WHEN** all splits are deleted
- **THEN** the transaction becomes a regular unsplit transaction
- **AND** the full amount is attributed to the wallet owner

#### Scenario: Partial split removal not allowed
- **GIVEN** a transaction with 3 splits
- **WHEN** attempting to delete only 1 split
- **THEN** the request is rejected with error code `MUST_DELETE_ALL_SPLITS`
- **AND** splits must be deleted together or updated to redistribute

### Requirement: Split Member Validation

The system SHALL validate that split members belong to the family.

#### Scenario: All split members must be in family
- **GIVEN** a transaction split request
- **WHEN** a member_id does not belong to the family
- **THEN** the request is rejected with error code `MEMBER_NOT_IN_FAMILY`
- **AND** the split is not created

#### Scenario: Duplicate members in splits
- **GIVEN** a split request with same member appearing twice
- **WHEN** the split is processed
- **THEN** the request is rejected with error code `DUPLICATE_SPLIT_MEMBER`
- **AND** each member can appear only once

#### Scenario: Creator can be in split
- **GIVEN** a transaction creator splits a transaction
- **WHEN** including themselves in the split
- **THEN** the split is valid
- **AND** creator's portion is tracked like other members

### Requirement: Split Reporting

The system SHALL provide split reporting for member contributions.

#### Scenario: View splits on transaction detail
- **GIVEN** a transaction with splits
- **WHEN** viewing transaction details
- **THEN** all splits are shown with member names and amounts
- **AND** percentages are displayed

#### Scenario: Member contribution report
- **GIVEN** multiple split transactions in a month
- **WHEN** generating member contribution report
- **THEN** each member's total contributed amount is calculated
- **AND** report shows who paid for what in shared expenses

#### Scenario: Split summary by member
- **GIVEN** a family member views their splits
- **WHEN** requesting split summary
- **THEN** all transactions where they are in a split are shown
- **AND** total amount they owe or are owed is calculated

### Requirement: Split Calculator

The system SHALL provide split calculation utilities for the UI.

#### Scenario: Calculate equal splits
- **GIVEN** a transaction amount and number of members
- **WHEN** requesting equal split calculation
- **THEN** each member's amount is calculated
- **AND** rounding differences are assigned to the first member

#### Scenario: Rounding adjustment
- **GIVEN** a transaction of 100 split equally among 3 members
- **WHEN** calculating splits
- **THEN** splits are [33.34, 33.33, 33.33]
- **AND** total still equals 100.00

#### Scenario: Convert percentages to amounts
- **GIVEN** percentages and transaction amount
- **WHEN** calculating amounts
- **THEN** each amount is percentage * transaction_amount
- **AND** rounding is handled to match total

### Requirement: Split History

The system SHALL track split changes in audit log.

#### Scenario: Split creation logged
- **GIVEN** splits are created for a transaction
- **WHEN** the splits are saved
- **THEN** an audit log entry is created
- **AND** before_data is null and after_data contains all splits

#### Scenario: Split update logged
- **GIVEN** splits are updated
- **WHEN** the update is saved
- **THEN** an audit log entry is created
- **AND** before_data and after_data show the changes

#### Scenario: Split deletion logged
- **GIVEN** splits are deleted
- **WHEN** the deletion is processed
- **THEN** an audit log entry is created
- **AND** before_data contains deleted splits and after_data is empty array
