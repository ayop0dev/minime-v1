# Theme Selection Specification V1

## Context

Theme selection is the mechanism by which a profile's Appearance State acquires a Theme reference.

This is a subsystem of the Appearance Domain.

Theme selection is a user-facing action performed through the Design editor.

---

## Purpose

This document defines:

- How a profile selects a Theme
- How the Appearance State is built from the selected Theme
- How Theme changes are applied
- How invalid states are handled

---

## Always-Live Rule

Minime V1 is always-live.

There are no drafts.

There is no preview/live split.

When a Theme is selected or customized, the Appearance State is updated immediately.

The Renderer always consumes the latest resolved Appearance State.

---

## One Profile, One Theme

```text
One Profile
=
One Active Theme
```

A profile always references exactly one Theme.

There is no theme-less profile state.

---

## What The Appearance State Stores

The Appearance State (stored per profile) contains:

```text
selected_theme_id
theme customization values
```

Profile ownership is via account_id.

There is no profile_id ownership.

There is no user_id ownership.

There is no owner_account_id field.

---

## Theme Selection Flow

Users select Themes through the Design editor.

Flow:

```text
Theme Catalog
        ↓
User Selection (Design editor)
        ↓
Appearance State Updated
        ↓
Renderer Consumes Updated State
```

---

## Theme States

A Theme reference within a profile passes through these states:

```text
Available (in catalog)
        ↓
Selected / Active
        ↓
Customized (optional)
        ↓
Switched (when replaced)
```

---

## State: Available

The Theme exists in the Theme Catalog.

It can be selected by any profile.

It is not yet attached to a profile.

---

## State: Selected / Active

The profile's Appearance State references this Theme.

Characteristics:

```text
Used by Renderer
Applies to Entire Profile
Visible to Visitors Immediately
```

Rule:

```text
One Profile = One Active Theme
```

---

## State: Customized

The user modifies Theme customization values within the Appearance State.

Examples of customizable values:

```text
Colors
Typography
Background
Radius
Shadows
```

Characteristics:

```text
Theme Identity Unchanged
Appearance State Updated Immediately
Changes Are Visible to Visitors Immediately
```

---

## State: Switched

The user selects a different Theme from the catalog.

Flow:

```text
Theme A (active)
        ↓
User Selects Theme B (Design editor)
        ↓
Appearance State Updated
        ↓
Theme B (active)
        ↓
Renderer Consumes New State
```

Only one Theme is active at a time.

---

## Initial Theme Assignment

When a profile is created:

```text
Profile Created
        ↓
Default Theme Assigned
        ↓
Appearance State Initialized
```

The platform selects the system default Theme.

---

## Appearance State Resolution Pipeline

When a Theme is selected or customized, the Appearance State is resolved.

```text
Theme Definition (from catalog)
        ↓
Theme Customization (from Appearance State)
        ↓
Constraint Validation
        ↓
Resolved Appearance State
        ↓
Renderer
```

---

## Step 1 — Load Theme Definition

The selected Theme Definition is loaded from the Theme Catalog.

The Theme Definition provides:

```text
Metadata
Defaults
Constraints
```

---

## Step 2 — Apply Customization

Profile-level customization values from the Appearance State are applied.

Examples:

```text
Primary Color
Typography
Background
Spacing
Radius Scale
Shadow Style
```

Customized values override Theme Definition defaults.

---

## Step 3 — Validate Against Constraints

Customized values must satisfy Theme Constraints.

Invalid values are rejected.

The Appearance State remains unchanged on validation failure.

---

## Step 4 — Resolved Appearance State

The system produces a resolved Appearance State.

Structure:

```text
Resolved Appearance State
├─ Background
├─ Colors
├─ Typography
├─ Spacing
├─ Radius
├─ Borders
├─ Shadows
├─ Animations
└─ Block Defaults
```

This is consumed by the Renderer.

---

## Theme Change Behavior

When Theme customization values change:

```text
Appearance State Updated
        ↓
Revalidate
        ↓
Re-resolve
        ↓
Renderer Consumes Updated State
```

---

## Theme Switching Behavior

When the selected Theme changes:

```text
Previous Theme Reference Cleared
        ↓
New Theme Definition Loaded
        ↓
Customization Reset to New Theme Defaults
        ↓
Validated
        ↓
Appearance State Updated
        ↓
Renderer Consumes New State
```

---

## Design Editor — Browse And Select

When a user browses Themes in the Design editor:

```text
User Views Theme Catalog
        ↓
User Previews Theme (transient Design-editor UI only)
        ↓
User Confirms Selection
        ↓
Appearance State Updated
```

Important rules:

- Theme preview during browsing is a transient UI interaction within the Design editor only.
- It is not a profile state.
- It is not a published/unpublished split.
- There is no preview/live profile split.
- Confirming selection immediately updates the live Appearance State.

---

## Failure Handling

If resolution fails:

```text
Use Last Valid Appearance State
```

or:

```text
Use System Default Theme
```

Invalid states must never reach the Renderer.

---

## Deterministic Resolution

Given identical inputs:

```text
Theme Definition
Customization Values
Constraints
```

The system must always produce the same resolved Appearance State.

Rule:

```text
Resolution Is Deterministic
```

---

## Performance Principle

Resolution occurs once per change.

Downstream systems reuse the resolved result.

Rule:

```text
Resolve Once
Consume Many
```

---

## Renderer Independence

Theme selection produces data.

The Renderer produces UI.

These are separate concerns.

The Renderer always consumes the latest resolved Appearance State.

---

## Not Supported In V1

```text
Multiple Active Themes Per Profile
Page-Level Themes
Sub Page Themes
Conditional Themes
Scheduled Theme Changes
User-Created Themes
Theme Cloning
Theme Export / Import
Preview / Live Profile Split
```

---

## V1 Principles

```text
One Profile = One Active Theme

Theme Selection Updates Appearance State Immediately

Minime Is Always-Live — No Preview / Live Split

Theme Definition Provides Defaults

Customization Is Stored In Appearance State

Resolution Is Deterministic

Renderer Consumes Resolved Appearance State

Invalid Values Are Rejected Before Rendering

Profiles Always Have An Active Theme

account_id Is The Ownership Identifier
```
