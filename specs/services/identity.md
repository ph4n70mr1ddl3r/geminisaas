# Identity Service Specification

## Overview

The Identity Service is responsible for user management, authentication, and authorization (RBAC) across the entire ERP system.

## Data Model (SQLite)

- `users`: id (UUID), username, password_hash (Argon2), email, active (BOOL).
- `roles`: id (UUID), name (e.g., 'ADMIN', 'FINANCE_MANAGER', 'PURCHASER').
- `permissions`: id (UUID), resource (e.g., 'ledger'), action (e.g., 'write').
- `user_roles`: maps users to roles.
- `role_permissions`: maps roles to permissions.
- `sessions`: maps active JWT tokens or sessions (if stateful).

## API Endpoints (Axum)

- `POST /auth/login`: Authenticate and return JWT.
- `POST /auth/logout`: Revoke session.
- `POST /auth/refresh`: Refresh JWT.
- `GET /users/me`: Current user info and permissions.
- `POST /admin/users`: Create/Manage users.
- `POST /admin/roles`: Create/Manage roles and permissions.

## Inter-service Interaction

- Acts as a central authority for token verification.
- Services can call `/auth/verify` or use a shared secret/public key for local JWT validation.
- Emits events when user/role changes occur (e.g., `UserCreated`, `RoleUpdated`).

## Security

- Password hashing via `argon2`.
- JWT signed with RS256 or ED25519 (using `jsonwebtoken` or `paseto`).
- Rate limiting on login attempts.
- Enforce MFA (Multi-Factor Authentication) as a later requirement.
