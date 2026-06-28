# Theme Catalog Specification V1

## Context

The Theme Catalog is a subsystem within the Theme Library.

The Theme Library is a subsystem of the Appearance Domain.

The Appearance Domain is responsible for the visual state of a Minime profile.

The Theme Catalog provides the platform-managed Theme Definitions that profiles may select through the Design editor.

---

## Purpose

The Theme Catalog is the single source of all Theme Definitions available to profiles.

Themes are platform assets.

Users select Themes through the Design editor.

Users do not create Themes in V1.

---

## Philosophy

```text
Themes Are Provided
Not Created
```

---

## Theme Catalog Structure

The Theme Catalog is a collection of Theme Definitions.

```text
Theme Catalog
├─ Theme A
├─ Theme B
├─ Theme C
└─ Theme N
```

Each Theme Definition is independently defined.

---

## What The Catalog Stores

The Theme Catalog stores Theme Definitions only.

```text
Theme Catalog Stores:
- Theme Definitions
- Theme Metadata
- Theme Defaults
- Theme Constraints
- Theme Availability State
```

The Theme Catalog does not store:

```text
- Profile Appearance State
- User Customization Values
- Block Content
- Profile Content
- Analytics
- Account Data
```

Profile-level customization is stored in the Appearance State.

See: theme.customization.specification.v1.md

---

## Catalog Separation Rule

The Theme Catalog stores:

```text
Theme Definitions
```

The Appearance State (per profile) stores:

```text
selected_theme_id
theme customization values
```

Separation is mandatory.

---

## Theme Definition

A Theme Definition represents a reusable visual preset.

```text
Theme Definition
├─ Metadata
├─ Defaults
├─ Configuration Rules
├─ Constraints
└─ Visual Identity
```

Theme Definitions are immutable catalog assets.

Structure details are defined in: theme.definition.specification.v1.md

---

## Theme Identity

Every Theme Definition must have a unique identity.

Fields:

```text
theme_id
theme_key
theme_name
theme_version
```

Theme identity remains stable across profile usage.

---

## Theme Availability

Themes may exist in one of two states.

### Available

Theme may be selected by profiles.

```text
Available
```

### Unavailable

Theme may not be selected by new profiles.

```text
Unavailable
```

Existing profiles using an unavailable Theme continue functioning without interruption.

---

## Theme Categories

Themes may optionally be grouped into categories.

Examples:

```text
Minimal
Creator
Business
Portfolio
Dark
Light
```

Categories are organizational only.

They do not affect rendering.

---

## Theme Defaults

Each Theme Definition provides default visual values.

Examples:

```text
Colors
Typography
Background
Radius
Spacing
Block Defaults
```

These defaults are the starting point before user customization is applied.

---

## Theme Constraints

Each Theme Definition defines customization boundaries.

Examples:

```text
Allowed Fonts
Allowed Radius Range
Allowed Background Types
Allowed Color Rules
```

Constraints protect Theme visual identity.

Constraint details are defined in: theme.constraints.specification.v1.md

---

## Theme Selection

Profiles select a Theme from the Catalog through the Design editor.

Flow:

```text
Theme Catalog
        ↓
Theme Selection (Design editor)
        ↓
Appearance State Updated
```

Selection behavior is defined in: theme.selection.specification.v1.md

---

## Default Theme

The platform defines a system default Theme.

Flow:

```text
Profile Created
        ↓
Default Theme Assigned
        ↓
Appearance State Initialized
```

All new profiles receive the default Theme.

---

## Theme Updates

Theme Definitions may be updated by the platform.

Example:

```text
Theme v1
        ↓
Theme v2
```

Update behavior is controlled by platform migration rules.

Profiles continue referencing the same Theme identity across versions.

---

## Theme Deprecation

Themes may be deprecated.

Characteristics:

```text
Not Selectable by New Profiles
Still Supported for Existing Profiles
```

---

## Theme Retirement

Themes may be retired.

Characteristics:

```text
Removed From Catalog
Migration Required for Affected Profiles
```

Retirement behavior is managed by platform migration rules.

---

## Not Supported In V1

```text
User-Created Themes
Theme Marketplace
Theme Uploads
Theme Imports
Theme Exports
Theme Sharing
Theme Selling
Community Themes
Theme Forking
```

---

## V1 Principles

```text
Theme Catalog Is A Subsystem Within The Appearance Domain

Themes Are Platform Assets

Users Select Themes Through The Design Editor

Users Do Not Create Themes

Catalog Stores Theme Definitions Only

Appearance State Stores Selection And Customization

Theme Identity Is Stable

Deprecated Themes Remain Functional For Existing Profiles

One Profile References One Theme At A Time
```
