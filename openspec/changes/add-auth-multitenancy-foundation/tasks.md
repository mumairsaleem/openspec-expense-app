# Implementation Tasks

## 1. Database Setup

- [ ] 1.1 Create PostgreSQL database schema
- [ ] 1.2 Create users table (id, email, password_hash, name, phone, profile_picture, notification_preferences, created_at, updated_at, deleted_at)
- [ ] 1.3 Create families table (id, name, currency, timezone, fiscal_year_start, super_admin_id, created_at, updated_at, deleted_at)
- [ ] 1.4 Create memberships table (id, user_id, family_id, role, joined_at, expires_at, created_at, updated_at)
- [ ] 1.5 Create refresh_tokens table (id, user_id, token_hash, expires_at, revoked_at, created_at)
- [ ] 1.6 Create password_reset_tokens table (id, user_id, token_hash, expires_at, used_at, created_at)
- [ ] 1.7 Create invites table (id, family_id, email, role, token_hash, invited_by, expires_at, accepted_at, created_at)
- [ ] 1.8 Create categories table (id, family_id, name, icon, color, created_at, updated_at, deleted_at)
- [ ] 1.9 Create audit_logs table (id, family_id, user_id, action, resource_type, resource_id, before_data, after_data, created_at)
- [ ] 1.10 Create indexes on foreign keys (user_id, family_id) and frequently queried fields (email, token hashes)
- [ ] 1.11 Set up row-level security policies for multi-tenancy
- [ ] 1.12 Enable pgcrypto extension for encrypted fields
- [ ] 1.13 Create database migration scripts

## 2. Backend - Authentication Service

- [ ] 2.1 Initialize Node.js project with TypeScript
- [ ] 2.2 Install dependencies (express/fastify, jsonwebtoken, bcrypt/argon2, pg/prisma, helmet, cors, express-rate-limit)
- [ ] 2.3 Set up TypeScript configuration with strict mode
- [ ] 2.4 Create project structure (routes/, controllers/, services/, middleware/, models/, utils/, config/)
- [ ] 2.5 Create environment configuration (.env with DATABASE_URL, JWT_SECRET, JWT_REFRESH_SECRET, etc.)
- [ ] 2.6 Implement password hashing utility (bcrypt/argon2)
- [ ] 2.7 Implement JWT token generation and verification utilities
- [ ] 2.8 Create user registration endpoint (POST /auth/register)
- [ ] 2.9 Create user login endpoint (POST /auth/login)
- [ ] 2.10 Create token refresh endpoint (POST /auth/refresh)
- [ ] 2.11 Create logout endpoint (POST /auth/logout)
- [ ] 2.12 Create password reset request endpoint (POST /auth/password-reset/request)
- [ ] 2.13 Create password reset completion endpoint (POST /auth/password-reset/complete)
- [ ] 2.14 Implement rate limiting on authentication endpoints
- [ ] 2.15 Create authentication middleware to verify JWT tokens
- [ ] 2.16 Create session timeout tracking middleware
- [ ] 2.17 Implement 2FA setup endpoint (POST /auth/2fa/setup)
- [ ] 2.18 Implement 2FA verification endpoint (POST /auth/2fa/verify)
- [ ] 2.19 Add error handling middleware with standardized error responses
- [ ] 2.20 Write unit tests for authentication service (Jest)

## 3. Backend - Family Service

- [ ] 3.1 Create family creation endpoint (POST /v1/families)
- [ ] 3.2 Implement default category creation on family creation
- [ ] 3.3 Create family list endpoint (GET /v1/families)
- [ ] 3.4 Create family details endpoint (GET /v1/families/:id)
- [ ] 3.5 Create family update endpoint (PUT /v1/families/:id)
- [ ] 3.6 Create family soft delete endpoint (DELETE /v1/families/:id)
- [ ] 3.7 Create family restoration endpoint (POST /v1/families/:id/restore)
- [ ] 3.8 Implement family context extraction middleware
- [ ] 3.9 Implement family membership validation middleware
- [ ] 3.10 Create background job for permanent deletion after 30 days
- [ ] 3.11 Add audit logging for all family CRUD operations
- [ ] 3.12 Write unit tests for family service (Jest)

## 4. Backend - Membership Service

- [ ] 4.1 Create membership creation on family creation (automatic SuperAdmin assignment)
- [ ] 4.2 Create invite endpoint (POST /v1/families/:id/invites)
- [ ] 4.3 Create invite acceptance endpoint (POST /v1/invites/:token/accept)
- [ ] 4.4 Create member list endpoint (GET /v1/families/:id/members)
- [ ] 4.5 Create role assignment endpoint (PUT /v1/families/:id/members/:userId/role)
- [ ] 4.6 Create SuperAdmin transfer endpoint (POST /v1/families/:id/transfer-ownership)
- [ ] 4.7 Create member removal endpoint (DELETE /v1/families/:id/members/:userId)
- [ ] 4.8 Implement RBAC middleware with permission matrix
- [ ] 4.9 Implement time-boxed Auditor expiry check
- [ ] 4.10 Create family context switching endpoint (POST /v1/user/active-family)
- [ ] 4.11 Add audit logging for all membership changes
- [ ] 4.12 Write unit tests for membership service (Jest)
- [ ] 4.13 Write integration tests for RBAC enforcement (Jest)

## 5. Backend - Email Service Integration

- [ ] 5.1 Set up email service provider (SendGrid/AWS SES)
- [ ] 5.2 Create email templates for registration confirmation
- [ ] 5.3 Create email templates for password reset
- [ ] 5.4 Create email templates for family invites
- [ ] 5.5 Create email templates for 2FA OTP
- [ ] 5.6 Implement email sending service with retry logic
- [ ] 5.7 Add email queue for asynchronous sending (optional: Bull/BullMQ)
- [ ] 5.8 Write tests for email service (mocked)

## 6. Backend - Constants and Error Codes

- [ ] 6.1 Create constants file for all text strings (USER_MESSAGES, ERROR_MESSAGES, etc.)
- [ ] 6.2 Define error codes (USER_EMAIL_EXISTS, INVALID_CREDENTIALS, INSUFFICIENT_PERMISSIONS, etc.)
- [ ] 6.3 Create error code to HTTP status mapping
- [ ] 6.4 Implement centralized error response formatter

## 7. Frontend - Project Setup

- [ ] 7.1 Initialize React project with TypeScript
- [ ] 7.2 Install dependencies (react-router-dom, axios/fetch, zustand/redux, react-hook-form, etc.)
- [ ] 7.3 Set up TypeScript configuration
- [ ] 7.4 Create project structure (components/, pages/, hooks/, services/, utils/, constants/, types/)
- [ ] 7.5 Set up environment configuration (.env with REACT_APP_API_BASE_URL)
- [ ] 7.6 Configure routing with react-router-dom
- [ ] 7.7 Create API client with interceptors for auth tokens

## 8. Frontend - Authentication Pages

- [ ] 8.1 Create registration page with form validation
- [ ] 8.2 Create login page with form validation
- [ ] 8.3 Create password reset request page
- [ ] 8.4 Create password reset completion page
- [ ] 8.5 Create 2FA setup page
- [ ] 8.6 Create 2FA verification page
- [ ] 8.7 Implement token storage (localStorage/sessionStorage with security considerations)
- [ ] 8.8 Implement automatic token refresh logic
- [ ] 8.9 Create protected route wrapper component
- [ ] 8.10 Create authentication context/store for user state
- [ ] 8.11 Add logout functionality
- [ ] 8.12 Write tests for authentication pages (React Testing Library)

## 9. Frontend - Family Management

- [ ] 9.1 Create family creation wizard page
- [ ] 9.2 Create family selection page for users with multiple families
- [ ] 9.3 Create family settings page (view/edit for authorized roles)
- [ ] 9.4 Create family context provider for active family state
- [ ] 9.5 Implement family context switching UI
- [ ] 9.6 Create family deletion confirmation dialog
- [ ] 9.7 Add default category display on family creation
- [ ] 9.8 Write tests for family pages (React Testing Library)

## 10. Frontend - Membership Management

- [ ] 10.1 Create member invite form/modal
- [ ] 10.2 Create invite acceptance page (public route)
- [ ] 10.3 Create member list page with role indicators
- [ ] 10.4 Create role assignment UI (dropdown/modal for authorized users)
- [ ] 10.5 Create member removal confirmation dialog
- [ ] 10.6 Create SuperAdmin transfer workflow UI
- [ ] 10.7 Display time-boxed expiry for Auditor role
- [ ] 10.8 Implement permission-based UI rendering (hide/disable actions based on role)
- [ ] 10.9 Write tests for membership pages (React Testing Library)

## 11. Frontend - Constants and Accessibility

- [ ] 11.1 Create constants file for all UI text strings
- [ ] 11.2 Define role labels, permission descriptions
- [ ] 11.3 Ensure WCAG AA compliance (color contrast, keyboard navigation, ARIA labels)
- [ ] 11.4 Add scalable font size support
- [ ] 11.5 Test with screen reader

## 12. Integration and End-to-End Testing

- [ ] 12.1 Set up Playwright or Cypress for E2E tests
- [ ] 12.2 Write E2E test for user registration and login flow
- [ ] 12.3 Write E2E test for family creation and member invitation flow
- [ ] 12.4 Write E2E test for role assignment and permission enforcement
- [ ] 12.5 Write E2E test for password reset flow
- [ ] 12.6 Write E2E test for 2FA setup and verification
- [ ] 12.7 Write E2E test for multi-family context switching

## 13. Documentation and Deployment Prep

- [ ] 13.1 Document API endpoints with request/response examples
- [ ] 13.2 Create database migration guide
- [ ] 13.3 Create environment setup instructions
- [ ] 13.4 Set up Docker configuration for local development
- [ ] 13.5 Create CI/CD pipeline with GitHub Actions
- [ ] 13.6 Run all tests and ensure they pass
- [ ] 13.7 Perform security audit (check for vulnerabilities, secure token storage, etc.)
- [ ] 13.8 Deploy to staging environment

## Dependencies and Parallelization Notes

**Can be done in parallel:**
- Tasks 2.x (Backend Auth) and 7.x (Frontend Setup) can start simultaneously
- Tasks 3.x (Backend Family) and 4.x (Backend Membership) can run in parallel after database is set up
- Tasks 8.x, 9.x, 10.x (Frontend pages) can be built in parallel once API client is ready

**Sequential dependencies:**
- Task 1 (Database) must complete before tasks 2, 3, 4
- Task 2.15-2.16 (Auth middleware) must complete before tasks 3 and 4
- Task 7.7 (API client) must complete before tasks 8, 9, 10
- Tasks 2-11 must complete before task 12 (E2E tests)
- Task 12 must complete before task 13 (Deployment)

**Critical path:**
Database → Auth Service → Family Service → Membership Service → Frontend Setup → Auth Pages → Family/Membership Pages → E2E Tests → Deployment
