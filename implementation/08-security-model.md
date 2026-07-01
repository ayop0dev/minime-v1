# Minime V1 — Security Model

## Status

Canonical. Final.

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
- provider authentication security rules (Google, Apple)
- authorization enforcement
- secret management
- data protection
- attack surface protection

Every application component must comply with this document.

---

## Trust Boundary

The backend is the only trusted execution environment.

- Frontend applications are untrusted.
- External providers are untrusted (including Google and Apple).
- Client requests are untrusted.
- Every external input must be validated before execution.

---

## Authentication Methods

V1 supports exactly two authentication methods:

```
Google Sign-In
Apple Sign-In
```

Authentication is provider-based only. Minime never authenticates users directly.

The following are not supported in V1 and must not be implemented:

```
Email OTP
Phone OTP
SMS Verification
WhatsApp Verification
Password Authentication
Facebook Login
X Login
LinkedIn Login
```

Authentication must execute in the backend. Authentication must complete before authorization executes.

---

## OAuth Flow Security

Both Google Sign-In and Apple Sign-In are implemented as backend-driven OAuth/OIDC authorization-code flows through a provider-agnostic core flow plus a per-provider Provider Adapter (`AuthService.startOAuth` / `handleOAuthCallback`; see `04-service-contracts.md` and `authentication.policy.v1.md` — "Provider Adapter Architecture"). These rules apply identically to every provider, present or future; they are enforced by the core flow, not by per-provider code.

Every OAuth flow must satisfy all of the following:

- **HTTPS only.** No step of the flow may occur over plain HTTP.
- **State parameter — provider-agnostic control.** A single-use, unguessable `state` value is generated at `startOAuth` and validated at `handleOAuthCallback`. A mismatch or missing `state` must reject the request before any token exchange occurs. This is the flow's CSRF protection, and it is enforced by core `AuthService` logic, not by any adapter.
- **Nonce — provider-agnostic control.** A `nonce` is generated at `startOAuth`, requested inside the `id_token`, and validated at `handleOAuthCallback`. This is the flow's replay protection for the identity token itself, and it too is a core, provider-agnostic control.
- **Provider/state-context consistency.** The `provider` supplied in the callback request data must be validated against the provider bound to the stored `state` context recorded at `startOAuth` time. A request whose `provider` does not match the state context's provider must be rejected before any Provider Adapter is resolved or any token exchange occurs.
- **Unsupported providers rejected.** Any `provider` value outside the V1 allowlist (`google`, `apple`) must be rejected at both `startOAuth` and `handleOAuthCallback`, before any adapter is resolved.
- **Replayed state/nonce rejected.** `state` and `nonce` are both single-use; a `state` or `nonce` value that has already been consumed, or that has expired, must never be accepted.
- **Backend token exchange only.** The authorization `code` is exchanged for tokens exclusively by the backend, inside the resolved Provider Adapter, in a direct server-to-provider call. The frontend never performs this exchange and never receives, stores, or transmits a client secret.
- **`id_token` signature validation.** The token's signature must be verified against the provider's published JWKS, inside the adapter's `validateIdToken` step.
- **Issuer validation.** The token's `iss` claim must match the provider's expected issuer.
- **Audience validation.** The token's `aud` claim must match this application's configured `client_id` for that provider.
- **Expiry validation.** The token's `exp` claim must not be in the past.
- A token or callback failing any one of these checks must be rejected with `401`/`400` as appropriate; no identity data is extracted from a failed validation.
- **Email is never canonical identity.** No step of any OAuth flow, for any provider, may use `email` to look up, match, or merge an Account. Identity resolution is always `(provider, provider_subject)`. See "Identity Merge Policy" in `authentication.policy.v1.md`.

---

## Provider Adapter Secret Ownership

OAuth provider secrets are owned and stored per Provider Adapter — never by core `AuthService`, and never shared across adapters.

- Each adapter (Google, Apple, and any future adapter) owns its own secret configuration. Core `AuthService` never reads, holds, or forwards a provider secret directly; it only invokes the adapter contract.
- No adapter secret may be exposed to the frontend, under any circumstance.
- Core `AuthService` never exposes provider tokens (`id_token`, access token, refresh token) to the frontend. Only Minime's own session tokens (access/refresh JWTs) ever reach the client.
- Provider access tokens and refresh tokens must not be persisted unless a future provider explicitly requires it and this security model is updated to say so. No such requirement exists today.

---

## Google Sign-In Security (Google Provider Adapter)

**From `authentication.policy.v1.md`:**

- The Google `id_token` must be verified against Google's public keys inside the Google adapter's `validateIdToken` step, per the OAuth Flow Security rules above.
- Client-side assertions must never be trusted.
- The `sub` claim from the verified token is the canonical Google identity identifier, normalized into `provider_subject` by the adapter's `normalizeIdentity` step.
- Google access tokens, refresh tokens, and ID tokens must never be stored in the database.
- The Google `email` claim may be normalized into `AuthenticationIdentity.email`, and other profile fields (name, picture, locale) into `AuthenticationIdentity.provider_profile`. `email` must never be the canonical identity key.
- The Google `client_id`/`client_secret` are Google-adapter-owned secret configuration (see "Provider Adapter Secret Ownership").
- If the token is expired or invalid, the request must be rejected with `401`.
- A successfully verified Google identity is considered verified immediately.

---

## Apple Sign-In Security (Apple Provider Adapter)

**From `authentication.policy.v1.md`:**

- The Apple `id_token` must be verified against Apple's public keys inside the Apple adapter's `validateIdToken` step, per the OAuth Flow Security rules above.
- Client-side assertions must never be trusted.
- The `sub` claim from the verified token is the canonical Apple identity identifier, normalized into `provider_subject` by the adapter's `normalizeIdentity` step.
- Apple authentication requires an Apple Developer Program account for production use.
- Apple web authentication uses a Services ID as the OAuth `client_id`.
- The Apple `client_secret` is a JWT generated server-side, signed with the Apple private key, constructed entirely inside the Apple adapter. It is never a static secret and is never present in any client-side code, build artifact, or environment exposed to the frontend.
- The Apple private key is Apple-adapter-owned secret configuration (see "Provider Adapter Secret Ownership"). It must never be exposed to the frontend, must never leave the backend execution environment, and must be stored as a backend-only secret (see Secret Management).
- Apple's `response_mode=form_post` callback delivery is an adapter/transport-layer detail, normalized into the same `{provider, code, state}` shape used by every other provider before the core callback flow runs. It never changes the public API contract (see `authentication.policy.v1.md` — "Callback Transport Normalization").
- Apple may return `name` and `email` only on the first authorization; this data must be captured at that moment, normalized into `email` and `provider_profile`, because it will not be returned again on subsequent sign-ins.
- Apple may return a private relay email. A relay email is valid optional metadata, stored identically to a direct email, and must never be used for identity matching.
- Apple access tokens, refresh tokens, and ID tokens must never be stored in the database.
- The Apple `email` claim must never be the canonical identity key. The system must never depend on Apple `email` for identity matching, account lookup, or duplicate detection.
- If the token is expired or invalid, the request must be rejected with `401`.
- A successfully verified Apple identity is considered verified immediately.

---

## Provider Linking, Unlinking, and Identity Merge Security

- Linking a provider requires the user to already be logged in and to explicitly start provider linking (`intent=link`); a provider must never be linked from unverified client-supplied data, and never inferred automatically.
- Duplicate-identity detection (lookup by `(provider, provider_subject)`) must execute before linking completes, both at `handleOAuthCallback` and again, defense-in-depth, at `POST /account/auth-providers/link`. A `(provider, provider_subject)` pair already bound to a different account must be rejected.
- `email` must never be used as a canonical identity, an automatic-merge signal, or a lookup key in any linking, unlinking, sign-in, or registration flow. The same email returned by two different providers must never cause an automatic merge; the identities remain separate unless a logged-in Account owner explicitly links them.
- There is no automatic account merge in V1, and no manual account merge endpoint in V1.
- Unlinking must be rejected if it would leave the account with zero `AuthenticationIdentity` records (see "No Password Handling" — V1 has no fallback authentication method, so at least one provider identity must always remain), regardless of whether the identity being removed is marked `primary_provider`.
- Unlinking must verify the target `AuthenticationIdentity` (identified by `auth_identity_id`) belongs to the authenticated account before deleting it.
- The `primary_provider` display preference, if implemented, must never be consulted by any authentication, authorization, or ownership decision, and changing or removing it must never affect authentication validity for any linked identity.

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

This outcome is achieved by two independent, complementary mechanisms — neither depends on the non-durable EventEmitter alone (see `06-event-contracts.md` — "V1 Event Durability Classification"):

1. **Session (refresh token) revocation** is durably guaranteed by the `account.deletion.cascade` job's session cleanup step, whose enqueue is guaranteed within 30 seconds by the Deletion Outbox Dispatcher regardless of process crashes (see `10-background-jobs.md` — Job 7, Job 8). A best-effort EventEmitter handler attempts this synchronously as a fast path but is never the sole guarantee.
2. **Access (JWT) revocation** cannot rely on session-table revocation alone, because a short-lived access token remains cryptographically valid until its own expiry regardless of whether the underlying Session row has been revoked. This gap is closed by the "Per-Request Account Status Check" below, which bounds it to a few seconds rather than the full access token TTL.

### Per-Request Account Status Check

Every authenticated request (bearing an access token) must, in addition to validating the JWT signature and expiry, verify that the account's `status = 'active'` before the request is allowed to proceed.

- The check reads `Account.status` from a Redis cache key `account:status:{account_id}`, TTL 5 seconds. On cache miss, it reads PostgreSQL and repopulates the cache.
- `AccountService.deleteAccount` and any administrative suspension command must invalidate this cache key as part of the same operation that changes `Account.status` (best-effort; the 5-second TTL is the actual bound, not the invalidation).
- If `Account.status` is `deleted` or `suspended`, the request is rejected (401) regardless of JWT or Session validity.
- This bounds the maximum window during which a still-valid access token can be used against a deleted or suspended account to at most 5 seconds, independent of session revocation timing or access token TTL.

**Session revocation triggers:**

- User-initiated logout (`POST /auth/logout`)
- User-initiated logout-all (`POST /auth/logout-all`)
- Account deletion (all sessions)
- Administrative suspension (all sessions)

---

## Token Rules

| Token | Rule |
|---|---|
| Access token | JWT, signed HS256 with a server-held secret (rotation: manual, operational — no automated key-rotation schedule in V1). TTL 15 minutes. Claims: `{sub: account_id, session_id, iat, exp, iss: "minime.ae", aud: "minime.ae"}`. Not stored in PostgreSQL or Redis. |
| Refresh token | Opaque random value, rotated on every use. TTL 30 days from last use. Only the hash is stored (`Session.refresh_token_hash`). |
| `state` / `nonce` | Single-use, short-lived, provider-agnostic controls. Generated at `startOAuth`; consumed at `handleOAuthCallback`. Bound to `provider` and `intent`. Must not be reused. Must not be stored in PostgreSQL. |
| `oauth_handoff_token` | Short-lived, single-use. Issued by backend after a validated OAuth callback with no matching account; consumed by registration completion. Carries only `{provider, provider_subject, email, email_verified, provider_profile}` — never raw provider tokens. Must not be stored in PostgreSQL or Redis beyond its TTL. |
| `link_handoff_token` | Short-lived, single-use. Issued by backend after a validated OAuth callback (`intent=link`) for an identity not yet linked to any account; bound to the linking account's `account_id`; consumed by `POST /account/auth-providers/link`. Carries only `{account_id, provider, provider_subject, email, email_verified, provider_profile}` — never raw provider tokens. Must not be stored in PostgreSQL or Redis beyond its TTL. |
| Provider access/refresh tokens | Never persisted anywhere in V1, beyond the duration of the single backend-to-provider exchange call inside the resolved Provider Adapter. |

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
- The public `GET /qr/{qr_code_id}` redirect route must never expose internal Account data; on failure it must return a generic not-found result.
- The `AccountQRCode.canonical_asset_id` storage key must never be exposed in API responses; clients receive only the constructed public asset URL via `StoragePlatform.buildPublicUrl`.
- Authentication credentials (`oauth_handoff_token`, `link_handoff_token`, `state`, `nonce`, refresh tokens) must never appear in API responses beyond the minimal disclosure required by the auth flow.
- `provider_subject` must never appear in any client-facing API response.
- `provider_profile` is non-authoritative provider metadata; if any part of it is surfaced to the account owner (e.g. in Settings), it must be presented as informational only and must never be treated as verified or authoritative data.

---

## Secret Management

Secrets must exist only in environment variables. Secrets must never be committed to source control.

Secrets include:

- JWT signing secret
- Database credentials
- Redis credentials
- Object Storage credentials
- Google OAuth client credentials (`client_id`, `client_secret`) — owned by the Google Provider Adapter
- Apple OAuth client credentials (Services ID `client_id`, Team ID, Key ID, and the Apple private key used to sign the `client_secret` JWT server-side at request time) — owned by the Apple Provider Adapter
- `AI_API_KEY` — AI provider API key

OAuth provider secrets are scoped per Provider Adapter. A future adapter introduces its own secret configuration without exposing or depending on any other adapter's secrets, and without requiring `AuthService` itself to hold any provider secret.

The Apple private key must never be exposed to the frontend, must never appear in any client-side bundle, and must never leave the backend execution environment.

Secrets must never appear in logs, events, responses, or job payloads.

---

## Redis Security

Redis must not store:

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

- Authentication tokens (access or refresh)
- Provider assertions
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

- `POST /api/v1/auth/oauth/start` — by IP
- `POST /api/v1/auth/oauth/callback` — by IP
- `POST /api/v1/auth/oauth/register` — by IP
- `POST /api/v1/account/auth-providers/link` — by account_id
- `DELETE /api/v1/account/auth-providers/{auth_identity_id}` — by account_id
- `GET /api/v1/usernames/availability` — by IP
- `POST /api/v1/usernames/reservations` — by IP
- Public rendering `/{username}` — by IP
- Public redirect `GET /out/{public_id}` — by IP
- Public redirect `GET /qr/{qr_code_id}` — by IP

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

AI responses must be treated as untrusted input. AI responses must be validated before use, including conformance to the configured current `output_schema_version`; non-conforming responses mark the Analysis Session as failed and are never applied. User confirmation is required before applying AI-generated changes. AI responses must never modify business state automatically.

`AIService.analyzeProfile(account_id)` is the single entry point for AI. It must not be invoked from authentication or authorization flows. Product Domains must never call `Provider.execute` directly. `Provider.execute(request)` must time out after `AI_TIMEOUT_MS`; timeout marks the Analysis Session as failed and returns empty suggestions, never throws.

AI API keys must never appear in logs, events, or responses.

---

## External Provider Security

Connections to external providers (Google, Apple) must use encrypted transport. Provider responses must be validated before business use. External providers must not become the source of truth.

---

## Security Prohibitions

Implementation must not:

- trust client input or client-side authorization
- expose secrets, stack traces, or infrastructure details
- bypass authentication or authorization
- bypass validation
- bypass repositories
- implement password authentication (V1 has no passwords)
- implement Email OTP, phone OTP, SMS, WhatsApp OTP, Facebook Login, X Login, or LinkedIn Login, or any `AuthEmailDelivery`
- trust an `id_token` without full backend verification (signature, issuer, audience, expiry, nonce) against the provider's public keys
- trust client-side OAuth results, a client-supplied `code`, or a client-supplied `provider_subject`
- exchange an OAuth authorization code anywhere other than the backend, inside the resolved Provider Adapter
- accept a `provider` value other than `google` or `apple` on any authentication or account-provider endpoint in V1
- expose `provider` as a route/path segment on any authentication endpoint — it is always request data
- proceed past `handleOAuthCallback` when the `provider` in the request body does not match the provider bound to the stored `state` context
- expose an Apple private key, a Google/Apple `client_secret`, `state`, `nonce`, `oauth_handoff_token`, or `link_handoff_token` to the frontend beyond what each flow step requires
- persist a provider access token or refresh token (no provider token persistence in V1, for any provider, unless a future version explicitly requires it and this document is updated)
- use `email` (any provider) as the canonical identity key, a uniqueness constraint, a login identifier, or an automatic account-merge signal
- perform automatic account merge under any circumstance in V1; there is no manual account merge endpoint in V1
- allow `primary_provider` to be consulted by any authentication, authorization, or ownership decision, or allow changing it to affect authentication validity
- allow `unlinkAuthenticationProvider` to remove an account's last remaining `AuthenticationIdentity`, regardless of which identity is marked `primary_provider`
- store a top-level `display_name` field on `AuthenticationIdentity` — provider-reported names live only inside `provider_profile`
- introduce a new authentication provider without implementing the full `AuthenticationProviderAdapter` contract and explicit allowlist update — no shortcuts, no shared/generic provider handling outside the adapter boundary
- allow linking or unlinking a provider to imply Social Account ownership verification, Connected Account creation, profile link creation, or social account import
- allow AI-generated changes without explicit user confirmation
- store access tokens or refresh tokens in plain text
- expose raw `OutLink.id` publicly (only `public_id` appears in public routes)
- allow a deleted account to create a new session
- allow user-triggered QR Code regeneration, customization, replacement, reset, or deletion
- key the QR Code record by `username`; the QR Code is always keyed by `account_id`
- store `username` or a username snapshot on the QR Code record; the redirect route must always resolve `username` live through `account_id`
- expose the `AccountQRCode.canonical_asset_id` storage key in API responses
- allow an Account to reach `active` status without a successfully created and persisted `AccountQRCode` record and canonical SVG asset
- generate the QR Code lazily on first dashboard view, first settings view, first download, first share, or first public profile visit
