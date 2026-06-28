# Minime Button Renderer Specification V1

## Purpose

The Button Renderer is responsible for rendering Button Blocks.

The Button Renderer renders content owned by the Button Block itself.

The renderer consumes the block content together with the Resolved Style produced by the Block Styling System, and returns a Render Object that carries both the content and the resolved style.

---

## Block Type

Supported block type:

```text
button
```

The Button Renderer processes only Button Blocks.

---

## Renderer Category

Button Renderer is a:

```text
Content Renderer
```

It renders block-owned content.

It does not resolve profile references.

It does not resolve connected account references.

---

## Ownership Model

### Button Block Owns

```text
label
url
```

The Button Block is the single source of truth for its content.

---

## Data Source

The renderer reads content directly from:

```text
Button Block
```

The renderer must not read button content from:

```text
Profile
Connected Accounts
Other Blocks
```

---

## Renderer Input

The renderer receives:

### Block

```text
Button Block
```

Containing:

```text
label
url
```

### Resolved Style

The renderer also receives the **Resolved Style** for this block, produced by the Block Styling System. The renderer attaches it to the Render Object unchanged and never reads the Theme directly.

---

## Resolution Flow

```text
Button Block
↓
Read Button Content
↓
Validate Content
↓
Create Render Object
↓
Return Result
```

---

## Successful Resolution

If a valid button exists:

```text
label
url
```

the renderer returns a Render Object.

---

## Render Object

Example:

```json
{
  "id": "block_button_1",
  "type": "button",
  "content": {
    "label": "Visit Website",
    "url": "https://example.com"
  },
  "resolved_style": {}
}
```

---

## Content Structure

### label

Represents the button text.

Example:

```json
{
  "label": "Visit Website"
}
```

---

### url

Represents the button destination.

Example:

```json
{
  "url": "https://example.com"
}
```

---

## Required Fields

Required:

```text
label
url
```

Both fields must exist.

---

## Empty Content

If:

```text
label = empty
```

or

```text
url = empty
```

the renderer must return:

```text
null
```

No Render Object should be generated.

---

## Invalid URL

Examples:

```text
Missing URL
Malformed URL
Invalid URL
Unsupported URL
```

The renderer must return:

```text
null
```

---

## URL Preservation

The Button Renderer must return the URL exactly as stored.

The renderer must not:

```text
Rewrite URLs
Shorten URLs
Track URLs
Replace URLs
Modify URLs
```

URL transformation belongs to later layers of the system.

---

## Label Preservation

The Button Renderer must return the label exactly as stored.

The renderer must not:

```text
Rewrite Labels
Translate Labels
Optimize Labels
Generate Labels
Shorten Labels
```

---

## Content Preservation

The renderer must preserve button content exactly as stored.

Example:

Stored:

```text
Book a Consultation
```

Returned:

```text
Book a Consultation
```

Exactly.

---

## Renderer Responsibilities

The Button Renderer may:

* Read button content
* Validate required fields
* Validate URL structure
* Normalize content
* Produce Render Objects

---

## Renderer Non-Responsibilities

The Button Renderer must not:

* Generate buttons
* Create labels
* Rewrite labels
* Analyze destinations
* Track clicks
* Store button data
* Edit button data

These responsibilities belong elsewhere.

---

## Analytics Independence

The Button Renderer must not know:

```text
Click Tracking
Analytics
Events
UTM Parameters
Conversion Metrics
```

Analytics belongs to separate systems.

---

## Routing Independence

The Button Renderer must not know:

```text
Outbound Routing
Redirect Systems
Tracking URLs
Link Wrappers
```

Example:

```text
/out/xxxxx
```

is not a rendering concern.

---

## Presentation Independence

The Button Renderer must not define:

```text
Color
Size
Shape
Border Radius
Spacing
Shadow
Animation
Icon Placement
```

Examples:

```text
Primary Button
Outlined Button
Rounded Button
Large Button
```

are resolved by the Block Styling System and attached to the Render Object as Resolved Style.

The renderer does not resolve or decide them.

---

## Theme Independence

The Button Renderer must not know:

```text
Theme
Layout
Template
Visual Style
```

The renderer returns content plus the Resolved Style it is given. It never reads the Theme.

---

## Multiple Button Blocks

V1 allows:

```text
Unlimited Button Blocks
```

Each Button Block is rendered independently.

---

## Rendering Failure

The renderer returns:

```text
null
```

when button content cannot be resolved safely.

The renderer must never throw public-facing errors.

---

## Public Profile Flow

```text
Button Block
↓
Button Renderer
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
Label Resolution
URL Resolution
URL Validation
Render Object Creation
Null-safe Rendering
```

Not Supported:

```text
Click Tracking
Analytics
Smart Routing
Conditional Buttons
Button Variants
Dynamic Labels
Dynamic URLs
```

---

## Final Rule

The Button Renderer answers one question:

```text
What button data should be available for presentation?
```

It does not answer:

```text
How should the Theme be resolved, or how should blocks be composed into a page?
```
