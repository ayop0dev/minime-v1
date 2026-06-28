# Minime Social Icons Renderer Specification V1

## Purpose

The Social Icons Renderer is responsible for rendering Social Icons blocks.

The Social Icons Renderer does not own social account data.

Social account data is stored by Connected Accounts, which persists the user-provided records. The user is the source of truth.

The renderer resolves account references and returns a Render Object that carries the resolved content — including the stored canonical `url` consumed exactly as saved — together with the Resolved Style produced by the Block Styling System.

The renderer never regenerates URLs, normalizes identifiers, applies platform rules, or rebuilds links. Rendering is presentation only.

---

## Block Type

Supported block type:

```text
social_icons
```

The Social Icons Renderer processes only Social Icons Blocks.

---

## Renderer Category

Social Icons Renderer is a:

```text
Collection Reference Renderer
```

It renders data owned by an external collection.

It does not render block-owned account data.

---

## Ownership Model

### Connected Accounts Store

```text
platform
username
url
metadata
```

Connected Accounts is the canonical store of these user-provided social account records. The user remains the source of truth; Connected Accounts persists the record the user provided. The renderer consumes the stored canonical `url` exactly as saved.

---

### Social Icons Block Owns

```text
selection
ordering
visibility
placement
existence
```

The block determines:

* Which accounts appear
* In what order
* Which accounts are hidden

The block does not own account information.

---

## Data Source

The renderer resolves:

```text
Connected Accounts
```

The renderer must never store account information inside the block.

---

## Renderer Input

The renderer receives:

### Block

```text
Social Icons Block
```

---

### Rendering Context

Including:

```text
Connected Accounts
```

### Resolved Style

The renderer also receives the **Resolved Style** for this block, produced by the Block Styling System. The renderer attaches it to the Render Object unchanged and never reads the Theme directly.

---

## Resolution Flow

```text
Social Icons Block
↓
Read Connected Accounts
↓
Apply Selection
↓
Apply Visibility
↓
Apply Ordering
↓
Create Render Object
↓
Return Result
```

---

## Resolution Rules

### Step 1

Load all connected accounts.

Example:

```text
Instagram
TikTok
LinkedIn
YouTube
```

---

### Step 2

Apply block selection.

Example:

```text
Selected:
Instagram
LinkedIn
```

---

### Step 3

Apply visibility rules.

Example:

```text
Instagram = visible
LinkedIn = hidden
```

Result:

```text
Instagram
```

---

### Step 4

Apply block ordering.

Example:

```text
Instagram
YouTube
TikTok
```

---

### Step 5

Create Render Object.

---

## Render Object

Example:

```json
{
  "id": "block_social_icons_1",
  "type": "social_icons",
  "content": {
    "accounts": [
      {
        "platform": "instagram",
        "username": "ahmed",
        "url": "https://instagram.com/ahmed"
      },
      {
        "platform": "linkedin",
        "username": "ahmed-hassan",
        "url": "https://linkedin.com/in/ahmed-hassan"
      }
    ]
  },
  "resolved_style": {}
}
```

---

## Content Structure

### accounts

Contains the final visible accounts after resolution.

Example:

```json
{
  "accounts": []
}
```

---

## Account Structure

Each account may contain:

```json
{
  "platform": "instagram",
  "username": "ahmed",
  "url": "https://instagram.com/ahmed"
}
```

---

## Required Fields

Required:

```text
platform
url
```

Optional:

```text
username
```

Additional fields may be added in future versions.

---

## Deleted Accounts

If a selected account no longer exists:

```text
Connected Account Deleted
```

the renderer must silently ignore it.

The renderer must not fail.

---

## Invalid Accounts

Examples:

```text
Missing Platform
Missing URL
Corrupted Record
Invalid Record
```

The renderer must exclude the account from the result.

The renderer must continue rendering remaining accounts.

---

## Empty Result

If no visible accounts remain after filtering:

```text
accounts = []
```

the renderer must return:

```text
null
```

No Render Object should be generated.

---

## Partial Rendering

The renderer may return a valid Render Object even if some accounts are invalid.

Example:

```text
Instagram = valid
TikTok = invalid
LinkedIn = valid
```

Result:

```text
Instagram
LinkedIn
```

Only.

---

## Renderer Responsibilities

The Social Icons Renderer may:

* Resolve connected accounts
* Apply selection
* Apply visibility
* Apply ordering
* Remove invalid accounts
* Produce Render Objects

---

## Renderer Non-Responsibilities

The Social Icons Renderer must not:

* Create accounts
* Collect or normalize accounts
* Verify accounts
* Connect accounts
* Edit accounts
* Store accounts
* Synchronize accounts

These responsibilities belong elsewhere.

---

## Presentation Independence

The Social Icons Renderer must not define:

```text
Icon Size
Icon Color
Icon Shape
Spacing
Alignment
Rows
Columns
Animations
Hover Effects
```

Examples:

```text
Circular Icons
Outlined Icons
Colored Icons
```

are resolved by the Block Styling System and attached to the Render Object as Resolved Style.

The renderer does not resolve or decide them.

---

## Theme Independence

The Social Icons Renderer must not know:

```text
Theme
Layout
Template
Visual Style
```

The renderer returns content plus the Resolved Style it is given. It never reads the Theme.

---

## Multiple Social Icons Blocks

V1 allows:

```text
Unlimited Social Icons Blocks
```

Each block is rendered independently.

Different blocks may display different account selections.

Example:

```text
Block A
→ Instagram + TikTok

Block B
→ LinkedIn + YouTube
```

Both are valid.

---

## Rendering Failure

The renderer returns:

```text
null
```

only when no renderable accounts remain.

The renderer must never throw public-facing errors.

---

## Public Profile Flow

```text
Social Icons Block
↓
Social Icons Renderer
↓
Render Object (content + resolved style)
↓
Rendering Engine
↓
Public Profile
```

---

## V1 Scope

Supported:

```text
Connected Account Resolution
Selection Filtering
Visibility Filtering
Ordering
Partial Rendering
Render Object Creation
```

Not Supported:

```text
Social Account Collection
Account Verification
Live Platform Sync
Dynamic Account Status
Follower Counts
Account Analytics
Conditional Visibility
```

---

## Final Rule

The Social Icons Renderer answers one question:

```text
Which social accounts should be available for presentation?
```

It does not answer:

```text
How should the Theme be resolved, or how should blocks be composed into a page?
```
