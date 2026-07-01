# Phase A.5 — Repository Governance Hardening Report

**Status:** Complete
**Scope:** Harden the repository's governance so that future Architecture Drift is structurally prevented rather than dependent on the next manual audit. No product redesign, no V1 scope expansion, and no implementation behavior change was performed in this phase.

Phase A (see `PHASE_A_ARCHITECTURE_SYNCHRONIZATION_REPORT.md`) resolved every known Architecture ↔ Implementation contradiction. Phase A.5 audits the governance structure Phase A itself introduced, corrects one class of error found in it (Implementation documents listed as canonical Owners), and adds the permanent structural defenses (ownership rules, cross-reference discipline, health rules, a PR validation gate, an enhanced APD template, and an implementation-agnostic linter rule set) requested for this phase.

---

## 1. Every Modified File

| # | File | Nature of change |
|---|---|---|
| 1 | `Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md` | Decision Hierarchy expanded with an explicit conflict-resolution rule and required workflow (G-02); Canonical Ownership table restructured into explicit Owner (Architecture) / Operational Specification (Implementation) columns, correcting two rows that previously listed an `implementation/` document as Owner (G-01) |
| 2 | `Architecture-Product-Definition/docs/ARCHITECTURE_PR_APPROVAL_DECISIONS.md` | Registry Format expanded with six new required fields; all 16 existing entries (APD-001–APD-016) retrofitted with `Supersedes`, `Affected Canonical Documents`, `Affected Implementation Documents`, `Dependent Documents`, `Migration Impact`, `Backward Compatibility`; Amendment Procedure updated to require the full template going forward (G-06) |
| 3 | `Architecture-Product-Definition/account-management/audit.logging.policy.v1.md` | Minor residual fix carried over from Phase A validation: "Logged Events" catalog note reconfirmed consistent (no new change beyond what Phase A already applied) |
| 4 | *(new)* `Architecture-Product-Definition/docs/REPOSITORY_GOVERNANCE.md` | New governance document: Cross Reference Policy (G-03), Repository Health Policy — 8 rules H-1 through H-8 (G-04), Architecture Drift Prevention — 6-validation PR gate (G-05) |
| 5 | *(new)* `ARCHITECTURE_LINTER_RULES.md` | New governance document: 13 implementation-agnostic linter rules (R-01 through R-13), each with Purpose / Detection Logic / Failure Condition / Expected Resolution, plus a mapping back to the Repository Health Policy and the Architecture Drift Prevention gate (G-07) |
| 6 | `PHASE_A5_GOVERNANCE_HARDENING_REPORT.md` | This report (new file) |

No file under `implementation/` was modified by this phase. (`implementation/README.md` shows as locally modified in `git status`, but that change predates this phase and Phase A — it was already present in the working tree before either governance phase began, per the repository's initial state, and neither phase touched it.) No product behavior, API contract, entity shape, or business rule was changed anywhere in this phase — only governance-document text.

---

## 2. Canonical Ownership Corrections (G-01)

Two rows in the Canonical Ownership table, as introduced in Phase A, violated the governing principle this phase exists to enforce: they listed an `implementation/` document as the Owner of an architectural concept.

| Concept | Before (Phase A) | After (Phase A.5) |
|---|---|---|
| Session lifecycle, token model, session management | Owner: `implementation/08-security-model.md` | Owner: `account-management/minime.account.management.system.specification.v1.md` ("Session Policy"). Operational Specification: `implementation/08-security-model.md`. |
| Public profile cache strategy | Owner: `implementation/09-caching-strategy.md` | Owner: `public-profile/public-profile.cache.policy.v1.md` (two-layer model, Smart Cache Invalidation). Operational Specification: `implementation/09-caching-strategy.md`. |

The table format itself was restructured repository-wide (all nine rows) into explicit **Owner (Architecture)** / **Operational Specification (Implementation)** columns, so this class of error is visually impossible to reintroduce silently — every row now has a column that must name an Architecture document, and a separate column reserved for the Implementation-tier operational detail. No behavior changed; only which document is recorded as authoritative for each concept.

---

## 3. Decision Hierarchy Improvements (G-02)

Added to `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md` — "Decision Hierarchy":

- An explicit, standalone **Conflict Resolution Rule**: *"No lower architectural layer may silently redefine any higher layer."*
- A mandatory, single-path **workflow diagram** for the case Implementation discovers a better engineering solution:

```text
Implementation discovers a better engineering solution
        ↓
Create an APD entry
        ↓
Update the Architecture document(s)
        ↓
Implementation remains synchronized
        ↓
Code follows the updated Architecture
```

No alternative workflow is permitted. This is the same workflow Phase A followed for five of its six contradiction resolutions (Session Model, Cache Invalidation, Account Deletion Cascade Order, the `AnalyticsEvent.account_id` field, and the audit-log catalog correction) — it is now recorded as the mandatory path rather than only having been followed once.

---

## 4. Cross Reference Policy (G-03)

Defined in `Architecture-Product-Definition/docs/REPOSITORY_GOVERNANCE.md` — Section 1:

- **Core rule:** every architectural concept has exactly one authoritative definition; every other document references it by file path and section name rather than restating it.
- Six sub-rules covering: one-definition-many-references, section-level (not file-level) reference precision, examples-must-trace-to-source, repeated ownership is forbidden, new concepts create their own authoritative home immediately, and the explicit Architecture-may-reference-Implementation-only-as-Operational-Specification-pointer rule (never as a rule source).
- One explicit exemption: a *consequence* of a rule (citing the source) is not a duplicate of the rule itself.

---

## 5. Repository Health Policy (G-04)

Defined in `REPOSITORY_GOVERNANCE.md` — Section 2. Eight unhealthy states the repository must never enter, each with Purpose / Detection / Required Resolution:

| ID | Rule |
|---|---|
| H-1 | Duplicated Ownership |
| H-2 | Duplicated Business Rules |
| H-3 | Conflicting Terminology |
| H-4 | Conflicting Lifecycle States |
| H-5 | V2 Behavior Described As V1 |
| H-6 | Implementation-Only Architecture |
| H-7 | Obsolete Supported Behavior |
| H-8 | Two Documents Defining The Same Canonical Decision |

---

## 6. Architecture Drift Prevention (G-05)

Defined in `REPOSITORY_GOVERNANCE.md` — Section 3. Every future Pull Request touching Architecture or Implementation must pass six validations before being considered architecture-complete:

1. **Ownership Validation** — every touched concept has exactly one Architecture-tier Owner.
2. **Consistency Validation** — no contradiction remains per Repository Health H-1 through H-8.
3. **Cross Reference Validation** — every reference resolves; no new duplicate rule introduced.
4. **APD Validation** — every qualifying change has a corresponding, fully-populated APD entry.
5. **Scope Validation** — no V1 scope expansion or undocumented new Product Domain.
6. **Version Validation** — every touched document has a valid, unambiguous status; no capability left describable as both V1 and V2 Scope.

A PR failing any validation must not be considered architecture-complete, and if merged anyway, must be explicitly flagged with a follow-up task rather than silently accepted — silent partial merges are the exact failure mode this gate exists to prevent.

---

## 7. Updated APD Template (G-06)

The Registry Format in `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` now requires, per entry:

`Decision ID` · `Decision Name` · `Status` · `Classification` · `Why it was required` · `Why it is NOT an architectural redesign` (satisfies "Reason this is not a redesign") · **`Supersedes`** · `Scope` · `Explicit Non-Goals` · **`Affected Canonical Documents`** · **`Affected Implementation Documents`** · **`Dependent Documents`** · **`Migration Impact`** · **`Backward Compatibility`**

(Bold = new in this phase.) All 16 existing entries (APD-001 through APD-016) were retrofitted with every new field — verified by direct count: `Supersedes` × 16, `Affected Canonical Documents` × 16, `Affected Implementation Documents` × 16, `Dependent Documents` × 16, `Migration Impact` × 16, `Backward Compatibility` × 16. Entries with no direct Architecture-tier grounding (e.g. APD-004 `apps/worker`, a pure deployment-topology decision) state this explicitly ("None directly — ...") rather than leaving the field blank, per the Amendment Procedure's new requirement that architectural grounding is never left unstated.

---

## 8. Architecture Linter Rules Summary (G-07)

`ARCHITECTURE_LINTER_RULES.md` (new, repository root) defines 13 implementation-agnostic rules:

| Rule | Name |
|---|---|
| R-01 | Duplicate Ownership |
| R-02 | Duplicate Canonical Definitions |
| R-03 | Broken References |
| R-04 | Missing APD |
| R-05 | Implementation Drift |
| R-06 | Unsupported V1 Features |
| R-07 | V2 Leakage |
| R-08 | Deprecated References |
| R-09 | Circular References |
| R-10 | Conflicting Terminology |
| R-11 | Missing Canonical Owner |
| R-12 | Missing Status |
| R-13 | Mixed Lifecycle States |

Each rule states Purpose, Detection Logic, Failure Condition, and Expected Resolution, and the document closes with an explicit mapping from these 13 rules onto the six Architecture Drift Prevention validations (Section 6 above), so the two documents function as one enforcement system: `REPOSITORY_GOVERNANCE.md` states *what* must be true; `ARCHITECTURE_LINTER_RULES.md` states *how* it is checked. The document is deliberately silent on tooling, CI, or scripting choices — it defines repository health, not a specific automation stack.

---

## 9. Final Repository Governance Assessment

**Verified during this phase (G-08):**

- Every row in the Canonical Ownership table now names an Architecture-tier document as Owner; no `implementation/` document appears in the Owner column anywhere in the table.
- Every one of the 16 APD entries carries all six newly-required fields (verified by direct count, Section 7 above), and each entry's `Affected Canonical Documents` field either names a real Architecture document or explicitly states "None directly" with a cited grounding principle — never left blank.
- The two documents introduced in this phase (`REPOSITORY_GOVERNANCE.md`, `ARCHITECTURE_LINTER_RULES.md`) exist at their stated paths and cross-reference each other and the Map/registry consistently.
- No `implementation/` file was modified by this phase; the one pre-existing local modification to `implementation/README.md` predates both Phase A and Phase A.5 and was left untouched, per the restriction against modifying implementation logic.

**One residual observation, not corrected in this phase (out of the explicit G-01 through G-08 scope):** `implementation/README.md` and the registry's own header both describe `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` as living "at the repository root," while the file actually resides at `Architecture-Product-Definition/docs/ARCHITECTURE_PR_APPROVAL_DECISIONS.md`. This is a pre-existing path-description inaccuracy, not introduced by Phase A or Phase A.5, and not a rule contradiction of the kind this phase was scoped to fix (it is a path-accuracy nit, not a duplicated rule or an ownership conflict). It is recorded here for visibility; per `ARCHITECTURE_LINTER_RULES.md` — R-03 ("Broken References"), a future pass should either move the file to match its described location or correct the description to match its actual location — this report takes no position on which correction is preferable.

---

## 10. Explicit Confirmations

- **Architecture is the sole Source of Truth.** Every concept in the Canonical Ownership table names an `Architecture-Product-Definition/` document (or the Map, or the APD registry itself) as Owner. No exception exists anywhere in the table after the corrections in Section 2.

- **Implementation no longer owns any architectural concept.** Every `implementation/` document that previously appeared in an Owner position has been demoted to Operational Specification, a role that is now structurally distinct in the table's column layout, not just in prose. The Decision Hierarchy's Conflict Resolution Rule makes this permanent: "No lower architectural layer may silently redefine any higher layer."

- **Future Architecture Drift is structurally prevented rather than manually monitored.** Three independent, mutually-reinforcing mechanisms now exist where previously only a manual audit did: (1) the Architecture Drift Prevention gate in `REPOSITORY_GOVERNANCE.md`, which every future Architecture- or Implementation-touching PR must pass across six named validations; (2) the enhanced APD template, which makes every implementation-enabling decision self-documenting (its supersession target, its affected documents on both tiers, its migration and compatibility impact) rather than requiring a future auditor to reconstruct that context; and (3) `ARCHITECTURE_LINTER_RULES.md`'s 13 concrete, checkable rules, each mapped to a specific Repository Health violation and a specific Drift Prevention validation, providing the detection logic the other two mechanisms assume exists.

---

**End of Phase A.5. No further refactoring performed, per instruction.**
