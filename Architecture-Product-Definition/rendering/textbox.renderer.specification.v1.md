# Minime Textbox Renderer Specification V1

## Purpose

The Textbox Renderer is responsible for rendering Textbox Blocks.

The Textbox Renderer renders content owned by the Textbox Block itself.

The renderer consumes the block content together with the Resolved Style produced by the Block Styling System, and returns a Render Object that carries both the content and the resolved style.

---

## Block Type

Supported block type:

```text
textbox
```

The Textbox Renderer processes only Textbox Blocks.

---

## Renderer Category

Textbox Renderer is a:

```text
Content Renderer
```

It renders block-owned content.

---

## Semantic Meaning

A Textbox Block represents:

```text
Section Content
```

Examples:

```text
Descriptions
Introductions
Summaries
Paragraphs
Explanations
Personal Notes
```

The Textbox Block is intended to contain readable content.

It is not intended to represent a heading.

---

## Ownership Model

### Textbox Block Owns

```text
value
```

The Textbox Block is the single source of truth for its content.

---

## Data Source

The renderer reads content directly from:

```text
Textbox Block
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
Textbox Block
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
Textbox Block
↓
Read Textbox Content
↓
Validate Content
↓
Create Render Object
↓
Return Result
```

---

## Successful Resolution

If valid textbox content exists:

```text
value
```

the renderer returns a Render Object.

---

## Render Object

Example:

```json
{
  "id": "block_textbox_1",
  "type": "textbox",
  "content": {
    "value": "I help founders build brands and digital products."
  },
  "resolved_style": {}
}
```

---

## Content Structure

### value

Represents the textbox content.

Example:

```json
{
  "value": "I help founders build brands and digital products."
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

The Textbox Renderer must return content exactly as stored.

The renderer must not:

```text
Rewrite Text
Summarize Text
Translate Text
Optimize Text
Generate Text
Expand Text
Shorten Text
```

---

## Formatting Preservation

The Textbox Renderer should preserve stored formatting.

Examples:

```text
Line Breaks
Paragraph Breaks
Spacing
```

must remain intact.

The renderer does not alter formatting.

---

## Semantic Preservation

The renderer preserves the semantic meaning of:

```text
Section Content
```

The renderer does not convert textbox content into:

```text
Headings
Buttons
Captions
Labels
```

---

## Relationship With Title

The Textbox Block and Title Block are independent.

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

The Textbox Renderer never reads Title content.

The Title Renderer never reads Textbox content.

---

## Renderer Responsibilities

The Textbox Renderer may:

* Read textbox content
* Validate content
* Normalize content
* Produce Render Objects

---

## Renderer Non-Responsibilities

The Textbox Renderer must not:

* Create content
* Rewrite content
* Analyze content
* Improve content
* Store content

These responsibilities belong elsewhere.

---

## Presentation Independence

The Textbox Renderer must not define:

```text
Font
Font Size
Color
Alignment
Line Height
Spacing
Animation
```

Examples:

```text
Large Paragraph
Centered Text
Muted Text
Rich Text Style
```

are resolved by the Block Styling System and attached to the Render Object as Resolved Style.

The renderer does not resolve or decide them.

---

## Theme Independence

The Textbox Renderer must not know:

```text
Theme
Layout
Template
Visual Style
```

The renderer returns content plus the Resolved Style it is given. It never reads the Theme.

---

## Multiple Textbox Blocks

V1 allows:

```text
Unlimited Textbox Blocks
```

Each Textbox Block is rendered independently.

---

## Rendering Failure

The renderer returns:

```text
null
```

when textbox content cannot be resolved safely.

The renderer must never throw public-facing errors.

---

## Public Profile Flow

```text
Textbox Block
↓
Textbox Renderer
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
Textbox Content Resolution
Formatting Preservation
Render Object Creation
Null-safe Rendering
```

Not Supported:

```text
Rich Text Rendering
Markdown Rendering
Generated Content
Conditional Content
Localized Content
AI Content Generation
```

---

## Final Rule

The Textbox Renderer answers one question:

```text
What section content should be available for presentation?
```

It does not answer:

```text
How should the Theme be resolved, or how should blocks be composed into a page?
```
