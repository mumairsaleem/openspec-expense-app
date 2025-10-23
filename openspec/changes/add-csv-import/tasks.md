# Implementation Tasks

## Prerequisites

This change depends on the completion of:
- `add-transaction-tracking` (creates transactions from import)
- `add-wallet-account-management` (associates imports with wallets)
- `add-categories-budgets` (maps categories in import)

## 1. Database Schema

- [ ] 1.1 Create import_history table
  - Columns: id (UUID), family_id (UUID FK), user_id (UUID FK), filename (VARCHAR), file_url (TEXT), total_rows (INTEGER), imported_rows (INTEGER), skipped_rows (INTEGER), status (ENUM: PENDING, IN_PROGRESS, COMPLETED, FAILED, ROLLED_BACK), error_details (JSONB), created_at, completed_at
  - Indexes: family_id, user_id, status, created_at
  - Foreign keys: family_id → families(id), user_id → users(id)
- [ ] 1.2 Create import_transactions table
  - Columns: id (UUID), import_history_id (UUID FK), transaction_id (UUID FK), source_row (INTEGER), created_at
  - Indexes: import_history_id, transaction_id
  - Foreign keys: import_history_id → import_history(id), transaction_id → transactions(id)
- [ ] 1.3 Create database migration scripts
- [ ] 1.4 Add row-level security policies for import tables

## 2. Backend - Constants and Types

- [ ] 2.1 Define IMPORT_STATUS constant (PENDING, IN_PROGRESS, COMPLETED, FAILED, ROLLED_BACK)
- [ ] 2.2 Define SUPPORTED_FILE_TYPES constant (CSV, XLSX, OFX)
- [ ] 2.3 Define DATE_FORMATS constant (YYYY-MM-DD, DD/MM/YYYY, MM/DD/YYYY)
- [ ] 2.4 Create TypeScript types for ImportHistory entity
- [ ] 2.5 Create TypeScript types for ImportTransaction entity
- [ ] 2.6 Create TypeScript types for ColumnMapping
- [ ] 2.7 Define error codes (FILE_TOO_LARGE, INVALID_FILE_TYPE, MISSING_REQUIRED_MAPPING, EMPTY_FILE, IMPORT_ALREADY_ROLLED_BACK, DUPLICATE_IMPORT_ROW)

## 3. Backend - File Upload Service

- [ ] 3.1 Create file upload endpoint for imports (POST /v1/imports/upload)
- [ ] 3.2 Validate file size (max 10MB)
- [ ] 3.3 Validate file type (CSV, XLSX, OFX)
- [ ] 3.4 Upload to temporary storage with expiry (24 hours)
- [ ] 3.5 Return file_id and file_url for processing
- [ ] 3.6 Write tests for upload service

## 4. Backend - CSV Parser Service

- [ ] 4.1 Install CSV parsing library (csv-parser or papaparse)
- [ ] 4.2 Create CSV parser service
- [ ] 4.3 Implement parseCSV method
  - Read CSV file from storage
  - Detect headers
  - Parse all rows
  - Return array of row objects
- [ ] 4.4 Implement detectColumns method
  - Analyze header row
  - Auto-detect standard column names (Date, Amount, Description, Type, etc.)
  - Return suggested mappings
- [ ] 4.5 Implement parseDateFormats method
  - Detect date format from sample values
  - Support YYYY-MM-DD, DD/MM/YYYY, MM/DD/YYYY
  - Return detected format
- [ ] 4.6 Implement parseAmountFormats method
  - Detect decimal and thousands separators
  - Handle currency symbols
  - Handle negative amounts (sign or parentheses)
  - Return parsed numeric value
- [ ] 4.7 Write unit tests for CSV parser

## 5. Backend - Excel Parser Service

- [ ] 5.1 Install Excel parsing library (xlsx or exceljs)
- [ ] 5.2 Create Excel parser service
- [ ] 5.3 Implement parseExcel method
  - Read Excel file from storage
  - List all sheet names
  - Parse selected sheet
  - Convert to row objects
- [ ] 5.4 Implement handleExcelDates method
  - Convert Excel serial dates to ISO dates
  - Handle different Excel date formats
- [ ] 5.5 Write unit tests for Excel parser

## 6. Backend - OFX Parser Service

- [ ] 6.1 Install OFX parsing library (ofx-js or similar)
- [ ] 6.2 Create OFX parser service
- [ ] 6.3 Implement parseOFX method
  - Read OFX file from storage
  - Extract STMTTRN or INVTRAN entries
  - Convert to row objects
- [ ] 6.4 Implement mapOFXFields method
  - Map TRNTYPE to transaction type
  - Map TRNAMT to amount
  - Map DTPOSTED to date
  - Map NAME/MEMO to merchant
- [ ] 6.5 Write unit tests for OFX parser

## 7. Backend - Deduplication Service

- [ ] 7.1 Create deduplication service
- [ ] 7.2 Implement findDuplicates method
  - Query transactions within ±7 days of import row date
  - Match by exact amount and merchant
  - Return matches with confidence score
- [ ] 7.3 Implement fuzzyMatch method
  - Match transactions with amount within $1
  - Use fuzzy string matching for merchant names
  - Return possible duplicates
- [ ] 7.4 Implement markDuplicates method
  - Tag import rows as duplicates
  - Allow user override
- [ ] 7.5 Write unit tests for deduplication service

## 8. Backend - Import Service

- [ ] 8.1 Create import service
- [ ] 8.2 Implement createImportSession method
  - Create import_history record with status=PENDING
  - Store file_url and metadata
  - Return import session ID
- [ ] 8.3 Implement parseImportFile method
  - Determine file type (CSV, Excel, OFX)
  - Call appropriate parser
  - Return parsed rows
- [ ] 8.4 Implement applyColumnMapping method
  - Map CSV columns to transaction fields
  - Apply date and amount parsing
  - Validate required fields
  - Return mapped transaction objects
- [ ] 8.5 Implement previewImport method
  - Parse and map first 10 rows
  - Run deduplication check
  - Suggest categories based on merchant
  - Return preview with warnings/errors
- [ ] 8.6 Implement executeImport method
  - Update import status to IN_PROGRESS
  - Create transactions in bulk
  - Link via import_transactions table
  - Update wallet balances
  - Update status to COMPLETED
  - Handle errors and rollback if needed
- [ ] 8.7 Implement rollbackImport method
  - Find all transactions linked to import
  - Soft delete each transaction
  - Recalculate wallet balances
  - Update import status to ROLLED_BACK
  - Create audit log entry
- [ ] 8.8 Implement getImportHistory method
  - List all imports for user or family
  - Filter by status, date range
  - Sort by created_at DESC
- [ ] 8.9 Implement exportErrorRows method
  - Generate CSV of rows that failed validation
  - Include error messages
  - Return download URL
- [ ] 8.10 Write unit tests for import service

## 9. Backend - Import Routes

- [ ] 9.1 Create import router
- [ ] 9.2 Create POST /v1/imports/upload endpoint
- [ ] 9.3 Create POST /v1/imports/parse endpoint
  - Accept file_id
  - Return parsed rows and suggested mappings
- [ ] 9.4 Create POST /v1/imports/preview endpoint
  - Accept file_id, column_mapping
  - Return preview of first 10 rows
- [ ] 9.5 Create POST /v1/imports endpoint
  - Accept file_id, column_mapping, wallet_id, scope
  - Execute import
  - Return import_history_id
- [ ] 9.6 Create GET /v1/imports/:id/status endpoint
  - Poll for import progress
  - Return current status and row counts
- [ ] 9.7 Create POST /v1/imports/:id/rollback endpoint
  - Rollback import
  - Return success confirmation
- [ ] 9.8 Create GET /v1/imports/history endpoint
  - List import history
- [ ] 9.9 Create GET /v1/imports/:id/errors endpoint
  - Download error rows CSV
- [ ] 9.10 Create GET /v1/imports/template endpoint
  - Download CSV template
- [ ] 9.11 Write integration tests for import routes

## 10. Frontend - Constants and Types

- [ ] 10.1 Define IMPORT_STATUS constant with labels
- [ ] 10.2 Define FILE_TYPES constant with labels and icons
- [ ] 10.3 Define DATE_FORMATS constant with examples
- [ ] 10.4 Create TypeScript types for ImportSession
- [ ] 10.5 Create TypeScript types for ColumnMapping
- [ ] 10.6 Create TypeScript types for ImportPreview
- [ ] 10.7 Define import-related UI messages

## 11. Frontend - Import API Service

- [ ] 11.1 Create import API service
- [ ] 11.2 Implement uploadFile API call
- [ ] 11.3 Implement parseFile API call
- [ ] 11.4 Implement previewImport API call
- [ ] 11.5 Implement executeImport API call
- [ ] 11.6 Implement getImportStatus API call (for polling)
- [ ] 11.7 Implement rollbackImport API call
- [ ] 11.8 Implement getImportHistory API call
- [ ] 11.9 Implement downloadErrors API call
- [ ] 11.10 Implement downloadTemplate API call

## 12. Frontend - Import Wizard

- [ ] 12.1 Create ImportWizard component (multi-step)
  - Step 1: File upload
  - Step 2: Column mapping
  - Step 3: Preview
  - Step 4: Confirmation
  - Step 5: Results
- [ ] 12.2 Create FileUploadStep component
  - Drag-drop file upload
  - File type validation
  - Show file details (name, size, type)
  - Upload progress indicator
  - Select target wallet
- [ ] 12.3 Create ColumnMappingStep component
  - Display CSV headers
  - Dropdowns to map each column
  - Required fields marked
  - Date format selector
  - Amount format selector
  - Preview sample row
- [ ] 12.4 Create ImportPreviewStep component
  - Table showing first 10 rows
  - Highlight validation errors
  - Show duplicate warnings
  - Show category suggestions
  - Summary: total rows, valid, duplicates, errors
- [ ] 12.5 Create ImportResultsStep component
  - Show import summary (imported, skipped, errors)
  - Link to view imported transactions
  - Link to download error rows
  - Option to rollback import
- [ ] 12.6 Implement wizard navigation (next, back, cancel)
- [ ] 12.7 Write tests for import wizard

## 13. Frontend - Import History Page

- [ ] 13.1 Create ImportHistory component
- [ ] 13.2 Display list of past imports
  - Filename, date, status, row counts
  - Action buttons (view details, rollback)
- [ ] 13.3 Add filter by status
- [ ] 13.4 Add rollback confirmation dialog
- [ ] 13.5 Show rollback success message
- [ ] 13.6 Write tests for import history component

## 14. Frontend - Import Progress

- [ ] 14.1 Create ImportProgress component
  - Progress bar showing rows processed
  - Real-time status updates via polling
  - Cancel import option (if feasible)
- [ ] 14.2 Implement polling mechanism
  - Poll every 2 seconds during import
  - Stop polling when status is COMPLETED or FAILED
- [ ] 14.3 Show error details if import fails

## 15. Frontend - CSV Template Download

- [ ] 15.1 Add "Download Template" button to import wizard
- [ ] 15.2 Generate sample CSV with example data
- [ ] 15.3 Provide bank-specific templates (optional)

## 16. Frontend - Accessibility

- [ ] 16.1 Ensure drag-drop upload has keyboard alternative
- [ ] 16.2 Add ARIA labels for wizard steps
- [ ] 16.3 Support keyboard navigation through mapping controls
- [ ] 16.4 Ensure error messages are announced to screen readers
- [ ] 16.5 Test with screen reader

## 17. Integration Testing

- [ ] 17.1 Write E2E test for CSV import flow (upload → map → preview → import)
- [ ] 17.2 Write E2E test for Excel import
- [ ] 17.3 Write E2E test for OFX import
- [ ] 17.4 Write E2E test for duplicate detection and skip
- [ ] 17.5 Write E2E test for import with validation errors
- [ ] 17.6 Write E2E test for import rollback
- [ ] 17.7 Write E2E test for viewing import history
- [ ] 17.8 Write E2E test for downloading error rows

## 18. Documentation

- [ ] 18.1 Document import API endpoints
- [ ] 18.2 Document supported file formats and column mappings
- [ ] 18.3 Document deduplication algorithm
- [ ] 18.4 Create user guide for CSV import
- [ ] 18.5 Create troubleshooting guide for common import errors

## Dependencies and Parallelization Notes

**Sequential dependencies:**
- Task 1 (Database) must complete first
- Tasks 4-7 (Parser and dedup services) must complete before task 8 (Import service)
- Task 8 (Import service) must complete before task 9 (Routes)
- Task 11 (Frontend API) must complete before tasks 12-15 (Components)

**Can be done in parallel:**
- Tasks 4, 5, 6, 7 (Different parser services) can be built in parallel
- Tasks 12, 13 (Different frontend pages) after API service exists
- Task 3 (File upload) can be done alongside parsers

**Critical path:**
Database → Parser Services → Import Service → Routes → Frontend API → Import Wizard → E2E Tests
