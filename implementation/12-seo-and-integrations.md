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
- GTM is optional by design: if no container ID is provided by the account owner, no snippet is injected for that account's public profile.
- GTM failure must not affect core profile rendering or analytics recording.
- GTM Container ID is user-provided account configuration. The account owner supplies their GTM Container ID through account settings; it is stored in `Account.settings.gtm_container_id`. It is not deployment-wide environment configuration.
- No integration metadata is stored as a standalone database entity. The GTM Container ID is stored as part of `Account.settings`.

---

### What Integrations Does NOT Own

- Analytics data (owned by Analytics domain via `AnalyticsEvent`)
- Any standalone database entity (GTM Container ID is stored in `Account.settings`, owned by Account domain)
- Any API endpoint
- Any background job

---

### Google Tag Manager (GTM)

**V1 scope:** GTM only.

**Container ID:** Provided by the account owner through account settings, stored as `Account.settings.gtm_container_id`. If absent or empty for an account, the GTM snippet is not injected for that account's public profile.

**Injection rule:** The GTM snippet is never injected directly into the public profile document's own DOM context. It is injected as a sandboxed `<iframe>` placed immediately after the opening `<body>` tag — see "GTM Security Rules" below for why, and `google-tag-manager.specification.v1.md` — "Security Principles" for the full threat model this responds to.

**Snippet format:**

```html
<!-- Google Tag Manager (sandboxed) -->
<iframe
  sandbox="allow-scripts"
  srcdoc="&lt;script&gt;(function(w,d,s,l,i){...})(window,document,'script','dataLayer','GTM-XXXXXX');&lt;/script&gt;"
  style="display:none" width="0" height="0" aria-hidden="true">
</iframe>
<!-- End Google Tag Manager -->
```

Where `GTM-XXXXXX` is the configured `gtm_container_id`. `sandbox="allow-scripts"` is the only sandbox token granted — `allow-same-origin`, `allow-top-navigation`, `allow-popups`, and `allow-forms` are never added.

**No body noscript fallback in V1:** the GTM `<noscript>` fallback (a `<img>` pixel) would need to be either inside the sandboxed iframe (harmless but useless, since it fires from the opaque origin, same as the script path) or outside it in the parent document (which would defeat the containment boundary by giving the parent page an unreviewed third-party image tag). V1 omits it; GTM tag firing for JavaScript-disabled visitors is not a V1 guarantee.

---

### GTM Security Rules

Per `08-security-model.md` and `google-tag-manager.specification.v1.md` — "Security Principles":

- GTM container ID is user-provided account configuration. It is accepted through the account settings API (authenticated endpoint), validated on write (format only), and stored in `Account.settings`. It must not be accepted as a raw parameter in public profile requests.
- Format-validating the Container ID is not a script-injection defense — it cannot be, since the account owner's own GTM workspace can add arbitrary-JavaScript tags that Minime never sees. The actual defense is containment: **the GTM snippet always executes inside the sandboxed iframe above, never in the parent document's origin.** A sandboxed iframe without `allow-same-origin` has a unique, opaque origin — scripts inside it cannot read/write the parent DOM, cannot access any Minime cookie or storage, and cannot navigate the top-level page. This holds regardless of what tags the account owner configures in their GTM workspace.
- The parent document's Content-Security-Policy `script-src` must not need to allow `googletagmanager.com` or any Google tagging domain — those are only ever loaded inside the sandboxed iframe's own document (via `srcdoc`), which is a separate origin from the parent page's CSP scope. This keeps the parent page's own CSP tight regardless of whether GTM is configured.
- GTM snippet is injected server-side, not client-side. Client-supplied script content is never trusted.
- GTM loading must be conditional: absent or empty `Account.settings.gtm_container_id` for the requested account means no iframe is injected; no error is raised.
- GTM provider failures (network errors, script load failures) must not affect profile rendering. The iframe loads asynchronously and must not block rendering.

---

### Integrations Implementation Rules

- GTM snippet injection executes inside the Rendering pipeline, as the final step of HTML assembly.
- GTM snippet is part of the cached render response.
- `Account.settings.gtm_container_id` is read from the account's stored settings at render time for each public profile.
- If `Account.settings.gtm_container_id` is present and non-empty for the requested account, the snippet is injected into that account's public profile response only.
- The snippet must be injected as a static server-rendered string, not as a client-side React/Next.js component that fetches configuration.

---

### Integrations Prohibitions

Implementation must not:

- Inject a GTM snippet without a valid GTM Container ID stored in `Account.settings.gtm_container_id` for the requested account.
- Store GTM configuration as a standalone database entity; `Account.settings` is the only permitted storage for the GTM Container ID.
- Expose the account's GTM container ID in public API responses, server logs, or client-side rendered code.
- Introduce additional integration providers (analytics, chat, etc.) without updating the frozen architecture.
- Block profile rendering on GTM load failure.
- Use GTM as a substitute for Minime's own analytics recording (see `04-service-contracts.md` — AnalyticsService).

---

### Future Integration Scope

V1 supports GTM only. Future integration providers (e.g., Meta Pixel, LinkedIn Insight Tag, custom webhooks) must be added in a future architecture revision. They must not be pre-implemented or prototyped in V1 implementation files.
