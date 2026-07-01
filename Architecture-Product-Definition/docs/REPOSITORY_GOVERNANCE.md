# Minime V1 — Repository Governance

## Status

Approved. Canonical governance document.

## Authority

This document is an Architecture-tier governance document, introduced in Phase A.5 (Repository Governance Hardening). It sits alongside `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md` and `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` at the top of the Decision Hierarchy (see the Map's "Decision Hierarchy" section). It does not redesign Minime, does not expand V1 scope, and does not change any product or implementation behavior. Its sole purpose is to make the repository self-governing against future silent Architecture Drift.

This document defines three things:

1. The **Cross Reference Policy** (G-03) — how documents must refer to shared concepts instead of duplicating them.
2. The **Repository Health Policy** (G-04) — the specific unhealthy states the repository must never enter, each with Purpose, Detection, and Required Resolution.
3. **Architecture Drift Prevention** (G-05) — the mandatory validation gate every future Pull Request touching Architecture or Implementation must pass before it can be considered architecture-complete.

The automated, rule-by-rule detection logic implementing the checks referenced throughout this document lives in `ARCHITECTURE_LINTER_RULES.md`. This document defines *what* must be true of the repository; `ARCHITECTURE_LINTER_RULES.md` defines *how* that truth is checked.

---

# 1. Cross Reference Policy (G-03)

## Core Rule

**Every architectural concept must have exactly one authoritative definition.** All other documents that touch that concept must reference the authoritative definition by name and file path — they must never restate, paraphrase, or duplicate the rule itself.

## Why This Exists

Every contradiction resolved in Phase A (Theme Customization, Session Model, SEO Indexing, Cache Invalidation, Account Deletion Cascade Order) had the same root cause: the same rule was written down twice, in two documents, by two different points in the project's history, and the two copies drifted apart because updating one did not obligate updating the other. A reference has no copy to drift — it either points at a document that is current, or it points at nothing (a broken reference, which is a distinct and separately-detectable failure; see `ARCHITECTURE_LINTER_RULES.md` — "Broken References").

## Rules

1. **One definition, many references.** If a rule already has an authoritative home (see the Canonical Ownership table in `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`), every other document that needs to mention that rule must do so by reference: `"see <path> — <section>"`. It must not restate the rule's substance (the specific values, the specific state machine, the specific list of supported/unsupported items) in its own words.
2. **Reference the narrowest authoritative section, not just the file.** A reference should name the section (`"Session Policy"`, `"Cascade Order"`, `"V1 Scope Notice"`) so the reader lands on the exact rule, not a whole document they must re-read to find it.
3. **Reference instead of repetition — including examples.** Illustrative examples (e.g. a worked cascade-order example, a worked cache-TTL calculation) may be repeated for readability only if they are drawn from, and clearly attributed to, the authoritative source. An example must never introduce a value, a step, or a state not present in the authoritative source, since a "helpful" divergent example is exactly how Phase A's contradictions were originally introduced.
4. **Repeated ownership is forbidden.** If two documents both currently state a rule as if each owns it, this is a Repository Health violation (see Rule H-1 below) and must be resolved by picking one Owner (per the Canonical Ownership table) and converting the other into a reference.
5. **New concepts create their own authoritative home immediately.** When a document introduces a genuinely new architectural concept not covered elsewhere, that document becomes the Owner by construction, and must be added to the Canonical Ownership table in the same change — see `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md` — "Canonical Ownership."
6. **Implementation may reference Architecture; Architecture may reference Implementation only as an Operational Specification pointer, never as a source of a rule.** An Architecture document may say "the exact numeric value is configured per `implementation/07-validation-rules.md`" (pointing at an Operational Specification for a value Architecture has already declared configurable) but must never say "see `implementation/X` for what this rule is" — that would make Implementation the Owner, which Rule G-01 in `PHASE_A5_GOVERNANCE_HARDENING_REPORT.md` and the Canonical Ownership table both forbid.

## What Is Exempt

Restating a rule in different words is not a violation when the restatement is a **consequence**, not a **redefinition** — e.g. a rendering document may say "because the architecture has no publishing workflow, rendering always reflects current persisted state" without that being a duplicate of the publishing-workflow rule itself, provided it cites the source (`implementation/README.md` — "No Publishing Workflow") rather than silently assuming it.

---

# 2. Repository Health Policy (G-04)

The repository must never allow any of the following eight unhealthy states. Each is defined with a Purpose (why it matters), Detection (how it is recognized), and Required Resolution (what must be done when it is found).

## H-1 — Duplicated Ownership

**Purpose:** Ownership must be singular so that "who is right when two documents disagree" always has one answer. Duplicated ownership is the precondition for every other health violation below — nearly all of them become possible only after two documents both believe they own the same rule.

**Detection:** Two or more documents each state a rule about the same concept as though each is the authoritative source (neither uses "see X" phrasing pointing at the other, or at a third document). Cross-reference the concept against the Canonical Ownership table in `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`; if the concept is listed, exactly one of the documents in conflict should match the listed Owner.

**Required Resolution:** Designate the correct Owner (adding it to the Canonical Ownership table if not already present), convert every other document's copy into a reference to the Owner, and file an APD entry if the correction changes any previously-stated V1 behavior.

## H-2 — Duplicated Business Rules

**Purpose:** A business rule (a validation constraint, a lifecycle transition, a numeric limit) copied into two places will eventually be edited in only one of them.

**Detection:** The same concrete rule (a specific enum, a specific numeric threshold, a specific list of allowed/forbidden values) appears verbatim or near-verbatim in two documents without one referencing the other as source.

**Required Resolution:** Same as H-1 — pick the Owner per the Cross Reference Policy, convert duplicates to references.

## H-3 — Conflicting Terminology

**Purpose:** The same word describing two different things (or two different words describing the same thing) forces every reader to privately reconcile terminology instead of the repository doing it once.

**Detection:** A term (e.g. "Profile," "Session," "Published") is used with a different meaning in two documents, or two different terms are used for what is actually the same canonical entity or concept.

**Required Resolution:** Establish the single canonical term in the Owner document for that concept; update all other documents to use it; where a document must reference an obsolete term for historical clarity, it must explicitly say so ("previously called X").

## H-4 — Conflicting Lifecycle States

**Purpose:** Two documents describing different state machines for the same entity (e.g. one document's Account states vs. another's) makes "what state can this entity actually be in" unanswerable without arbitration.

**Detection:** An entity's lifecycle (its named states and transitions) is described inconsistently across documents — different state names, a different transition graph, or states present in one document and absent from another with no cross-reference.

**Required Resolution:** The entity's Owner document (per Canonical Ownership, or `platform/data/canonical.entities.map.v1.md` if unlisted) states the lifecycle once; all other documents reference it.

## H-5 — V2 Behavior Described As V1

**Purpose:** This was the single largest class of confusion found in the Phase A audit (Theme Customization). A capability that does not exist yet must never read as though it already does.

**Detection:** A document describes a capability in present tense, as an available action, or under a "V1" heading, when the capability is not present anywhere in `implementation/` and is not reachable through any approved V1 API.

**Required Resolution:** Tag the capability explicitly as **V2 Scope** at every point it is mentioned (per the Document Version Status Policy in `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`), and, if the surrounding document is otherwise about a real V1 capability, add a V1 Scope Notice at the top disambiguating which parts of the document are V1 and which are V2, exactly as done for the six Appearance documents in Phase A (APD-011).

## H-6 — Implementation-Only Architecture

**Purpose:** An architectural concept must never exist only in `implementation/` with no Architecture-tier statement of the rule it operationalizes — this makes Implementation the de facto (if unofficial) canonical owner, which the Decision Hierarchy forbids.

**Detection:** A rule with clear product/business meaning (not a pure mechanism detail like a specific hash function or queue name) exists in `implementation/` with no corresponding Architecture document stating it, or with only an APD entry as its sole architectural grounding rather than a substantive Architecture document.

**Required Resolution:** Write the rule into the appropriate Architecture document (creating one if none exists for that domain), file the APD entry recording the addition per the Post-Freeze Clarification Policy, and update the Canonical Ownership table.

## H-7 — Obsolete Supported Behavior

**Purpose:** A document that still describes a capability as active/supported after that capability has actually been removed or superseded (elsewhere in the repository, or in fact) misleads every reader who has not independently verified against every other document.

**Detection:** A document states a capability is supported, but a Repository Health or Linter check (or a human reviewer) finds another Architecture or Implementation document explicitly stating it is not supported, removed, or was never implemented.

**Required Resolution:** Resolve at the source per the Governing Principle inherited from Phase A ("every contradiction must be resolved at its source; never solve contradictions by adding explanatory notes somewhere else") — update the obsolete document directly, and apply the Document Version Status Policy (mark the section, or the whole document, Deprecated/Superseded if the obsolete content cannot simply be corrected in place).

## H-8 — Two Documents Defining The Same Canonical Decision

**Purpose:** Even outside ordinary business rules, two independent "canonical" declarations of the same decision (two documents each claiming Frozen/Approved status over the same concept) is a governance failure distinct from H-1: it means the repository's own status metadata is self-contradictory, not just its content.

**Detection:** Two documents both carry a `Status: Frozen` or `Status: Approved` header while defining the same concept differently, or the Canonical Ownership table lists a concept whose Owner document does not match what a search of the repository finds actually defining it.

**Required Resolution:** Immediate reconciliation against the Canonical Ownership table; the table itself is amended if it was the source of the ambiguity (as done for the Session Model and Cache Strategy rows in Phase A.5 — see Section G-01 of `PHASE_A5_GOVERNANCE_HARDENING_REPORT.md`).

---

# 3. Architecture Drift Prevention (G-05)

## Purpose

Phase A and Phase A.5 eliminated the Architecture Drift that had already accumulated. This section exists to make future drift structurally difficult to introduce, rather than something that can only be caught by the next full manual audit. It defines the mandatory validation gate for every future change.

## The Rule

**Every future Pull Request that affects any document under `Architecture-Product-Definition/`, this Map, `ARCHITECTURE_PR_APPROVAL_DECISIONS.md`, or `implementation/` must pass all six validations below before it may be considered architecture-complete.** A PR that fails any validation is incomplete, regardless of whether its code or its immediate document change is otherwise correct.

### 1. Ownership Validation

Confirm every concept touched by the PR has, after the PR, exactly one Owner document, and that the Owner is an Architecture document (never `implementation/`) per the Canonical Ownership table. If the PR touches a concept not yet in the table, the PR must add it.

### 2. Consistency Validation

Confirm the PR does not leave any document asserting a capability, state, or rule that contradicts another document's assertion about the same concept (see Repository Health Policy H-1 through H-8 above). This includes re-reading every document listed as a "Dependent Document" or "Affected Canonical Document" in any APD the PR touches or introduces.

### 3. Cross Reference Validation

Confirm every new or modified reference to another document resolves to a real file and a real section (no broken references — see `ARCHITECTURE_LINTER_RULES.md` — "Broken References"), and confirm the PR did not introduce a new duplicate of a rule that already has an Owner (per the Cross Reference Policy above).

### 4. APD Validation

Confirm that if the PR introduces an entity, background job, deployable, repository contract, transaction coordinator, advisory lock, unique index, security boundary, or any correction of a prior Architecture ↔ Implementation contradiction, a corresponding APD entry exists in `ARCHITECTURE_PR_APPROVAL_DECISIONS.md`, using the enhanced template (see Section G-06 of `PHASE_A5_GOVERNANCE_HARDENING_REPORT.md` and the Registry Format in `ARCHITECTURE_PR_APPROVAL_DECISIONS.md`). A PR that changes behavior without a corresponding APD entry fails this validation regardless of code quality.

### 5. Scope Validation

Confirm the PR does not expand V1 product scope, introduce a new Product Domain, or redesign an existing one, unless it is explicitly a Product Vision-level change accompanied by an update to `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`'s Product Scope section — not merely an APD entry, since APD entries are implementation-enabling clarifications only and are explicitly forbidden from expanding scope (per `implementation/README.md` — "Post-Freeze Clarification Policy").

### 6. Version Validation

Confirm every document touched by the PR carries a valid, single, unambiguous status per the Document Version Status Policy (Draft/Approved/Frozen/Deprecated/Superseded/Archived), and that no capability is left describable as both V1 and V2 Scope in the same document without one of those being explicitly marked as the error being corrected.

## Consequence of Failure

**If any of the six validations fails, the PR must not be considered architecture-complete.** It may still be merged as a code-only or documentation-only partial change at the team's discretion, but it must be explicitly flagged as leaving unresolved Architecture Drift, and a follow-up task must be opened to complete the remaining validation(s) before the next Architecture freeze checkpoint. Silent partial merges — merging a PR that fails one of these validations without flagging it — are exactly the failure mode this section exists to prevent.

---

# 4. Relationship To Other Governance Documents

```text
MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md
   Decision Hierarchy, Canonical Ownership, Document Version Status Policy
        │
        ▼
REPOSITORY_GOVERNANCE.md  (this document)
   Cross Reference Policy, Repository Health Policy, Architecture Drift Prevention
        │
        ▼
ARCHITECTURE_PR_APPROVAL_DECISIONS.md
   The enforcement record — every APD entry is evidence that the workflow in this
   document's Section 3 (Architecture Drift Prevention) was actually followed
        │
        ▼
ARCHITECTURE_LINTER_RULES.md
   The automated, implementation-agnostic detection logic for every Repository
   Health rule and Drift Prevention validation defined above
```

This document does not repeat content already stated in the Map or the APD registry; where a rule is already established there, this document references it rather than restating it — consistent with the Cross Reference Policy it defines.
