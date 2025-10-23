# Category Management Specification

## ADDED Requirements

### Requirement: Default Categories

The system SHALL create default expense categories when a family is created.

#### Scenario: Default categories on family creation
- **GIVEN** a new family is created
- **WHEN** the family creation completes
- **THEN** default categories are created: Housing, Food, Transport, Utilities, Healthcare, Education, Entertainment, Others
- **AND** each category has a predefined icon and color
- **AND** categories are associated with the family

### Requirement: Custom Category Creation

The system SHALL allow Admin and SuperAdmin to create custom categories.

#### Scenario: Admin creates custom category
- **GIVEN** a user with Admin role
- **WHEN** creating a category with name, icon, and color
- **THEN** the category is created for the family
- **AND** all family members can use it for transactions

#### Scenario: Member cannot create categories
- **GIVEN** a user with Member role
- **WHEN** attempting to create a category
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

#### Scenario: Category name must be unique per family
- **GIVEN** a family with existing category "Groceries"
- **WHEN** creating another category named "Groceries"
- **THEN** the request is rejected with error code `CATEGORY_NAME_EXISTS`
- **AND** the user is prompted to use a different name

#### Scenario: Category requires name
- **GIVEN** a user creates a category without a name
- **WHEN** the creation request is submitted
- **THEN** the request is rejected with error code `MISSING_REQUIRED_FIELDS`

### Requirement: Category Update

The system SHALL allow Admin and SuperAdmin to update category details.

#### Scenario: Update category name, icon, or color
- **GIVEN** an Admin updates a category
- **WHEN** changing name, icon, or color
- **THEN** the category is updated
- **AND** all transactions using this category reflect the new details
- **AND** an audit log entry is created

#### Scenario: Cannot update default category names
- **GIVEN** a default category like "Housing"
- **WHEN** attempting to rename it
- **THEN** the name can be updated (no restriction)
- **AND** the category behaves like any custom category

#### Scenario: Member cannot update categories
- **GIVEN** a user with Member role
- **WHEN** attempting to update a category
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

### Requirement: Category Deletion

The system SHALL support soft deletion of categories with transaction reassignment.

#### Scenario: Delete category with no transactions
- **GIVEN** a category with zero associated transactions
- **WHEN** an Admin deletes the category
- **THEN** the category is soft-deleted
- **AND** it is hidden from category lists

#### Scenario: Delete category with existing transactions
- **GIVEN** a category with existing transactions
- **WHEN** attempting to delete the category
- **THEN** the user is prompted to reassign transactions to another category
- **AND** after reassignment, the category is soft-deleted

#### Scenario: Cannot delete if not reassigned
- **GIVEN** a category with transactions
- **WHEN** deleting without providing replacement category
- **THEN** the request is rejected with error code `CATEGORY_HAS_TRANSACTIONS`
- **AND** the user must specify a replacement category_id

#### Scenario: "Others" category cannot be deleted
- **GIVEN** the default "Others" category
- **WHEN** attempting to delete it
- **THEN** the request is rejected with error code `CANNOT_DELETE_DEFAULT_CATEGORY`
- **AND** at least one catch-all category must remain

### Requirement: Category Listing

The system SHALL allow all family members to view categories.

#### Scenario: List all active categories
- **GIVEN** a family member requests category list
- **WHEN** the request is processed
- **THEN** all active (non-deleted) categories are returned
- **AND** categories are sorted alphabetically by name

#### Scenario: Include deleted categories for Admin
- **GIVEN** an Admin requests categories with include_deleted=true
- **WHEN** the request is processed
- **THEN** both active and deleted categories are returned
- **AND** deleted categories are marked with deleted_at timestamp

#### Scenario: Guest can view categories
- **GIVEN** a user with Guest role
- **WHEN** requesting category list
- **THEN** all active categories are visible
- **AND** Guest needs categories for viewing transaction details

### Requirement: Category Details

The system SHALL provide category information with usage statistics.

#### Scenario: View category details
- **GIVEN** a user views category details
- **WHEN** requesting a specific category
- **THEN** name, icon, color, and creation date are returned
- **AND** transaction count for the category is included
- **AND** total spent in the category for current month is shown

#### Scenario: Category usage in budgets
- **GIVEN** a category is used in active budgets
- **WHEN** viewing category details
- **THEN** associated budgets are listed
- **AND** current budget utilization is shown

### Requirement: Category Icons and Colors

The system SHALL support customizable icons and colors for visual categorization.

#### Scenario: Select icon from predefined set
- **GIVEN** a user creates or updates a category
- **WHEN** selecting an icon
- **THEN** a predefined set of category icons is available
- **AND** icon names follow a standard format (e.g., "icon-home", "icon-food")

#### Scenario: Select color from palette
- **GIVEN** a user creates or updates a category
- **WHEN** selecting a color
- **THEN** a color picker or predefined palette is provided
- **AND** colors are stored as hex codes (e.g., "#FF5733")

#### Scenario: Default icon and color if not specified
- **GIVEN** a category is created without icon or color
- **WHEN** the category is saved
- **THEN** default icon is "icon-tag" and default color is "#CCCCCC"

### Requirement: Category Suggestions

The system SHALL provide category suggestions based on merchant.

#### Scenario: Suggest category from merchant history
- **GIVEN** a user enters a merchant name when creating a transaction
- **WHEN** the merchant has been used before
- **THEN** the previously used category for that merchant is suggested
- **AND** the suggestion is based on family transaction history

#### Scenario: No suggestion for new merchant
- **GIVEN** a user enters a new merchant name
- **WHEN** no history exists
- **THEN** no category suggestion is made
- **AND** the user selects category manually
