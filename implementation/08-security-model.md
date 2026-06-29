# Minime V1 — Security Model

## Status

Reconciled. Supersedes `09-security-model.md`.

---

## Architecture Authority

```
Architecture-Product-Definition/account/authentication.policy.v1.md
Architecture-Product-Definition/account/account.model.specification.v1.md
Architecture-Product-Definition/account/account.deletion.policy.v1.md
Architecture-Product-Definition/platform/platform.architecture.specification.v1.md
implementation/03-canonical-data-model.md
implementation/04-service-contracts.md
```

---

## Purpose

Defines the canonical security model for Minime V1.

The security model defines:

- trust boundaries
- authentication methods and their security requirements
- session and token lifecycle
- OTP-specific security rules
- Google Sign-In security rules
- authorization enforcement
- secret management
- data protection
- attack surface protection

Every application component must comply with this document.

---

## Trust Boundary

The backend is the only trusted execution environment.

- Frontend applications are untrusted.
- External providers are untrusted (including Google).
- Client requests are untrusted.
- Every external input must be validated before execution.

---

## Authentication Methods

V1 supports exactly two authentication methods:

```
Email OTP
Google Sign-In
```

The following are not supported in V1 and must not be implemented:

```
Phone OTP
SMS Verification
WhatsApp Verification
Password Authentication
Apple Sign-In
Facebook Login
X Login
LinkedIn Login
```

Authentication must execute in the backend. Authentication must complete before authorization executes.

---

## OTP Security

**OTP constants:** See `07-validation-rules.md` — Authentication constants (format, lifetime, attempt limits, cooldown).

**Security enforcement requirements:**

- Plain-text OTP values must never be stored. Only the hash is stored in `OtpVerification.code_hash`.
- The hash algorithm is an implementation choice. The plain-text OTP must be discarded after hashing.
- OtpVerification records must be hard-deleted after successful verification or after expiry.
- The attempt counter (`OtpVerification.attempts`) must be incremented on every failed verification.
- When `attempts` reaches 5, the OtpVerification record must be treated as invalid (same as expired).
- OTP codes must never appear in logs, events, error responses, or email subjects.
- The OTP response must not reveal whether an Account exists for the given email.

**Resend cooldown enforcement:**

Before creating a new OtpVerification record, the system must check whether an existing record for the same identifier was created within the last 60 seconds. If so, the new request must be rejected with `429 Too Many Requests`.

---

## Google Sign-In Security

**From `authentication.policy.v1.md`:**

- The Google ID token must be verified against Google's public keys on the backend.
- Client-side assertions must never be trusted.
- The `sub` claim from the verified token is the canonical Google identity identifier.
- Google tokens must never be stored in the database.
- If the token is expired or invalid, the request must be rejected with `401`.
- A successfully verified Google identity is considered verified immediately. No secondary OTP is required.

---

## Session Security

Sessions belong to one authenticated Account.

**Refresh token rules:**

- Refresh tokens are rotated on every use. The old hash is invalidated; a new hash is stored.
- Refresh tokens are stored as hashes only (`Session.refresh_token_hash`). Plain-text refresh tokens are never stored.
- Expired sessions must not be accepted (`expires_at < NOW()`).
- Revoked sessions must not be accepted (`revoked_at IS NOT NULL`).
- Session ownership belongs to the Authentication domain.

**Session revocation on account deletion:**

Per `account.deletion.policy.v1.md`, all active sessions must be invalidated at the moment account deletion is confirmed. No new session may be created for a deleted account.

**Session revocation triggers:**

- User-initiated logout (`POST /auth/logout`)
- User-initiated logout-all (`POST /auth/logout-all`)
- Account deletion (all sessions)
- Administrative suspension (all sessions)

---

## Token Rules

| Token | Rule |
|---|---|
| Access token | Short-lived JWT. Not stored in PostgreSQL or Redis. |
| Refresh token | Rotated on every use. Only the hash is stored (`Session.refresh_token_hash`). |
| Verification token | Short-lived, single-use. Issued after OTP verification; consumed by registration or sign-in. |
| Google token | Short-lived, single-use. Issued by backend after Google ID token verification; consumed by registration. |

Access tokens must not appear in logs.

Refresh tokens must not appear in logs, events, or API responses except during creation or rotation.

---

## No Password Handling

V1 has no password authentication. There are no passwords in the system.

- No password fields exist in any entity.
- No password hashing is performed.
- No password reset flow exists.
- No password validation rules exist.

If any implementation introduces a password field, hash, or flow, it contradicts the architecture and must be removed.

---

## Authorization

Authorization verifies Account ownership.

- Authorization must execute after authentication and before business execution.
- Every private endpoint verifies the authenticated `account_id` matches the resource's `account_id`.
- Business entities must not be accessible across Accounts.
- Authorization decisions belong to Product Domain services.
- There is no User entity, no Profile entity, and no profile_id. Account is the single ownership concept.

---

## Data Ownership Protection

- Every request accessing private data must verify Account ownership.
- Public resources must expose only public-safe fields.
- The internal `id` of OutLink must never be exposed publicly — routing uses `public_id` only.
- Authentication credentials (OTP codes, verification tokens, refresh tokens) must never appear in API responses beyond the minimal disclosure required by the auth flow.

---

## Secret Management

Secrets must exist only in environment variables. Secrets must never be committed to source control.

Secrets include:

- JWT signing secret
- Database credentials
- Redis credentials
- Object Storage credentials
- Google OAuth client credentials

Secrets must never appear in logs, events, responses, or job payloads.

---

## Redis Security

Redis must not store:

- OTP codes (plain text or hash)
- Access tokens
- Refresh tokens
- Canonical business data

Redis stores only:

- Rendered profile cache (TTL-bound)
- Temporary runtime state (TTL-bound)

Redis access must require authentication. Redis must not be publicly accessible.

---

## Object Storage Security

- Storage keys must remain internal. Clients must never receive raw storage keys.
- Uploaded files must verify MIME type, file size, and authenticated ownership before processing.
- Executable file types must not be accepted.
- Business authorization must execute before storage access.

---

## Database Security

- Database access must occur only through repositories.
- Services must not execute raw database access directly.
- Database credentials must remain private.
- Database connections must use encrypted transport.

---

## Input Protection

Every external input must be validated before execution per `07-validation-rules.md`.

Invalid input must terminate request processing before business logic executes.

---

## Output Protection

Responses must expose only approved public data.

Responses must never expose:

- Secrets or credentials
- Stack traces
- Repository or Prisma implementation details
- Internal infrastructure details
- OTP codes
- Refresh tokens (except during creation or rotation)

---

## File Security

- Uploaded files must verify MIME type before storage.
- Maximum file size must be enforced.
- Uploaded file ownership must be verified via authenticated account_id.
- Files are stored in Object Storage. Binary content must not be stored in PostgreSQL.
- Executable file types must be rejected.

---

## Logging Security

Logs must not contain:

- OTP codes
- Authentication tokens (access or refresh)
- Secrets or credentials
- Sensitive personal information not required for diagnostics

Security events must be logged:

- Authentication failures
- Authorization failures
- Token revocations
- Account deletions
- Unexpected server failures
- Background job failures

---

## Rate Limiting

Rate limiting must protect:

- `POST /api/v1/auth/otp/request` — by identifier (email)
- `POST /api/v1/auth/otp/verify` — by identifier (email)
- `POST /api/v1/auth/google` — by IP
- `POST /api/v1/auth/session` — by identifier
- `GET /api/v1/usernames/availability` — by IP
- `POST /api/v1/usernames/reservations` — by IP
- Public rendering `/{username}` — by IP
- Public redirect `GET /out/{public_id}` — by IP

Rate limiting policies belong to the backend. Clients must not control rate limiting behavior.

---

## Cross-Origin Requests

Cross-origin requests must use an explicit allow list. Origins not present in the allow list must be rejected. Credentialed cross-origin requests require explicit configuration.

---

## HTTP Security Headers

Production deployments must configure:

- Content-Type protection (`X-Content-Type-Options: nosniff`)
- Clickjacking protection (`X-Frame-Options: DENY`)
- Transport security (`Strict-Transport-Security`)
- Referrer policy

Security headers belong to the deployment layer.

---

## Background Job Security

Background jobs must:

- execute with backend permissions
- validate business ownership before mutating business entities
- not bypass Product Domain validation

Background jobs must not:

- access HTTP request state
- expose secrets in job payloads
- bypass repositories

---

## AI Security

AI responses must be treated as untrusted input. AI responses must be validated before use. User confirmation is required before applying AI-generated changes. AI responses must never modify business state automatically.

---

## External Provider Security

Connections to external providers (Google) must use encrypted transport. Provider responses must be validated before business use. External providers must not become the source of truth.

---

## Security Prohibitions

Implementation must not:

- trust client input or client-side authorization
- expose secrets, stack traces, or infrastructure details
- bypass authentication or authorization
- bypass validation
- bypass repositories
- store plain-text OTP codes
- implement password authentication (V1 has no passwords)
- implement phone OTP or SMS authentication
- trust Google ID tokens without backend verification against Google's public keys
- allow AI-generated changes without explicit user confirmation
- store access tokens or refresh tokens in plain text
- expose raw `OutLink.id` publicly (only `public_id` appears in public routes)
- allow a deleted account to create a new session
- expose OTP codes in logs, events, or responses
