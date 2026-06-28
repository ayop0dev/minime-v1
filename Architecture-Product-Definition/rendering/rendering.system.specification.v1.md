# Block Renderer Specification V1

## Purpose

This document defines the Block Renderer used by Minime.

The Block Renderer is responsible for transforming Block Data and Resolved Styles into Render Objects.

A Render Object carries the block's normalized content together with its Resolved Style. The Renderer does not build final UI; the Rendering Engine composes the final output from Render Objects.

The Renderer does not make styling decisions.

The Renderer does not resolve inheritance.

The Renderer does not process Theme configuration.

The Renderer only renders finalized data.

---

# Philosophy

The Renderer is a data-transformation layer.

It receives prepared inputs and produces a Render Object.

Rule:

```text
Renderer
Does Not Decide

Renderer
Transforms Into A Render Object
```

---

# Responsibilities

The Block Renderer is responsible for:

* Transforming blocks into Render Objects
* Normalizing content
* Attaching the Resolved Style it is given
* Producing Render Objects (content + resolved style)
* Handling block-specific transformation logic

---

# Out Of Scope

The Block Renderer is not responsible for:

* Theme resolution
* Theme configuration
* Theme validation
* Style inheritance
* Style overrides
* Style resolution
* Layout management
* Analytics
* Out Links
* Content management

These responsibilities belong to other modules.

---

# Renderer Inputs

The Renderer receives:

```text
Block Data
Resolved Style
```

The Renderer requires both inputs.

---

# Renderer Contract

The Renderer consumes:

```text
Resolved Block Style
```

The Renderer never consumes:

```text
Theme Defaults
Theme Configuration
Theme Constraints
Block Overrides
```

directly.

All styling decisions must already be resolved.

---

# Rendering Pipeline

Rendering follows a fixed sequence.

```text
Block Content
        +
Resolved Style
        ↓
Renderer
        ↓
Render Object (content + resolved style)
        ↓
Rendering Engine
        ↓
Final Output
```

The Renderer never modifies input data.

---

# Block Structure

The Renderer receives:

```text
Block
├─ Type
├─ Content
├─ Settings
└─ Resolved Style
```

Only renderable data is supplied.

---

# Block Type Rendering

Each block type owns its own rendering logic.

Examples:

```text
Image
Avatar
Name
Bio
Button
Text
Divider
Social Icons
```

Each renderer interprets content differently.

---

# Image Rendering

Input:

```text
Image Source
Resolved Style
```

Output:

```text
Image Render Object
```

---

# Avatar Rendering

Input:

```text
Avatar Source
Resolved Style
```

Output:

```text
Avatar Render Object
```

---

# Name Rendering

Input:

```text
Text Content
Resolved Style
```

Output:

```text
Name Render Object
```

---

# Bio Rendering

Input:

```text
Bio Content
Resolved Style
```

Output:

```text
Bio Render Object
```

---

# Button Rendering

Input:

```text
Button Label
Out Link Reference
Resolved Style
```

Output:

```text
Button Render Object
```

---

# Text Rendering

Input:

```text
Text Content
Resolved Style
```

Output:

```text
Text Render Object
```

---

# Divider Rendering

Input:

```text
Divider Settings
Resolved Style
```

Output:

```text
Divider Render Object
```

---

# Social Icons Rendering

Input:

```text
Icon Data
Resolved Style
```

Output:

```text
Social Icons Render Object
```

---

# Style Consumption

The Renderer treats all style values as final.

Example:

Input:

```text
Radius = 24px
Shadow = Soft
Color = Black
```

Renderer behavior:

```text
Attach Values To The Render Object
```

The Renderer never asks:

```text
Where Did This Value Come From?
```

Source tracking is outside Renderer scope.

---

# Theme Independence

The Renderer has no knowledge of:

```text
Theme Identity
Theme Catalog
Theme Configuration
Theme Lifecycle
```

The Renderer operates only on resolved values.

---

# Inheritance Independence

The Renderer has no knowledge of:

```text
Inheritance
Overrides
Theme Defaults
```

These concerns are resolved before rendering begins.

---

# Layout Independence

The Renderer does not determine:

```text
Block Position
Page Structure
Layout Direction
Layout Strategy
```

Layout decisions belong to the Rendering Domain.

---

# Rendering Determinism

Given identical inputs:

```text
Block Data
Resolved Style
```

the Renderer must produce:

```text
Identical Output
```

Rule:

```text
Rendering Must Be Deterministic
```

---

# Stateless Rendering

Rendering is stateless.

Rule:

```text
Same Input
=
Same Output
```

The Renderer must not store rendering state.

---

# Error Handling

If rendering fails:

```text
Block Error
```

the failure should remain isolated.

Rule:

```text
One Block Failure
≠
Profile Failure
```

Other blocks must continue rendering.

---

# Performance Principle

The Renderer should not perform:

```text
Theme Resolution
Style Resolution
Inheritance Resolution
Validation
```

Rule:

```text
Render Only
```

All expensive calculations occur upstream.

---

# Relationship With Appearance system

Flow:

```text
Appearance system (Theme Library + Appearance State)
        ↓
Resolved Appearance State
```

The Renderer does not consume this output directly.

---

# Relationship With Block Styling System

Flow:

```text
Theme
        ↓
Block Styling
        ↓
Resolved Style
        ↓
Renderer
```

The Renderer consumes only the final resolved style.

---

# Relationship With Rendering Engine

Flow:

```text
Renderer
        ↓
Render Objects (content + resolved style)
        ↓
Rendering Engine
        ↓
Page Assembly
```

The Rendering Engine controls composition and layout.

The Renderer controls block transformation into Render Objects only.

---

# Not Supported In V1

```text
Runtime Theme Resolution
Runtime Style Resolution
Conditional Rendering Rules
Theme-Aware Rendering
Cross-Block Rendering Logic
AI Rendering Logic
```

---

# V1 Principles

```text
Renderer Consumes Final Data

Renderer Consumes Resolved Styles

Renderer Does Not Resolve Themes

Renderer Does Not Resolve Overrides

Renderer Does Not Resolve Inheritance

Renderer Is Stateless

Renderer Is Deterministic

Renderer Is Layout Independent

Renderer Renders Blocks Only

Render Only
```
