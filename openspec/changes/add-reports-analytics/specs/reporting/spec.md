# Reporting Specification

## ADDED Requirements

### Requirement: Monthly Summary Report

The system SHALL generate monthly spending summaries with trends.

#### Scenario: Generate current month summary
- **GIVEN** a family member requests monthly summary
- **WHEN** no month is specified
- **THEN** current month summary is generated
- **AND** includes total income, total expenses, net savings
- **AND** includes top spending categories
- **AND** includes MoM comparison with previous month

#### Scenario: MoM percentage change
- **GIVEN** previous month expenses were 50,000 and current month is 60,000
- **WHEN** generating monthly summary
- **THEN** MoM change is +20%
- **AND** increase is highlighted in red
- **AND** decrease would be shown in green

#### Scenario: First month has no MoM comparison
- **GIVEN** a family's first month of data
- **WHEN** generating monthly summary
- **THEN** MoM fields are null or show "N/A"
- **AND** no comparison percentage is calculated

#### Scenario: Member sees personal and family data
- **GIVEN** a user with Member role requests monthly summary
- **WHEN** the report is generated
- **THEN** family transactions are included
- **AND** their personal transactions are included
- **AND** other members' personal transactions are excluded

#### Scenario: Admin sees all data
- **GIVEN** a user with Admin role requests monthly summary
- **WHEN** the report is generated
- **THEN** all family transactions are included
- **AND** all personal transactions from all members are included

### Requirement: Category Breakdown Report

The system SHALL provide spending breakdown by category.

#### Scenario: Generate category breakdown for month
- **GIVEN** transactions across multiple categories in October
- **WHEN** generating category breakdown
- **THEN** each category shows total spent and percentage of total
- **AND** categories are sorted by amount DESC
- **AND** pie chart data is included

#### Scenario: Top categories highlighted
- **GIVEN** a category breakdown report
- **WHEN** identifying top spending areas
- **THEN** the top 5 categories are highlighted
- **AND** all other categories are grouped into "Others"

#### Scenario: Empty categories excluded
- **GIVEN** categories with zero transactions
- **WHEN** generating breakdown
- **THEN** categories with zero spending are excluded
- **AND** only categories with transactions are shown

#### Scenario: Income vs Expense categories
- **GIVEN** both income and expense transactions exist
- **WHEN** generating category breakdown
- **THEN** expense categories and income categories are shown separately
- **AND** each group has its own pie chart

### Requirement: Member Contribution Report

The system SHALL generate reports showing who paid for shared expenses.

#### Scenario: Calculate member contributions
- **GIVEN** split transactions among family members
- **WHEN** generating member contribution report for a month
- **THEN** each member's total contribution is calculated
- **AND** percentage of total family spending is shown
- **AND** member names are sorted by contribution amount DESC

#### Scenario: Include non-split transactions
- **GIVEN** a family transaction without splits
- **WHEN** calculating member contributions
- **THEN** the creator is credited with 100% of that transaction
- **AND** unsplit transactions are included in totals

#### Scenario: Personal transactions excluded from contributions
- **GIVEN** a member has personal and family transactions
- **WHEN** generating contribution report
- **THEN** only family-scoped transactions are included
- **AND** personal transactions do not count toward contributions

#### Scenario: Zero contribution members shown
- **GIVEN** a family member with no transactions in the period
- **WHEN** generating contribution report
- **THEN** the member is listed with 0 contribution
- **AND** all members are visible for transparency

### Requirement: Custom Date Range Reports

The system SHALL allow reports for custom date ranges.

#### Scenario: Generate report for date range
- **GIVEN** a user specifies start_date=2025-01-01 and end_date=2025-03-31
- **WHEN** generating the report
- **THEN** only transactions within the date range are included
- **AND** the report title shows "Q1 2025" or custom date range

#### Scenario: Year-to-date report
- **GIVEN** a user requests YTD report
- **WHEN** the report is generated
- **THEN** start_date is set to family's fiscal_year_start
- **AND** end_date is set to today
- **AND** all transactions from fiscal year start to now are included

#### Scenario: Quarterly report
- **GIVEN** a user selects Q2 2025
- **WHEN** generating the report
- **THEN** transactions from April 1 to June 30, 2025 are included

#### Scenario: Invalid date range
- **GIVEN** end_date is before start_date
- **WHEN** attempting to generate report
- **THEN** the request is rejected with error code `INVALID_DATE_RANGE`

### Requirement: Budget Adherence Report

The system SHALL provide budget vs actual reports.

#### Scenario: Budget adherence summary
- **GIVEN** multiple budgets for a month
- **WHEN** generating budget adherence report
- **THEN** each budget shows amount, spent, percentage, and status (on track/exceeded)
- **AND** overall adherence rate is calculated
- **AND** categories over budget are highlighted in red

#### Scenario: Categories without budgets
- **GIVEN** a category has transactions but no budget
- **WHEN** generating adherence report
- **THEN** the category is listed separately as "No budget set"
- **AND** spending amount is shown for awareness

#### Scenario: Historical budget adherence
- **GIVEN** a user requests budget adherence for last 6 months
- **WHEN** the report is generated
- **THEN** each month shows adherence percentage
- **AND** trends show improving or worsening budget discipline

### Requirement: Dashboard Metrics

The system SHALL provide real-time dashboard metrics.

#### Scenario: Dashboard shows current month metrics
- **GIVEN** a user views the dashboard
- **WHEN** the page loads
- **THEN** current month spend, budget status, and recent transactions are shown
- **AND** quick action buttons for adding transactions are visible

#### Scenario: Budget status cards
- **GIVEN** active budgets exist
- **WHEN** displaying dashboard
- **THEN** top 3-5 budgets are shown with progress bars
- **AND** color coding indicates health (green/yellow/red)

#### Scenario: Recent transactions list
- **GIVEN** recent transactions exist
- **WHEN** loading dashboard
- **THEN** the 10 most recent transactions are displayed
- **AND** clicking a transaction navigates to details

#### Scenario: Spending trend chart
- **GIVEN** historical transaction data
- **WHEN** rendering dashboard
- **THEN** a line or bar chart shows daily/weekly spending trend for current month
- **AND** the chart is interactive with tooltips

### Requirement: Report Filtering and Drill-Down

The system SHALL support filtering and drill-down in reports.

#### Scenario: Filter by category
- **GIVEN** a monthly report
- **WHEN** clicking on a category in the breakdown
- **THEN** detailed transactions for that category are shown
- **AND** user can drill down to transaction details

#### Scenario: Filter by wallet
- **GIVEN** a spending report
- **WHEN** filtering by a specific wallet
- **THEN** only transactions from that wallet are included in calculations

#### Scenario: Filter by member
- **GIVEN** an Admin views member contribution report
- **WHEN** clicking on a member
- **THEN** all transactions by that member are shown
- **AND** splits they participated in are highlighted

### Requirement: Report Permissions

The system SHALL enforce RBAC for report access.

#### Scenario: Member generates own reports
- **GIVEN** a user with Member role
- **WHEN** generating reports
- **THEN** family transactions and own personal transactions are included
- **AND** other members' personal transactions are excluded

#### Scenario: Admin generates family-wide reports
- **GIVEN** a user with Admin role
- **WHEN** generating reports
- **THEN** all family and personal transactions are included
- **AND** full visibility into family finances

#### Scenario: Guest cannot generate reports
- **GIVEN** a user with Guest role
- **WHEN** attempting to generate reports
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`
- **AND** Guest role is read-only for assigned data only

#### Scenario: Auditor generates read-only reports
- **GIVEN** a user with Auditor role within validity period
- **WHEN** generating reports
- **THEN** all family data is included
- **AND** private_notes are excluded
- **AND** export capabilities are available

### Requirement: Report Caching

The system SHALL cache frequently accessed reports for performance.

#### Scenario: Cache monthly reports
- **GIVEN** a monthly report is generated
- **WHEN** the same report is requested again within 1 hour
- **THEN** the cached version is returned
- **AND** response time is under 100ms

#### Scenario: Invalidate cache on new transaction
- **GIVEN** a cached current month report
- **WHEN** a new transaction is created
- **THEN** the cache for current month is invalidated
- **AND** next request generates fresh data

#### Scenario: Cache historical reports longer
- **GIVEN** a report for a past closed month
- **WHEN** the report is generated
- **THEN** it is cached for 24 hours
- **AND** historical data rarely changes
