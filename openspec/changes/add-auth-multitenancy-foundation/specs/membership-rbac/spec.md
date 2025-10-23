# Membership and RBAC Specification

## ADDED Requirements

### Requirement: Family Membership Creation

The system SHALL create a membership record when a user creates or joins a family.

#### Scenario: SuperAdmin membership on family creation
- **GIVEN** a user creates a new family
- **WHEN** the family is created successfully
- **THEN** a membership record is created
- **AND** the user is assigned the SuperAdmin role
- **AND** the membership has no expiry date

#### Scenario: Member joins via invite acceptance
- **GIVEN** a user accepts a family invite
- **WHEN** the acceptance is processed
- **THEN** a membership record is created
- **AND** the user is assigned the role specified in the invite
- **AND** the user can access family resources based on their role

### Requirement: Role Assignment

The system SHALL allow authorized users to assign and revoke roles.

#### Scenario: SuperAdmin assigns any role
- **GIVEN** a SuperAdmin of a family
- **WHEN** assigning a role to another member
- **THEN** the role is updated
- **AND** an audit log entry is created
- **AND** the change takes effect immediately

#### Scenario: Admin assigns non-SuperAdmin roles
- **GIVEN** a user with Admin role
- **WHEN** assigning Member, Guest, or Auditor role
- **THEN** the role is updated
- **AND** an audit log entry is created

#### Scenario: Admin cannot assign SuperAdmin role
- **GIVEN** a user with Admin role
- **WHEN** attempting to assign SuperAdmin role
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`
- **AND** no role change occurs

#### Scenario: Member cannot assign roles
- **GIVEN** a user with Member role
- **WHEN** attempting to assign any role
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

#### Scenario: Only one SuperAdmin per family
- **GIVEN** a family with a SuperAdmin
- **WHEN** attempting to assign SuperAdmin to another user
- **THEN** the request is rejected with error code `SUPERADMIN_ALREADY_EXISTS`
- **AND** the existing SuperAdmin must transfer ownership first

### Requirement: SuperAdmin Transfer

The system SHALL allow SuperAdmin to transfer ownership to another family member.

#### Scenario: Successful ownership transfer
- **GIVEN** a SuperAdmin transfers ownership to an existing Admin
- **WHEN** the transfer is confirmed
- **THEN** the current SuperAdmin becomes an Admin
- **AND** the target user becomes SuperAdmin
- **AND** an audit log entry is created
- **AND** the new SuperAdmin has 2FA enabled or is required to enable it

#### Scenario: Cannot transfer to non-member
- **GIVEN** a SuperAdmin attempts to transfer to a user not in the family
- **WHEN** the transfer request is submitted
- **THEN** the request is rejected with error code `USER_NOT_FAMILY_MEMBER`

### Requirement: Member Invitation

The system SHALL allow Admin and SuperAdmin to invite new members.

#### Scenario: Send family invite
- **GIVEN** an Admin or SuperAdmin invites a user by email
- **WHEN** specifying the role (Member, Guest, or Auditor)
- **THEN** an invite record is created with a unique token
- **AND** an email is sent with the invite link
- **AND** the invite expires after 7 days

#### Scenario: Invite existing user
- **GIVEN** an Admin invites an email that's already a registered user
- **WHEN** the invite is sent
- **THEN** the invite email is sent to the existing user
- **AND** they can accept to join the family immediately

#### Scenario: Invite new user
- **GIVEN** an Admin invites an email that's not registered
- **WHEN** the invite is sent
- **THEN** the invite email includes registration instructions
- **AND** the user must register before accepting the invite

#### Scenario: Expired invite
- **GIVEN** a user has an invite older than 7 days
- **WHEN** attempting to accept the invite
- **THEN** the request is rejected with error code `INVITE_EXPIRED`
- **AND** the user is prompted to request a new invite

#### Scenario: Admin cannot invite SuperAdmin
- **GIVEN** a user with Admin role
- **WHEN** attempting to send an invite with SuperAdmin role
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

### Requirement: Member Removal

The system SHALL allow Admin and SuperAdmin to remove members from the family.

#### Scenario: Admin removes Member or Guest
- **GIVEN** an Admin of a family
- **WHEN** removing a user with Member or Guest role
- **THEN** the membership is deleted
- **AND** the user loses access to family resources
- **AND** an audit log entry is created

#### Scenario: Admin cannot remove SuperAdmin
- **GIVEN** an Admin attempts to remove the SuperAdmin
- **WHEN** the removal request is submitted
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`

#### Scenario: SuperAdmin can remove any member
- **GIVEN** a SuperAdmin of a family
- **WHEN** removing any non-SuperAdmin member
- **THEN** the membership is deleted
- **AND** the user loses access immediately

#### Scenario: SuperAdmin cannot remove themselves
- **GIVEN** a SuperAdmin attempts to remove their own membership
- **WHEN** the removal request is submitted
- **THEN** the request is rejected with error code `CANNOT_REMOVE_SELF`
- **AND** they must transfer ownership or delete the family instead

### Requirement: Time-Boxed Auditor Access

The system SHALL support temporary Auditor role with automatic expiry.

#### Scenario: Create time-boxed Auditor membership
- **GIVEN** an Admin assigns Auditor role with expiry date
- **WHEN** the membership is created
- **THEN** the expires_at field is set
- **AND** the Auditor can access family data until expiry

#### Scenario: Expired Auditor access denied
- **GIVEN** an Auditor's membership has expired
- **WHEN** they attempt to access family resources
- **THEN** the request is rejected with error code `MEMBERSHIP_EXPIRED`
- **AND** their role is automatically revoked

#### Scenario: Extend Auditor access
- **GIVEN** an Admin extends an Auditor's expiry date
- **WHEN** the update is processed
- **THEN** the expires_at is updated
- **AND** the Auditor continues to have access

### Requirement: Permission Matrix Enforcement

The system SHALL enforce role-based permissions according to the defined permission matrix.

#### Scenario: SuperAdmin full access
- **GIVEN** a user with SuperAdmin role
- **WHEN** accessing any family resource
- **THEN** full CRUD permissions are granted

#### Scenario: Admin manages resources
- **GIVEN** a user with Admin role
- **WHEN** accessing categories, budgets, wallets, or transactions
- **THEN** CRUD permissions are granted
- **AND** access to policy and retention settings is denied

#### Scenario: Member limited access
- **GIVEN** a user with Member role
- **WHEN** creating or editing transactions
- **THEN** they can CRUD their own transactions
- **AND** they can view family transactions
- **AND** they can propose changes to shared transactions

#### Scenario: Guest read-only access
- **GIVEN** a user with Guest role
- **WHEN** accessing any family resource
- **THEN** only read access is granted to assigned wallets and reports
- **AND** all write operations are denied with error code `INSUFFICIENT_PERMISSIONS`

#### Scenario: Auditor view-only access
- **GIVEN** a user with Auditor role
- **WHEN** accessing family resources
- **THEN** read-only access is granted to all reports and ledgers
- **AND** private notes fields are excluded from responses
- **AND** all write operations are denied

### Requirement: RBAC Middleware

The system SHALL provide middleware to enforce role-based permissions on routes.

#### Scenario: Permission check passes
- **GIVEN** a request to a protected route with required role
- **WHEN** the middleware validates the user's role
- **THEN** the user's role is sufficient for the operation
- **AND** the request proceeds to the route handler

#### Scenario: Permission check fails
- **GIVEN** a request to a route requiring Admin role
- **WHEN** the user has only Member role
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`
- **AND** status 403 is returned

#### Scenario: Multiple role support
- **GIVEN** a route allows Admin or SuperAdmin
- **WHEN** a user with either role makes a request
- **THEN** the request is authorized
- **AND** the route handler executes

### Requirement: Family Context Switching

The system SHALL allow users in multiple families to switch between family contexts.

#### Scenario: Select active family
- **GIVEN** a user belongs to multiple families
- **WHEN** they select a family as their active context
- **THEN** the family_id is stored in their session
- **AND** subsequent requests use that family context
- **AND** they see data only from the selected family

#### Scenario: Invalid family selection
- **GIVEN** a user attempts to select a family they don't belong to
- **WHEN** the context switch is requested
- **THEN** the request is rejected with error code `NOT_FAMILY_MEMBER`
- **AND** the active context remains unchanged

### Requirement: Membership List

The system SHALL allow authorized users to view family membership.

#### Scenario: List all family members
- **GIVEN** a user with Admin or SuperAdmin role
- **WHEN** requesting the family member list
- **THEN** all active members are returned with their roles and join dates
- **AND** expired memberships are excluded

#### Scenario: Member views member list
- **GIVEN** a user with Member role
- **WHEN** requesting the member list
- **THEN** all members are visible with names and roles only
- **AND** contact details are hidden

#### Scenario: Guest cannot view member list
- **GIVEN** a user with Guest role
- **WHEN** requesting the member list
- **THEN** the request is rejected with error code `INSUFFICIENT_PERMISSIONS`
