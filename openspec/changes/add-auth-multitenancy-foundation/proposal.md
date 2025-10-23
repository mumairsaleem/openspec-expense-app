# Add Authentication and Multi-Tenancy Foundation

## Why

The Home Expense & Investment Tracker (HEIT) requires a secure, multi-tenant foundation where users can create and join family accounts with role-based access control. Without authentication and tenancy management, no other features can function. This is the foundational layer that all subsequent features will build upon.

## What Changes

- Add user registration and login with JWT-based authentication
- Add family (tenant) creation and management with one SuperAdmin per family
- Add user-family membership management with role-based permissions
- Add middleware for authentication verification and family context extraction
- Add password hashing with bcrypt/argon2
- Add refresh token rotation for session management
- Add 2FA support for SuperAdmin accounts

## Impact

**New Capabilities:**
- `user-authentication`: User registration, login, JWT token management, password reset
- `family-tenancy`: Family creation, settings management, tenant isolation
- `membership-rbac`: User-family memberships, role assignment, permission enforcement

**Affected Code:**
- New: Backend authentication routes (`/auth/*`)
- New: Backend family routes (`/v1/families/*`)
- New: Backend membership routes (`/v1/memberships/*`)
- New: Database schema (users, families, memberships tables)
- New: Authentication middleware
- New: RBAC middleware
- New: Frontend authentication pages (login, register, family setup)
- New: Frontend family context management

**External Dependencies:**
- JWT library (jsonwebtoken)
- Password hashing (bcrypt or argon2)
- Email service for invites and password reset
- 2FA provider for SuperAdmin accounts

**Security Considerations:**
- JWT tokens with 15-minute TTL + refresh tokens
- Password hashing with bcrypt/argon2
- Rate limiting on authentication endpoints
- Session timeout after 30 minutes of inactivity
- 2FA enforcement for SuperAdmin role
