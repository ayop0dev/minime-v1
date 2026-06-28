# Minime Block Renderers Specification V1

**Status:** Canonical
**Version:** V1
**Layer:** Rendering
**Parent:** `rendering.architecture.canon.v1.md`

This specification defines every block-type renderer in one place. Each renderer
transforms one block type into a normalized **Render Object** for the Rendering
Engine. The architectural principles that govern all renderers live in the
Rendering Architecture Canon; this document defines only the per-renderer
content contracts.

---

## Shared Renderer Rules

These rules apply to **every** renderer below and are not repeated per renderer.

- **Input.** Each renderer receives its block plus the **Resolved Style** for
  that block (already computed by the Block Styling System). The renderer
  attaches Resolved Style to the Render Object **unchanged** and never reads the
  Theme, Layout, or Template directly.
- **Output shape.** Every Render Object has the form:
  ```json
  { "id": "<block_id>", "type": "<block_type>", "content": { … }, "resolved_style": { … } }
  ```
- **Content preservation.** Renderers return stored content exactly as saved.
  They never rewrite, translate, summarize, optimize, generate, shorten, expand,
  or reformat content. Stored line breaks and paragraph breaks are preserved.
- **Presentation & theme independence.** Renderers never decide fonts, color,
  size, shape, radius, spacing, alignment, shadow, or animation. Those are
  Resolved Style values, attached unchanged.
- **Null-safe failure.** A renderer returns `null` when its content cannot be
  resolved safely. It never throws public-facing errors. A single block failure
  must never prevent the rest of the profile from rendering.
- **Common flow.** `Block → Renderer → Render Object → Rendering Engine → Public Profile`.

---

## Renderer Summary

| Renderer | Block type | Category | Reads content from | Render Object `content` | Required | Multiplicity |
|---|---|---|---|---|---|---|
| Avatar | `avatar` | Reference | `ProfileContent.avatar` | `{ url }` | `url` | Max 1 |
| Name | `name` | Reference | `ProfileContent.display_name` | `{ value }` | `value` | Max 1 |
| Bio | `bio` | Reference | `ProfileContent.bio` | `{ value }` | `value` | Max 1 |
| Image | `image` | Content | Image Block | `{ url, alt }` | `url` (`alt` optional) | Unlimited |
| Button | `button` | Content | Button Block | `{ label, url }` | `label`, `url` | Unlimited |
| Divider | `divider` | Content | Divider Block | `{}` (empty) | none | Unlimited |
| Title | `title` | Content | Title Block | `{ value }` | `value` | Unlimited |
| Textbox | `textbox` | Content | Textbox Block | `{ value }` | `value` | Unlimited |
| Social Icons | `social_icons` | Collection Reference | Connected Accounts | `{ accounts:[…] }` | per account: `platform`, `url` | Unlimited |

**Categories** (defined in the Rendering Canon):
- **Reference** — reads a Profile-owned field; the block owns placement only.
- **Content** — reads the block's own stored content.
- **Collection Reference** — reads an external collection; the block owns
  selection, ordering, and visibility only.

---

## Reference Renderers

Reference renderers read Profile-owned content. The block owns placement; Profile
owns the value. The renderer must never read the value from the block. Each
returns `null` if the referenced Profile field is null or empty.

### Avatar Renderer
- Reads `ProfileContent.avatar`. Content: `{ "url": "…" }`.
- Returns `null` if the avatar is null, empty, or the asset is missing/invalid.
- Must not upload, edit, crop, resize, generate, or store avatars.

### Name Renderer
- Reads `ProfileContent.display_name`. Content: `{ "value": "…" }`.
- Returns `null` if the name is null or empty.

### Bio Renderer
- Reads `ProfileContent.bio`. Content: `{ "value": "…" }`.
- Preserves stored line/paragraph breaks. No rich-text or Markdown rendering in V1.
- Returns `null` if the bio is null or empty.

---

## Content Renderers

Content renderers read the block's own stored content directly. They must not
read from Profile, Connected Accounts, or other blocks.

### Image Renderer
- Owns nothing; Image Block owns `image` and `alt_text`.
- Content: `{ "url": "…", "alt": "…" }`. `url` required, `alt` optional.
- If `alt` is missing, return `alt: ""`. Never auto-generate alt text.
- Returns `null` if the image is null/empty or the asset is missing/invalid.

### Button Renderer
- Button Block owns `label` and `url`. Content: `{ "label": "…", "url": "…" }`.
- Both fields required; returns `null` if either is empty or the URL is malformed.
- Preserves the URL exactly as stored. Never shortens, wraps, tracks, or rewrites
  it — outbound routing and click tracking belong to the Out Links and Analytics
  domains, not Rendering.

### Divider Renderer
- Represents the existence of a visual separator. Content is intentionally `{}`.
- The **only** renderer permitted to return a valid Render Object with empty content.
- No labels/captions/text in V1 — use a Title Block for section headings.
- Returns `null` only if the block itself is invalid.

### Title Renderer
- Semantic meaning: **section heading**. Title Block owns `value`.
- Content: `{ "value": "…" }`. Returns `null` if `value` is null or empty.
- Independent of Textbox — no parent/child relationship; never reads Textbox content.
- No heading levels or variants in V1.

### Textbox Renderer
- Semantic meaning: **section content** (paragraphs, descriptions). Textbox Block owns `value`.
- Content: `{ "value": "…" }`. Preserves stored formatting (line/paragraph breaks).
- Returns `null` if `value` is null or empty. No rich-text/Markdown rendering in V1.
- Independent of Title — never reads Title content.

---

## Collection Reference Renderer

### Social Icons Renderer
- **Connected Accounts** is the canonical store of social account records
  (`platform`, `username`, `url`, metadata); the user is the source of truth.
- The **Social Icons Block** owns selection, ordering, and visibility only — not
  account data.
- **Resolution order:** load all connected accounts → apply block selection →
  apply visibility → apply ordering → build Render Object.
- Content: `{ "accounts": [ { "platform", "username", "url" }, … ] }`.
  Per account, `platform` and `url` are required; `username` optional.
- Consumes the stored canonical `url` **exactly as saved**. Never regenerates
  URLs, normalizes identifiers, applies platform rules, or rebuilds links.
- **Partial rendering:** invalid or deleted accounts are silently excluded; valid
  accounts still render. Returns `null` only when no renderable accounts remain.
- Multiple Social Icons Blocks may each show different account selections.
- Not supported in V1: account collection/verification, live sync, dynamic
  status, follower counts, per-account analytics, conditional visibility.
