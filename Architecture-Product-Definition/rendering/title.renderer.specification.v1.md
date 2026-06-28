# Minime Title Renderer Specification V1

## Purpose

The Title Renderer is responsible for rendering Title Blocks.

The Title Renderer renders content owned by the Title Block itself.

The renderer consumes the block content together with the Resolved Style produced by the Block Styling System, and returns a Render Object that carries both the content and the resolved style.

---

## Block Type

Supported block type:

```text
title
```

The Title Renderer processes only Title Blocks.

---

## Renderer Category

Title Renderer is a:

```text
Content Renderer
```

It renders block-owned content.

---

## Semantic Meaning

A Title Block represents:

```text
Section Heading
```

Examples:

```text
About Me
Projects
Featured Links
Contact
Experience
Services
```

The Title Block is intended to introduce a section.

It is not intended for paragraph content.

---

## Ownership Model

### Title Block Owns

```text
value
```

The Title Block is the single source of truth for its content.

---

## Data Source

The renderer reads content directly from:

```text
Title Block
```

The renderer does not read data from:

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
Title Block
```

Containing:

```text
value
```

### Resolved Style

The renderer also receives the **Resolved Style** for this block, produced by the Block Styling System. The renderer attaches it to the Render Object unchanged and never reads the Theme directly.

---

## Resolution Flow

```text
Title Block
↓
Read Title Content
↓
Validate Content
↓
Create Render Object
↓
Return Result
```

---

## Successful Resolution

If valid title content exists:

```text
value
```

the renderer returns a Render Object.

---

## Render Object

Example:

```json
{
  "id": "block_title_1",
  "type": "title",
  "content": {
    "value": "Projects"
  },
  "resolved_style": {}
}
```

---

## Content Structure

### value

Represents the title text.

Example:

```json
{
  "value": "Projects"
}
```

---

## Required Fields

Required:

```text
value
```

---

## Empty Content

If:

```text
value = empty
```

or

```text
value = null
```

the renderer must return:

```text
null
```

No Render Object should be generated.

---

## Invalid Content

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

The Title Renderer must return content exactly as stored.

The renderer must not:

```text
Rewrite Titles
Translate Titles
Optimize Titles
Generate Titles
Shorten Titles
Expand Titles
```

---

## Semantic Preservation

The renderer preserves the semantic meaning of:

```text
Section Heading
```

The renderer does not convert titles into:

```text
Paragraphs
Descriptions
Captions
Buttons
```

---

## Renderer Responsibilities

The Title Renderer may:

* Read title content
* Validate content
* Normalize content
* Produce Render Objects

---

## Renderer Non-Responsibilities

The Title Renderer must not:

* Create titles
* Suggest titles
* Rewrite titles
* Store titles
* Analyze titles

These responsibilities belong elsewhere.

---

## Presentation Independence

The Title Renderer must not define:

```text
Font
Font Weight
Font Size
Color
Spacing
Alignment
Decoration
Animation
```

Examples:

```text
Large Heading
Centered Heading
Bold Heading
Underlined Heading
```

are resolved by the Block Styling System and attached to the Render Object as Resolved Style.

The renderer does not resolve or decide them.

---

## Theme Independence

The Title Renderer must not know:

```text
Theme
Layout
Template
Visual Style
```

The renderer returns content plus the Resolved Style it is given. It never reads the Theme.

---

## Relationship With Textbox

The Title Block and Textbox Block are independent.

Valid:

```text
Title
↓
Textbox
```

But:

```text
No Parent Relationship
No Child Relationship
```

The Title Renderer never reads Textbox content.

The Textbox Renderer never reads Title content.

---

## Multiple Title Blocks

V1 allows:

```text
Unlimited Title Blocks
```

Each Title Block is rendered independently.

---

## Rendering Failure

The renderer returns:

```text
null
```

when title content cannot be resolved safely.

The renderer must never throw public-facing errors.

---

## Public Profile Flow

```text
Title Block
↓
Title Renderer
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
Title Resolution
Semantic Heading Representation
Render Object Creation
Null-safe Rendering
```

Not Supported:

```text
Heading Levels
Title Variants
Generated Titles
Conditional Titles
Localized Titles
```

---

## Final Rule

The Title Renderer answers one question:

```text
What section heading data should be available for presentation?
```

It does not answer:

```text
How should the Theme be resolved, or how should blocks be composed into a page?
```
