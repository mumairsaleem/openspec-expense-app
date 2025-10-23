# Add CSV Import for Transactions

## Why

Users often have existing transaction history in bank statements (CSV, Excel, OFX formats). Manually entering hundreds of transactions is time-consuming and error-prone. CSV import allows users to quickly populate their transaction history, making HEIT immediately useful with historical data. Smart deduplication prevents duplicate entries when re-importing.

## What Changes

- Add CSV/Excel file upload for transaction import
- Add OFX (Quicken format) support for investment accounts
- Add column mapping wizard for flexible CSV formats
- Add smart deduplication based on date, amount, and merchant
- Add import preview before committing
- Add import history with rollback capability
- Add support for common bank statement date formats
- Add RBAC enforcement for import operations

## What Changes

- Add CSV/Excel file upload for transaction import
- Add OFX (Quicken format) support for investment accounts
- Add column mapping wizard for flexible CSV formats
- Add smart deduplication based on date, amount, and merchant
- Add import preview before committing
- Add import history with rollback capability
- Add support for common bank statement date formats
- Add RBAC enforcement for import operations

## Impact

**New Capabilities:**
- `transaction-import`: CSV/Excel/OFX import, column mapping, deduplication, import history

**Affected Code:**
- New: Backend import routes (`/v1/imports/*`)
- New: CSV parsing service
- New: OFX parsing service
- New: Deduplication service
- New: Database schema (import_history table, import_transactions table)
- New: Frontend import wizard pages
- Modified: Transaction service (bulk create capability)

**Affected Specs:**
- Builds on: `transaction-management` (creates transactions)
- Builds on: `wallet-management` (associates with wallets)
- Builds on: `category-management` (maps to categories)
- Builds on: `membership-rbac` (permission enforcement)

**Database Changes:**
- New table: import_history (id, family_id, user_id, filename, file_url, total_rows, imported_rows, skipped_rows, status, error_details, rollback_id, created_at)
- New table: import_transactions (id, import_history_id, transaction_id, source_row, created_at)

**External Dependencies:**
- CSV parsing library (csv-parser or papaparse)
- Excel parsing library (xlsx or exceljs)
- OFX parsing library (ofx-js or similar)
- File upload storage

**Breaking Changes:**
- None (new feature)
