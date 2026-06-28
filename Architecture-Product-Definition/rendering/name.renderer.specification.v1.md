# Minime Name Renderer Specification V1

## Purpose

The Name Renderer is responsible for rendering name reference blocks.

The Name Renderer does not own name content.

Name content is owned by Profile Content.

The renderer resolves the reference and returns a Render Object that carries the resolved content together with the Resolved Style produced by the Block Styling System.

---

## Block Type

Supported block type:

```text
name
```

The Name Renderer processes only Name Blocks.

---

## Renderer Category

Name Renderer is a:

```text
Reference Renderer
```

It renders profile-owned content.

It does not render block-owned content.

---

## Ownership Model

### Profile Content Owns

```text
ProfileContent.display_name
```

---

### Name Block Owns

```text
Placement
Visibility
Existence
```

The block determines whether the name appears in the profile.

The block does not own the display name itself.

---

## Data Source

The Name Renderer resolves:

```text
ProfileContent.display_name
```

The renderer must never read name content from the block.

---

## Renderer Input

The renderer receives:

### Block

```text
Name Block
```

---

### Rendering Context

Including:

```text
Profile
```

Profile Content contains the display name.

### Resolved Style

The renderer also receives the **Resolved Style** for this block, produced by the Block Styling System. The renderer attaches it to the Render Object unchanged and never reads the Theme directly.

---

## Resolution Flow

```text
Name Block
↓
Read ProfileContent.display_name
↓
Resolve Name
↓
Create Render Object
↓
Return Result
```

---

## Successful Resolution

If a valid display name exists:

```text
ProfileContent.display_name
```

the renderer returns a Render Object.

---

## Render Object

Example:

```json
{
  "id": "block_name_1",
  "type": "name",
  "content": {
    "value": "Ahmed Hassan"
  },
  "resolved_style": {}
}
```

---

## Content Structure

### value

Represents the resolved profile display name.

Example:

```json
{
  "value": "Ahmed Hassan"
}
```

---

## Required Content

Required:

```text
value
```

---

## Empty Name

If:

```text
ProfileContent.display_name = null
```

or

```text
ProfileContent.display_name = empty
```

the renderer must return:

```text
null
```

No Render Object should be generated.

---

## Invalid Name

Examples:

```text
Missing Value
Invalid Value
Corrupted Data
```

The renderer must return:

```text
null
```

---

## Content Preservation

The Name Renderer must return the display name exactly as stored.

The renderer must not:

```text
Rewrite
Reformat
Capitalize
Shorten
Translate
Generate
Enhance
```

the display name.

---

## Renderer Responsibilities

The Name Renderer may:

* Resolve profile name references
* Validate name availability
* Produce Render Objects

---

## Renderer Non-Responsibilities

The Name Renderer must not:

* Create names
* Edit names
* Suggest names
* Optimize names
* Store names
* Modify names

These responsibilities belong elsewhere.

---

## Presentation Independence

The Name Renderer must not define:

```text
Font
Weight
Size
Color
Alignment
Spacing
Animation
```

Examples:

```text
Large Name
Centered Name
Bold Name
```

are resolved by the Block Styling System and attached to the Render Object as Resolved Style.

The renderer does not resolve or decide them.

---

## Theme Independence

The Name Renderer must not know:

```text
Theme
Layout
Template
Visual Style
```

The renderer returns content plus the Resolved Style it is given. It never reads the Theme.

---

## Multiple Name Blocks

V1 allows:

```text
Maximum 1 Name Block
```

Therefore:

```text
Maximum 1 Name Renderer Execution
```

per profile render cycle.

---

## Rendering Failure

The renderer returns:

```text
null
```

when name content cannot be resolved safely.

The renderer must never throw public-facing errors.

---

## Public Profile Flow

```text
Name Block
↓
Name Renderer
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
Profile Name Resolution
Render Object Creation
Null-safe Rendering
```

Not Supported:

```text
Generated Names
Alternative Names
Localized Names
Conditional Names
Multiple Names
```

---

## Final Rule

The Name Renderer answers one question:

```text
What name data should be available for presentation?
```

It does not answer:

```text
How should the Theme be resolved, or how should blocks be composed into a page?
```
