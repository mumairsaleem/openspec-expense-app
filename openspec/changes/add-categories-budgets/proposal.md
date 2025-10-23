# Add Categories and Budget Management

## Why

Users need to organize transactions into categories and set spending limits to control expenses. Categories provide structure for reporting and analysis, while budgets help families stay within spending limits with proactive alerts. This feature is essential for financial discipline and awareness.

## What Changes

- Add default category creation on family setup (already specified in family-tenancy)
- Add custom category creation with icons and colors
- Add budget creation with monthly/quarterly/yearly periods
- Add budget threshold alerts at 50%, 80%, and 100%
- Add budget rollover options (carry forward unused budget or reset)
- Add budget usage tracking and reporting
- Add RBAC enforcement for category and budget management

## Impact

**New Capabilities:**
- `category-management`: Category CRUD, custom categories with icons/colors
- `budget-management`: Budget CRUD, threshold alerts, usage tracking, rollover

**Affected Code:**
- Modified: Family creation service (default categories already specified, now fully implemented)
- New: Backend category routes (`/v1/categories/*`)
- New: Backend budget routes (`/v1/budgets/*`)
- New: Database schema (budgets table, budget_alerts table)
- New: Frontend category management pages
- New: Frontend budget management pages
- New: Budget alert notification system

**Affected Specs:**
- Builds on: `family-tenancy` (default categories on family creation)
- Builds on: `transaction-management` (transactions use categories)
- Builds on: `membership-rbac` (permission enforcement)

**Database Changes:**
- Categories table already exists (created with family)
- New table: budgets (id, family_id, category_id, amount, period, start_date, end_date, rollover_enabled, alert_thresholds, created_at, updated_at, deleted_at)
- New table: budget_alerts (id, budget_id, threshold, triggered_at, notified_users)

**External Dependencies:**
- Email service for budget alert notifications
- Background job scheduler for budget period rollover

**Breaking Changes:**
- None (new feature)
