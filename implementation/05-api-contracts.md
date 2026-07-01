# Minime V1 — API Contracts

## Status

Canonical. Final.

---

## Architecture Authority

```
Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md
Architecture-Product-Definition/account/account.model.specification.v1.md
Architecture-Product-Definition/account/authentication.policy.v1.md
Architecture-Product-Definition/account/username.policy.v1.md
Architecture-Product-Definition/account-management/connected.accounts.specification.v1.md
Architecture-Product-Definition/profile-content/profile.content.specification.v1.md
Architecture-Product-Definition/blocks/block.system.specification.v1.md
Architecture-Product-Definition/out-links/out-link.model.specification.v1.md
Architecture-Product-Definition/analytics/analytics.model.specification.v1.md
implementation/03-canonical-data-model.md
```

---

## Part 1 — Global API Rules

### API Style

REST. JSON. UTF-8.

Every endpoint must use HTTPS.

### Versioning

All management API endpoints begin with:

```
/api/v1/
```

### Request Rules

- Every request body must be validated.
- Every authenticated request must validate the authenticated Account.
- Request DTOs must be explicit.

### Response Shape

Successful single-resource responses:

```json
{ "data": {} }
```

Successful collection responses:

```json
{ "data": [], "meta": {} }
```

Error responses:

```json
{ "error": { "code": "string", "message": "string" } }
```

Internal stack traces must never be returned.

### HTTP Status Codes

```
200   Successful read
201   Successful creation
204   Successful operation, no response body
400   Invalid request (malformed input)
401   Authentication required or invalid
403   Authenticated but not authorized
404   Resource does not exist
409   Business constraint violation (conflict)
422   Validation failure
429   Rate limit exceeded
500   Unexpected server error
```

### HTTP Methods

- `GET` — read; must be idempotent
- `POST` — create or action
- `PATCH` — partial update
- `DELETE` — delete; must be idempotent

`PUT` is not used.

### URL Rules

- Lowercase kebab-case
- Resource-oriented paths
- Resource identifiers in path parameters
- Filtering, sorting, pagination in query parameters
- No profile_id in any URL

### Authentication Contract

- Protected endpoints require a valid Bearer access token.
- Authentication executes before authorization.
- Public endpoints do not require authentication.
- Authentication failure returns `401 Unauthorized`.

### Authorization Contract

- Authorization executes after successful authentication.
- Authorization verifies Account ownership before accessing private resources.
- Authorization failure returns `403 Forbidden`.

### Ownership Verification

Every private endpoint verifies the authenticated account_id matches the resource's account_id.

"Account" is the single ownership concept. There is no User entity. There is no Profile entity.

### Pagination

Collection endpoints support:

```
?page=1&limit=20
```

Response meta:

```json
{ "page": 1, "limit": 20, "total": 120, "pages": 6 }
```

### Rate Limiting

Authentication endpoints must be rate limited.

Violations return `429 Too Many Requests`.

### File Upload

Avatar and image block uploads use:

```
Content-Type: multipart/form-data
```

Binary content stored in Object Storage. Only the storage key is persisted in PostgreSQL.

### Execution Flow

Every request must follow:

```
HTTP Request
    ↓
Controller
    ↓
Validation
    ↓
Authentication
    ↓
Authorization
    ↓
Service
    ↓
Repository → PostgreSQL
    ↓
Publish Event
    ↓
HTTP Response
```

No endpoint may bypass this flow.

### API Prohibitions

- No internal database identifiers exposed beyond approved entity ids
- No repository implementations exposed
- No Prisma models exposed
- No stack traces
- No secrets
- No access tokens after the authentication response
- No profile_id in any request or response field

---

## Part 2 — Authentication Domain API

**Base path:** `/api/v1/auth`

**Technology note:** Google Sign-In and Apple Sign-In are the only V1 authentication methods, implemented as standard OAuth/OIDC authorization-code flows with backend token exchange. Email OTP, Phone OTP, SMS, WhatsApp, Password, Facebook Login, X Login, and LinkedIn Login are not supported.

**Route stability note:** Every route in this section is provider-agnostic. `provider` is always request data (JSON body), never part of the path. This route structure must never change when a future provider adapter is added — only the V1 allowlist (`google`, `apple`) changes, and only by configuration, not by redesigning these routes. See `authentication.policy.v1.md` — "Provider Adapter Architecture".

---

### POST /api/v1/auth/oauth/start

Start an OAuth authorization-code flow with the given provider.

**Authentication:** Not required for `intent=authenticate`. Required for `intent=link`.

**Request:**

```json
{
  "provider": "google",
  "intent": "authenticate"
}
```

`provider` must be `google` or `apple`. `intent` is `"authenticate"` (default) or `"link"`.

**Response:**

```json
{
  "data": {
    "authorization_url": "string",
    "state": "string"
  }
}
```

**Errors:** `400` (invalid `provider` or `intent`), `401` (if `intent=link` without a valid session)

**Rules:**
- Calls `AuthService.startOAuth({provider, intent, account_id?})`.
- `provider` is validated against the V1 allowlist before any Provider Adapter is resolved. An unsupported value is rejected with `400` — there is no per-provider route to 404 on, since the route itself never varies by provider.
- Generates a single-use `state` value and a `nonce`; persists both server-side with a short TTL, bound to `provider`, `intent`, and (for `link`) to `account_id`. This is the "state context" referenced by the callback endpoint.
- The frontend navigates the browser to `authorization_url`. That URL includes `client_id`, `redirect_uri`, `response_type=code`, `scope`, `state`, and `nonce`. It never includes a client secret.
- `redirect_uri` is adapter-owned configuration; it may point at a provider-specific transport relay (see Apple note below) but always results in exactly one call to `POST /api/v1/auth/oauth/callback`.

---

### POST /api/v1/auth/oauth/callback

Complete an OAuth authorization-code flow for any supported provider. This single route handles every provider; it never changes shape when a new provider adapter is added.

**Apple transport note:** Apple delivers its callback via `response_mode=form_post` directly to Apple's configured redirect URI, rather than as a browser-navigable redirect with query parameters. This is an adapter-specific transport detail, not a backend route difference: the redirect target normalizes the form-encoded payload into the same `{provider, code, state}` JSON shape before this endpoint is ever called. The backend contract below is identical for every provider.

**Authentication:** Not required (state-bound, not session-bound).

**Request:**

```json
{
  "provider": "apple",
  "code": "string",
  "state": "string"
}
```

**Response (existing account, sign-in):**

```json
{
  "data": {
    "access_token": "jwt",
    "refresh_token": "string",
    "account": { "id": "uuid", "username": "string" }
  }
}
```

**Response (new account — no existing identity):**

```json
{
  "data": {
    "needs_registration": true,
    "oauth_handoff_token": "string"
  }
}
```

**Response (`intent=link`, success):**

```json
{
  "data": {
    "needs_linking": true,
    "link_handoff_token": "string"
  }
}
```

**Errors:** `400`, `401`, `409` (duplicate identity — already linked to a different account), `422`, `429`

**Rules:**
- Calls `AuthService.handleOAuthCallback({provider, code, state})`.
- Look up the persisted state context by `state`. Reject on missing, expired, or already-consumed state (CSRF + replay protection).
- Validate that `provider` in the request body matches the provider bound to the state context. Reject on mismatch — a request claiming a different provider than the one `startOAuth` recorded for this `state` must never proceed.
- Reject if `provider` is not in the V1 allowlist, independently of the check already performed at `startOAuth` time.
- Resolve the Provider Adapter for `provider`; exchange `code` for tokens via a backend-to-provider call only (never client-side); for Apple, the `client_secret` used here is a JWT signed server-side with the Apple private key (adapter-owned secret).
- Validate the returned `id_token` inside the adapter: signature, issuer, audience, expiry, and `nonce` match.
- Normalize the validated identity into `{provider, provider_subject, email?, email_verified?, provider_profile?}` only after successful validation.
- Perform duplicate-identity detection by `(provider, provider_subject)` before creating any Account, session, or linked identity. `email` is never consulted for this lookup.
- `intent=authenticate`: existing identity → create session, return tokens; no identity → return `needs_registration: true` with a short-lived, single-use `oauth_handoff_token` (never the raw provider tokens).
- `intent=link`: requires the authenticated `account_id` bound to the state context at `startOAuth` time; reject with `409` if `(provider, provider_subject)` already belongs to a different account; if it already belongs to the same account, call `AuthService.refreshOAuthIdentityMetadata` and return the sign-in response shape; otherwise return `needs_linking: true` with a short-lived, single-use `link_handoff_token`, to be finalized via `POST /api/v1/account/auth-providers/link`.
- Provider access tokens and refresh tokens must never be stored in PostgreSQL or Redis.
- Publish `auth.provider.completed` on success, `auth.provider.failed` on failure.

---

### POST /api/v1/auth/oauth/register

Complete registration using a validated OAuth handoff token + username reservation.

**Authentication:** Not required.

**Request:**

```json
{
  "provider": "google",
  "oauth_handoff_token": "string",
  "reservation_id": "uuid"
}
```

**Response:**

```json
{
  "data": {
    "access_token": "jwt",
    "refresh_token": "string",
    "account": { "id": "uuid", "username": "string" }
  }
}
```

**Errors:** `400`, `401`, `409`, `422`, `429`

**Rules:**
- `provider` (body) must be validated against the V1 allowlist and must match the provider embedded in `oauth_handoff_token`; mismatch is rejected with `400`.
- Validate `oauth_handoff_token` (issued by `POST /auth/oauth/callback`); reject if expired or already used.
- Verify reservation exists and has not expired.
- Calls `AuthService.completeRegistrationWithOAuth({reservation_id, oauth_handoff_token})`, which creates Account + AuthenticationIdentity (`provider`, `provider_subject`, `email`, `email_verified`, `provider_profile` from the validated token) + ProfileContent in one transaction.
- Immediately and synchronously create the Account's `AccountQRCode` record and canonical SVG asset (`AccountService.createQrCode`) as a required blocking step. The Account must not be treated as active, and this endpoint must not return success, unless QR Code creation succeeds.
- Delete UsernameReservation.
- Create Session; issue tokens.
- Publish `auth.registration.completed`.

---

### POST /api/v1/auth/session/refresh

Rotate refresh token; issue new access token.

**Authentication:** Refresh token (in request body).

**Request:**

```json
{ "refresh_token": "string" }
```

**Response:**

```json
{
  "data": {
    "access_token": "jwt",
    "refresh_token": "string"
  }
}
```

**Errors:** `400`, `401`, `429`

**Rules:**
- Validate refresh token against stored hash.
- Reject expired or revoked sessions.
- Rotate refresh token (old hash invalidated, new hash stored).
- Issue new access token.
- Publish `auth.session.refreshed`.

---

### POST /api/v1/auth/logout

Revoke the current session.

**Authentication:** Required.

**Request:** None.

**Response:**

```json
{ "data": { "status": "logged-out" } }
```

**Errors:** `401`, `403`

**Rules:**
- Revoke current refresh token (set Session.revoked_at).
- Publish `auth.logout`.

---

### POST /api/v1/auth/logout-all

Revoke all active sessions for the account.

**Authentication:** Required.

**Request:** None.

**Response:**

```json
{ "data": { "status": "all-sessions-revoked" } }
```

**Errors:** `401`, `403`

**Rules:**
- Set revoked_at on all active, non-expired sessions for the account.
- Publish `auth.session.revoked` (one event covers all).

---

### GET /api/v1/auth/sessions

List active sessions for the account.

**Authentication:** Required.

**Response:**

```json
{ "data": [] }
```

**Errors:** `401`, `403`

**Rules:**
- Return only sessions owned by the authenticated account.
- Exclude revoked and expired sessions.
- Never expose refresh tokens or internal session identifiers.

---

### Authentication API Prohibitions

- No Email OTP endpoints of any kind: no `/auth/otp/request`, no `/auth/otp/verify`, no `/auth/email/*`, no `/claim/complete-with-otp`, no `/auth/session` with a `verification_token`.
- No phone OTP, SMS, or WhatsApp OTP endpoints.
- No password endpoints (no `/auth/password/*`, no `/auth/login` with a password body).
- No Facebook, X, or LinkedIn login endpoints.
- No provider-specific route segments of any kind: no `/auth/{provider}/start`, no `/auth/{provider}/callback`, no `/auth/register/{provider}`, no `/auth/google/*`, no `/auth/apple/*`, and no equivalent for any future provider. `provider` is always request data inside `POST /auth/oauth/start`, `POST /auth/oauth/callback`, and `POST /auth/oauth/register` — never a path segment.
- No `provider` value is accepted beyond the V1 allowlist (`google`, `apple`) on any route — unsupported values are rejected with `400` before any Provider Adapter is resolved or any provider call is made.
- The route structure defined above must never change when a future provider is added; only the allowlist changes.
- Never expose `oauth_handoff_token` or `link_handoff_token` beyond the single response each is issued in.
- Never expose a `state`, `nonce`, or `code_verifier` value to any client other than the one that initiated the flow.
- Never expose refresh tokens except during session creation or rotation.
- Never expose `provider_subject` in any response. It is an internal identity key, not a client-facing value.

---

## Part 3 — Username Domain API

**Base path:** `/api/v1/usernames`

Username is set at registration and is not editable in V1. These endpoints support the registration flow only.

---

### GET /api/v1/usernames/availability

Check if a username is available.

**Authentication:** Not required.

**Query parameters:** `?username=string`

**Response:**

```json
{ "data": { "username": "string", "available": true } }
```

**Errors:** `400`, `422`, `429`

**Rules:**
- Validate format: a-z/0-9, length 3–30, lowercase only.
- Check against reserved block lists.
- Check Account.username (permanent reservation) and UsernameReservation (temporary).
- Do not expose the owner of an unavailable username.
- Do not create a reservation.

---

### POST /api/v1/usernames/reservations

Reserve a username as part of the registration flow.

**Authentication:** Not required.

**Request:**

```json
{ "username": "string" }
```

**Response:**

```json
{
  "data": {
    "reservation_id": "uuid",
    "username": "string",
    "expires_at": "datetime"
  }
}
```

**Errors:** `400`, `409`, `422`, `429`

**Rules:**
- Validate format.
- Validate availability.
- Create UsernameReservation.
- Return reservation_id for use in POST /auth/oauth/register.
- Publish `username.reserved`.

---

### Username API Prohibitions

- No `PATCH /api/v1/usernames` endpoint. Username changes are not supported in V1.
- No `POST /api/v1/usernames` (assign) endpoint. Assignment happens through registration.
- No endpoint that reveals the Account owner of a username.

---

## Part 4 — Account Domain API

**Base path:** `/api/v1/account`

All endpoints require authentication.

---

### GET /api/v1/account

Get the authenticated account.

**Response:**

```json
{
  "data": {
    "id": "uuid",
    "username": "string",
    "status": "active",
    "created_at": "datetime",
    "updated_at": "datetime"
  }
}
```

**Errors:** `401`, `403`, `404`

**Rules:**
- Return the authenticated account only.
- Do not return deleted accounts.
- Do not return authentication tokens.
- Do not return profile content.
- No `user_id` field. No `profile_id` field.

---

### GET /api/v1/account/settings

Get account settings.

**Response:**

```json
{ "data": { "settings": {} } }
```

**Errors:** `401`, `403`

**Rules:**
- Return Account.settings only.
- Do not return appearance configuration (that is ProfileContent).

---

### PATCH /api/v1/account/settings

Update account settings.

**Request:**

```json
{ "settings": {} }
```

**Response:**

```json
{ "data": { "settings": {} } }
```

**Errors:** `400`, `401`, `403`, `422`

**Rules:**
- Validate settings payload.
- Update Account.settings only.
- Publish `account.settings.updated` after persistence.

---

### GET /api/v1/account/qr-code

Get the account's QR Code record and asset URL. Read-only.

**Response:**

```json
{
  "data": {
    "qr_code": {
      "qr_code_id": "string",
      "qr_url": "https://minime.ae/qr/{qr_code_id}",
      "asset_url": "string"
    }
  }
}
```

**Errors:** `401`, `403`

**Rules:**
- Returns the existing `AccountQRCode` record only. Never generates, mutates, or deletes a record.
- `asset_url` is constructed via `StoragePlatform.buildPublicUrl(canonical_asset_id)`; the raw storage key is never returned.
- Used to render the QR Code directly in `Settings / Account / QR Code` and to power the Download, Share, and Copy profile link actions.
- This endpoint does not support a PNG query parameter for persistence; any PNG export is derived on read and never stored as a second record.

There is no `PATCH /api/v1/account/qr-code`, `POST /api/v1/account/qr-code/regenerate`, or `DELETE /api/v1/account/qr-code`. The QR Code is never user-editable, regenerable, or deletable.

---

### GET /api/v1/account/auth-providers

List the authenticated account's linked authentication providers.

**Response:**

```json
{
  "data": [
    { "auth_identity_id": "uuid", "provider": "google", "email": "string | null", "linked_at": "datetime", "is_primary": true },
    { "auth_identity_id": "uuid", "provider": "apple", "email": "string | null", "linked_at": "datetime", "is_primary": false }
  ]
}
```

**Errors:** `401`, `403`

**Rules:**
- Returns the account's `AuthenticationIdentity` records, keyed by `auth_identity_id` (needed to address `DELETE /account/auth-providers/{auth_identity_id}`). Never exposes `provider_subject` or any provider token.
- `email` reflects provider-reported metadata only; it is informational and may be `null`.
- `is_primary` reflects the account's `primary_provider` display preference only; see "Primary Provider" rules below. It is never used by any other endpoint to determine ownership or authorization.

---

### POST /api/v1/account/auth-providers/link

Finalize linking a provider that was just verified via the OAuth flow.

**Authentication:** Required.

**Request:**

```json
{ "link_handoff_token": "string" }
```

**Response:**

```json
{ "data": { "auth_identity_id": "uuid", "provider": "google", "status": "linked" } }
```

**Errors:** `400`, `401`, `403`, `409` (duplicate identity — already linked to a different account)

**Rules:**
- The user must already be logged in. Linking is never inferred from an unauthenticated flow.
- The flow to obtain `link_handoff_token` is: `POST /api/v1/auth/oauth/start` with `{"provider": "...", "intent": "link"}` (authenticated), followed by the provider redirect resolving to `POST /api/v1/auth/oauth/callback`. See Part 2 — Authentication Domain API.
- Calls `AuthService.linkAuthenticationProvider({account_id, link_handoff_token})`.
- Validates `link_handoff_token`: signature, expiry, single-use, and that its bound `account_id` matches the authenticated caller.
- Rejects with `409` if `(provider, provider_subject)` is already bound to a different account.
- Does not affect any other linked identity.
- Publish `auth.provider.linked`.

---

### DELETE /api/v1/account/auth-providers/{auth_identity_id}

Unlink an authentication provider identity from the account.

**Path parameters:** `auth_identity_id` — the identity record to remove, as returned by `GET /api/v1/account/auth-providers`.

**Response:**

```json
{ "data": { "status": "unlinked", "auth_identity_id": "uuid" } }
```

**Errors:** `401`, `403` (identity does not belong to the caller), `404` (identity not found), `409` (last remaining identity)

**Rules:**
- Calls `AuthService.unlinkAuthenticationProvider({account_id, auth_identity_id})`.
- Must verify the identity belongs to the authenticated account before deleting it.
- Must reject with `409` if this is the account's last remaining `AuthenticationIdentity`, regardless of whether it is marked `primary_provider`.
- Does not affect the account, its data, or any other linked identity.
- Does not revoke existing Sessions created via the unlinked provider; Session validity is independent of `AuthenticationIdentity` existence once issued, per `08-security-model.md`.
- Publish `auth.provider.unlinked`.

---

### Account Authentication Provider API Prohibitions

- No endpoint accepts a raw `provider_subject` from the client. It is always derived from a verified `id_token` during an OAuth callback.
- No endpoint allows unlinking the last remaining provider.
- No endpoint allows linking a provider value other than `google` or `apple` in V1.
- No endpoint allows `email` to be supplied or used to find, match, or merge an account. Identity resolution is always `(provider, provider_subject)`.
- No endpoint performs automatic account merge. There is no manual account merge endpoint in V1.
- No endpoint allows `DELETE /account/auth-providers/{auth_identity_id}` for an identity not owned by the caller.

---

### POST /api/v1/account/deletion

Delete account. Immediate and final. There is no grace period and no cancellation.

**Request:**

```json
{ "confirmation": true }
```

**Response:**

```json
{ "data": { "status": "deleted" } }
```

**Errors:** `400`, `401`, `403`, `409`, `422`

**Rules:**
- `confirmation` must be `true`.
- Deletion is immediate and final. There is no grace period in V1.
- Cascade executes per `04-service-contracts.md` — AccountService.deleteAccount.
- Account record is retained permanently (referenced by audit logs).
- Username is permanently reserved; not released in V1.
- Publish `account.deleted` after persistence.

---

### Account API Prohibitions

- No `POST /api/v1/account/deactivate`. `suspended` status is administrative-only.
- No `POST /api/v1/account/reactivate`. No such operation in V1.
- No `POST /api/v1/account/deletion/cancel`. Deletion is final.
- No `user_id` in any request or response.
- No `profile_id` in any request or response.

---

## Part 5 — Profile Domain API

**Base path:** `/api/v1/profile`

All endpoints require authentication. Authorization verifies account ownership.

Profile content belongs to Account. There is no Profile entity and no profile_id.

---

### GET /api/v1/profile

Get the authenticated account's profile content and blocks.

**Response:**

```json
{
  "data": {
    "display_name": "string",
    "bio": "string | null",
    "avatar_url": "string | null",
    "contact": {
      "email": "string | null",
      "phone": "string | null",
      "whatsapp": "string | null",
      "location": "string | null"
    },
    "appearance": {
      "selected_theme_id": "string | null",
      "customizations": {}
    }
  }
}
```

**Errors:** `401`, `403`, `404`

**Rules:**
- Return the authenticated account's ProfileContent.
- Do not return deleted blocks here; blocks are a separate endpoint.
- No `profile_id` in response.

---

### PATCH /api/v1/profile

Update profile content fields.

**Request:**

```json
{
  "display_name": "string",
  "bio": "string | null",
  "contact": {
    "email": "string | null",
    "phone": "string | null",
    "whatsapp": "string | null",
    "location": "string | null"
  }
}
```

**Response:**

```json
{ "data": { "display_name": "string", "bio": "string | null", "contact": {} } }
```

**Errors:** `400`, `401`, `403`, `422`

**Rules:**
- Validate bio length per `07-validation-rules.md`.
- Update ProfileContent fields only.
- Invalidate rendering cache (`profile:public:{username}`) after persistence.
- Publish `profile.updated` after persistence.

---

### POST /api/v1/profile/avatar

Upload or replace avatar image.

**Content-Type:** `multipart/form-data`

**Request:** Binary image file.

**Response:**

```json
{ "data": { "avatar_url": "string" } }
```

**Errors:** `400`, `401`, `403`, `413`, `422`

**Rules:**
- Validate file type and file size.
- Upload to Object Storage via StoragePlatform.
- Update ProfileContent.avatar_storage_key with returned storage key.
- Invalidate rendering cache.
- Publish `profile.avatar.updated` after persistence.

---

### POST /api/v1/profile/images

Upload an image for use in an Image Block. Returns the `image_id` to include in the block's content.

**Authentication:** Required.

**Content-Type:** `multipart/form-data`

**Request:** Binary image file.

**Response:**

```json
{
  "data": {
    "image_id": "uuid",
    "mime_type": "string",
    "width": 0,
    "height": 0,
    "file_size": 0
  }
}
```

**Errors:** `400`, `401`, `403`, `413`, `422`

**Rules:**
- Validate file MIME type and file size per `07-validation-rules.md` — ImageAsset constants.
- Upload to Object Storage via StoragePlatform (assetType: 'image').
- Create ImageAsset record; return `image_id`.
- Storage key must never appear in the response.
- Publish `profile.image.uploaded` after persistence.

---

### DELETE /api/v1/profile/images/{imageId}

Delete an image asset. Only allowed when no active block references the image.

**Authentication:** Required.

**Response:**

```json
{ "data": { "status": "deleted" } }
```

**Errors:** `401`, `403`, `404`, `409`

**Rules:**
- Validate ownership via account_id.
- Reject with `409` if any active (non-soft-deleted) block references this `image_id`. The user must delete the block first.
- Hard Delete: delete ImageAsset record and call StoragePlatform.delete(storage_key).
- Publish `profile.image.deleted` after persistence.

---

### PATCH /api/v1/profile/appearance

Update appearance configuration.

**Request:**

```json
{
  "selected_theme_id": "string | null",
  "customizations": {}
}
```

**Response:**

```json
{
  "data": {
    "selected_theme_id": "string | null",
    "customizations": {}
  }
}
```

**Errors:** `400`, `401`, `403`, `422`

**Rules:**
- Validate selected_theme_id exists in theme catalog.
- Update ProfileContent.appearance_config.
- Invalidate rendering cache.
- Publish `profile.appearance.updated` after persistence.

---

### GET /api/v1/profile/blocks

Get all active blocks for the authenticated account.

**Response:**

```json
{ "data": [] }
```

**Errors:** `401`, `403`

**Rules:**
- Return blocks ordered by sort_order ascending.
- Exclude soft-deleted blocks (deleted_at IS NOT NULL).

---

### POST /api/v1/profile/blocks

Add a block.

**Request:**

```json
{
  "type": "button",
  "content": { "label": "string", "url": "string" },
  "settings": {},
  "sort_order": 1
}
```

**Response:**

```json
{ "data": { "block": {} } }
```

**Errors:** `400`, `401`, `403`, `409`, `422`

**Rules:**
- Validate `type` against approved BlockType enum.
- Validate `content` and `settings` against per-type schema (from 03-canonical-data-model.md).
- Enforce identity block instance limit: max 1 per account for avatar, name, bio types.
- Enforce MAX_BLOCKS_PER_ACCOUNT total active blocks per account. Reject with `409` if limit is reached.
- Invalidate rendering cache after persistence.
- Publish `profile.block.added` after persistence.
- Field names: `type` (not `blockType`), `content` (not `data`), `sort_order` (not `displayOrder`).

---

### PATCH /api/v1/profile/blocks/{blockId}

Update a block's content or settings.

**Request:**

```json
{
  "content": {},
  "settings": {}
}
```

**Response:**

```json
{ "data": { "block": {} } }
```

**Errors:** `400`, `401`, `403`, `404`, `422`

**Rules:**
- Validate block ownership via account_id.
- Validate content and settings against per-type schema.
- `type` and `sort_order` are not updatable here.
- Invalidate rendering cache after persistence.
- Publish `profile.block.updated` after persistence.

---

### DELETE /api/v1/profile/blocks/{blockId}

Soft delete a block.

**Response:**

```json
{ "data": { "status": "deleted" } }
```

**Errors:** `401`, `403`, `404`, `409`

**Rules:**
- Validate block ownership via account_id.
- Set Block.deleted_at (soft delete).
- Invalidate rendering cache after persistence.
- Publish `profile.block.deleted` after persistence.

---

### PATCH /api/v1/profile/blocks/order

Reorder blocks.

**Request:**

```json
{
  "blocks": [
    { "block_id": "uuid", "sort_order": 1 },
    { "block_id": "uuid", "sort_order": 2 }
  ]
}
```

**Response:**

```json
{ "data": { "blocks": [] } }
```

**Errors:** `400`, `401`, `403`, `422`

**Rules:**
- Validate all block_ids belong to the authenticated account.
- Update all sort_order values in one transaction.
- Invalidate rendering cache after persistence.
- Publish `profile.block.reordered` after persistence.

---

### Cache Invalidation

Profile mutations must invalidate profile cache per `09-caching-strategy.md`. Invalidation executes after successful persistence, not before.

---

### Profile API Prohibitions

- No `POST /api/v1/profile/publish` endpoint.
- No `POST /api/v1/profile/unpublish` endpoint.
- No `GET /api/v1/profile/preview` endpoint. There is no draft/published concept.
- No `profile_id` in any request or response.
- No `blockType` or `displayOrder` field names. Use `type` and `sort_order`.
- No `data` field for block content. Use `content` and `settings`.
- No `GET /api/v1/profile/images` listing endpoint. There is no image browser or media library.
- No image search, tagging, folder, album, or gallery endpoint.
- No image editing, cropping, or transformation endpoint.
- Storage keys must never appear in any image-related API response.

---

## Part 6 — Connected Accounts Domain API

**Base path:** `/api/v1/connected-accounts`

Connected Accounts are user-entered social links stored as `{platform, username, url, sort_order}`. These are not OAuth provider connections.

All endpoints require authentication.

---

### GET /api/v1/connected-accounts

List the authenticated account's connected accounts.

**Response:**

```json
{ "data": [] }
```

**Errors:** `401`, `403`

**Rules:**
- Return ConnectedAccounts ordered by sort_order.
- All belong to the authenticated account.
- Do not expose OAuth tokens, provider secrets, or connection status.

---

### GET /api/v1/connected-accounts/{id}

Get one connected account.

**Response:**

```json
{
  "data": {
    "id": "uuid",
    "platform": "instagram",
    "username": "string",
    "url": "string",
    "sort_order": 1
  }
}
```

**Errors:** `401`, `403`, `404`

**Rules:**
- Validate ownership via account_id.

---

### POST /api/v1/connected-accounts

Add a social link. The system validates the handle and generates the canonical URL.

**Request:**

```json
{
  "platform": "instagram",
  "username": "string"
}
```

**Response:**

```json
{
  "data": {
    "id": "uuid",
    "platform": "instagram",
    "username": "string",
    "url": "string",
    "sort_order": 1
  }
}
```

**Errors:** `400`, `401`, `403`, `409`, `422`

**Rules:**
- Validate platform against SocialPlatform enum.
- Validate username format against platform-specific rules (via SocialAccountsService).
- System generates canonical url from platform rules. Client must not provide url.
- Prevent duplicate `(account_id, platform, username)`.
- Publish `connected-account.added` after persistence.

---

### PATCH /api/v1/connected-accounts/{id}

Update the username for a connected account. Only username is editable.

**Request:**

```json
{ "username": "string" }
```

**Response:**

```json
{
  "data": {
    "id": "uuid",
    "platform": "instagram",
    "username": "string",
    "url": "string",
    "sort_order": 1
  }
}
```

**Errors:** `400`, `401`, `403`, `404`, `409`, `422`

**Rules:**
- Validate ownership via account_id.
- Validate new username against platform-specific rules.
- Regenerate url from platform rules.
- platform is not editable. To change platform: DELETE + POST.
- Publish `connected-account.updated` after persistence.

---

### DELETE /api/v1/connected-accounts/{id}

Remove a connected account. Hard Delete — permanent.

**Response:**

```json
{ "data": { "status": "removed" } }
```

**Errors:** `401`, `403`, `404`

**Rules:**
- Validate ownership via account_id.
- Hard Delete: no soft delete, no deleted_at, no archive.
- Publish `connected-account.removed` after deletion.

---

### PATCH /api/v1/connected-accounts/order

Reorder connected accounts.

**Request:**

```json
{
  "ordered_ids": ["uuid", "uuid", "uuid"]
}
```

**Response:**

```json
{ "data": { "connected_accounts": [] } }
```

**Errors:** `400`, `401`, `403`, `422`

**Rules:**
- Validate all IDs belong to the authenticated account.
- Update sort_order values in one transaction.
- Publish `connected-account.reordered` after persistence.

---

### Connected Accounts API Prohibitions

- No OAuth authorization code flow.
- No `POST /api/v1/connected-accounts/{provider}/connect`.
- No `POST /api/v1/connected-accounts/{id}/refresh`.
- No `POST /api/v1/connected-accounts/{id}/revoke`.
- No `connection_status`, `is_verified`, `revoked_at`, or `provider_metadata` fields.
- No `/api/v1/social-accounts` resource. Social Accounts domain has no REST API.
- Client must never provide a url field. URL is always system-generated.

---

## Part 7 — Out Links Domain API

**Public redirect path:** `/out/{public_id}`

**Management read path:** `/api/v1/out-links`

**Creation model:** OutLinks are created internally by the system when profile blocks create a Clickable State. There is no user-facing endpoint to create or archive an OutLink. Creation and archiving are triggered by ProfileService as part of block lifecycle (addBlock, updateBlock, deleteBlock).

Management read endpoints require authentication. Public redirect does not.

---

### GET /api/v1/out-links

List active out links for the authenticated account. Read-only.

**Authentication:** Required.

**Response:**

```json
{ "data": [] }
```

**Errors:** `401`, `403`

**Rules:**
- Return OutLinks with status = 'active' or 'created'.
- Do not return archived OutLinks.
- Ordered by created_at descending.

---

### GET /api/v1/out-links/{id}

Get one out link. Read-only.

**Authentication:** Required.

**Response:**

```json
{
  "data": {
    "id": "uuid",
    "public_id": "string",
    "block_id": "uuid",
    "block_type": "string",
    "destination_url": "string",
    "label_snapshot": "string | null",
    "status": "active",
    "created_at": "datetime"
  }
}
```

**Errors:** `401`, `403`, `404`

**Rules:**
- Validate ownership via account_id.
- Return internal id (for analytics lookups) and public_id (for `/out/` routing).

---

### GET /out/{public_id}

Public redirect. Resolves a managed out link and redirects the visitor. Unauthenticated.

**Response:** `302 Found` (redirect to destination_url)

**Errors:** `404 Not Found`, `410 Gone` (if archived)

**Rules:**
- Resolve by public_id.
- Return 404 if not found.
- Return 410 if status = 'archived'.
- Redirect immediately; do not wait for analytics recording.
- Enqueue BullMQ job to record `link_click` event (non-blocking, after redirect).
- Never expose the internal `id`. Routing uses `public_id` only.

---

### Out Links API Prohibitions

- No `POST /api/v1/out-links`. OutLinks are created internally by the system when Clickable States are created.
- No `POST /api/v1/out-links/{id}/archive`. Archiving is system-triggered, not user-initiated.
- No `POST /api/v1/out-links/{id}/enable`.
- No `POST /api/v1/out-links/{id}/disable`.
- No `PATCH /api/v1/out-links/{id}`. All content fields are immutable after creation.
- No `DELETE /api/v1/out-links/{id}`.
- No `is_enabled` field in any request or response.
- Do not block redirects for analytics processing.
- Redirect must use `public_id`, not internal `id`.

---

## Part 8 — Analytics Domain API

**Base path:** `/api/v1/analytics`

All endpoints require authentication.

Analytics responses contain aggregated metrics only. Raw event records and visitor identity are never exposed.

---

### GET /api/v1/analytics/summary

Get aggregated summary for the authenticated account.

**Query parameters:** `?from=date&to=date`

**Response:**

```json
{
  "data": {
    "profile_views": 0,
    "link_clicks": 0
  }
}
```

**Errors:** `401`, `403`, `422`

**Rules:**
- Return aggregated data for account_id only.
- Metrics derive from `profile_view` and `link_click` events only.
- Do not expose raw AnalyticsEvent records.

---

### GET /api/v1/analytics/profile

Get profile view analytics for the authenticated account.

**Query parameters:** `?from=date&to=date`

**Response:**

```json
{
  "data": {
    "total_views": 0,
    "by_device": {
      "desktop": 0,
      "mobile": 0,
      "tablet": 0
    },
    "by_date": []
  }
}
```

**Errors:** `401`, `403`, `422`

**Rules:**
- Validate date range.
- Aggregate profile_view events for account_id.
- device_category breakdown: See `03-canonical-data-model.md` — DeviceCategory enum.
- No visitor identity, no country, no referrer in response.

---

### GET /api/v1/analytics/out-links/{id}

Get click analytics for a specific out link.

**Query parameters:** `?from=date&to=date`

**Response:**

```json
{
  "data": {
    "out_link_id": "uuid",
    "total_clicks": 0,
    "by_date": []
  }
}
```

**Errors:** `401`, `403`, `404`, `422`

**Rules:**
- Validate OutLink ownership via account_id.
- Validate date range.
- Aggregate link_click events for out_link_id.
- Do not expose raw event records.

---

### Analytics API Prohibitions

- No public endpoint for event ingestion (analytics recording is internal).
- No raw AnalyticsEvent records in responses.
- No visitor identity (no username, no IP, no session).
- No country or referrer fields in responses.
- No `qr_scan` or `out_link_click` event types. Valid types: See `03-canonical-data-model.md` — AnalyticsEventType enum.

---

## Part 9 — AI Domain API

**Base path:** `/api/v1/account/ai`

All endpoints require authentication and operate only on the authenticated account. There is no way to request analysis for another account.

This is a synchronous API: `POST /api/v1/account/ai/analysis` blocks until the analysis completes, fails, or reuses a prior result — it does not return a `pending` status to the client. `AnalysisSession.status = 'pending'` (`03-canonical-data-model.md`) is an internal transient row state during execution of that single request; it is never observed by the client. There is no polling endpoint in V1 because there is nothing to poll for — the request either finishes or errors.

---

### POST /api/v1/account/ai/analysis

Trigger "Analyze My Profile." Maps directly to `AIService.analyzeProfile({account_id})` (`04-service-contracts.md` / `11-platform-services.md` — AIService Interface).

**Request body:** None. The account is taken from the authenticated session; there is no client-supplied input.

**Timeout:** The client and any intermediating proxy must allow at least 60 seconds for this request. This is a long-running synchronous call, not a fire-and-forget trigger.

**Response (200 — new analysis executed, or prior result reused):**

```json
{
  "data": {
    "analysis_session_id": "uuid",
    "status": "completed",
    "reused": false,
    "report": {},
    "scores": {},
    "suggestions": [ { "area": "string", "suggestion": "string", "reason": "string" } ],
    "recommendations": {},
    "completed_at": "2026-01-01T00:00:00Z"
  }
}
```

`reused: true` indicates Analysis Session Reuse occurred (matching `input_hash`, `analysis_version`, and `output_schema_version` against the latest completed session) rather than a new provider call — see `ai.architecture.specification.v1.md` — "Analysis Session."

**Response (200 — analysis failed: provider timeout, provider error, or invalid structured output):**

```json
{
  "data": {
    "analysis_session_id": "uuid",
    "status": "failed",
    "suggestions": []
  }
}
```

**Response (200 — AI disabled: `AI_API_KEY` absent or empty):**

```json
{
  "data": {
    "status": "disabled",
    "suggestions": []
  }
}
```

Note there is no `analysis_session_id` in the `disabled` case — no Analysis Session row is ever created when AI is disabled, unlike `failed`, which always has a persisted (failed) session (`AIService.analyzeProfile`, `11-platform-services.md`).

Both `failed` and `disabled` are `200`, not error statuses — neither is a client error, and neither must surface as an HTTP error or block the rest of the product. The frontend distinguishes three empty-`suggestions` cases by `status` alone, never by the emptiness of `suggestions`: `completed` with `suggestions: []` (nothing to improve — a real, positive result), `failed` (something went wrong, offer retry), and `disabled` (the feature is off, do not offer retry).

**Errors:** `401`, `403`, `429` (see "Rate Limiting" below)

**Rules:**
- Exactly one `POST` call executes exactly one `AIService.analyzeProfile` invocation; there is no batching or queuing of multiple analyses per request.
- A confirmed AI suggestion is never applied automatically by this endpoint or by any AI endpoint. The user must separately call the relevant Product Domain endpoint (e.g. `PATCH /api/v1/profile`) to act on a suggestion, exactly as if they had typed the change themselves. There is no `applySuggestion` or `acceptSuggestion` endpoint in V1.

---

### GET /api/v1/account/ai/analysis/latest

Return the latest completed Analysis Session for the authenticated account, for display without re-running analysis (e.g. on page reload). Read-only; never triggers a new analysis and never calls `Provider.execute`.

**Response (200):**

```json
{
  "data": {
    "analysis_session_id": "uuid",
    "status": "completed",
    "report": {},
    "scores": {},
    "suggestions": [],
    "recommendations": {},
    "completed_at": "2026-01-01T00:00:00Z"
  }
}
```

**Response (404):** No completed Analysis Session exists yet for this account (the user has never run "Analyze My Profile," or every prior attempt failed).

**Errors:** `401`, `403`, `404`

**Rules:**
- Returns only `status = 'completed'` sessions. A `failed` session is never returned here — the client already received the `failed` outcome synchronously from the `POST` call that produced it, and a failed session is not meaningful to redisplay later.
- Selects the most recent `completed` session by `completed_at`, regardless of whether it is still "reusable" (matching the current `input_hash`/`analysis_version`/`output_schema_version`) — this endpoint displays history, it does not determine reuse eligibility. Reuse eligibility is decided only inside `AIService.analyzeProfile` on the next `POST`.

---

### Rate Limiting

`POST /api/v1/account/ai/analysis` is rate-limited per account because each non-reused call has a real provider cost. Limit: 10 requests per account per rolling 24-hour window. Exceeding it returns `429` with `{ "error": { "code": "ai_rate_limited", "message": "string" } }`. `GET /api/v1/account/ai/analysis/latest` is not rate-limited (read-only, no provider cost).

---

### AI API Prohibitions

- No endpoint accepts free-text prompts, custom instructions, or provider/model selection from the client — `AIService` owns all of that internally (`ai.architecture.specification.v1.md`).
- No endpoint exposes `provider_key` or `model_key` — those are internal diagnostic fields only (`11-platform-services.md` — Analysis Session Entity).
- No `applySuggestion`, `acceptSuggestion`, `rejectSuggestion`, or `dismissSuggestion` endpoint exists in V1 — per-suggestion tracking is V2 scope (`platform/data/canonical.entities.map.v1.md` — §22).
- No polling endpoint (`GET .../analysis/{id}/status` or similar) exists in V1 — see the synchronous-API note above.

---

## Part 10 — Public Rendering

**Path:** `/{username}`

The public profile page. Rendered server-side. Outside the `/api/v1/` base path.

**Authentication:** Not required.

**Response:** Rendered HTML profile page.

**Errors:** `404 Not Found` (username does not exist or account is deleted)

**Rules:**
- Resolve username to an active Account.
- Return 404 for deleted or suspended accounts.
- RenderingService assembles profile from ProfileContent, Blocks, ConnectedAccounts, OutLinks, ImageAssets (all read-only).
- For `button` blocks, the tracked link uses `/out/{public_id}` for the single OutLink.
- For `social_icons` blocks, each social icon renders with its own `/out/{public_id}` tracked link. Raw destination URLs must not appear in public render payloads for tracked links. Icons with no active OutLink are omitted.
- Response is cached per `09-caching-strategy.md`.
- No publishing state. The rendered output always reflects current persisted state.
- There is no preview path. The authenticated dashboard shows live data.

---

### GET /qr/{qr_code_id}

Public QR Code redirect. Resolves a QR Code record and redirects the visitor to the owning Account's current public profile. Outside the `/api/v1/` base path. Unauthenticated.

**Response:** `302 Found` (redirect to `/{username}`)

**Errors:** `404 Not Found`

**Rules:**
- Resolve `qr_code_id` to its `AccountQRCode` record via `AccountService.resolveQrCode`.
- Resolve `account_id` from the record, then read `Account.username` live through `account_id` — the QR Code record never stores `username` or a username snapshot.
- Resolution flow: `qr_code_id → AccountQRCode → account_id → Account.username → redirect to /{username}`.
- Redirect to `/{username}`. Never render a separate public profile page at this route.
- Return `404` if the QR Code record does not exist, the Account is deleted or suspended, or no active public profile route resolves. The response must never expose internal Account data.
- The route parameter is `qr_code_id`, never `username`.
- This route performs no analytics recording and no scan tracking in V1.
