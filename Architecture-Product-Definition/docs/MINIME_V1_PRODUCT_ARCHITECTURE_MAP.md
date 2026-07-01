# MINIME V1 PRODUCT ARCHITECTURE MAP

---

# Product at a Glance

Product Name

Minime

Current Version

V1

Product Category

Personal Branding Operating System

Primary Purpose

Enable anyone to build and manage a professional personal brand from a single account.

Target Users

- Creators
- Professionals
- Entrepreneurs
- Freelancers
- Students
- Job Seekers

Primary Output

A publicly accessible personal profile.

Core User Flow

Create Account
→ Verify Identity
→ Choose Username
→ Add Social Accounts
→ Connect Accounts
→ Create Profile
→ Arrange Blocks
→ Share
→ Track Analytics

Architecture Style

Account-Centric

Aggregate Root

Account

Primary Identifier

`account_id`

Rendering Model

Stateless

Social Accounts Setup

- Smart Mode
- Manual Mode

Profile Ownership

Every resource belongs to exactly one account.

---

# Purpose

This document is a practical architectural overview of Minime V1.

It describes the product structure: the complete product scope, the major domains, the end-to-end product journey, the architectural boundaries, and the relationships between the core components.

It exists to help engineers, reviewers, and AI understand how the product is organized. Specifications across the repository are expected to stay consistent with this overview.

This document intentionally remains implementation-independent. It defines what Minime is, how it is organized, and how every domain contributes to the overall product architecture.

**Architectural View:** This document is the conceptual overview of the product. It describes domain ownership and domain dependency order. It does not describe runtime execution sequences, persistence structures, or technical infrastructure. Persistence structures and runtime flows are documented separately.

When a new feature, domain, folder, or specification is introduced, it should have a clear place within this overview before detailed specifications are created.

---

# Product Vision

Minime is a Personal Branding Operating System that enables individuals to build, manage, and continuously improve their online presence from a single account.

Rather than simply aggregating links, Minime enables users to build a structured, professional, and evolving public identity — combining social account setup, profile creation, visual profile building, sharing, analytics, and AI assistance into one unified experience.

V1 focuses on delivering the simplest possible experience for building and managing a personal brand while establishing a scalable architectural foundation for expanded AI capabilities in future versions.

---

# Design Philosophy

The following principles shape every architectural and product decision within Minime V1. Each principle is directly reflected in the architecture and the specifications throughout this repository.

- **User-provided content is the Source of Truth.** The only source of truth is the content the user intentionally provides or confirms. No system — Social Accounts, Connected Accounts, OAuth, AI, platform import, synchronization, or any external platform — ever becomes the source of truth. Every other system may normalize, format, enrich, synchronize, verify, or suggest, but never replaces user intent automatically.

- **Account-centric.** Every resource belongs to exactly one account. The account is the single aggregate root throughout the entire product lifecycle.

- **Single responsibility.** Every domain owns one clearly defined responsibility. No domain absorbs the concerns of another.

- **Stateless rendering.** The public profile is assembled from current profile content at the point of access. The rendering layer never stores or owns data.

- **Observational analytics.** Analytics collects and reports on activity without influencing product state.

- **User-provided social accounts.** Social Accounts collects user-provided account identifiers, normalizes them, and generates canonical public profile URLs. It never searches platforms, verifies account existence, or contacts external services.

- **Near-live public updates.** Persisted profile changes propagate to the visitor-facing profile as near-live updates, with a maximum cache delay of 60 seconds. There are no draft, unpublished, scheduled, preview, or separate live profile states.

---

# Product Boundaries

The following boundaries define the scope of the product — what Minime is and what it is not.

## Minime Is

Minime is a Personal Branding Operating System.

Minime enables individuals to build, manage, and continuously improve their professional online presence from a single account.

Minime combines social account setup, profile creation, visual profile building, sharing, analytics, and AI assistance into one unified experience.

Minime is built around an account-centric architecture where every resource belongs to exactly one account.

Minime is designed to simplify personal branding while remaining scalable for expanded AI capabilities in future versions.

---

## Minime Is Not

Minime is not a social network.
Minime is not a blogging platform.
Minime is not a general-purpose website builder.
Minime is not a content management system (CMS).
Minime is not an e-commerce platform.
Minime is not a team collaboration platform.
Minime is not a project management tool.
Minime does not replace existing social platforms.
Instead, it connects and organizes a user's public presence into a single destination.

---

# Product Goals

The primary objective of Minime V1 is to make personal branding accessible, simple, and fast without sacrificing flexibility or scalability.

V1 establishes the architectural foundation of the product while delivering a complete end-to-end experience for building and managing a personal brand.

## Primary Goals

- Enable anyone to create a professional public profile within minutes.
- Provide a single destination for an individual's online presence.
- Simplify social account setup and profile creation.
- Make profile customization easy through reusable content blocks.
- Deliver a fast, mobile-first profile experience.
- Provide practical analytics for measuring profile performance.
- Introduce AI assistance that reduces manual work without increasing complexity.
- Establish a scalable architecture that future versions can extend without redesigning the core system.

---

# Product Scope

The following sections define the official functional scope of Minime V1.

Everything included below is considered part of the V1 product architecture.

---

## Product Domains

- Account
- Authentication
- Username
- Social Accounts
- Connected Accounts
- Profile
- Rendering
- Out Links
- Analytics

> **Rendering** produces the visitor experience and serves it as the public rendering surface (the public profile) through public URLs. Public Profile is not a separate domain; it is the public rendering surface produced by Rendering.
>
> **Account** is a single domain that owns account identity, account management, settings, and the QR code entry point. Settings, QR Code, and Account Management are not standalone domains.
>
> **Profile** is a single domain that owns the profile as one business concept — its profile fields, its blocks, and its design (appearance). Profile fields, blocks, and design are implementation concerns within Profile, not separate business ownership boundaries.
>
> Theme Library and Background Configuration are implementation concerns within Profile design, not standalone domains.

---

## Cross-Cutting Product Capabilities

Cross-Cutting Product Capabilities span all Product Domains without being owned by any single domain.

They are not Product Domains.

They are not Platform Services.

They may read from multiple domains and produce output consumed by Rendering.

They own no canonical entity.

| Capability   | Folder          | Primary Consumer |
| ------------ | --------------- | ---------------- |
| SEO          | `/seo`          | Rendering        |
| Integrations | `/integrations` | Rendering        |

### SEO

SEO generates metadata definitions for all public profiles.

SEO reads from Profile and Account.

SEO produces metadata definitions (HTML title, canonical URL, robots directives, Open Graph, Twitter Card, Structured Data) that Rendering emits into the final HTML output.

SEO owns no canonical entity. SEO never writes to any domain. SEO never renders HTML directly.

Classification: Cross-Cutting Product Capability.

### Integrations

Integrations defines which external providers may interact with public pages and governs their lifecycle.

Integrations owns: provider registration, validation, configuration, loading policies, and security rules.

Rendering loads approved providers according to Integrations eligibility rules.

Integrations is optional — Minime operates correctly without any active provider.

Integrations owns no canonical entity. Integrations never writes to any Product Domain.

Classification: Cross-Cutting Product Capability.

---

## AI Assistance (V1)

AI is intentionally introduced in V1 with a focused and practical scope.

Included capability:

- "Analyze My Profile" — a single, on-demand Analysis Session triggered only by explicit user request. There is no AI chat and no background AI.

The following are explicitly outside V1 scope:

- Username suggestions
- Social account setup assistance
- AI-assisted onboarding

AI assists users.
AI never replaces user decisions.

AI is optional. The platform operates fully with AI completely disabled. AI is included in V1 because it provides immediate, practical user value within an intentionally narrow scope — consistent with the principle that Minime grows by necessity, never by speculation.

---

## Future Capabilities

The following capabilities are intentionally outside the scope of V1.

- Advanced Personal Branding AI
- AI Content Generation
- AI Profile Optimization
- AI Performance Insights
- Collaboration
- Marketplace
- Public API
- Third-Party Integrations
- Advanced Automation


# Product Journey

The following journey describes the complete lifecycle of a Minime profile in V1.

Every core domain within the repository participates in one or more stages of this journey.

Visitor
    │
    ▼
Create Account
    │
    ▼
Verify Identity
    │
    ▼
Choose Username
    │
    ▼
Add Social Accounts
    │
    ▼
Save Connected Accounts
    │
    ▼
Create Profile Content
    │
    ▼
Arrange Blocks
    │
    ▼
Customize Appearance
    │
    ▼
Generate Public Profile
    │
    ▼
Access Public Profile
    │
    ▼
Share Public Link
    │
    ▼
Visitors Interact
    │
    ▼
Track Analytics
    │
    ▼
Update Profile

Each stage of this journey is represented by one or more architecture domains within the repository.

---

# Domain Coverage

The following table maps every stage of the Product Journey to the architecture domain responsible for that stage.

Every domain should have a clear responsibility within the overall product lifecycle.

| Product Journey Stage      | Responsible Domain |
| -------------------------- | ------------------ |
| Create Account             | Account            |
| Verify Identity               | Authentication     |
| Choose Username            | Username           |
| Add Social Accounts        | Social Accounts    |
| Save Connected Accounts    | Connected Accounts |
| Create Profile Content     | Profile            |
| Arrange Blocks             | Profile            |
| Customize Appearance       | Profile            |
| Generate Public Profile    | Rendering          |
| Access Public Profile      | Rendering          |
| Share Public Link          | Rendering          |
| Route Outbound Links       | Out Links          |
| Track Analytics            | Analytics          |

Every architecture domain should own at least one stage of the Product Journey.

No domain should exist without a clearly defined responsibility within the product lifecycle.

---

# Product Domains

The following domains define the complete functional architecture of Minime V1.

Each domain has a single, well-defined responsibility within the product.

Together, these domains describe the complete architecture of Minime.

| Domain             | Responsibility                                                         |
| ------------------ | ---------------------------------------------------------------------- |
| Account            | Owns and manages every resource within the system, including account management, account preferences and settings, and the QR code entry point to the public profile. |
| Authentication     | Verifies account identity and controls access.                         |
| Username           | Manages each account's unique public identity.                         |
| Social Accounts    | Collects user-provided social account identifiers, normalizes input, applies platform rules, and generates canonical public profile URLs (Smart Mode and Manual Mode). |
| Connected Accounts | Stores the social account records produced by Social Accounts Setup.   |
| Profile            | Owns the profile as one business concept — its profile fields, its blocks, and its design (appearance) — and all content displayed on the public profile. |
| Rendering          | Generates the public profile from current profile content and serves it as the public rendering surface through public URLs. |
| Out Links          | Routes outbound traffic and tracks link clicks.                        |
| Analytics          | Collects and reports profile and link performance metrics.             |

Every specification within the repository belongs to one of these domains.

New domains should only be introduced when their responsibility cannot reasonably be incorporated into an existing domain while preserving clear architectural boundaries.

---

# Domain Relationships

> [Conceptual Architecture View — Domain Dependency Flow]
>
> This flow shows domain dependency order, not runtime execution sequence.

The following flow illustrates how the core domains interact throughout the lifecycle of a Minime profile.

Each domain is responsible for one stage of the architecture and hands responsibility to the next.

Authentication
        │
        ▼
Account
        │
        ▼
Username
        │
        ▼
Social Accounts
        │
        ▼
Connected Accounts
        │
        ▼
Profile
        │
        ▼
Rendering
        │
        ▼
Out Links
        │
        ▼
Analytics

Profile owns the profile fields, blocks, and design (appearance) as one business concept; these are stages within the Profile domain, not separate domains.

Rendering produces the public profile and serves it as the public rendering surface. The public profile is not a separate domain stage; it is the surface produced by Rendering.

## Account Domain Scope

Account preferences and settings, account management, and the QR code entry point to the public profile are all owned by the Account domain. They are not separate domains.

```text
Account
 ├─ account management
 ├─ settings (account preferences and configuration)
 └─ QR code (scannable entry point to the public profile)
```

## Relationship Principles

* Every domain has a single responsibility.
* Domains communicate through clearly defined architectural boundaries.
* A domain may depend on previous stages but must never bypass the architecture.
* Rendering never modifies content.
* Rendering depends on current profile content.
* Analytics is observational only.
* Analytics never modifies content.
* Social Accounts collects and normalizes user-provided identifiers only.
* Social Accounts never searches platforms or verifies account existence.
* Profile design (appearance) affects presentation only.
* Account remains the single aggregate root throughout the entire product lifecycle.


# Repository Structure

The Product Map defines the architecture.

The repository structure is the physical implementation of that architecture.

Each domain is implemented as an independent specification folder containing all documents related to that responsibility.

| Domain             | Repository Folder(s)                                            | Primary Specification                              |
| ------------------ | -------------------------------------------------------------- | -------------------------------------------------- |
| Account            | `/account`, `/account-management`, `/settings`, `/qr-code`     | `account.model.specification.v1.md`                |
| Authentication     | `/account`                                                     | `authentication.policy.v1.md`                      |
| Username           | `/account`                                                     | `username.policy.v1.md`                            |
| Social Accounts    | `/social-accounts`                                             | `social.accounts.setup.specification.v1.md`        |
| Connected Accounts | `/account-management`                                          | `connected.accounts.specification.v1.md`           |
| Profile            | `/profile-content`, `/blocks`, `/appearance`, `/block-styling` | `profile.content.specification.v1.md`              |
| Rendering          | `/rendering`, `/public-profile`                                | `rendering.architecture.canon.v1.md`               |
| Out Links          | `/out-links`                                                   | `out-link.system.specification.v1.md`              |
| Analytics          | `/analytics`                                                   | `analytics.system.specification.v1.md`             |

The Account domain spans the `/account`, `/account-management`, `/settings`, and `/qr-code` folders. The Profile domain spans the `/profile-content`, `/blocks`, `/appearance`, and `/block-styling` folders. The Rendering domain spans the `/rendering` and `/public-profile` folders, where `/public-profile` is the public rendering surface. These multiple folders are implementation concerns within a single business domain; they are not separate domains.

## Repository Principles

* Every specification belongs to exactly one domain.
* Each domain has one primary specification.
* Supporting specifications extend the primary specification of their domain.
* A domain may span more than one folder when those folders are implementation concerns of the same business domain.
* Cross-domain references should remain minimal and explicitly documented.
* Repository organization should reflect this architectural overview.

# Architecture Canon

The following architectural principles define the immutable foundation of Minime V1.

Every specification, policy, rule, and future architecture decision must comply with these principles.

---

## Source of Truth

* User-provided content is the single Source of Truth across the entire Minime architecture.
* The Source of Truth is the content the user intentionally provided or confirmed.
* Connected Accounts, OAuth, AI, platform import, synchronization, and external platforms never become the Source of Truth.
* Storage domains (such as Connected Accounts) persist the canonical representation of user-provided data; they store user intent but never originate or replace it.
* Every non-user system may normalize, format, enrich, synchronize, verify, or suggest — but never replaces user intent automatically.

---

## User Ownership Canon

The user owns the content.

The system standardizes the content.

The system never replaces user intent automatically.

* AI may suggest.
* OAuth may verify.
* Import may enrich.
* Sync may update only when explicitly allowed.

None of them may silently replace user intent.

---

## Identity

* Account is the single aggregate root.
* Profile is not a persisted entity.
* User is not a persisted entity.
* Every owned resource references `account_id`.

---

## Ownership

* Every resource belongs to exactly one account.
* Ownership is immutable unless explicitly transferred by the system.
* `account_id` is the canonical ownership identifier across the entire repository.

---

## Social Accounts

Social Accounts replaces the retired Discovery Engine concept. Minime V1 does not discover, search, or verify social accounts.

The Social Accounts domain is responsible for:

* **Smart Mode** — automating URL construction from a single user-provided username or social URL.
* **Manual Mode** — platform-specific manual entry with normalization and URL generation.
* **Input Normalization** — trimming, extracting identifiers from supported URLs, and applying platform formatting rules.
* **Platform Rules** — per-platform formatting definitions (identifier type, accepted input, display prefix, canonical URL template, normalization rules).
* **URL Generation** — producing canonical public profile URLs from normalized identifiers.
* **Social Account Storage** — persisting normalized social account records.
* **OAuth Boundary** — OAuth is an optional future Account / Connected Accounts capability only; it is never part of onboarding or Social Accounts Setup, and never required to create a social account.

Social Accounts never performs account discovery, platform search, account existence checks, confidence scoring, scraping, browser automation, platform API requests, or search engine queries. Removing internet access must not change its behavior.

---

## Rendering

* Rendering is completely stateless.
* Rendering never stores data.
* Rendering never owns data.
* Rendering never modifies content.
* Rendering consumes stored canonical values as saved; it never regenerates URLs, normalizes identifiers, applies platform rules, or rebuilds links. Rendering is presentation only.
* Rendering always represents the latest persisted profile state.

---

## No Publishing Workflow

Minime V1 has no publishing workflow.

A profile is not "published" in V1.
A profile exists once an account is activated and a valid username is bound to it.

Persisted profile changes propagate to the visitor-facing profile as near-live updates, with a maximum cache delay of 60 seconds.

There are no draft, unpublished, scheduled, preview, or separate live profile states in V1.

---

## Analytics

* Analytics is observational only.
* Analytics never modifies user content.
* Analytics never changes user data.

---

## Architecture

* Every domain has a single responsibility.
* Every specification belongs to exactly one domain.
* Every domain owns one primary specification.
* Cross-domain coupling should remain minimal.
* The Product Map is the practical architectural overview of the product.

---

## Decision Hierarchy

This section defines the canonical precedence order for every decision made about Minime. When two documents at different layers appear to disagree, the higher layer wins, and the lower layer must be corrected to match it — never the reverse.

```text
Product Vision
    ↓
Architecture  (Architecture-Product-Definition/, including this Map and ARCHITECTURE_PR_APPROVAL_DECISIONS.md)
    ↓
Approved APD Decisions  (ARCHITECTURE_PR_APPROVAL_DECISIONS.md — implementation-enabling clarifications only)
    ↓
Implementation Specifications  (implementation/)
    ↓
Code
    ↓
Deployment Configuration
```

Rules that follow from this hierarchy:

- **Product Vision** justifies the existence of a domain or capability. It never specifies mechanism.
- **Architecture** (this Map, and every document under `Architecture-Product-Definition/`) is the single source of truth for what the system is, its domain boundaries, and its permanent invariants. Architecture is authoritative over Implementation.
- **Approved APD Decisions** sit between Architecture and Implementation. An APD entry may adopt an Implementation decision into Architecture (promoting it upward) or correct Architecture to match Implementation when Implementation is demonstrably the better engineering decision — but every such adoption must be recorded in `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` before or alongside the change, per `implementation/README.md` — "Post-Freeze Clarification Policy." An unrecorded divergence is not a valid APD-level decision, regardless of where else it appears.
- **Implementation Specifications** (`implementation/`) operationalize Architecture. They must never silently redefine Architecture. If an Implementation document contains a better engineering decision than the frozen Architecture, the correct action is to promote that decision into Architecture (via a new APD entry and, where the contradiction is severe, a direct edit to the relevant Architecture document) — never to leave Architecture and Implementation silently disagreeing, and never to weaken the better Implementation decision merely to preserve an obsolete Architecture document.
- **Code** must match Implementation Specifications exactly. Any code-level deviation from an Implementation Specification is a bug, not a new source of truth.
- **Deployment Configuration** (environment variables, infrastructure, CI/CD) supplies runtime values only where an owning specification has explicitly declared a value configurable (see `platform.architecture.specification.v1.md` — "Configuration Authority"). It never creates business rules.

Every engineer, at every layer, must be able to answer: "which layer wins here?" — and the answer is always: the higher one in this list.

### Conflict Resolution Rule (Phase A.5)

**No lower architectural layer may silently redefine any higher layer.**

This is absolute. Implementation may never become the canonical owner of an architectural concept merely by describing it first, describing it more precisely, or describing it more currently than Architecture does. If Implementation and Architecture disagree, that disagreement is always a defect to be resolved — never a tolerated dual-authority state, and never resolved by treating Implementation as authoritative in place of Architecture.

### Required Workflow When Implementation Discovers a Better Engineering Solution

There is exactly one valid path from "Implementation knows something Architecture doesn't yet say" to "the repository is consistent again." No alternative workflow may exist.

```text
Implementation discovers a better engineering solution
        ↓
Create an APD entry in ARCHITECTURE_PR_APPROVAL_DECISIONS.md
   (states the reason, scope, and why it is not a redesign)
        ↓
Update the Architecture document(s) that own the affected concept
   (per the Canonical Ownership table below — the Architecture document
   is corrected in the same change as the APD entry, never left to drift)
        ↓
Implementation remains synchronized
   (implementation/ already matches, or is confirmed unchanged, since the
   direction of correction was Architecture adopting Implementation's decision)
        ↓
Code follows the updated Architecture
   (via the already-synchronized Implementation Specification — no direct
   code change is triggered by this workflow alone unless Implementation
   itself required a correction, which is a separate, explicit step)
```

Skipping any step in this workflow — in particular, changing Implementation behavior without an APD entry, or leaving an APD entry unaccompanied by the corresponding Architecture update — reintroduces exactly the class of silent Architecture Drift that Phase A was created to eliminate. See `ARCHITECTURE_LINTER_RULES.md` — "Missing APD" and "Implementation Drift" for the automated checks that detect a skipped step.

---

## Canonical Ownership

Every architectural concept has exactly one owner document, **and that owner document is always an Architecture document** (a file under `Architecture-Product-Definition/`, this Map, or `ARCHITECTURE_PR_APPROVAL_DECISIONS.md`). An Implementation document (`implementation/`) may never be listed as the Owner of an architectural concept — it may only be listed as an **Operational Specification**, i.e. the document that carries the concrete mechanism, field-level detail, exact numeric values, or API contract that operationalizes what the Owner document already establishes as a rule. This distinction is load-bearing: it is what "Implementation must never silently redefine Architecture" means in practice, applied table-row by table-row.

**Correction record (Phase A.5 — G-01):** Two rows in this table, as originally introduced in Phase A, listed an `implementation/` document as the "Canonical Owner Document" (Session lifecycle/token model, and Public profile cache strategy). This was itself a violation of the governing principle the table exists to enforce, discovered during the Phase A.5 governance-hardening audit. Both rows are corrected below: the Architecture document that already states the governing rule (rewritten during Phase A to carry that rule) is now the listed Owner, and the `implementation/` document is demoted to Operational Specification. No behavior changed — only which document is recorded as authoritative.

All other documents that mention a concept in this table must reference its Owner document rather than redefining the rule. This table exists specifically to prevent the class of contradiction found during the Phase A architecture-synchronization audit, where the same concept (a session model, a cache invalidation strategy, an indexing eligibility rule, a deletion cascade order, a customization capability) was described differently in two places because no single document was designated as authoritative.

| Concept | Owner (Architecture) | Operational Specification (Implementation) | Notes |
|---|---|---|---|
| Session lifecycle, token model, session management | `account-management/minime.account.management.system.specification.v1.md` ("Session Policy") | `implementation/08-security-model.md` ("Session Security," "Token Rules") | The Owner document states the rule (token types, TTLs, session management capabilities). The Operational Specification carries the exact cryptographic/algorithmic detail (JWT signing, hash storage). Neither may be updated without the other being checked for consistency (see APD-012). |
| Theme selection vs. Theme customization scope | `appearance/themes/theme.customization.specification.v1.md` ("V1 Scope Notice") | `implementation/03-canonical-data-model.md` (`ProfileContent.appearance_config.customizations`) | The Owner document is the single place that states what is V1 vs. V2 Scope for customization. Every other Appearance document defers to it (see APD-011). |
| Block-level style overrides | `block-styling/block-style.model.specification.v1.md` (closed property catalog) | `implementation/07-validation-rules.md` (`style_overrides` validation) | Distinct from theme-wide customization above; never conflate the two. |
| Public profile cache strategy | `public-profile/public-profile.cache.policy.v1.md` (two-layer model, Smart Cache Invalidation) | `implementation/09-caching-strategy.md` ("Cache Invalidation," "Freshness Budget Propagation") | The Owner document states the strategy and the invalidation principle. The Operational Specification carries the exact Redis key names, TTL arithmetic, and the full mutating-operation trigger list (see APD-014). |
| SEO indexing eligibility | `seo/seo.indexing.policy.v1.md` | `implementation/12-seo-and-integrations.md` ("SEO Implementation Rules") | Keyed exclusively on `Account.status`; no other document may introduce a competing visibility concept (see APD-013). |
| Account deletion cascade order and rationale | `account/account.deletion.policy.v1.md` (Section 4) | `implementation/10-background-jobs.md` (Job 7) | The Owner document carries the idempotency rationale at its source. The Operational Specification must match this section's step order exactly (see APD-015). |
| Canonical entity map (what persists, who owns it) | `platform/data/canonical.entities.map.v1.md` | `implementation/03-canonical-data-model.md` (field-level realization) | The Operational Specification must never introduce an entity absent from the Owner document without an APD entry. |
| Post-freeze implementation-enabling decisions | `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` | — (this registry sits between Architecture and Implementation in the Decision Hierarchy; it has no separate Operational Specification) | The only valid registry for entity/job/deployable/repository-contract/transaction-coordinator/advisory-lock/unique-index/security-boundary additions. |
| Product domain boundaries and responsibilities | This document (`MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`) — "Product Domains," "Domain Coverage" | `implementation/02-repository-structure.md` (folder-to-domain mapping) | No other document may introduce, rename, merge, or split a domain independently. |

If a future document needs to describe one of these concepts, it must reference the Owner document by name rather than restating the rule. If a document must describe a concept not yet listed here, the Architecture document describing it becomes the de facto canonical Owner and should be added to this table — as Owner, never as Operational Specification — in the same change. An `implementation/` document must never be added to this table as an Owner under any circumstance; see `ARCHITECTURE_LINTER_RULES.md` — "Missing Canonical Owner" and "Implementation Drift" for the automated check.

---

## Document Version Status Policy

Every document in `Architecture-Product-Definition/` and `implementation/` must declare exactly one of the following statuses, and must never mix lifecycle states within itself (e.g. a document must never describe both current and removed behavior without explicitly labeling which is which):

| Status | Meaning |
|---|---|
| **Draft** | Under active discussion. Not yet authoritative. Must not be relied upon for implementation. |
| **Approved** | Reviewed and accepted as correct for the version it targets. Equivalent to **Frozen** for all V1 architecture documents already carrying a `Status: Approved` header as of this policy's introduction — no retroactive relabeling of existing "Approved" documents is required by this policy alone. |
| **Frozen** | Approved and explicitly locked against further change except through the Post-Freeze Clarification Policy and the APD registry. This is the intended steady state for `Architecture-Product-Definition/` documents. |
| **Deprecated** | Superseded in part; still contains some currently-accurate content but also describes behavior that no longer applies. Must state, at the point of deprecation, exactly which sections are obsolete and what replaces them. |
| **Superseded** | Entirely replaced by a named successor document. Retained only for historical traceability; must link to its successor and must not be used as an implementation reference. |
| **Archived** | Retired from the active specification set. Retained for historical record only. |

**Mixed lifecycle states are forbidden.** A document must never simultaneously describe a capability as both "Supported" and "Not Supported" for the same version without one of those statements being an error to correct (this was the root cause of the Theme Customization, Session Model, SEO Indexing, and Cache Invalidation contradictions resolved in Phase A — see `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` APD-011 through APD-015). Every deferred capability must be explicitly marked **V2 Scope** at the point it is mentioned — never left ambiguous as to which version it belongs to.

Documents updated during Phase A (see `PHASE_A_ARCHITECTURE_SYNCHRONIZATION_REPORT.md`) now carry explicit V1/V2 Scope notices at every point of prior ambiguity. Extending this explicit status-per-section labeling to every remaining document in the repository is a recommended follow-up (Phase B), not a requirement of this policy's introduction.

---

## Post-Freeze Approved Engineering Decisions

Since this architecture was frozen, a set of implementation-enabling engineering decisions has been approved to make this Product Map concrete enough to build against (e.g. specific entities, background jobs, deployables, and security boundaries required by `implementation/`). These decisions do not redesign this architecture and do not change anything stated above.

The canonical, authoritative registry of those decisions is:

```
ARCHITECTURE_PR_APPROVAL_DECISIONS.md
```

at the repository root. This Product Map does not duplicate that registry's content — refer to it directly for the full list of approved decisions, their scope, and their non-goals.
