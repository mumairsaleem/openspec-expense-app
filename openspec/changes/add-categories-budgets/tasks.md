# Implementation Tasks

## Prerequisites

This change depends on the completion of:
- `add-auth-multitenancy-foundation` (authentication, families, RBAC)
- `add-transaction-tracking` (transactions use categories for tracking)

Note: Default category creation was specified in `add-auth-multitenancy-foundation` but is fully implemented here.

## 1. Database Schema

- [ ] 1.1 Ensure categories table exists from family creation (already created in add-auth-multitenancy-foundation)
- [ ] 1.2 Create budgets table
  - Columns: id (UUID), family_id (UUID FK), category_id (UUID FK), amount (DECIMAL), period (ENUM: MONTHLY, QUARTERLY, YEARLY), start_date (DATE), end_date (DATE), rollover_enabled (BOOLEAN), alert_thresholds (JSONB array of integers), is_active (BOOLEAN), created_at, updated_at, deleted_at
  - Indexes: family_id, category_id, period, start_date, end_date, is_active
  - Foreign keys: family_id → families(id), category_id → categories(id)
  - Constraint: UNIQUE(family_id, category_id, period, start_date) WHERE deleted_at IS NULL
- [ ] 1.3 Create budget_alerts table
  - Columns: id (UUID), budget_id (UUID FK), threshold (INTEGER), triggered_at (TIMESTAMP), notified_users (JSONB array of user IDs)
  - Indexes: budget_id, triggered_at
  - Foreign keys: budget_id → budgets(id)
- [ ] 1.4 Add row-level security policies for budgets
- [ ] 1.5 Create database migration scripts
- [ ] 1.6 Create default categories seed data (Housing, Food, Transport, Utilities, Healthcare, Education, Entertainment, Others)

## 2. Backend - Constants and Types

- [ ] 2.1 Define BUDGET_PERIODS constant (MONTHLY, QUARTERLY, YEARLY)
- [ ] 2.2 Define DEFAULT_CATEGORIES constant with name, icon, color
- [ ] 2.3 Define DEFAULT_ALERT_THRESHOLDS constant [50, 80, 100]
- [ ] 2.4 Create TypeScript types for Category entity
- [ ] 2.5 Create TypeScript types for Budget entity
- [ ] 2.6 Create TypeScript types for BudgetAlert entity
- [ ] 2.7 Define error codes (CATEGORY_NAME_EXISTS, CATEGORY_HAS_TRANSACTIONS, CANNOT_DELETE_DEFAULT_CATEGORY, BUDGET_ALREADY_EXISTS)
- [ ] 2.8 Create validation schemas for category and budget CRUD

## 3. Backend - Default Category Creation

- [ ] 3.1 Implement createDefaultCategories utility
  - Create all default categories with predefined icons and colors
  - Associate with family_id
  - Call during family creation workflow
- [ ] 3.2 Update family creation service to call createDefaultCategories
- [ ] 3.3 Write tests for default category creation

## 4. Backend - Category Service

- [ ] 4.1 Create category service with CRUD methods
- [ ] 4.2 Implement createCategory method
  - Validate uniqueness of category name within family
  - Set default icon and color if not provided
  - Create audit log entry
- [ ] 4.3 Implement updateCategory method
  - Validate permissions (Admin/SuperAdmin only)
  - Allow updates to name, icon, color
  - Create audit log entry
- [ ] 4.4 Implement deleteCategory method (soft delete)
  - Check for associated transactions
  - Require reassignment category_id if transactions exist
  - Prevent deletion of "Others" category
  - Reassign transactions to new category
  - Soft delete category
- [ ] 4.5 Implement getCategories method
  - Filter by family_id
  - Include deleted categories if requested by Admin
  - Sort alphabetically by name
- [ ] 4.6 Implement getCategoryById method
  - Return category details
  - Include transaction count
  - Include total spent in current month
  - Include associated budgets
- [ ] 4.7 Implement getCategorySuggestion method
  - Query transaction history for merchant
  - Return most frequently used category for that merchant
- [ ] 4.8 Write unit tests for category service

## 5. Backend - Budget Service

- [ ] 5.1 Create budget service with CRUD methods
- [ ] 5.2 Implement createBudget method
  - Validate category exists and belongs to family
  - Calculate start_date and end_date based on period
  - Set default alert_thresholds if not provided
  - Check for existing active budget for same category/period
  - Create budget
- [ ] 5.3 Implement updateBudget method
  - Validate permissions (Admin/SuperAdmin only)
  - Allow updates to amount, alert_thresholds, rollover_enabled
  - Prevent changes to category_id and period
  - Create audit log entry
- [ ] 5.4 Implement deleteBudget method (soft delete)
  - Validate permissions
  - Soft delete budget
  - Create audit log entry
- [ ] 5.5 Implement getBudgets method
  - Filter by family_id, category_id, period
  - Include/exclude past budgets based on parameter
  - Return with current usage percentage and remaining amount
  - Sort by start_date DESC
- [ ] 5.6 Implement getBudgetById method
  - Return budget details
  - Calculate current usage
  - Include recent transactions in category
  - Include alert history
- [ ] 5.7 Implement calculateBudgetUsage method
  - Query expense transactions in category within date range
  - Exclude deleted transactions
  - Calculate total spent and percentage
  - Return usage data
- [ ] 5.8 Implement checkThresholds method
  - Calculate current usage percentage
  - Check against alert_thresholds
  - Return list of newly crossed thresholds
- [ ] 5.9 Write unit tests for budget service

## 6. Backend - Budget Alert Service

- [ ] 6.1 Create budget alert service
- [ ] 6.2 Implement triggerAlert method
  - Create budget_alert record
  - Send email notifications to Admins and creator
  - Respect user notification preferences
  - Mark threshold as triggered for this period
- [ ] 6.3 Implement checkAlertNeeded method
  - Query budget_alerts for current period
  - Return true if threshold not yet triggered
  - Return false if already alerted
- [ ] 6.4 Implement sendAlertNotification method
  - Generate email content with budget details
  - Include current usage, remaining amount
  - Mark alert as critical for 100% threshold
  - Send email via email service
- [ ] 6.5 Implement getAlertHistory method
  - Return all alerts for a budget
  - Include threshold and triggered_at
  - Sort by triggered_at DESC
- [ ] 6.6 Write unit tests for alert service

## 7. Backend - Budget Rollover Service

- [ ] 7.1 Create budget rollover service
- [ ] 7.2 Implement calculateRollover method
  - Get current period spending
  - Calculate unused amount (budget - spent)
  - If rollover_enabled, add to next period amount
  - Prevent negative amounts (min 0)
  - Return next period amount
- [ ] 7.3 Implement createNextPeriod method
  - Calculate new start_date and end_date based on period
  - Use rollover calculation for amount
  - Copy alert_thresholds and rollover_enabled
  - Create new budget record
- [ ] 7.4 Implement processBudgetRollover background job
  - Query budgets where end_date = yesterday
  - For each active budget, create next period
  - Handle quarterly and yearly periods
  - Align yearly budgets with fiscal_year_start
- [ ] 7.5 Write unit tests for rollover service

## 8. Backend - Transaction Integration

- [ ] 8.1 Update transaction service to trigger budget checks
  - After creating transaction, check affected category budgets
  - Calculate new budget usage
  - Check for threshold crossings
  - Trigger alerts if thresholds crossed
- [ ] 8.2 Update transaction update to recalculate budgets
  - If category or amount changes, recalculate old and new category budgets
  - Trigger alerts if needed
- [ ] 8.3 Update transaction delete to recalculate budgets
  - Recalculate budget usage after deletion
  - No alerts needed for decreasing usage
- [ ] 8.4 Write integration tests for transaction-budget interaction

## 9. Backend - Category Routes

- [ ] 9.1 Create category router
- [ ] 9.2 Create POST /v1/categories endpoint (Admin/SuperAdmin only)
- [ ] 9.3 Create GET /v1/categories endpoint (all members)
- [ ] 9.4 Create GET /v1/categories/:id endpoint
- [ ] 9.5 Create PUT /v1/categories/:id endpoint (Admin/SuperAdmin only)
- [ ] 9.6 Create DELETE /v1/categories/:id endpoint (Admin/SuperAdmin only)
- [ ] 9.7 Create GET /v1/categories/suggest endpoint (for merchant-based suggestion)
- [ ] 9.8 Write integration tests for category routes

## 10. Backend - Budget Routes

- [ ] 10.1 Create budget router
- [ ] 10.2 Create POST /v1/budgets endpoint (Admin/SuperAdmin only)
- [ ] 10.3 Create GET /v1/budgets endpoint (all members can view)
- [ ] 10.4 Create GET /v1/budgets/:id endpoint
- [ ] 10.5 Create GET /v1/budgets/:id/usage endpoint (detailed usage with transactions)
- [ ] 10.6 Create PUT /v1/budgets/:id endpoint (Admin/SuperAdmin only)
- [ ] 10.7 Create DELETE /v1/budgets/:id endpoint (Admin/SuperAdmin only)
- [ ] 10.8 Create GET /v1/budgets/:id/alerts endpoint (alert history)
- [ ] 10.9 Create GET /v1/reports/budget-adherence endpoint
- [ ] 10.10 Write integration tests for budget routes

## 11. Frontend - Constants and Types

- [ ] 11.1 Create BUDGET_PERIODS constant with labels
- [ ] 11.2 Create CATEGORY_ICONS constant with icon names
- [ ] 11.3 Create COLOR_PALETTE constant for category colors
- [ ] 11.4 Create TypeScript types for Category
- [ ] 11.5 Create TypeScript types for Budget
- [ ] 11.6 Define category and budget UI messages

## 12. Frontend - Category API Service

- [ ] 12.1 Create category API service
- [ ] 12.2 Implement createCategory API call
- [ ] 12.3 Implement getCategories API call
- [ ] 12.4 Implement getCategoryById API call
- [ ] 12.5 Implement updateCategory API call
- [ ] 12.6 Implement deleteCategory API call
- [ ] 12.7 Implement getCategorySuggestion API call

## 13. Frontend - Budget API Service

- [ ] 13.1 Create budget API service
- [ ] 13.2 Implement createBudget API call
- [ ] 13.3 Implement getBudgets API call
- [ ] 13.4 Implement getBudgetById API call
- [ ] 13.5 Implement getBudgetUsage API call
- [ ] 13.6 Implement updateBudget API call
- [ ] 13.7 Implement deleteBudget API call
- [ ] 13.8 Implement getBudgetAlerts API call
- [ ] 13.9 Implement getBudgetAdherence API call

## 14. Frontend - Category Management Pages

- [ ] 14.1 Create CategoryList component
  - Display categories in a grid with icons and colors
  - Show transaction count per category
  - Add "Create Category" button (Admin only)
  - Add click to edit (Admin only)
- [ ] 14.2 Create CategoryForm component (create/edit)
  - Form fields: name, icon selector, color picker
  - Implement form validation
  - Handle submission
- [ ] 14.3 Create IconSelector component
  - Display predefined icon set
  - Allow icon selection
  - Preview selected icon
- [ ] 14.4 Create ColorPicker component
  - Show color palette
  - Support custom hex input
  - Preview selected color
- [ ] 14.5 Create CategoryDeleteModal component
  - Show warning if category has transactions
  - Provide dropdown to select reassignment category
  - Prevent deletion of "Others"
- [ ] 14.6 Write tests for category components

## 15. Frontend - Budget Management Pages

- [ ] 15.1 Create BudgetList component
  - Display budgets with usage bars/charts
  - Show amount, period, usage percentage, remaining
  - Color code based on utilization (green < 50%, yellow 50-80%, red > 80%)
  - Add "Create Budget" button (Admin only)
  - Filter by period and category
- [ ] 15.2 Create BudgetForm component (create/edit)
  - Form fields: category, amount, period, alert_thresholds, rollover_enabled
  - Show period date range preview
  - Implement form validation
  - Handle submission
- [ ] 15.3 Create BudgetDetails component
  - Show budget information
  - Display usage chart (budget vs actual)
  - Show spending trend over period
  - List recent transactions in category
  - Show alert history
  - Add edit and delete buttons (Admin only)
- [ ] 15.4 Create BudgetUsageCard component (reusable)
  - Show category name with icon
  - Progress bar showing usage
  - Amount spent / total amount
  - Remaining amount
- [ ] 15.5 Create BudgetAlertNotification component
  - Display in-app when threshold crossed
  - Show category, usage, remaining
  - Dismiss button
- [ ] 15.6 Write tests for budget components

## 16. Frontend - Dashboard Budget Widgets

- [ ] 16.1 Create BudgetSummaryWidget for dashboard
  - Show top 3-5 budgets with highest utilization
  - Quick view of overall budget health
  - Link to full budget page
- [ ] 16.2 Create BudgetAdherenceChart
  - Pie or donut chart showing budget categories
  - Color coded by utilization level
  - Interactive with click to details
- [ ] 16.3 Write tests for dashboard widgets

## 17. Frontend - Accessibility

- [ ] 17.1 Ensure all forms have proper labels and ARIA attributes
- [ ] 17.2 Support keyboard navigation for category and budget management
- [ ] 17.3 Add screen reader announcements for budget alerts
- [ ] 17.4 Ensure color contrast meets WCAG AA (especially for category colors)
- [ ] 17.5 Test with screen reader

## 18. Integration Testing

- [ ] 18.1 Write E2E test for creating custom category
- [ ] 18.2 Write E2E test for creating monthly budget
- [ ] 18.3 Write E2E test for budget threshold alert (50%, 80%, 100%)
- [ ] 18.4 Write E2E test for budget rollover at period end
- [ ] 18.5 Write E2E test for category deletion with reassignment
- [ ] 18.6 Write E2E test for budget adherence report
- [ ] 18.7 Write E2E test for transaction triggering budget alert

## 19. Documentation

- [ ] 19.1 Document category API endpoints
- [ ] 19.2 Document budget API endpoints
- [ ] 19.3 Document budget alert thresholds and notification logic
- [ ] 19.4 Document budget rollover calculations
- [ ] 19.5 Add category and budget management to user guide

## Dependencies and Parallelization Notes

**Sequential dependencies:**
- Task 1 (Database) must complete first
- Task 3 (Default categories) must complete before task 9 (routes can use them)
- Tasks 4-7 (Backend services) must complete before tasks 9-10 (routes)
- Task 8 (Transaction integration) requires both transaction service and budget service
- Tasks 12-13 (Frontend API) must complete before tasks 14-16 (components)

**Can be done in parallel:**
- Tasks 4, 5, 6, 7 (Different backend services) after database is ready
- Tasks 9, 10 (Different route groups) after services are done
- Tasks 14, 15, 16 (Different frontend pages) after API service exists
- Task 17 (Accessibility) can be done alongside frontend work

**Critical path:**
Database → Backend Services → Routes → Frontend API → Frontend Components → E2E Tests
