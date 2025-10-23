# Implementation Tasks

## Prerequisites

This change depends on the completion of:
- `add-transaction-tracking` (transaction data for reports)
- `add-categories-budgets` (budget and category data for reports)
- `add-wallet-account-management` (wallet data)

## 1. Database Optimization

- [ ] 1.1 Add indexes for report queries on transactions table (date, category_id, wallet_id, family_id)
- [ ] 1.2 Add indexes for date range queries (composite index on family_id + date)
- [ ] 1.3 Create export_history table
  - Columns: id (UUID), user_id (UUID FK), family_id (UUID FK), export_type (ENUM: PDF, EXCEL, CSV), date_range_start (DATE), date_range_end (DATE), file_url (TEXT), expires_at (TIMESTAMP), created_at
  - Indexes: user_id, family_id, created_at, expires_at
  - Foreign keys: user_id → users(id), family_id → families(id)
- [ ] 1.4 Create database migration scripts

## 2. Backend - Constants and Types

- [ ] 2.1 Define EXPORT_TYPES constant (PDF, EXCEL, CSV)
- [ ] 2.2 Define REPORT_TYPES constant (MONTHLY, CATEGORY_BREAKDOWN, MEMBER_CONTRIBUTIONS, BUDGET_ADHERENCE)
- [ ] 2.3 Create TypeScript types for Report entities
- [ ] 2.4 Create TypeScript types for Export entities
- [ ] 2.5 Define error codes (INVALID_DATE_RANGE, EXPORT_TOO_LARGE)

## 3. Backend - Reporting Service

- [ ] 3.1 Create reporting service
- [ ] 3.2 Implement generateMonthlySummary method
  - Calculate total income, total expenses, net savings
  - Calculate MoM percentage change
  - Identify top spending categories
  - Apply RBAC filtering
- [ ] 3.3 Implement generateCategoryBreakdown method
  - Group transactions by category
  - Calculate totals and percentages
  - Sort by amount DESC
  - Generate pie chart data
- [ ] 3.4 Implement generateMemberContributions method
  - Calculate each member's contributions from splits
  - Include unsplit family transactions
  - Calculate percentages
  - Sort by contribution DESC
- [ ] 3.5 Implement generateBudgetAdherence method
  - Load all budgets for period
  - Calculate usage for each budget
  - Determine on-track vs exceeded status
  - Calculate overall adherence rate
- [ ] 3.6 Implement getDashboardMetrics method
  - Current month spend
  - Top budgets with progress
  - Recent 10 transactions
  - Spending trend data
- [ ] 3.7 Implement date range utilities (YTD, quarterly, custom)
- [ ] 3.8 Implement report caching with TTL
- [ ] 3.9 Write unit tests for reporting service

## 4. Backend - Export Service

- [ ] 4.1 Create export service
- [ ] 4.2 Install PDF generation library (Puppeteer or PDFKit)
- [ ] 4.3 Install Excel generation library (ExcelJS)
- [ ] 4.4 Implement generatePDF method
  - Create HTML template for report
  - Render charts as images or SVG
  - Convert to PDF with proper formatting
  - Return file buffer
- [ ] 4.5 Implement generateExcel method
  - Create workbook with multiple sheets
  - Add transactions sheet with formatting
  - Add summary sheet with formulas
  - Add category and budget sheets
  - Return file buffer
- [ ] 4.6 Implement generateCSV method
  - Convert data to CSV with proper escaping
  - Handle special characters
  - Use UTF-8 encoding
  - Return file buffer
- [ ] 4.7 Implement uploadExportFile method
  - Upload file to object storage
  - Generate presigned download URL (24 hour expiry)
  - Store in export_history
- [ ] 4.8 Implement deleteExpiredExports background job
  - Query exports where expires_at < now
  - Delete files from storage
  - Delete history records
- [ ] 4.9 Implement queueExportJob for large datasets
  - Add to background job queue
  - Process asynchronously
  - Notify user when ready
- [ ] 4.10 Write unit tests for export service

## 5. Backend - PDF Templates

- [ ] 5.1 Create HTML template for monthly report PDF
- [ ] 5.2 Create HTML template for category breakdown PDF
- [ ] 5.3 Create HTML template for member contributions PDF
- [ ] 5.4 Add CSS for print-friendly layout
- [ ] 5.5 Add chart rendering (using Chart.js or similar)
- [ ] 5.6 Add family branding (name, logo placeholder)
- [ ] 5.7 Add page numbers and headers/footers

## 6. Backend - Report Routes

- [ ] 6.1 Create report router
- [ ] 6.2 Create GET /v1/reports/monthly endpoint
- [ ] 6.3 Create GET /v1/reports/category-breakdown endpoint
- [ ] 6.4 Create GET /v1/reports/member-contributions endpoint
- [ ] 6.5 Create GET /v1/reports/budget-adherence endpoint
- [ ] 6.6 Create GET /v1/reports/dashboard endpoint
- [ ] 6.7 Add query params for date ranges, filters
- [ ] 6.8 Write integration tests for report routes

## 7. Backend - Export Routes

- [ ] 7.1 Create export router
- [ ] 7.2 Create POST /v1/exports/pdf endpoint
  - Accept report type and parameters
  - Generate PDF
  - Upload and return download URL
- [ ] 7.3 Create POST /v1/exports/excel endpoint
- [ ] 7.4 Create POST /v1/exports/csv endpoint
- [ ] 7.5 Create GET /v1/exports/history endpoint
  - List user's export history
  - Admin can see all family exports
- [ ] 7.6 Create GET /v1/exports/:id/download endpoint
  - Validate authentication
  - Check expiry
  - Return presigned URL or redirect
- [ ] 7.7 Write integration tests for export routes

## 8. Frontend - Constants and Types

- [ ] 8.1 Define EXPORT_TYPES constant with labels
- [ ] 8.2 Define REPORT_TYPES constant with labels
- [ ] 8.3 Create TypeScript types for Report data
- [ ] 8.4 Create TypeScript types for Dashboard metrics
- [ ] 8.5 Define report and export UI messages

## 9. Frontend - Reporting API Service

- [ ] 9.1 Create reporting API service
- [ ] 9.2 Implement getMonthlySummary API call
- [ ] 9.3 Implement getCategoryBreakdown API call
- [ ] 9.4 Implement getMemberContributions API call
- [ ] 9.5 Implement getBudgetAdherence API call
- [ ] 9.6 Implement getDashboardMetrics API call
- [ ] 9.7 Implement exportPDF API call
- [ ] 9.8 Implement exportExcel API call
- [ ] 9.9 Implement exportCSV API call
- [ ] 9.10 Implement getExportHistory API call

## 10. Frontend - Chart Components

- [ ] 10.1 Install chart library (Recharts or Chart.js)
- [ ] 10.2 Create PieChart component (for category breakdown)
- [ ] 10.3 Create BarChart component (for MoM comparisons)
- [ ] 10.4 Create LineChart component (for spending trends)
- [ ] 10.5 Create ProgressBar component (for budget usage)
- [ ] 10.6 Make all charts responsive and touch-friendly
- [ ] 10.7 Add tooltips and interactivity
- [ ] 10.8 Write tests for chart components

## 11. Frontend - Dashboard Page

- [ ] 11.1 Create Dashboard component
- [ ] 11.2 Add summary cards (current month spend, income, net savings)
- [ ] 11.3 Add budget status section (top budgets with progress bars)
- [ ] 11.4 Add recent transactions list
- [ ] 11.5 Add spending trend chart
- [ ] 11.6 Add quick action buttons (add transaction, pay bill)
- [ ] 11.7 Add family/personal toggle
- [ ] 11.8 Implement responsive grid layout
- [ ] 11.9 Add loading skeletons
- [ ] 11.10 Write tests for Dashboard component

## 12. Frontend - Reports Pages

- [ ] 12.1 Create ReportsPage component (with tabs for different reports)
- [ ] 12.2 Create MonthlySummary tab
  - Show income, expenses, net savings
  - Show MoM comparison with percentage change
  - Show top categories
  - Add date selector for different months
- [ ] 12.3 Create CategoryBreakdown tab
  - Show pie chart
  - Show table with amounts and percentages
  - Add drill-down to transactions
- [ ] 12.4 Create MemberContributions tab
  - Show bar chart or table
  - Show each member's contribution
  - Show percentage of total
- [ ] 12.5 Create BudgetAdherence tab
  - Show budget vs actual for each category
  - Highlight exceeded budgets
  - Show overall adherence rate
  - Add historical view
- [ ] 12.6 Add date range picker (month, quarter, YTD, custom)
- [ ] 12.7 Add export buttons (PDF, Excel, CSV) on each tab
- [ ] 12.8 Write tests for report components

## 13. Frontend - Export Functionality

- [ ] 13.1 Create ExportDialog component
  - Select export type (PDF, Excel, CSV)
  - Select columns (for CSV/Excel)
  - Confirm and trigger export
- [ ] 13.2 Add export button to report pages
- [ ] 13.3 Show loading indicator during export generation
- [ ] 13.4 Handle export completion notification
- [ ] 13.5 Auto-download file or provide download link
- [ ] 13.6 Show export history in settings/profile
- [ ] 13.7 Write tests for export components

## 14. Frontend - Print Styles

- [ ] 14.1 Create print.css stylesheet
- [ ] 14.2 Hide navigation, buttons, and interactive elements for print
- [ ] 14.3 Optimize charts for black and white printing
- [ ] 14.4 Add page break optimizations
- [ ] 14.5 Test print functionality across browsers

## 15. Frontend - Accessibility

- [ ] 15.1 Ensure all charts have text alternatives
- [ ] 15.2 Add ARIA labels for data visualizations
- [ ] 15.3 Support keyboard navigation for report tabs
- [ ] 15.4 Ensure color contrast meets WCAG AA (especially in charts)
- [ ] 15.5 Test with screen reader

## 16. Backend - Report Caching

- [ ] 16.1 Implement caching layer (Redis or in-memory)
- [ ] 16.2 Cache monthly reports for 1 hour
- [ ] 16.3 Cache historical reports for 24 hours
- [ ] 16.4 Invalidate cache on new/updated/deleted transactions
- [ ] 16.5 Invalidate cache on budget changes
- [ ] 16.6 Write tests for cache invalidation

## 17. Integration Testing

- [ ] 17.1 Write E2E test for viewing monthly summary report
- [ ] 17.2 Write E2E test for category breakdown with drill-down
- [ ] 17.3 Write E2E test for member contributions report
- [ ] 17.4 Write E2E test for budget adherence report
- [ ] 17.5 Write E2E test for exporting report as PDF
- [ ] 17.6 Write E2E test for exporting report as Excel
- [ ] 17.7 Write E2E test for exporting report as CSV
- [ ] 17.8 Write E2E test for print functionality
- [ ] 17.9 Write E2E test for dashboard metrics

## 18. Documentation

- [ ] 18.1 Document reporting API endpoints
- [ ] 18.2 Document export API endpoints
- [ ] 18.3 Document report calculation methodologies
- [ ] 18.4 Document export file formats and column definitions
- [ ] 18.5 Add reports and analytics to user guide

## Dependencies and Parallelization Notes

**Sequential dependencies:**
- Task 1 (Database) must complete first
- Tasks 3-4 (Backend services) must complete before tasks 6-7 (routes)
- Task 9 (Frontend API) must complete before tasks 10-13 (components)

**Can be done in parallel:**
- Tasks 3, 4 (Backend services) after database
- Tasks 5 (Templates) can be done alongside task 4 (Export service)
- Tasks 10, 11, 12, 13 (Different frontend sections) after API service
- Task 14 (Print styles) can be done alongside task 12
- Task 16 (Caching) can be added after task 3 is complete

**Critical path:**
Database → Backend Services → Routes → Frontend API → Components → E2E Tests
