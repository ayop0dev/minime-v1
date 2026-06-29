# Minime V1 — API Contracts

## Status

Reconciled. Supersedes `06-api-contracts.md` and `14-` through `21-` domain API files.

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

Authentication and OTP endpoints must be rate limited.

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

**Technology note:** Email OTP and Google Sign-In are the only V1 authentication methods. Phone OTP is not supported.

---

### POST /api/v1/auth/otp/request

Request an email OTP. Rate limited.

**Authentication:** Not required.

**Request:**

```json
{ "email": "string" }
```

**Response:**

```json
{ "data": { "status": "otp-requested" } }
```

**Errors:** `400`, `422`, `429`

**Rules:**
- Validate email format.
- Rate limit by identifier.
- Do not reveal whether an Account exists.
- Publish `auth.otp.requested` after OTP creation.
- OTP format, lifetime, and resend cooldown: See `07-validation-rules.md` — Authentication constants.

---

### POST /api/v1/auth/otp/verify

Verify an email OTP. Returns a short-lived verification token.

**Authentication:** Not required.

**Request:**

```json
{ "email": "string", "code": "string" }
```

**Response:**

```json
{ "data": { "verification_token": "string", "expires_at": "datetime" } }
```

**Errors:** `400`, `401`, `422`, `429`

**Rules:**
- Validate OTP format and reject after maximum attempts: See `07-validation-rules.md` — Authentication constants.
- Reject expired OTP records.
- Reject invalid codes.
- Hard delete OtpVerification record after successful verification.
- Publish `auth.otp.verified`.
- verification_token is short-lived and single-use.

---

### POST /api/v1/auth/session

Create a session using a verified email OTP token (sign-in flow for existing accounts).

**Authentication:** Not required.

**Request:**

```json
{ "email": "string", "verification_token": "string" }
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

**Errors:** `400`, `401`, `422`, `429`

**Rules:**
- Validate verification_token.
- Resolve the Account by email (via AuthCredential of type 'email').
- If no Account exists for this email, return `404` — account must be registered first.
- Create Session record.
- Issue access token (short-lived JWT, not stored).
- Issue refresh token (stored as hash in Session).
- Publish `auth.session.created`.

---

### POST /api/v1/auth/google

Authenticate using a Google ID token. Handles both sign-in (existing account) and initiates registration (new account).

**Authentication:** Not required.

**Request:**

```json
{ "id_token": "string" }
```

**Response (existing account):**

```json
{
  "data": {
    "access_token": "jwt",
    "refresh_token": "string",
    "account": { "id": "uuid", "username": "string" }
  }
}
```

**Response (new account — no existing record):**

```json
{
  "data": {
    "needs_registration": true,
    "google_token": "string"
  }
}
```

**Errors:** `400`, `401`, `422`, `429`

**Rules:**
- Backend must verify the ID token against Google's public keys.
- Do not trust client-side token assertions.
- If Account exists (AuthCredential of type 'google' with matching sub): create session, return tokens.
- If no Account exists: return `needs_registration: true` with a short-lived google_token for registration.
- Publish `auth.google.authenticated`.

---

### POST /api/v1/auth/register

Complete registration using a verified OTP token + username reservation.

**Authentication:** Not required.

**Request:**

```json
{
  "email": "string",
  "verification_token": "string",
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
- Validate verification_token and reservation_id.
- Verify reservation exists and has not expired.
- Create Account + AuthCredential (email) + ProfileContent in one transaction.
- Delete UsernameReservation.
- Create Session; issue tokens.
- Publish `auth.registration.completed`.

---

### POST /api/v1/auth/register/google

Complete registration using Google authentication + username reservation.

**Authentication:** Not required.

**Request:**

```json
{
  "google_token": "string",
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
- Validate google_token (issued by POST /auth/google).
- Verify reservation exists and has not expired.
- Create Account + AuthCredential (google) + ProfileContent in one transaction.
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

- No `channel` field on OTP endpoints. Email is the only OTP channel in V1.
- No phone OTP endpoints.
- No SMS endpoints.
- Never expose OTP codes in responses.
- Never expose refresh tokens except during session creation or rotation.

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
- Return reservation_id for use in POST /auth/register.
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

Get QR code configuration.

**Response:**

```json
{ "data": { "qr_code": {} } }
```

**Errors:** `401`, `403`

---

### PATCH /api/v1/account/qr-code

Update QR code configuration.

**Request:**

```json
{ "qr_code": {} }
```

**Response:**

```json
{ "data": { "qr_code": {} } }
```

**Errors:** `400`, `401`, `403`, `422`

**Rules:**
- Validate qr_code payload.
- Publish `account.qr.updated` after persistence.

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

## Part 9 — Public Rendering

**Path:** `/{username}`

The public profile page. Rendered server-side. Outside the `/api/v1/` base path.

**Authentication:** Not required.

**Response:** Rendered HTML profile page.

**Errors:** `404 Not Found` (username does not exist or account is deleted)

**Rules:**
- Resolve username to an active Account.
- Return 404 for deleted or suspended accounts.
- RenderingService assembles profile from ProfileContent, Blocks, ConnectedAccounts, OutLinks, ImageAssets (all read-only).
- For `button` blocks and `social_icons` blocks, tracked links in the rendered output use `/out/{public_id}` format. Raw destination URLs must not appear in public render payloads for tracked links.
- Response is cached per `09-caching-strategy.md`.
- No publishing state. The rendered output always reflects current persisted state.
- There is no preview path. The authenticated dashboard shows live data.
