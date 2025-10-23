# Budget Management Specification

## ADDED Requirements

### Requirement: Budget Creation

The system SHALL allow Admin and SuperAdmin to create budgets for categories.

#### Scenario: Create monthly budget
- **GIVEN** a user with Admin role
- **WHEN** creating a budget for category with amount=10000, period=MONTHLY
- **THEN** the budget is created
- **AND** start_date and end_date are set for the current month
- **AND** alert_thresholds default to [50, 80, 100]

#### Scenario: Create quarterly budget
- **GIVEN** a user creates a budget with period=QUARTERLY
- **WHEN** the budget is created
- **THEN** end_date is 3 months after start_date
- **AND** the budget tracks spending over the quarter

#### Scenario: Create yearly budget
- **GIVEN** a user creates a budget with period=YEARLY
- **WHEN** the budget is created
- **THEN** end_date is 1 year after start_date
- **AND** the budget aligns with the family's fiscal year if specified

#### Scenario: Member cannot create budgets
- **GIVEN** a user with Member role
- **WHEN** attempting to create a budget
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

#### Scenario: Budget requires category and amount
- **GIVEN** a user creates a budget without category_id or amount
- **WHEN** the creation request is submitted
- **THEN** the request is rejected with error code `MISSING_REQUIRED_FIELDS`

#### Scenario: One active budget per category per period
- **GIVEN** a category already has an active monthly budget
- **WHEN** creating another monthly budget for the same category with overlapping dates
- **THEN** the request is rejected with error code `BUDGET_ALREADY_EXISTS`
- **AND** the user must update the existing budget instead

### Requirement: Budget Update

The system SHALL allow Admin and SuperAdmin to update budget details.

#### Scenario: Update budget amount
- **GIVEN** an Admin updates a budget
- **WHEN** changing the amount
- **THEN** the budget is updated
- **AND** current utilization percentage is recalculated
- **AND** an audit log entry is created

#### Scenario: Update alert thresholds
- **GIVEN** a budget with default thresholds [50, 80, 100]
- **WHEN** changing thresholds to [60, 90, 100]
- **THEN** the budget is updated
- **AND** future alerts use the new thresholds

#### Scenario: Toggle rollover option
- **GIVEN** a budget with rollover_enabled=false
- **WHEN** setting rollover_enabled=true
- **THEN** unused budget at period end will carry forward
- **AND** next period budget = original amount + unused

#### Scenario: Cannot change category or period
- **GIVEN** a user attempts to update a budget
- **WHEN** trying to change category_id or period
- **THEN** the request is rejected with error code `IMMUTABLE_FIELD`
- **AND** the user must delete and create new budget if needed

### Requirement: Budget Deletion

The system SHALL support soft deletion of budgets.

#### Scenario: Delete budget
- **GIVEN** an Admin deletes a budget
- **WHEN** the deletion is confirmed
- **THEN** the budget is soft-deleted with deleted_at timestamp
- **AND** no future alerts are sent
- **AND** historical data is preserved for reporting

#### Scenario: Member cannot delete budgets
- **GIVEN** a user with Member role
- **WHEN** attempting to delete a budget
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

### Requirement: Budget Usage Tracking

The system SHALL track budget utilization based on transactions.

#### Scenario: Calculate budget usage
- **GIVEN** a budget of 10,000 for Groceries in October
- **WHEN** transactions totaling 7,500 are created in that category
- **THEN** budget usage is 75%
- **AND** remaining budget is 2,500

#### Scenario: Only expense transactions count
- **GIVEN** a budget tracks spending
- **WHEN** calculating usage
- **THEN** only transactions with type=EXPENSE are included
- **AND** income transactions do not affect budget usage

#### Scenario: Usage includes date range filter
- **GIVEN** a monthly budget for October
- **WHEN** calculating usage
- **THEN** only transactions with date between start_date and end_date are counted

#### Scenario: Deleted transactions excluded
- **GIVEN** a transaction is soft-deleted
- **WHEN** recalculating budget usage
- **THEN** the deleted transaction is excluded
- **AND** budget usage decreases accordingly

### Requirement: Budget Threshold Alerts

The system SHALL send alerts when spending crosses threshold percentages.

#### Scenario: 50% threshold alert
- **GIVEN** a budget with alert_thresholds=[50, 80, 100]
- **WHEN** spending reaches 50% of budget
- **THEN** an alert is triggered
- **AND** Admins and budget creator are notified via email
- **AND** the alert is logged in budget_alerts table

#### Scenario: 80% threshold alert
- **GIVEN** existing spending of 75%
- **WHEN** a new transaction increases spending to 82%
- **THEN** an 80% alert is triggered
- **AND** notification email is sent

#### Scenario: 100% threshold alert
- **GIVEN** spending reaches or exceeds 100% of budget
- **WHEN** the threshold is crossed
- **THEN** a budget exceeded alert is sent
- **AND** the alert is marked as critical

#### Scenario: Each threshold alerts once per period
- **GIVEN** 50% alert was already sent this month
- **WHEN** spending is still above 50% but below 80%
- **THEN** no additional 50% alert is sent
- **AND** the system tracks which thresholds have been triggered

#### Scenario: Reset alerts on new period
- **GIVEN** a new budget period starts
- **WHEN** the period rolls over
- **THEN** all alert triggers are reset
- **AND** alerts can be sent again for the new period

#### Scenario: Alert respects user notification preferences
- **GIVEN** a user has disabled budget alert emails
- **WHEN** a threshold is crossed
- **THEN** only in-app notification is shown
- **AND** email is not sent

### Requirement: Budget Rollover

The system SHALL support carrying forward unused budget to the next period.

#### Scenario: Rollover enabled with unused budget
- **GIVEN** a monthly budget of 10,000 with rollover_enabled=true
- **WHEN** only 8,000 is spent and the period ends
- **THEN** next month's budget becomes 12,000 (10,000 + 2,000 unused)
- **AND** the new budget period is created automatically

#### Scenario: Rollover disabled resets budget
- **GIVEN** a monthly budget with rollover_enabled=false
- **WHEN** the period ends
- **THEN** next month's budget is 10,000 (original amount)
- **AND** unused amount is not carried forward

#### Scenario: Overspending with rollover
- **GIVEN** a budget of 10,000 with 12,000 spent and rollover enabled
- **WHEN** the period ends
- **THEN** next month's budget is 8,000 (10,000 - 2,000 overspent)
- **AND** overspending reduces next period budget

#### Scenario: Cannot rollover below zero
- **GIVEN** severe overspending that would result in negative budget
- **WHEN** calculating rollover
- **THEN** next period budget is set to 0
- **AND** Admins are notified of budget depletion

### Requirement: Budget Period Management

The system SHALL automatically manage budget periods and transitions.

#### Scenario: Monthly period auto-advance
- **GIVEN** a monthly budget with end_date=2025-10-31
- **WHEN** the background job runs on 2025-11-01
- **THEN** a new budget period is created for November
- **AND** the amount is based on rollover setting

#### Scenario: Quarterly period advance
- **GIVEN** a quarterly budget ending on 2025-12-31
- **WHEN** the period ends
- **THEN** the next quarter budget (Jan-Mar) is created
- **AND** start_date is 2026-01-01

#### Scenario: Yearly period advance
- **GIVEN** a yearly budget aligned with fiscal year
- **WHEN** the fiscal year ends
- **THEN** the new fiscal year budget is created
- **AND** fiscal_year_start from family settings is used

#### Scenario: Inactive budget not renewed
- **GIVEN** a budget marked as inactive
- **WHEN** the period ends
- **THEN** no new period is created
- **AND** the budget expires

### Requirement: Budget Listing

The system SHALL allow users to view budgets based on permissions.

#### Scenario: List all active budgets
- **GIVEN** a family member requests budget list
- **WHEN** the request is processed
- **THEN** all active budgets for the current period are returned
- **AND** each budget includes usage percentage and remaining amount

#### Scenario: Filter by category
- **GIVEN** multiple budgets exist
- **WHEN** filtering by category_id
- **THEN** only budgets for that category are returned

#### Scenario: Filter by period
- **GIVEN** budgets for monthly, quarterly, and yearly periods
- **WHEN** filtering by period=MONTHLY
- **THEN** only monthly budgets are returned

#### Scenario: Member can view all budgets
- **GIVEN** a user with Member role
- **WHEN** requesting budget list
- **THEN** all family budgets are visible
- **AND** Members need visibility for spending awareness

#### Scenario: Include historical budgets
- **GIVEN** an Admin requests budgets with include_past=true
- **WHEN** the request is processed
- **THEN** past period budgets are included
- **AND** budgets are sorted by start_date DESC

### Requirement: Budget Details

The system SHALL provide detailed budget information with spending breakdown.

#### Scenario: View budget details
- **GIVEN** a user views a specific budget
- **WHEN** requesting budget details
- **THEN** amount, period, usage, thresholds, and rollover setting are shown
- **AND** recent transactions in the category are listed

#### Scenario: Budget vs actual chart
- **GIVEN** a budget detail view
- **WHEN** rendering the budget visualization
- **THEN** a chart shows budget amount vs actual spending
- **AND** daily/weekly spending trends are shown

#### Scenario: Alert history
- **GIVEN** a budget has triggered alerts
- **WHEN** viewing budget details
- **THEN** all alert history is shown with timestamps
- **AND** which thresholds were triggered and when

### Requirement: Budget Reporting

The system SHALL provide budget adherence reports.

#### Scenario: Overall budget adherence
- **GIVEN** a family has multiple budgets
- **WHEN** generating budget adherence report
- **THEN** percentage of budgets within limit is shown
- **AND** categories with overspending are highlighted

#### Scenario: Month-over-month budget comparison
- **GIVEN** historical budget data
- **WHEN** generating MoM report
- **THEN** budget utilization is compared across months
- **AND** trends show improving or worsening adherence

#### Scenario: Budget performance metrics
- **GIVEN** a budget report is requested
- **WHEN** calculating metrics
- **THEN** metrics include: average utilization, number of exceeded budgets, total saved
- **AND** metrics are calculated for the requested time period
