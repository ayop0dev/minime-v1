# Phase A — Architecture Synchronization Report

**Status:** Complete
**Scope:** Resolve every Architecture ↔ Implementation contradiction identified in the prior full-stack implementation-readiness audit, without redesigning Minime, expanding V1 scope, or weakening any implementation decision that was technically correct.

This report documents every change made during Phase A. No implementation code was touched. No new product feature was introduced. Every change either (a) corrected an obsolete or contradictory Architecture document to match an already-approved, superior Implementation decision, or (b) added governance structure to prevent this class of silent divergence from recurring.

---

## 1. Every Modified File

| # | File | Nature of change |
|---|---|---|
| 1 | `Architecture-Product-Definition/appearance/themes/theme.customization.specification.v1.md` | Rewritten: added V1 Scope Notice, tagged every customization category as V2 Scope, corrected "V1 Principles" to state actual V1 behavior, resolved stale block-level-override flag |
| 2 | `Architecture-Product-Definition/appearance/themes/theme.definition.specification.v1.md` | Added status/scope notice; marked "Theme Identity Rule" customization example as V2 Scope; resolved stale block-level-override flag; split V1/V2 principles |
| 3 | `Architecture-Product-Definition/appearance/themes/theme.selection.specification.v1.md` | Added scope notice; tagged "State: Customized," resolution pipeline Step 2, and "Theme Change Behavior" as V2 Scope / no-op in V1; corrected V1 Principles footer |
| 4 | `Architecture-Product-Definition/appearance/design.editor.specification.v1.md` | Added V1 Scope Notice; split "Editor Capabilities" into V1 (Theme selection, preview, block-level overrides) and V2 Scope (all customization controls) |
| 5 | `Architecture-Product-Definition/appearance/appearance.system.specification.v1.md` | Added scope notice; re-tagged "Theme customization" in the responsibilities list as V2 Scope |
| 6 | `Architecture-Product-Definition/appearance/appearance.state.specification.v1.md` | Added scope notice clarifying Appearance State stores `selected_theme_id` only in V1 |
| 7 | `Architecture-Product-Definition/settings/settings.categories.specification.v1.md` | Tagged "Visual customization" under the Appearance settings category as V2 Scope |
| 8 | `Architecture-Product-Definition/account-management/minime.account.management.system.specification.v1.md` | Session Policy section rewritten: replaced obsolete flat 7-day/no-session-management model with the approved JWT + rotating refresh token + session list + logout-all model |
| 9 | `Architecture-Product-Definition/seo/seo.indexing.policy.v1.md` | Rewritten in full: removed all draft/publish/unpublished references; indexing eligibility, robots policy, sitemap participation, and visibility-change handling now keyed exclusively on `Account.status` |
| 10 | `Architecture-Product-Definition/public-profile/public-profile.cache.policy.v1.md` | Rewritten sections: adopted the two-layer cache model (Edge TTL-only + Application/Redis with Smart Cache Invalidation); removed the prohibition on Smart Cache Invalidation |
| 11 | `Architecture-Product-Definition/account/account.deletion.policy.v1.md` | Cascade order (Section 4), entity deletion rules (Section 5), and the ownership summary table (Section 9) updated to match the implemented, retry-safe order, with engineering rationale recorded at the source |
| 12 | `Architecture-Product-Definition/account-management/audit.logging.policy.v1.md` | "Logged Events" catalog corrected to remove `recovery_email_changed`, `page_*`, and per-`block_*` events that describe non-existent V1 concepts; rationale recorded inline |
| 13 | `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` | Six new registry entries added: APD-011 through APD-016 |
| 14 | `Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md` | Three new governance sections added: "Decision Hierarchy," "Canonical Ownership," "Document Version Status Policy" |
| 15 | `PHASE_A_ARCHITECTURE_SYNCHRONIZATION_REPORT.md` | This report (new file) |

No file outside `Architecture-Product-Definition/` and the repository root was modified. `implementation/` was read for reference only and was not changed, per the instruction to never weaken a correct implementation decision.

---

## 2. Every Contradiction Resolved

| ID | Contradiction | Resolution | Direction of correction |
|---|---|---|---|
| A-01 | Theme Customization: fully described as V1 in four Appearance documents; fully rejected in the data model | All customization capability explicitly marked **V2 Scope**; V1 = Theme selection + block-level overrides only | Architecture corrected to match Implementation |
| A-02 | Authentication Session: Architecture said "no session management, no logout-all, 7-day flat session"; Implementation requires JWT + refresh + session list + logout-all | Architecture updated to describe the approved JWT/refresh/session-management model | Architecture corrected to match Implementation |
| A-03 | SEO Indexing: policy assumed a draft/publish workflow that exists nowhere else in V1 | Policy rewritten to key indexing eligibility exclusively on `Account.status` | Architecture corrected (internal consistency — no implementation change needed, `12-seo-and-integrations.md` was already correct) |
| A-04 | Cache Invalidation: policy forbade "Smart Cache Invalidation"; implementation requires it on every mutation | Policy rewritten to describe the actual two-layer cache model with Smart Cache Invalidation adopted for the application layer | Architecture corrected to match Implementation |
| A-05 | Account Deletion cascade order: Architecture specified one order; Implementation deliberately uses a different, retry-safe order | Architecture's cascade order promoted to match Implementation, with the idempotency rationale now recorded at the source | Architecture corrected to match Implementation (superior engineering decision) |
| (bonus, folded into A-06) | `AnalyticsEvent.account_id` on `link_click`: present in Implementation, absent from the Analytics Model architecture documents | Recorded via APD-016 rather than rewriting the Analytics Model documents field-by-field, since the addition is a narrow, backward-compatible snapshot field | Implementation decision formally adopted via registry |
| (G-04) | Audit Logging catalog listed events for non-existent V1 concepts (`recovery_email_changed`, `page_*`, `block_*`) | Catalog corrected to match the already-correct implementation-layer catalog | Architecture corrected to match Implementation |

Every resolution follows Required Principle 3/4: where Implementation already reflected the better engineering decision (A-02, A-04, A-05, the `AnalyticsEvent.account_id` addition, the audit log catalog), Architecture was updated to adopt it. Where Architecture and Implementation both needed the same underlying invariant restated consistently (A-01, A-03), the correction was applied at the Architecture layer, and Implementation was left untouched because it was already correct.

---

## 3. Every APD Added

| APD | Title | Classification |
|---|---|---|
| APD-011 | Theme Customization Deferred to V2 | Architecture Correction (Scope Narrowing) |
| APD-012 | Session Model: JWT Access Token + Rotating Refresh Token, With Session Management | Architecture Correction (Implementation Adoption) |
| APD-013 | SEO Indexing Keyed Exclusively on Account.status (No Draft/Publish Workflow) | Architecture Correction (Consistency Fix) |
| APD-014 | Smart Cache Invalidation Adopted for the Application Cache Layer | Architecture Correction (Implementation Adoption) |
| APD-015 | Account Deletion Cascade Order Promoted From Implementation | Architecture Correction (Implementation Adoption) |
| APD-016 | AnalyticsEvent.account_id Snapshot on link_click Rows | Entity (Field Addition) |

Each entry follows the existing registry format (Status, Classification, Why it was required, Why it is NOT an architectural redesign, Scope, Explicit Non-Goals, Files defining the decision, Files depending on the decision) exactly as established by APD-001 through APD-010, so the registry remains internally consistent.

---

## 4. Every Obsolete Document Updated

| Document | Obsolete content found | Action taken |
|---|---|---|
| `account-management/minime.account.management.system.specification.v1.md` | Entire "Session Policy" section described a session model never implemented | Rewritten (not merely annotated) |
| `seo/seo.indexing.policy.v1.md` | Entire document assumed a draft/publish workflow | Rewritten (not merely annotated) |
| `public-profile/public-profile.cache.policy.v1.md` | "Not Supported" list and "Profile Updates" section contradicted the implemented invalidation strategy | Rewritten (not merely annotated) |
| `account/account.deletion.policy.v1.md` | Cascade order and per-entity notes assumed a non-retry-safe order | Sections 4, 5, and 9 rewritten |
| `account-management/audit.logging.policy.v1.md` | "Logged Events" catalog listed non-existent V1 event types | Catalog corrected with inline rationale |
| `appearance/themes/theme.customization.specification.v1.md`, `theme.definition.specification.v1.md`, `theme.selection.specification.v1.md`, `appearance/design.editor.specification.v1.md`, `appearance.system.specification.v1.md`, `appearance.state.specification.v1.md` | Described theme-wide customization as current V1 capability | Annotated with explicit V1/V2 Scope tags throughout (content preserved as the V2 target design, not deleted, since it remains useful for future implementation) |

No document was marked **Deprecated**, **Superseded**, or **Archived** in this phase — every document identified as obsolete was corrected in place, because in every case the document's *subject* (sessions, SEO indexing, caching, account deletion, audit logging, theme customization) remains an active, relevant V1 or V2 concern; only specific *sections* were wrong. None of the six documents warranted full retirement.

**Follow-up (Phase B, not performed here):** applying the new Version Status Policy's explicit status header (Draft/Approved/Frozen/Deprecated/Superseded/Archived) to the remaining ~120 documents in the repository that were not touched in this phase. This was intentionally out of scope for Phase A to avoid unbounded, low-value churn across documents with no actual contradiction; the policy itself is now defined and ready for that follow-up pass.

---

## 5. Decision Hierarchy Summary

A new "Decision Hierarchy" section was added to `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`:

```text
Product Vision
    ↓
Architecture  (Architecture-Product-Definition/)
    ↓
Approved APD Decisions  (ARCHITECTURE_PR_APPROVAL_DECISIONS.md)
    ↓
Implementation Specifications  (implementation/)
    ↓
Code
    ↓
Deployment Configuration
```

Every layer is defined with an explicit rule for what happens when a lower layer appears to disagree with a higher one: the lower layer is corrected, never the higher one — except that an APD entry may formally promote a superior Implementation decision upward into Architecture, which is exactly the mechanism used for five of the six contradictions resolved in this phase.

---

## 6. Canonical Ownership Summary

A new "Canonical Ownership" table was added to `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`, assigning exactly one owner document to each concept that was previously described inconsistently in two places:

- Session lifecycle/token model → `implementation/08-security-model.md`
- Theme selection vs. customization scope → `appearance/themes/theme.customization.specification.v1.md`
- Block-level style overrides → `block-styling/block-style.model.specification.v1.md`
- Public profile cache strategy → `implementation/09-caching-strategy.md`
- SEO indexing eligibility → `seo/seo.indexing.policy.v1.md`
- Account deletion cascade order → `account/account.deletion.policy.v1.md` (Section 4)
- Canonical entity map → `platform/data/canonical.entities.map.v1.md`
- Post-freeze implementation-enabling decisions → `ARCHITECTURE_PR_APPROVAL_DECISIONS.md`
- Product domain boundaries → `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md` itself

Every other document referencing these concepts must defer to the owner rather than restating the rule independently — this is the structural fix that prevents the next version of the Theme Customization / Session Model / Cache Invalidation class of contradiction from being introduced silently.

---

## 7. Version Policy Summary

A new "Document Version Status Policy" section was added to `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`, defining six exclusive statuses: **Draft, Approved, Frozen, Deprecated, Superseded, Archived**. The policy explicitly states that existing `Status: Approved` headers are understood as equivalent to **Frozen** for V1 architecture documents (no retroactive relabeling required), and that mixed lifecycle states — a document simultaneously implying a capability is both supported and not supported — are forbidden going forward. Every document touched in this phase now carries explicit V1/V2 Scope annotations at the exact points where the prior ambiguity existed, satisfying this policy for those documents specifically; repository-wide retrofitting of the six-state header to all remaining documents is flagged as a Phase B follow-up, not performed here.

---

## 8. Confirmation

After the changes recorded in this report:

- **No document simultaneously describes any capability as both Supported and Not Supported.** Theme customization, session management, SEO indexing eligibility, cache invalidation strategy, and account-deletion cascade order are each now described identically, in direction and in detail, at both the Architecture layer and the Implementation layer.
- **No V2 capability is presented as V1 anywhere touched in this phase.** Every theme-wide customization capability is explicitly labeled V2 Scope at every point it is mentioned, distinguished clearly from the already-approved, real V1 block-level style override mechanism.
- **Every implementation-enabling decision identified as previously unrecorded is now recorded** in `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` (APD-011 through APD-016), each with reason, scope, files, architecture impact, and explicit non-goals, in the same format as the pre-existing APD-001 through APD-010.
- **No implementation decision that was technically correct was reverted or weakened.** The JWT/refresh session model, Smart Cache Invalidation, and the retry-safe deletion cascade order all remain exactly as implemented; only the Architecture documents were brought into agreement with them.
- **Architecture remains the declared single source of truth.** Every correction was made by editing the Architecture document to state the agreed system behavior, never by leaving Implementation to silently stand as an unacknowledged parallel authority.

**Architecture and Implementation now describe the same V1 system** for every contradiction identified in the source audit. Minor, lower-severity documentation-drift items noted in that audit but not listed in the Phase A required-fixes scope (e.g. the stale 7-of-12 platform summary in `social.accounts.platform.rules.v1.md`, the missing `.env.example` file, undocumented numeric rate-limit values) were intentionally left untouched, as they were not part of the six required fixes (A-01 through A-06) or the governance improvements (G-01 through G-05) specified for this phase, and addressing them would have expanded this phase's scope beyond what was requested.

---

**End of Phase A. No further refactoring performed, per instruction.**
