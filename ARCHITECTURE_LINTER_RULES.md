# Minime — Architecture Linter Rules

## Status

Approved. Canonical governance document.

## Authority

This document defines automated, repository-wide validation rules for the Minime specification repository. It is introduced in Phase A.5 (Repository Governance Hardening) as the concrete detection layer for the Repository Health Policy and Architecture Drift Prevention gate defined in `Architecture-Product-Definition/docs/REPOSITORY_GOVERNANCE.md`.

**This document defines repository health only. It is implementation-agnostic.** It does not test, lint, or validate `implementation/` code, Prisma schemas, TypeScript, or any executable artifact. It validates the *documents* — Architecture and Implementation Specifications as text — against each other and against the governance rules established in `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md` and `REPOSITORY_GOVERNANCE.md`. Whether these rules are eventually enforced by a script, a CI job, or a manual reviewer checklist is an execution detail outside this document's scope; this document defines only *what* must be checked and *why*.

Every rule below follows the same structure: **Purpose**, **Detection Logic**, **Failure Condition**, **Expected Resolution**.

---

## R-01 — Duplicate Ownership

**Purpose:** Ownership of an architectural concept must be singular (see `REPOSITORY_GOVERNANCE.md` — Repository Health Policy, H-1). Duplicate ownership is the root cause of every contradiction resolved in Phase A.

**Detection Logic:** For each concept listed in the Canonical Ownership table (`MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`), scan all documents for statements defining that concept's rule (not merely mentioning it). Flag any document, other than the table's listed Owner, that defines the concept's substance rather than referencing the Owner by name and path.

**Failure Condition:** Two or more documents each state the same rule's substance (not just a cross-reference) without one of them being the designated Owner.

**Expected Resolution:** Convert every non-Owner occurrence into a reference (`"see <path> — <section>"`); if the concept is not yet in the Canonical Ownership table, add it, designating exactly one document as Owner.

---

## R-02 — Duplicate Canonical Definitions

**Purpose:** A "canonical" declaration (an entity shape, an enum, a lifecycle) must exist in exactly one authoritative form. Two independently-maintained copies of the same definition, even if currently identical, will diverge the next time only one copy is edited.

**Detection Logic:** Search for structurally similar field lists, enum value lists, or state-machine diagrams describing the same named entity or concept across two or more documents. Compare against `platform/data/canonical.entities.map.v1.md` and `implementation/03-canonical-data-model.md` as the two expected canonical homes (conceptual and field-level, respectively) — any third copy is a duplicate.

**Failure Condition:** A third (or further) definition of an already-doubly-homed (conceptual + field-level) entity or enum exists anywhere in the repository.

**Expected Resolution:** Delete the third copy; replace with a reference to the conceptual home (Architecture) and, where field-level detail is needed, the field-level home (`implementation/03-canonical-data-model.md`).

---

## R-03 — Broken References

**Purpose:** A reference that does not resolve is worse than no reference — it gives false confidence that a rule has an authoritative home.

**Detection Logic:** Extract every inline reference of the form `` `path/to/file.md` `` or "see `X` — `section`" across all documents. Confirm the file exists at that path and, where a section name is given, that a heading matching that name exists in the target file.

**Failure Condition:** A referenced file does not exist, or a referenced section heading is not found in the target file.

**Expected Resolution:** Correct the path or section name; if the target was renamed, retitled, or deprecated, update the reference to point at its current name or successor (see `R-08` — Deprecated References).

---

## R-04 — Missing APD

**Purpose:** Every implementation-enabling decision (entity, background job, deployable, repository contract, transaction coordinator, advisory lock, unique index, security boundary) or Architecture correction must be traceable to a registry entry, per `implementation/README.md` — "Post-Freeze Clarification Policy" and the APD Validation check in `REPOSITORY_GOVERNANCE.md`.

**Detection Logic:** For each entity/job/deployable/contract/lock/index/boundary introduced or changed in `implementation/`, confirm a corresponding `APD-0NN` entry exists in `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` whose **Affected Implementation Documents** field names the changed file. For each correction made to an Architecture document during a governance-hardening pass, confirm a corresponding APD exists whose **Affected Canonical Documents** field names it.

**Failure Condition:** A qualifying change exists in `implementation/` or `Architecture-Product-Definition/` with no matching APD entry.

**Expected Resolution:** Author the missing APD entry using the full Registry Format (including `Supersedes`, `Affected Canonical Documents`, `Affected Implementation Documents`, `Dependent Documents`, `Migration Impact`, `Backward Compatibility`) before the change can be considered architecture-complete.

---

## R-05 — Implementation Drift

**Purpose:** This is the master check for the Decision Hierarchy's central rule: "No lower architectural layer may silently redefine any higher layer." It is the automated counterpart to the manual audit performed in Phase A.

**Detection Logic:** For every rule stated in an Architecture document, confirm the corresponding `implementation/` document (if one operationalizes that rule) does not state a materially different value, order, threshold, or capability list. Pay particular attention to: enumerated "Supported" / "Not Supported" lists, numeric thresholds (TTLs, limits, rate limits), ordered sequences (cascade steps, pipeline stages), and state machines.

**Failure Condition:** An Architecture document and its corresponding Implementation Specification disagree on the substance of a rule, and no APD entry reconciles the two (per `R-04`).

**Expected Resolution:** Follow the Required Workflow in `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md` — "Decision Hierarchy": create an APD, update the Architecture document to adopt the correct behavior (never silently prefer Implementation), confirm Implementation remains synchronized.

---

## R-06 — Unsupported V1 Features

**Purpose:** A document must never claim V1 supports a capability that does not exist anywhere in the approved `implementation/` specification set or the closed set of V1 API endpoints.

**Detection Logic:** For each capability stated as "Supported" or described in present tense under a V1-labeled section, confirm at least one of: (a) a corresponding API endpoint exists in `implementation/05-api-contracts.md`; (b) a corresponding service command exists in `implementation/04-service-contracts.md`; (c) a corresponding field/validation exists in `implementation/03-canonical-data-model.md` / `07-validation-rules.md`. A capability with none of these is unsupported regardless of what its own Architecture document claims.

**Failure Condition:** A document claims V1 support for a capability absent from all three implementation cross-checks above.

**Expected Resolution:** Either implement the capability (out of scope for a documentation-only governance pass) or correct the document to mark the capability V2 Scope (see `R-07`).

---

## R-07 — V2 Leakage

**Purpose:** A V2-only capability described without an explicit V2 Scope tag reads as available now. This was the single largest class of contradiction found in Phase A (Theme Customization).

**Detection Logic:** Cross-reference every capability flagged by `R-06` as failing the implementation cross-check. Confirm each such capability carries an explicit, visible **V2 Scope** tag at the point it is described (per the Document Version Status Policy), not merely a note elsewhere in the document or in a different document.

**Failure Condition:** A capability fails the `R-06` implementation cross-check and has no V2 Scope tag at its point of description.

**Expected Resolution:** Add an explicit V2 Scope tag at every point the capability is mentioned, plus a document-level V1 Scope Notice if the surrounding document is predominantly about the capability (as done for the six Appearance documents in APD-011).

---

## R-08 — Deprecated References

**Purpose:** A reference to a document, section, or concept that has since been marked Deprecated, Superseded, or Archived (per the Document Version Status Policy) must be updated to point at the current successor, or explicitly acknowledged as historical.

**Detection Logic:** For every reference (see `R-03`), check the target document's Status header. If the target is Deprecated or Superseded, confirm the referencing document either (a) points at the stated successor instead, or (b) explicitly marks its own reference as historical/traceability-only.

**Failure Condition:** A live (non-historical) document references a Deprecated or Superseded document/section as though it were current guidance.

**Expected Resolution:** Update the reference to the successor document or section; if no successor exists, escalate as a Repository Health violation (H-7, Obsolete Supported Behavior) rather than leaving a dangling reference to obsolete guidance.

---

## R-09 — Circular References

**Purpose:** Two documents that each claim to be the authoritative source for the same rule, each pointing at the other as though deferring to it, leaves the rule with no actual home.

**Detection Logic:** Build a directed graph of "Owner reference" edges (document A says "see document B for this rule"). Detect cycles of length 2 or more.

**Failure Condition:** A cycle exists in the Owner-reference graph for any single concept.

**Expected Resolution:** Break the cycle by designating one document as the true Owner per the Canonical Ownership table and converting every other document in the cycle into a one-directional reference to it.

---

## R-10 — Conflicting Terminology

**Purpose:** See `REPOSITORY_GOVERNANCE.md` — Repository Health Policy, H-3. Consistent terminology is what makes automated detection of the other rules in this document possible at all — a linter (automated or human) cannot compare two rules about "Session" and "Login Context" as the same concept unless terminology is already normalized.

**Detection Logic:** Maintain a canonical glossary derived from the Canonical Ownership table and the Entity Catalog (`platform/data/canonical.entities.map.v1.md`). Flag any document using a term not in the glossary to refer to a glossary concept, or using a glossary term to mean something the glossary does not define.

**Failure Condition:** A non-canonical synonym is used for a glossary concept without an explicit "(also called X in this document)" note tracing it back to the canonical term.

**Expected Resolution:** Replace the non-canonical term with the glossary term, or add the canonical term to the glossary if the document's usage is in fact the more correct one (requires an APD if this changes an already-published term's meaning).

---

## R-11 — Missing Canonical Owner

**Purpose:** Every architectural concept must appear in the Canonical Ownership table with an Architecture-tier document as Owner. A concept with no listed Owner is a concept that can silently acquire an Implementation-tier owner by default, which is exactly the failure mode corrected in Phase A.5 (G-01).

**Detection Logic:** For every concept substantively defined in two or more documents (per `R-01`/`R-02`), confirm it appears as a row in the Canonical Ownership table in `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`, and that the listed Owner column names an `Architecture-Product-Definition/` document (or this Map, or `ARCHITECTURE_PR_APPROVAL_DECISIONS.md`) — never an `implementation/` document.

**Failure Condition:** A cross-referenced concept has no Canonical Ownership row, or its row's Owner column names an `implementation/` file.

**Expected Resolution:** Add the missing row, or correct the Owner column to name the appropriate Architecture document and move the `implementation/` document to the Operational Specification column (as performed for the Session Model and Cache Strategy rows in Phase A.5).

---

## R-12 — Missing Status

**Purpose:** Every document must declare an explicit status under the Document Version Status Policy so a reader never has to guess whether a document is authoritative.

**Detection Logic:** Confirm every document under `Architecture-Product-Definition/` and `implementation/` carries a `Status:` (or `## Status`) header whose value is one of: Draft, Approved, Frozen, Deprecated, Superseded, Archived.

**Failure Condition:** A document has no status header, or a status value outside the six approved values.

**Expected Resolution:** Add the missing header. For documents predating the Document Version Status Policy that already carry `Status: Approved`, no change is required — the policy explicitly treats existing `Approved` headers as equivalent to `Frozen`.

---

## R-13 — Mixed Lifecycle States

**Purpose:** See `REPOSITORY_GOVERNANCE.md` — Repository Health Policy, H-5 and the Document Version Status Policy's explicit prohibition. A document must never simultaneously imply a capability is both current and not-yet-available without one of those statements being flagged as the error.

**Detection Logic:** Within a single document, detect a capability described both as present-tense/available ("Users may configure X") and as explicitly unsupported ("X is not supported in V1") without a clear V1/V2 disambiguation between the two statements.

**Failure Condition:** Both a "supported" and a "not supported" statement exist for the same capability in the same document without one being explicitly marked as describing a different version (V1 vs. V2) or as the historical state being corrected.

**Expected Resolution:** Apply explicit V1/V2 Scope tags at both points of description (per `R-07`), or, if the conflict is a genuine leftover contradiction rather than a V1/V2 distinction, resolve it at its source per the Governing Principle ("every contradiction must be resolved at its source; never solve contradictions by adding explanatory notes somewhere else").

---

## Rule Summary Table

| Rule | Name | Health Policy Cross-Reference |
|---|---|---|
| R-01 | Duplicate Ownership | H-1 |
| R-02 | Duplicate Canonical Definitions | H-2, H-8 |
| R-03 | Broken References | (Cross Reference Policy) |
| R-04 | Missing APD | (APD Validation) |
| R-05 | Implementation Drift | (Decision Hierarchy master check) |
| R-06 | Unsupported V1 Features | H-6 |
| R-07 | V2 Leakage | H-5 |
| R-08 | Deprecated References | (Version Status Policy) |
| R-09 | Circular References | (Cross Reference Policy) |
| R-10 | Conflicting Terminology | H-3 |
| R-11 | Missing Canonical Owner | H-1, H-6 |
| R-12 | Missing Status | (Version Status Policy) |
| R-13 | Mixed Lifecycle States | H-4, H-5 |

## Relationship To Architecture Drift Prevention

The six validations required by every future Pull Request (`REPOSITORY_GOVERNANCE.md` — "Architecture Drift Prevention") map onto these rules as follows:

- **Ownership Validation** → R-01, R-11
- **Consistency Validation** → R-02, R-05, R-10, R-13
- **Cross Reference Validation** → R-03, R-09
- **APD Validation** → R-04
- **Scope Validation** → R-06, R-07
- **Version Validation** → R-08, R-12, R-13

A Pull Request that would fail any rule in this document fails the corresponding validation in the Architecture Drift Prevention gate.
