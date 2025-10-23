# Transaction Import Specification

## ADDED Requirements

### Requirement: CSV File Upload

The system SHALL allow users to upload CSV files for transaction import.

#### Scenario: Upload CSV file
- **GIVEN** a user with Member or Admin role
- **WHEN** uploading a CSV file for import
- **THEN** the file is uploaded to temporary storage
- **AND** file validation is performed (max 10MB, valid CSV format)
- **AND** an import session is created

#### Scenario: File size validation
- **GIVEN** a user uploads a file larger than 10MB
- **WHEN** the upload is processed
- **THEN** the request is rejected with error code `FILE_TOO_LARGE`
- **AND** the user is prompted to split the file or reduce data

#### Scenario: Invalid file type
- **GIVEN** a user uploads a non-CSV/Excel file
- **WHEN** the upload is validated
- **THEN** the request is rejected with error code `INVALID_FILE_TYPE`
- **AND** supported types are listed (CSV, XLSX, OFX)

#### Scenario: Member can import to personal wallet only
- **GIVEN** a user with Member role
- **WHEN** selecting target wallet for import
- **THEN** only personal wallets are available
- **AND** family wallets are restricted

#### Scenario: Admin can import to any wallet
- **GIVEN** a user with Admin role
- **WHEN** selecting target wallet
- **THEN** both personal and family wallets are available

### Requirement: Column Mapping

The system SHALL provide a column mapping wizard for flexible CSV formats.

#### Scenario: Auto-detect standard columns
- **GIVEN** a CSV with headers Date, Description, Amount, Type
- **WHEN** the mapping wizard loads
- **THEN** columns are auto-mapped to transaction fields
- **AND** user can confirm or adjust mappings

#### Scenario: Manual column mapping
- **GIVEN** a CSV with non-standard headers
- **WHEN** auto-detection fails or is partial
- **THEN** user can manually map each CSV column to transaction fields
- **AND** required fields are marked (Date, Amount, Type)

#### Scenario: Required field validation
- **GIVEN** a user completes column mapping
- **WHEN** required fields (Date, Amount) are not mapped
- **THEN** the mapping is rejected with error code `MISSING_REQUIRED_MAPPING`
- **AND** user is prompted to map missing fields

#### Scenario: Optional field mapping
- **GIVEN** a CSV with optional columns
- **WHEN** mapping Category, Wallet, Merchant, Notes
- **THEN** these fields can be left unmapped
- **AND** default values or suggestions are used

#### Scenario: Date format detection
- **GIVEN** a CSV with date column
- **WHEN** analyzing date values
- **THEN** common formats are detected (YYYY-MM-DD, DD/MM/YYYY, MM/DD/YYYY)
- **AND** user can select or confirm date format

#### Scenario: Debit/Credit to Type mapping
- **GIVEN** a CSV with Dr/Cr or Debit/Credit column
- **WHEN** mapping to transaction type
- **THEN** Dr/Debit maps to EXPENSE and Cr/Credit maps to INCOME
- **AND** user can confirm or reverse mapping

### Requirement: Import Preview

The system SHALL provide a preview of transactions before import.

#### Scenario: Preview first 10 rows
- **GIVEN** a CSV file is uploaded and mapped
- **WHEN** user requests preview
- **THEN** the first 10 rows are parsed and displayed
- **AND** each row shows how it will be imported as a transaction

#### Scenario: Validation errors highlighted
- **GIVEN** some rows have invalid data
- **WHEN** displaying preview
- **THEN** rows with errors are highlighted
- **AND** error messages explain the issue (invalid date, negative amount for expense)

#### Scenario: Category suggestions
- **GIVEN** a row with merchant but no category
- **WHEN** previewing import
- **THEN** category is auto-suggested based on merchant history
- **AND** user can accept or change suggestion

#### Scenario: Duplicate detection preview
- **GIVEN** some rows match existing transactions
- **WHEN** displaying preview
- **THEN** duplicates are marked with warning icon
- **AND** user can choose to skip or import anyway

### Requirement: Deduplication

The system SHALL detect and prevent duplicate transaction imports.

#### Scenario: Exact duplicate detection
- **GIVEN** an existing transaction with same date, amount, and merchant
- **WHEN** importing a CSV row with identical values
- **THEN** the row is flagged as duplicate
- **AND** skipped by default with option to override

#### Scenario: Fuzzy duplicate detection
- **GIVEN** an existing transaction on same date with similar amount (within $1)
- **WHEN** importing a row
- **THEN** the row is flagged as possible duplicate
- **AND** user can review and decide

#### Scenario: Duplicate detection window
- **GIVEN** duplicate detection is performed
- **WHEN** checking for matches
- **THEN** only transactions within ±7 days are considered
- **AND** reduces false positives from recurring transactions

#### Scenario: Skip duplicates by default
- **GIVEN** an import with detected duplicates
- **WHEN** user confirms import
- **THEN** duplicate rows are skipped
- **AND** skipped_rows count is tracked

#### Scenario: Force import duplicates
- **GIVEN** a user wants to import despite duplicates
- **WHEN** selecting "import all" option
- **THEN** duplicate rows are imported
- **AND** a warning is logged

### Requirement: Import Execution

The system SHALL execute the import and create transactions.

#### Scenario: Import transactions in bulk
- **GIVEN** a user confirms import
- **WHEN** the import is executed
- **THEN** all valid rows are created as transactions
- **AND** wallet balances are updated
- **AND** import_history record is created

#### Scenario: Transaction creation with defaults
- **GIVEN** a row without category
- **WHEN** creating the transaction
- **THEN** category defaults to "Others"
- **AND** scope defaults to PERSONAL for Member, FAMILY for Admin

#### Scenario: Import failure handling
- **GIVEN** an error occurs during import (e.g., database error)
- **WHEN** the import is executing
- **THEN** the import is rolled back
- **AND** no partial data is saved
- **AND** error details are logged

#### Scenario: Import progress tracking
- **GIVEN** a large import is in progress
- **WHEN** processing rows
- **THEN** progress is reported (rows processed / total)
- **AND** user sees real-time updates

#### Scenario: Import success summary
- **GIVEN** an import completes successfully
- **WHEN** the import finishes
- **THEN** a summary is shown (total rows, imported, skipped, errors)
- **AND** user can view import history

### Requirement: Import History

The system SHALL maintain a log of all imports with rollback capability.

#### Scenario: Create import history record
- **GIVEN** an import is executed
- **WHEN** the import completes
- **THEN** a record is created in import_history
- **AND** includes filename, total_rows, imported_rows, skipped_rows, status

#### Scenario: Link transactions to import
- **GIVEN** transactions are created from import
- **WHEN** the import completes
- **THEN** each transaction is linked via import_transactions table
- **AND** the link allows rollback

#### Scenario: View import history
- **GIVEN** a user views their import history
- **WHEN** requesting past imports
- **THEN** all imports from last 90 days are shown
- **AND** each import shows summary and status

#### Scenario: Rollback import
- **GIVEN** a user wants to undo an import
- **WHEN** selecting rollback for an import
- **THEN** all transactions created in that import are soft-deleted
- **AND** wallet balances are recalculated
- **AND** import status is updated to ROLLED_BACK

#### Scenario: Cannot rollback twice
- **GIVEN** an import has been rolled back
- **WHEN** attempting to rollback again
- **THEN** the request is rejected with error code `IMPORT_ALREADY_ROLLED_BACK`

#### Scenario: Rollback creates audit trail
- **GIVEN** an import is rolled back
- **WHEN** the rollback completes
- **THEN** an audit log entry is created
- **AND** includes user, import_id, and rollback timestamp

### Requirement: Excel File Support

The system SHALL support Excel (.xlsx) file imports.

#### Scenario: Upload Excel file
- **GIVEN** a user uploads an .xlsx file
- **WHEN** the file is processed
- **THEN** the first sheet is parsed as CSV
- **AND** column mapping wizard is shown

#### Scenario: Select specific sheet
- **GIVEN** an Excel file with multiple sheets
- **WHEN** uploading the file
- **THEN** user can select which sheet to import
- **AND** sheet names are displayed for selection

#### Scenario: Excel date format handling
- **GIVEN** Excel file with date columns
- **WHEN** parsing dates
- **THEN** Excel serial dates are correctly converted to ISO dates
- **AND** date format detection works as with CSV

### Requirement: OFX File Support

The system SHALL support OFX (Quicken) file imports for investment accounts.

#### Scenario: Upload OFX file
- **GIVEN** a user uploads an .ofx file
- **WHEN** the file is parsed
- **THEN** transactions are extracted from OFX format
- **AND** mapping wizard shows OFX fields

#### Scenario: OFX transaction mapping
- **GIVEN** an OFX file with investment transactions
- **WHEN** mapping fields
- **THEN** TRNTYPE maps to transaction type
- **AND** TRNAMT maps to amount
- **AND** DTPOSTED maps to date
- **AND** NAME or MEMO maps to merchant

#### Scenario: OFX for bank statements
- **GIVEN** an OFX file from a bank account
- **WHEN** importing transactions
- **THEN** STMTTRN entries are parsed
- **AND** balance information is shown but not imported (manual reconciliation preferred)

### Requirement: Date Format Support

The system SHALL support multiple date formats in imports.

#### Scenario: ISO format dates
- **GIVEN** a CSV with dates in YYYY-MM-DD format
- **WHEN** parsing dates
- **THEN** all dates are correctly interpreted

#### Scenario: US format dates
- **GIVEN** a CSV with dates in MM/DD/YYYY format
- **WHEN** user selects US date format
- **THEN** dates are correctly parsed
- **AND** 12/25/2025 is interpreted as December 25, 2025

#### Scenario: European format dates
- **GIVEN** a CSV with dates in DD/MM/YYYY format
- **WHEN** user selects European date format
- **THEN** dates are correctly parsed
- **AND** 25/12/2025 is interpreted as December 25, 2025

#### Scenario: Invalid date handling
- **GIVEN** a CSV row with invalid date (e.g., 32/13/2025)
- **WHEN** parsing the row
- **THEN** the row is marked as error
- **AND** error message indicates invalid date

### Requirement: Amount Format Support

The system SHALL handle different amount formats and currencies.

#### Scenario: Decimal amounts
- **GIVEN** amounts with comma as decimal separator (e.g., 1.234,56)
- **WHEN** parsing amounts
- **THEN** the format is detected and converted correctly

#### Scenario: Thousands separator
- **GIVEN** amounts with thousands separator (e.g., 1,234.56 or 1.234,56)
- **WHEN** parsing amounts
- **THEN** separators are correctly interpreted based on locale

#### Scenario: Negative amounts
- **GIVEN** amounts with negative sign or parentheses (e.g., -100 or (100))
- **WHEN** parsing amounts
- **THEN** negative values are correctly identified
- **AND** can map to expense type if needed

#### Scenario: Currency symbols
- **GIVEN** amounts with currency symbols (e.g., $1,234.56, PKR 1234)
- **WHEN** parsing amounts
- **THEN** currency symbols are stripped
- **AND** numeric value is extracted

### Requirement: Error Handling and Validation

The system SHALL provide clear error messages for import issues.

#### Scenario: Row-level errors
- **GIVEN** some rows have validation errors
- **WHEN** the import is executed
- **THEN** valid rows are imported
- **AND** error rows are skipped with details logged

#### Scenario: Error row export
- **GIVEN** an import with skipped rows due to errors
- **WHEN** the import completes
- **THEN** user can download a CSV of error rows
- **AND** each row includes error reason

#### Scenario: Empty file validation
- **GIVEN** a user uploads an empty CSV
- **WHEN** the file is validated
- **THEN** the request is rejected with error code `EMPTY_FILE`

#### Scenario: Header row detection
- **GIVEN** a CSV file
- **WHEN** parsing the file
- **THEN** the first row is treated as headers
- **AND** user can specify if file has no headers

### Requirement: Import Permissions

The system SHALL enforce RBAC for import operations.

#### Scenario: Member imports to own wallet
- **GIVEN** a user with Member role
- **WHEN** importing transactions
- **THEN** transactions are scoped as PERSONAL
- **AND** can only be assigned to Member's wallets

#### Scenario: Admin imports to any wallet
- **GIVEN** a user with Admin role
- **WHEN** importing transactions
- **THEN** transactions can be scoped as FAMILY or PERSONAL
- **AND** can be assigned to any wallet in the family

#### Scenario: Guest cannot import
- **GIVEN** a user with Guest role
- **WHEN** attempting to import transactions
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

#### Scenario: Auditor cannot import
- **GIVEN** a user with Auditor role
- **WHEN** attempting to import
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`
- **AND** Auditor is read-only

### Requirement: Import Templates

The system SHALL provide downloadable CSV templates for common banks.

#### Scenario: Download import template
- **GIVEN** a user wants to import transactions
- **WHEN** requesting a template
- **THEN** a sample CSV is provided with correct column headers
- **AND** includes example rows

#### Scenario: Bank-specific templates
- **GIVEN** common bank formats are known
- **WHEN** user selects their bank
- **THEN** a pre-configured template with column mappings is provided
- **AND** reduces manual mapping effort
