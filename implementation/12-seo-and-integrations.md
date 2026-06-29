# Minime V1 — SEO and Integrations

## Status

Canonical. Final.

---

## Architecture Authority

```
Architecture-Product-Definition/seo/seo.architecture.specification.v1.md
Architecture-Product-Definition/integrations/integrations.architecture.specification.v1.md
Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md
implementation/03-canonical-data-model.md
implementation/04-service-contracts.md
implementation/09-caching-strategy.md
```

---

## Purpose

Defines implementation contracts for the two V1 Cross-Cutting Capabilities: SEO and Integrations.

SEO and Integrations are Cross-Cutting Capabilities — they are not Product Domains, not Platform Services, and they introduce no new business behavior. They are consumed by the Rendering pipeline and operate as read-only derivations of existing domain data.

---

## V1 Cross-Cutting Capabilities

| Capability | Owner | Status |
|---|---|---|
| SEO | Cross-cutting, consumed by Rendering | This document |
| Integrations | Cross-cutting, consumed by Rendering | This document |

No additional Cross-Cutting Capabilities exist in V1.

---

## SEO

### Architectural Position

SEO is a Cross-Cutting Product Capability.

- SEO produces HTML metadata for the Public Profile.
- SEO consumes data from Account, ProfileContent, and the assembled render object.
- SEO never writes to any domain entity.
- SEO never owns persisted data.
- SEO metadata is computed at render time; it is not stored.
- SEO is part of the Rendering pipeline, not a standalone service.

---

### What SEO Does NOT Own

- Profile content (owned by Profile domain)
- Canonical URLs (owned by Account domain via `Account.username`)
- Analytics (owned by Analytics domain)
- Any database entity
- Any background job

---

### SEO Metadata Produced

For every public profile response at `GET /{username}`, the following metadata is produced:

| Tag | Source | Notes |
|---|---|---|
| `<title>` | `ProfileContent.display_name` | Format: `{display_name} — Minime` |
| `<meta name="description">` | `ProfileContent.bio` | Truncated to 160 chars; empty string if bio is null |
| `<link rel="canonical">` | `Account.username` | Format: `{APP_BASE_URL}/{username}` |
| `<meta name="robots">` | Static | `index, follow` for all active accounts |
| `<meta property="og:title">` | `ProfileContent.display_name` | Same value as `<title>` |
| `<meta property="og:description">` | `ProfileContent.bio` | Same value as meta description |
| `<meta property="og:url">` | `Account.username` | Same value as canonical URL |
| `<meta property="og:image">` | `ProfileContent.avatar_storage_key` | Constructed via `StoragePlatform.buildPublicUrl(key)`; omitted if no avatar |
| `<meta property="og:type">` | Static | `website` |
| `<meta name="twitter:card">` | Static | `summary_large_image` if avatar present, else `summary` |
| `<meta name="twitter:title">` | `ProfileContent.display_name` | Same value as og:title |
| `<meta name="twitter:description">` | `ProfileContent.bio` | Same value as meta description |
| `<meta name="twitter:image">` | `ProfileContent.avatar_storage_key` | Same as og:image; omitted if no avatar |
| JSON-LD `Person` | Account + ProfileContent | See schema below |

---

### Structured Data (JSON-LD)

SEO produces a `Person` schema (schema.org) for each public profile:

```json
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "{ProfileContent.display_name}",
  "url": "{APP_BASE_URL}/{Account.username}",
  "image": "{avatar_url | omitted if no avatar}",
  "description": "{ProfileContent.bio | omitted if null}",
  "sameAs": ["{ConnectedAccount.url}", ...]
}
```

`sameAs` array: list of `ConnectedAccount.url` values for the account. Omitted if no connected accounts exist.

---

### SEO Implementation Rules

- SEO metadata generation executes inside the Rendering pipeline, after the render object is assembled.
- SEO metadata is included in the HTML `<head>` of the public profile response.
- SEO metadata is part of the cached render response. Cache keys and TTL: See `09-caching-strategy.md`.
- `APP_BASE_URL` is environment-configured. It must not be hardcoded.
- Robots directive is always `index, follow` for accounts with `status = 'active'`. Deleted or suspended accounts must return `noindex, nofollow`.
- Canonical URL always uses the account's current username. Username is immutable in V1, so canonical URLs are stable.
- Avatar URL is constructed via `StoragePlatform.buildPublicUrl(avatar_storage_key)`. See `11-platform-services.md`.

---

### SEO Prohibitions

Implementation must not:

- Store SEO metadata in any database entity.
- Create a SEO-specific entity, table, or repository.
- Allow the SEO layer to modify any domain entity.
- Serve stale SEO metadata independently of the profile cache; SEO metadata and profile content must be co-cached.
- Introduce a `noindex` rule for any reason other than account suspension or deletion.
- Expose internal entity IDs (UUIDs) in any SEO output.

---

## Integrations

### Architectural Position

Integrations is a Cross-Cutting Product Capability.

- V1 scope: Google Tag Manager (GTM) only.
- GTM is optional by design: if no container ID is configured, no snippet is injected.
- GTM failure must not affect core profile rendering or analytics recording.
- Integration configuration is never user data; it is environment/deployment configuration.
- No integration data is stored in any database entity.

---

### What Integrations Does NOT Own

- Analytics data (owned by Analytics domain via `AnalyticsEvent`)
- Any database entity
- Any API endpoint
- Any background job
- User configuration (GTM container ID is deployment configuration, not user preference)

---

### Google Tag Manager (GTM)

**V1 scope:** GTM only.

**Container ID:** `GTM_CONTAINER_ID` environment variable. If absent or empty, GTM is not loaded.

**Injection rule:** The GTM snippet is injected into the `<head>` of the public profile HTML response, immediately after the opening `<head>` tag.

**Snippet format:**

```html
<!-- Google Tag Manager -->
<script>(function(w,d,s,l,i){...})(window,document,'script','dataLayer','GTM-XXXXXX');</script>
<!-- End Google Tag Manager -->
```

Where `GTM-XXXXXX` is the configured `GTM_CONTAINER_ID`.

**Body noscript tag:** The GTM `<noscript>` fallback is injected immediately after the opening `<body>` tag.

---

### GTM Security Rules

Per `08-security-model.md`:

- GTM container ID is a deployment-level configuration value, not a user-provided value. It must not be accepted from client requests.
- GTM snippet is injected server-side, not client-side. Client-supplied script content is never trusted.
- GTM loading must be conditional: absent `GTM_CONTAINER_ID` means no snippet is injected; no error is raised.
- GTM provider failures (network errors, script load failures) must not affect profile rendering. GTM is loaded asynchronously and must not block rendering.

---

### Integrations Implementation Rules

- GTM snippet injection executes inside the Rendering pipeline, as the final step of HTML assembly.
- GTM snippet is part of the cached render response.
- `GTM_CONTAINER_ID` is read once at application startup. It does not change at runtime without restart.
- If `GTM_CONTAINER_ID` is present, the snippet is injected unconditionally for all public profile responses, regardless of account or profile content.
- The snippet must be injected as a static server-rendered string, not as a client-side React/Next.js component that fetches configuration.

---

### Integrations Prohibitions

Implementation must not:

- Allow users to configure their own GTM container ID.
- Store GTM configuration in any database entity.
- Expose `GTM_CONTAINER_ID` in API responses, logs, or client-side code.
- Introduce additional integration providers (analytics, chat, etc.) without updating the frozen architecture.
- Block profile rendering on GTM load failure.
- Use GTM as a substitute for Minime's own analytics recording (see `04-service-contracts.md` — AnalyticsService).

---

### Future Integration Scope

V1 supports GTM only. Future integration providers (e.g., Meta Pixel, LinkedIn Insight Tag, custom webhooks) must be added in a future architecture revision. They must not be pre-implemented or prototyped in V1 implementation files.
