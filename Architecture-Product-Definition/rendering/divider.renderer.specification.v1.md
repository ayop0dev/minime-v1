# Minime Divider Renderer Specification V1

## Purpose

The Divider Renderer is responsible for rendering Divider Blocks.

The Divider Renderer represents the intent of visual separation between blocks.

The renderer returns a Render Object indicating the existence of a divider, carrying the Resolved Style produced by the Block Styling System.

The renderer does not resolve or decide how the divider should appear; it attaches the resolved style it is given.

---

## Block Type

Supported block type:

```text
divider
```

The Divider Renderer processes only Divider Blocks.

---

## Renderer Category

Divider Renderer is a:

```text
Content Renderer
```

It renders block-owned content.

---

## Ownership Model

### Divider Block Owns

```text
Existence
Placement
```

The Divider Block represents the user's intention to insert a visual separator.

---

## Data Source

The renderer reads content directly from:

```text
Divider Block
```

The renderer does not read data from:

```text
Profile
Connected Accounts
Other Blocks
```

---

## Divider Content

In V1:

```text
Divider Block
```

contains no user-owned content.

The existence of the block itself is the content.

---

## Renderer Input

The renderer receives:

### Block

```text
Divider Block
```

### Resolved Style

The renderer also receives the **Resolved Style** for this block, produced by the Block Styling System. The renderer attaches it to the Render Object unchanged and never reads the Theme directly.

---

## Resolution Flow

```text
Divider Block
↓
Validate Block
↓
Create Render Object
↓
Return Result
```

---

## Successful Resolution

If a valid Divider Block exists:

```text
Divider Block
```

the renderer returns a Render Object.

---

## Render Object

Example:

```json
{
  "id": "block_divider_1",
  "type": "divider",
  "content": {},
  "resolved_style": {}
}
```

---

## Content Structure

Divider content is intentionally empty.

Example:

```json
{}
```

No additional content is required.

---

## Required Fields

Required:

```text
None
```

The block itself is sufficient.

---

## Empty Content Rules

Divider is the only block in V1 allowed to return a valid Render Object with empty content.

Example:

```json
{
  "id": "block_divider_1",
  "type": "divider",
  "content": {},
  "resolved_style": {}
}
```

is valid.

---

## Content Preservation

There is no content to preserve.

The renderer simply represents the divider's existence.

---

## Renderer Responsibilities

The Divider Renderer may:

* Validate block existence
* Produce Render Objects

---

## Renderer Non-Responsibilities

The Divider Renderer must not:

* Create divider styles
* Create divider text
* Generate labels
* Generate captions
* Generate content

These responsibilities belong elsewhere.

---

## Presentation Independence

The Divider Renderer must not define:

```text
Line Style
Line Width
Thickness
Color
Spacing
Padding
Margin
Animation
```

Examples:

```text
Solid Line
Dashed Line
Gradient Line
Minimal Divider
Decorative Divider
```

are resolved by the Block Styling System and attached to the Render Object as Resolved Style.

The renderer does not resolve or decide them.

---

## Theme Independence

The Divider Renderer must not know:

```text
Theme
Layout
Template
Visual Style
```

The renderer returns the Resolved Style it is given. It never reads the Theme.

---

## Divider Labels

V1 does not support:

```text
Divider Titles
Divider Labels
Divider Captions
Divider Text
```

Examples:

```text
About Me
Projects
Contact
```

must use:

```text
Title Block
```

not Divider Block.

---

## Multiple Divider Blocks

V1 allows:

```text
Unlimited Divider Blocks
```

Each Divider Block is rendered independently.

---

## Rendering Failure

The renderer returns:

```text
null
```

only when the Divider Block itself is invalid or unavailable.

---

## Public Profile Flow

```text
Divider Block
↓
Divider Renderer
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
Divider Existence
Render Object Creation
Null-safe Rendering
```

Not Supported:

```text
Divider Labels
Divider Captions
Divider Text
Divider Variants
Conditional Dividers
Decorative Dividers
```

---

## Final Rule

The Divider Renderer answers one question:

```text
Should a visual separator exist at this position?
```

It does not answer:

```text
How should the Theme be resolved, or how should blocks be composed into a page?
```
