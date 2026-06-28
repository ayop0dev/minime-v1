# Rendering Architecture Canon V1

**Status:** Canonical
**Version:** V1
**Authority:** Level 4 — Domain Specification
**Layer:** Rendering

---

## Purpose

This document is the constitutional entry point for the Minime Rendering Architecture.

It explains the complete Rendering layer as an architectural system.

A first-time reader who reads only this document shall understand:

* Why Rendering exists
* What Rendering owns
* What Rendering never owns
* Everything Rendering consumes
* Everything Rendering produces
* How Rendering interacts with the repository
* Every architectural principle governing Rendering

Detailed behavioral specifications exist for each component and are linked at the end of this document.

---

## Constitutional Position

Rendering is a Product Domain.

It is the fourteenth domain in the Minime V1 product architecture.

It occupies the stage between stored canonical profile data and the public-facing profile experience.

```text
Product Journey Position

... → Profile Content → Blocks → Appearance → Rendering → Public Profile → ...
```

Rendering is a consumer domain. It reads from every upstream domain and produces presentation output.

It owns no business data.

It produces no canonical records.

---

## Rendering Philosophy

```text
Rendering transforms architecture into presentation.
Rendering never owns business meaning.
Rendering never owns canonical data.
Rendering remains stateless.
Rendering consumes only.
Rendering never becomes a source of truth.
```

Rendering is the translation layer between canonical data and visible profile output.

The user's data exists before Rendering begins. The user's data exists after Rendering ends. Rendering is the temporary bridge between them.

A correct Rendering implementation is invisible. It adds no data. It removes no data. It only presents what already exists.

---

## Rendering Responsibilities

Rendering owns:

```text
Selecting the correct renderer for each block type (Renderer Registry)

Transforming block data into normalized Render Objects (Block Renderers)

Resolving reference blocks to their canonical profile content (Reference Renderers)

Resolving collection blocks to their canonical external data (Collection Reference Renderers)

Attaching Resolved Style to each Render Object (already computed by Block Styling System)

Composing Render Objects into a complete page structure (Rendering Engine)

Applying Layout configuration to the composed profile (Layout System)

Assembling and delivering the final public profile output (Profile Presentation System)

Emitting SEO metadata into the final HTML output (per SEO Domain definitions)

Loading approved integration providers during page generation (per Integrations Domain decisions)
```

---

## Rendering Non-Responsibilities

Rendering does not own:

```text
Profile content (owned by Profile Content)

Block definitions and block data (owned by Blocks)

Block ordering decisions (owned by Profile Content via sort_order)

Social account records (owned by Connected Accounts)

Theme selection (owned by Appearance)

Theme customization (owned by Appearance)

Style inheritance resolution (owned by Block Styling System)

Style override processing (owned by Block Styling System)

Resolved Style computation (owned by Block Styling System — Rendering only consumes the result)

Layout creation (Layout definitions are external inputs)

SEO rules and indexing policies (owned by SEO Domain)

Integration provider registration, validation, and eligibility (owned by Integrations Domain)

Analytics collection (owned by Analytics)

Business rule validation

Canonical entity ownership

Data persistence

Authentication

Storage
```

---

## Rendering Inputs

Rendering consumes the following inputs. For each input, the producer, owner, and read authority are explicit.

### Input 1 — Profile Content

**Producer:** Profile Content domain
**Owner:** Account (via Profile Content)
**What Rendering reads:**
- `display_name` (consumed by Name Block Renderer via Name Block reference)
- `avatar` (consumed by Avatar Block Renderer via Avatar Block reference)
- `bio` (consumed by Bio Block Renderer via Bio Block reference)
- `blocks` list (all blocks, with their `block_id`, `type`, `sort_order`, `data`, `settings`)
- `appearance_state` (feeds Block Styling System)

**Read authority:** Rendering reads Profile Content. It does not own it. It does not modify it.

---

### Input 2 — Blocks

**Producer:** Blocks domain (block-level content produced by user)
**Owner:** Account (via Blocks domain)
**What Rendering reads:**
- Per-block `type` (determines renderer selection)
- Per-block `sort_order` (determines composition order)
- Per-block `data` and `settings` (consumed by the specific block renderer)

**Read authority:** Rendering reads Block data. It does not own it. It does not modify it.

---

### Input 3 — Appearance State and Theme

**Producer:** Appearance domain (selection); Theme Catalog (definitions)
**Owner:** Account (for selection); System (for theme definitions)
**What Rendering reads:**
- `selected_theme_id` (which theme applies)
- `theme_customization_values` (user-applied overrides to theme defaults)
- Theme definitions (from Theme Catalog — colors, typography, spacing, radius, shadow)

**Read authority:** Rendering reads Appearance State and Theme definitions. The Block Styling System resolves styles from this input before Rendering begins.

---

### Input 4 — Resolved Style (Block Styling System output)

**Producer:** Block Styling System (internal to Rendering pipeline, upstream of Block Renderers)
**Owner:** Not independently owned — derived from Theme and Appearance State
**What Rendering reads:**
- Pre-resolved final style values per block (colors, typography, spacing, radius, border, shadow, alignment)

**Read authority:** Resolved Style is an intermediate computation product consumed by Block Renderers. Renderers attach it to Render Objects unchanged.

**Key rule:** Rendering consumes Resolved Style. Rendering never computes Resolved Style. Style resolution happens upstream in the Block Styling System.

---

### Input 5 — Connected Accounts

**Producer:** Social Accounts Setup (processing); Connected Accounts domain (canonical storage)
**Owner:** Account (via Connected Accounts)
**What Rendering reads:**
- `platform`, `username`, `url` for each Connected Account (consumed by Social Icons Renderer)

**Read authority:** Rendering reads Connected Accounts as a collection reference. It does not own the records. It does not modify them. It never regenerates URLs, normalizes identifiers, or applies platform rules. It consumes the stored canonical `url` exactly as saved.

---

### Input 6 — SEO Definitions

**Producer:** SEO Domain (Cross-Cutting Product Capability)
**Owner:** SEO Domain
**What Rendering reads:**
- HTML title rule
- Meta description rule
- Canonical URL rule
- Robots directive rule
- Open Graph metadata rules
- Twitter Card metadata rules
- Structured data rules

**Read authority:** Rendering reads SEO definitions and emits the resulting metadata into final HTML output. Rendering does not own SEO rules. Rendering does not invent metadata independently.

---

### Input 7 — Integration Eligibility Decisions

**Producer:** Integrations Domain (Cross-Cutting Product Capability)
**Owner:** Integrations Domain
**What Rendering reads:**
- Which providers are eligible to load for this page
- Provider loading configuration

**Read authority:** Rendering reads Integrations eligibility decisions and executes provider loading. Rendering does not own provider eligibility. Rendering does not bypass eligibility rules.

---

### Input 8 — Layout Configuration

**Producer:** Layout System (internal to Rendering)
**Owner:** Rendering (Layout is an internal Rendering concern)
**What Rendering reads:**
- Profile width
- Profile container definition
- Global alignment rules
- Global spacing rules
- Profile structure

**Read authority:** Layout is owned by the Rendering layer. It is an internal input to the Rendering Engine, not an external dependency.

---

## Rendering Processing Pipeline

The complete Rendering Architecture processes inputs through six sequential stages.

```text
Stage 1 — Style Preparation
Canonical Data (Appearance State + Theme)
↓
Block Styling System
↓
Resolved Style (per block, final values)

Stage 2 — Renderer Selection
Block Type
↓
Renderer Registry
↓
Renderer (one per block type)

Stage 3 — Block Rendering
Block Data + Resolved Style
↓
Block Renderer (type-specific)
↓
Render Object (content + resolved style)

Stage 4 — Composition
Render Objects (ordered list)
+ Layout Configuration
↓
Rendering Engine
↓
Composed Profile Structure

Stage 5 — Assembly
Composed Profile Structure
↓
Profile Presentation System
↓
Public Profile UI

Stage 6 — Cross-Cutting Output
SEO Definitions → Rendering → HTML Metadata (emitted into final page)
Integration Decisions → Rendering → Provider Activation (loaded into final page)
```

### Full Pipeline View

```text
Appearance State + Theme
        ↓
Block Styling System
        ↓
Resolved Style
        ↓
Blocks (ordered) + Block Data
        ↓
Renderer Registry → Block Renderer (type-specific)
        ↓
Render Objects (content + resolved style)
        ↓
Rendering Engine + Layout
        ↓
Profile Presentation System
        ↓
Public Profile UI
        +
        ↓ (in parallel)
SEO Definitions → HTML Metadata embedded in page
Integration Decisions → Provider Loading embedded in page
```

### Stage Responsibilities

| Stage | Component | Input | Output |
|---|---|---|---|
| Style Preparation | Block Styling System | Appearance State, Theme | Resolved Style per block |
| Renderer Selection | Renderer Registry | Block type | Renderer reference |
| Block Rendering | Block Renderer (type-specific) | Block Data + Resolved Style | Render Object |
| Composition | Rendering Engine | Ordered Render Objects + Layout | Composed profile |
| Assembly | Profile Presentation System | Composed profile | Public Profile UI |
| SEO Output | Rendering Engine | SEO Definitions | HTML metadata |
| Provider Loading | Rendering Engine | Integration Decisions | Active providers |

---

## Rendering Outputs

Rendering produces presentation artifacts only. It never produces canonical data.

### Output 1 — Public Profile UI

The complete rendered visual profile.

Contains all visible blocks in sort_order, with styles applied, composed using the selected layout.

This is the primary Rendering output.

---

### Output 2 — SEO Metadata (embedded in HTML)

HTML title, meta description, canonical URL, robots directives, Open Graph metadata, Twitter Card metadata, structured data.

These are emitted by Rendering into the final HTML document per SEO Domain definitions.

Rendering does not own these values. Rendering emits them.

---

### Output 3 — Active Integration Providers (embedded in page)

Approved provider activations loaded during page generation per Integrations Domain eligibility decisions.

Rendering does not own provider eligibility. Rendering executes loading.

---

### Output 4 — Render Objects (intermediate)

The internal per-block data structures produced by Block Renderers.

Render Objects are not a final output — they are consumed by the Rendering Engine in Stage 4.

Each Render Object contains:
- `id` — original block_id
- `type` — block type
- `content` — normalized, renderable content
- `resolved_style` — final style values (computed upstream, carried unchanged)

---

## Rendering Contracts

### Rendering ← Profile Content

**Producer:** Profile Content domain
**Consumer:** Rendering (read-only)
**Ownership:** Profile Content owns display_name, avatar, bio, blocks list, appearance_state
**Read:** Rendering reads Profile Content to populate block renderers
**Write:** Rendering never writes to Profile Content
**Dependency direction:** Rendering depends on Profile Content. Profile Content does not depend on Rendering.
**Non-goal:** Rendering does not modify, validate, or store profile content.

---

### Rendering ← Appearance

**Producer:** Appearance domain
**Consumer:** Rendering (read-only, via Block Styling System)
**Ownership:** Appearance owns theme selection, theme customization, appearance state
**Read:** Block Styling System reads Appearance State to produce Resolved Style
**Write:** Rendering never writes to Appearance
**Dependency direction:** Rendering depends on Appearance. Appearance does not depend on Rendering.
**Non-goal:** Rendering does not select themes, apply themes, or define design tokens.

---

### Rendering ← Block Styling System

**Producer:** Block Styling System (Theme + Appearance State → Resolved Style)
**Consumer:** Block Renderers (attach Resolved Style to Render Objects)
**Ownership:** Resolved Style is an intermediate computation — not independently owned
**Read:** Block Renderers receive Resolved Style and attach it unchanged to Render Objects
**Write:** Block Renderers never recompute or modify Resolved Style
**Dependency direction:** Block Renderers depend on Block Styling System output. Block Styling System does not depend on Block Renderers.
**Non-goal:** Block Renderers do not resolve inheritance, process overrides, or apply theme defaults.

---

### Rendering ← Connected Accounts

**Producer:** Connected Accounts domain
**Consumer:** Social Icons Renderer (read-only)
**Ownership:** Connected Accounts owns all social account records (platform, username, url)
**Read:** Social Icons Renderer reads Connected Account records to populate social icons content
**Write:** Rendering never writes to Connected Accounts
**Dependency direction:** Social Icons Renderer depends on Connected Accounts. Connected Accounts does not depend on Rendering.
**Non-goal:** Rendering does not regenerate URLs, normalize identifiers, or apply platform rules. It consumes stored canonical values exactly as saved.

---

### Rendering ← SEO

**Producer:** SEO Domain (Cross-Cutting Product Capability)
**Consumer:** Rendering Engine (reads SEO definitions; emits resulting metadata into HTML)
**Ownership:** SEO Domain owns metadata generation rules. Rendering owns HTML emission.
**Read:** Rendering reads SEO definitions to determine what metadata to emit
**Write:** Rendering never writes to SEO definitions
**Dependency direction:** Rendering depends on SEO. SEO does not depend on Rendering.
**Non-goal:** Rendering does not invent metadata independently. SEO does not emit HTML directly.

---

### Rendering ← Integrations

**Producer:** Integrations Domain (Cross-Cutting Product Capability)
**Consumer:** Rendering Engine (reads eligibility decisions; activates approved providers)
**Ownership:** Integrations Domain owns provider eligibility. Rendering owns provider loading execution.
**Read:** Rendering reads Integrations eligibility decisions to determine which providers to load
**Write:** Rendering never writes to Integrations configuration
**Dependency direction:** Rendering depends on Integrations. Integrations does not depend on Rendering.
**Non-goal:** Rendering does not determine provider eligibility. Integrations does not execute provider loading.
**Failure rule:** Provider failures must never interrupt profile rendering. Failure isolation is required.

---

## Block Renderer Categories

Block Renderers are organized into three categories.

### Category 1 — Reference Renderers

Read a profile-owned field and produce a Render Object referencing that content.

```text
Avatar Renderer     → reads ProfileContent.avatar
Name Renderer       → reads ProfileContent.display_name
Bio Renderer        → reads ProfileContent.bio
```

The block owns placement in the profile. Profile Content owns the actual content.

---

### Category 2 — Content Renderers

Render the block's own stored content directly.

```text
Image Renderer      → reads block-stored image URL and alt text
Button Renderer     → reads block-stored label and Out Link reference
Divider Renderer    → reads block-stored settings (visual separator)
Title Renderer      → reads block-stored title text
Textbox Renderer    → reads block-stored text content
```

The block owns the content. The renderer normalizes and structures it.

---

### Category 3 — Collection Reference Renderers

Read an external collection and produce a Render Object containing the resolved collection items.

```text
Social Icons Renderer   → reads Connected Accounts (filtered by selection, ordered, visibility applied)
```

Connected Accounts owns the collection. The block owns selection preferences. The renderer resolves and structures the result.

---

## Rendering Principles

### Stateless Rendering

Rendering stores no state between requests.

The same profile content always produces the same rendered output.

Rendering maintains no session, no cache, no persistent rendering state.

Each render is an independent execution.

---

### Read-Only Architecture

Rendering reads canonical data.

Rendering never writes canonical data.

Rendering never modifies, validates, or updates business records.

Reading never transfers ownership.

---

### Deterministic Rendering

Given identical canonical inputs, Rendering must always produce identical output.

No randomness is permitted in rendering logic.

Rendering behavior must not depend on timing, location, request origin, or session state.

---

### Presentation Independence

Rendering decisions are not business decisions.

Rendering presents what canonical data says.

Rendering never overrides business meaning.

Rendering never applies editorial judgment to content.

---

### Implementation Independence

Rendering specifications describe responsibility and intent.

They do not prescribe implementation technology, framework, or infrastructure.

The Rendering Architecture remains valid regardless of which rendering technology is used.

---

### Single-Direction Data Flow

Canonical data flows into Rendering.

Presentation flows out.

Rendering never returns data upstream to canonical stores.

```text
Canonical Data → Rendering → Presentation Output

Never:

Rendering → Canonical Data
```

---

### Fail-Safe Block Rendering

A block that cannot render must return null.

A single block failure must never prevent the remaining profile from rendering.

Profile rendering must be resilient to individual block failures.

---

### Consumer Independence

Each Block Renderer operates independently.

One renderer must never require another renderer.

One renderer must never require data from another block.

Renderer Registry resolves each block independently.

---

## Rendering Constraints

```text
Rendering never stores canonical data.
Rendering never creates canonical records.
Rendering never modifies canonical records.
Rendering never validates business rules.
Rendering never owns canonical entities.
Rendering never mutates Product Domains.
Rendering never bypasses Platform Services.
Rendering never becomes a source of truth.
Rendering never applies business logic.
Rendering never overrides user data.
Rendering never selects themes.
Rendering never resolves style inheritance.
Rendering never processes style overrides.
Rendering never determines SEO rules.
Rendering never determines provider eligibility.
Rendering never authenticates users.
Rendering never persists events.
Rendering never owns AI suggestions.
Rendering never stores analytics.
```

---

## Rendering Future Evolution

### What May Extend

Future rendering evolution may extend:

* Additional block renderers (new block types)
* Additional layout configurations
* Additional SEO metadata formats
* Additional integration providers
* Performance improvements to the rendering pipeline
* Caching strategies within the rendering layer

---

### What May Never Violate

Future evolution may never violate:

* Stateless rendering (Rendering must never become stateful)
* Read-only architecture (Rendering must never write to canonical stores)
* Single-direction data flow (Canonical data → Rendering only)
* Deterministic rendering (Same inputs must always produce same outputs)
* Implementation independence (Architecture must remain technology-independent)
* Ownership boundaries (Rendering must never assume ownership of data it reads)
* Dependency direction (Upstream domains must never depend on Rendering)

---

## Rendering Non-Goals

Rendering is not responsible for:

```text
Business ownership
Data persistence
Authentication
Storage management
Analytics collection
SEO rule definition
SEO indexing decisions
Provider configuration
Provider eligibility determination
Block editing
Block validation
Style resolution
Theme selection
Theme customization
Content creation
Username management
Account management
Social account processing
Connected account management
Canonical entity ownership
Routing decisions
Public profile URL ownership
QR Code generation
AI suggestion generation
```

These responsibilities belong to their respective domains.

---

## Specification Index

The following specifications define detailed behavior for each Rendering component.

| Specification | Component | Description |
|---|---|---|
| `rendering.system.specification.v1.md` | Block Renderer | Block-to-Render Object transformation |
| `render-object.specification.v1.md` | Render Object | Normalized data structure contract |
| `renderer.registry.specification.v1.md` | Renderer Registry | Block type to renderer mapping |
| `rendering.engine.specification.v1.md` | Rendering Engine | Render Object composition and layout |
| `layout.system.specification.v1.md` | Layout System | Profile structure definitions |
| `minime.profile.presentation.system.specification.v1.md` | Profile Presentation | Page assembly and public profile output |
| `avatar.renderer.specification.v1.md` | Avatar Renderer | Reference renderer for ProfileContent.avatar |
| `name.renderer.specification.v1.md` | Name Renderer | Reference renderer for ProfileContent.display_name |
| `bio.renderer.specification.v1.md` | Bio Renderer | Reference renderer for ProfileContent.bio |
| `image.renderer.specification.v1.md` | Image Renderer | Content renderer for image blocks |
| `button.renderer.specification.v1.md` | Button Renderer | Content renderer for button blocks |
| `divider.renderer.specification.v1.md` | Divider Renderer | Content renderer for divider blocks |
| `title.renderer.specification.v1.md` | Title Renderer | Content renderer for title blocks |
| `textbox.renderer.specification.v1.md` | Textbox Renderer | Content renderer for textbox blocks |
| `social-icons.renderer.specification.v1.md` | Social Icons Renderer | Collection reference renderer for Connected Accounts |

---

## Cross-Domain References

Rendering interacts with the following specifications in other domains:

| Specification | Domain | Rendering Relationship |
|---|---|---|
| `profile-content/profile.content.specification.v1.md` | Profile Content | Provides display_name, avatar, bio, blocks, appearance_state |
| `blocks/block.system.specification.v1.md` | Blocks | Provides block definitions and data |
| `block-styling/block-styling.system.specification.v1.md` | Block Styling | Produces Resolved Style consumed by Block Renderers |
| `account-management/connected.accounts.specification.v1.md` | Connected Accounts | Provides social account records for Social Icons Renderer |
| `appearance/appearance.system.specification.v1.md` | Appearance | Provides Appearance State feeding Block Styling |
| `seo/seo.architecture.specification.v1.md` | SEO (Cross-Cutting) | Defines metadata that Rendering emits into HTML |
| `integrations/integrations.architecture.specification.v1.md` | Integrations (Cross-Cutting) | Defines provider eligibility; Rendering executes loading |

---

## Canonical Statement

Rendering is a consumer-only domain.

It transforms canonical data into presentation output without modifying, storing, or owning any part of that data.

Its architecture is stateless, deterministic, read-only, and implementation-independent.

It is the single domain responsible for converting profile content and appearance configuration into a publicly accessible profile experience — and nothing more.
