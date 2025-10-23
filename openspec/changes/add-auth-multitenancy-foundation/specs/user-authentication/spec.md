# User Authentication Specification

## ADDED Requirements

### Requirement: User Registration

The system SHALL allow new users to register with email and password.

#### Scenario: Successful registration
- **GIVEN** a new user provides valid email and password
- **WHEN** the registration request is submitted
- **THEN** a user account is created
- **AND** the password is hashed using bcrypt or argon2
- **AND** a confirmation email is sent
- **AND** the user can log in immediately

#### Scenario: Duplicate email rejection
- **GIVEN** a user tries to register with an existing email
- **WHEN** the registration request is submitted
- **THEN** the request is rejected with error code `USER_EMAIL_EXISTS`
- **AND** no new account is created

#### Scenario: Invalid email format
- **GIVEN** a user provides an invalid email format
- **WHEN** the registration request is submitted
- **THEN** the request is rejected with error code `INVALID_EMAIL_FORMAT`

#### Scenario: Weak password rejection
- **GIVEN** a user provides a password shorter than 8 characters
- **WHEN** the registration request is submitted
- **THEN** the request is rejected with error code `WEAK_PASSWORD`
- **AND** a message indicates minimum password requirements

### Requirement: User Login

The system SHALL authenticate users with email and password and return JWT tokens.

#### Scenario: Successful login
- **GIVEN** a user provides valid credentials
- **WHEN** the login request is submitted
- **THEN** an access token (JWT) with 15-minute expiry is returned
- **AND** a refresh token with 7-day expiry is returned
- **AND** the tokens include user_id and email claims

#### Scenario: Invalid credentials
- **GIVEN** a user provides incorrect email or password
- **WHEN** the login request is submitted
- **THEN** the request is rejected with error code `INVALID_CREDENTIALS`
- **AND** no tokens are returned
- **AND** the response does not indicate whether email or password was incorrect

#### Scenario: Rate limiting on failed attempts
- **GIVEN** a user has 5 failed login attempts within 15 minutes
- **WHEN** another login attempt is made
- **THEN** the request is rejected with error code `RATE_LIMIT_EXCEEDED`
- **AND** the user must wait before retrying

### Requirement: Token Refresh

The system SHALL allow users to obtain new access tokens using valid refresh tokens.

#### Scenario: Successful token refresh
- **GIVEN** a user has a valid refresh token
- **WHEN** a refresh request is submitted
- **THEN** a new access token is returned
- **AND** a new refresh token is returned (rotation)
- **AND** the old refresh token is invalidated

#### Scenario: Expired refresh token
- **GIVEN** a user has an expired refresh token
- **WHEN** a refresh request is submitted
- **THEN** the request is rejected with error code `TOKEN_EXPIRED`
- **AND** the user must log in again

#### Scenario: Invalid refresh token
- **GIVEN** a user provides an invalid or revoked refresh token
- **WHEN** a refresh request is submitted
- **THEN** the request is rejected with error code `INVALID_TOKEN`

### Requirement: User Logout

The system SHALL allow users to invalidate their refresh tokens on logout.

#### Scenario: Successful logout
- **GIVEN** a user is logged in
- **WHEN** a logout request is submitted with refresh token
- **THEN** the refresh token is revoked
- **AND** future refresh attempts with that token fail
- **AND** the access token remains valid until expiry

### Requirement: Password Reset

The system SHALL allow users to reset forgotten passwords via email.

#### Scenario: Password reset request
- **GIVEN** a user requests password reset for a registered email
- **WHEN** the reset request is submitted
- **THEN** a password reset email is sent with a unique token
- **AND** the token expires after 1 hour
- **AND** the response does not reveal whether the email exists

#### Scenario: Password reset completion
- **GIVEN** a user has a valid reset token
- **WHEN** a new password is submitted with the token
- **THEN** the password is updated
- **AND** all existing refresh tokens are revoked
- **AND** the reset token is invalidated
- **AND** the user can log in with the new password

#### Scenario: Expired reset token
- **GIVEN** a user has an expired reset token
- **WHEN** attempting to reset password
- **THEN** the request is rejected with error code `TOKEN_EXPIRED`
- **AND** the user must request a new reset token

### Requirement: Two-Factor Authentication for SuperAdmin

The system SHALL require two-factor authentication for users with SuperAdmin role.

#### Scenario: 2FA setup required
- **GIVEN** a user is promoted to SuperAdmin role
- **WHEN** they next log in
- **THEN** they are required to set up 2FA
- **AND** they cannot access family resources until 2FA is enabled

#### Scenario: 2FA verification on login
- **GIVEN** a SuperAdmin user has 2FA enabled
- **WHEN** they provide valid credentials
- **THEN** an OTP is sent to their email or authenticator app
- **AND** they must provide the OTP to complete login
- **AND** OTP expires after 5 minutes

#### Scenario: Invalid OTP
- **GIVEN** a SuperAdmin user is at the 2FA verification step
- **WHEN** they provide an incorrect OTP
- **THEN** the request is rejected with error code `INVALID_OTP`
- **AND** they can retry up to 3 times before lockout

### Requirement: Session Timeout

The system SHALL enforce session timeout after 30 minutes of inactivity.

#### Scenario: Session expiry check
- **GIVEN** a user's last activity was more than 30 minutes ago
- **WHEN** they make an API request with their access token
- **THEN** the request is rejected with error code `SESSION_EXPIRED`
- **AND** they must refresh their token or log in again

#### Scenario: Activity tracking
- **GIVEN** a user makes an API request
- **WHEN** the request is authenticated successfully
- **THEN** the last activity timestamp is updated
- **AND** the session timeout is reset

### Requirement: Authentication Middleware

The system SHALL provide middleware to verify JWT tokens on protected routes.

#### Scenario: Valid token accepted
- **GIVEN** a request includes a valid access token in Authorization header
- **WHEN** the middleware processes the request
- **THEN** the user_id is extracted and attached to the request context
- **AND** the request proceeds to the route handler

#### Scenario: Missing token rejected
- **GIVEN** a request to a protected route has no Authorization header
- **WHEN** the middleware processes the request
- **THEN** the request is rejected with error code `UNAUTHORIZED`
- **AND** status 401 is returned

#### Scenario: Invalid token rejected
- **GIVEN** a request includes an invalid or expired access token
- **WHEN** the middleware processes the request
- **THEN** the request is rejected with error code `INVALID_TOKEN`
- **AND** status 401 is returned
