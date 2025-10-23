# Family Tenancy Specification

## ADDED Requirements

### Requirement: Family Creation

The system SHALL allow authenticated users to create a new family (tenant) and become its SuperAdmin.

#### Scenario: Successful family creation
- **GIVEN** an authenticated user creates a family
- **WHEN** providing name, currency, timezone, and fiscal year start
- **THEN** a new family is created with a unique UUID
- **AND** the user is assigned as the family SuperAdmin
- **AND** a membership record links the user to the family
- **AND** default categories are created for the family

#### Scenario: Multiple families per user
- **GIVEN** a user already belongs to one family
- **WHEN** they create another family
- **THEN** both families exist independently
- **AND** the user has separate roles in each family
- **AND** data is isolated between families

#### Scenario: Missing required fields
- **GIVEN** a user attempts to create a family without required fields
- **WHEN** the creation request is submitted
- **THEN** the request is rejected with error code `MISSING_REQUIRED_FIELDS`
- **AND** the response indicates which fields are missing

### Requirement: Family Settings Management

The system SHALL allow SuperAdmin to update family settings.

#### Scenario: Update family settings
- **GIVEN** a SuperAdmin of a family
- **WHEN** updating name, currency, timezone, or fiscal year start
- **THEN** the family settings are updated
- **AND** an audit log entry is created
- **AND** all family members see the updated settings

#### Scenario: Non-SuperAdmin cannot update settings
- **GIVEN** a user with Admin or Member role
- **WHEN** attempting to update family settings
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`
- **AND** no changes are made

#### Scenario: Invalid currency code
- **GIVEN** a SuperAdmin updates family currency
- **WHEN** providing an invalid ISO currency code
- **THEN** the request is rejected with error code `INVALID_CURRENCY`
- **AND** the family settings remain unchanged

### Requirement: Family Deletion

The system SHALL allow SuperAdmin to delete their family with a 30-day soft delete window.

#### Scenario: Soft delete family
- **GIVEN** a SuperAdmin requests family deletion
- **WHEN** the deletion is confirmed
- **THEN** the family is marked as deleted with deleted_at timestamp
- **AND** all family members lose access immediately
- **AND** the family can be restored within 30 days
- **AND** an audit log entry is created

#### Scenario: Permanent deletion after 30 days
- **GIVEN** a family has been soft-deleted for 30 days
- **WHEN** the background job runs
- **THEN** the family and all related data are permanently deleted
- **AND** the deletion cannot be undone

#### Scenario: Family restoration
- **GIVEN** a family was soft-deleted within the last 30 days
- **WHEN** the SuperAdmin requests restoration
- **THEN** the deleted_at timestamp is cleared
- **AND** all family members regain access
- **AND** all data is restored

#### Scenario: Non-SuperAdmin cannot delete family
- **GIVEN** a user with Admin or Member role
- **WHEN** attempting to delete the family
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

### Requirement: Family Context Extraction

The system SHALL provide middleware to extract and validate family context from requests.

#### Scenario: Valid family context
- **GIVEN** a request includes family_id in the URL or body
- **WHEN** the middleware processes the request
- **THEN** the family is retrieved from the database
- **AND** the family_id is attached to the request context
- **AND** the request proceeds to the route handler

#### Scenario: User not a member of family
- **GIVEN** a user requests access to a family they don't belong to
- **WHEN** the middleware validates membership
- **THEN** the request is rejected with error code `NOT_FAMILY_MEMBER`
- **AND** status 403 is returned

#### Scenario: Deleted family access attempt
- **GIVEN** a user tries to access a soft-deleted family
- **WHEN** the middleware processes the request
- **THEN** the request is rejected with error code `FAMILY_DELETED`
- **AND** status 404 is returned

### Requirement: Family List

The system SHALL allow users to list all families they belong to.

#### Scenario: List user's families
- **GIVEN** a user belongs to multiple families
- **WHEN** requesting their family list
- **THEN** all families where they have active membership are returned
- **AND** each family includes name, currency, timezone, and user's role
- **AND** deleted families are excluded

#### Scenario: No families
- **GIVEN** a user has no family memberships
- **WHEN** requesting their family list
- **THEN** an empty array is returned

### Requirement: Family Details

The system SHALL allow family members to view family details based on their role.

#### Scenario: View family details
- **GIVEN** a family member requests family details
- **WHEN** the request is authenticated and authorized
- **THEN** family settings are returned (name, currency, timezone, fiscal_year_start)
- **AND** SuperAdmin can see all settings including data retention policy
- **AND** other roles see public settings only

#### Scenario: Guest sees limited details
- **GIVEN** a user with Guest role views family details
- **WHEN** the request is processed
- **THEN** only name and currency are visible
- **AND** sensitive settings are hidden

### Requirement: Multi-Tenancy Data Isolation

The system SHALL ensure complete data isolation between families at the database level.

#### Scenario: Row-level security enforcement
- **GIVEN** all database queries
- **WHEN** accessing any family-scoped table
- **THEN** family_id filter is automatically applied
- **AND** users cannot access data from other families
- **AND** database policies enforce isolation even if application logic fails

#### Scenario: Family context required
- **GIVEN** a request to create or update family-scoped data
- **WHEN** no family_id is provided in the request context
- **THEN** the request is rejected with error code `FAMILY_CONTEXT_REQUIRED`
- **AND** no data is created or modified

### Requirement: Default Categories on Family Creation

The system SHALL create default expense categories when a family is created.

#### Scenario: Default categories created
- **GIVEN** a new family is created
- **WHEN** the creation completes
- **THEN** default categories are created: Housing, Food, Transport, Utilities, Healthcare, Education, Entertainment, Others
- **AND** each category has a unique color and icon
- **AND** categories are owned by the family
