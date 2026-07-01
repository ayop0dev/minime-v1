# AI Suggestions Specification V1

## Status

**Version:** V1

**Status:** Canonical

**Layer:** Platform

**Platform Service:** AI

---

# Purpose

This specification defines how the AI Platform produces suggestions within Minime.

Suggestions are the primary output of the AI Platform.

Suggestions assist users.

They never control users.

Suggestions are advisory rather than authoritative.

---

# Architectural Philosophy

The AI Suggestions model follows seven principles.

## Suggestions Never Become Truth

Every suggestion begins as a non-canonical proposal.

Nothing becomes Product Data until explicitly approved by the user.

---

## Suggestions Never Execute

Suggestions do not perform actions.

Suggestions do not modify Product Data.

Suggestions do not publish content.

Suggestions recommend.

Users decide.

---

## Suggestions Are Context-Driven

Every suggestion is produced from available platform context.

Suggestions should never ignore known platform knowledge.

Better context produces better suggestions.

---

## Suggestions Must Explain Value

Whenever practical, a suggestion should communicate why it exists.

Users should understand:

* what is being suggested
* why it is recommended
* what benefit it may provide

Transparent suggestions build trust.

---

## Suggestions Should Be Reusable

Identical context should not require repeated generation of identical suggestions.

Previously generated suggestions may be reused whenever still applicable.

Architecture should prefer reuse over regeneration.

---

## Suggestions Should Improve Over Time

**V2 scope:** Persistent learning from user decisions (storing accepted/rejected signals to improve future suggestions) is outside V1 scope. No `AiDecision` or suggestion history entity exists in V1.

In V1, accepted suggestions are written to the responsible Product Domain as ordinary domain data (e.g. an accepted bio suggestion is saved to `ProfileContent.bio`). No separate decision record is created. Rejected suggestions are discarded without persistence.

Future versions may introduce a suggestion history entity to enable confidence-based learning.

---

## Suggestions Should Minimize AI Consumption

Suggestions should be generated only when they provide meaningful additional value.

Repeated reasoning over unchanged context should be avoided.

Architecture should optimize usefulness per inference.

---

# Suggestion Lifecycle

In V1, suggestions are generated through an on-demand Analysis Session triggered by the explicit "Analyze My Profile" user action. There are no implicit triggers and no background suggestion generation.

Conceptually every suggestion follows the same lifecycle.

```text
Explicit "Analyze My Profile" User Request

↓

Context Constructed (profile, blocks, connected accounts, analytics summary)

↓

AIService.analyzeProfile → Provider.execute(request)

↓

Suggestions Generated

↓

Presented To User

↓

Accepted or Rejected

↓

Platform Learns (V2 — no suggestion history persisted in V1)
```

The lifecycle ends with the user's decision. In V1, accepted suggestions are applied as ordinary domain mutations (e.g. accepted bio is saved to `ProfileContent.bio`). No suggestion record is created.

---

# Suggestion Categories

The AI Platform may produce many kinds of suggestions.

Examples include:

## Content Suggestions

Examples:

* improve biography
* rewrite description
* shorten text
* generate headline

---

## Structural Suggestions

Examples:

* reorder buttons
* reorganize sections
* simplify navigation
* merge similar content

---

## Branding Suggestions

Examples:

* improve consistency
* strengthen messaging
* clarify positioning
* improve visual hierarchy

---

## Growth Suggestions

Examples:

* add missing links
* complete profile
* publish more information
* improve discoverability

---

## Quality Suggestions

Examples:

* broken links
* missing information
* inconsistent formatting
* duplicate content

---

## Optimization Suggestions

Examples:

* reduce unnecessary blocks
* improve readability
* improve mobile experience
* simplify layout

---

# Suggestion Confidence

Suggestions may have different confidence levels.

Confidence reflects how strongly available evidence supports a recommendation.

Confidence never guarantees correctness.

Final judgment always belongs to the user.

---

# User Decisions

Every suggestion eventually reaches one of three outcomes.

## Accepted

The user approves the suggestion.

If Product Data changes:

The Product Domain performs the update.

The suggestion itself never changes Product Data.

---

## Rejected

The user explicitly rejects the suggestion.

**V2 scope:** Rejection signals as persistent knowledge are outside V1 scope. In V1, rejections are discarded — no record is created. Future versions may persist rejection signals to avoid repeating unwanted suggestions.

---

## Ignored

The user takes no action.

Ignoring a suggestion should not automatically be interpreted as rejection.

**V2 scope:** Tracking ignored suggestion outcomes is outside V1 scope.

---

# Learning From Decisions

**V2 scope.** Persistent learning from user decisions is outside V1. V1 does not store suggestion history or decision signals. This section describes future intended behavior.

In a future version, the AI Platform may reuse user decisions whenever appropriate:

```text
User consistently prefers short biographies.

↓

Future biography suggestions become shorter.
```

```text
User repeatedly rejects emoji usage.

↓

Future content avoids unnecessary emojis.
```

Learning would occur through observed user preference rather than autonomous behavior. V1 does not implement this.

---

# Relationship with Data

Data owns accepted results.

Suggestions own proposals.

Approval transfers changes into Product Data through the responsible Product Domain.

---

# Relationship with Events

**V2 scope.** AI suggestion events (`ai.suggestion.generated`, `ai.suggestion.accepted`, `ai.suggestion.rejected`) are outside V1 scope. The V1 Event Catalog (`implementation/06-event-contracts.md`) does not include these events.

In V1, accepted suggestions produce ordinary Product Domain events (e.g. `profile.updated` when an accepted bio suggestion is saved). No AI-specific event is emitted.

---

# Relationship with Analytics

**V2 scope.** Analytics measurement of suggestion performance (acceptance rate, rejection rate, ignored suggestions, effectiveness trends) is outside V1 scope. The V1 Analytics System consumes only `profile.viewed` and `out-link.clicked` events (`implementation/06-event-contracts.md`). No suggestion analytics exist in V1.

---

# Relationship with Context

Suggestions depend entirely on context quality.

Poor context produces unreliable suggestions.

Context construction always precedes reasoning.

---

# Relationship with Models

Suggestions are independent from specific AI models.

The architecture defines:

* inputs
* outputs
* responsibilities

Model selection remains an implementation decision.

---

# Cost Optimization

The AI Platform minimizes suggestion cost by following this order.

```text
Existing Suggestion (Analysis Session Reuse / in-session)

↓

Existing Context (available Product Data)

↓

Deterministic Rule

↓

Small Model

↓

Large Model
```

Escalation occurs only when necessary.

**Note:** "Existing User Decision" as a reuse source is outside V1 scope (no suggestion history is persisted in V1). Suggestions from the latest completed Analysis Session whose Input Hash, `analysis_version`, and `output_schema_version` all match (Analysis Session Reuse), or generated earlier in the same session, may still be reused.

---

# Failure Philosophy

If suggestion generation fails:

* Product Domains continue operating.
* Product Data remains correct.
* Existing suggestions remain usable.
* Platform functionality remains unaffected.

Suggestions are enhancements rather than requirements.

---

# Implementation Independence

This specification intentionally avoids defining:

* prompt templates
* provider APIs
* model routing
* inference engines
* response formats
* confidence algorithms
* caching strategies
* storage mechanisms

Those belong to implementation.

---

# Architectural Non-Goals

The AI Suggestions Platform is not responsible for:

* autonomous execution
* automatic publishing
* Product ownership
* business validation
* permissions
* workflow automation
* replacing user judgment

Suggestions remain advisory only.

---

# Canonical Rules

The AI Suggestions model follows these rules.

1. Suggestions are never canonical.
2. Suggestions never execute actions.
3. Suggestions depend on context.
4. User approval is required for Product changes.
5. User decisions improve future suggestions (V2 — no suggestion history persisted in V1).
6. Existing knowledge should be reused before generating new reasoning.
7. Suggestions should explain their value whenever practical.
8. AI consumption should be minimized through reuse.
9. Suggestions remain model-independent.
10. The user remains the final decision maker.

---

# Canonical Principle

The purpose of the AI Platform is not to think instead of the user.

Its purpose is to reduce effort, increase understanding, and provide meaningful recommendations.

Knowledge comes from the platform.

Intelligence comes from reasoning.

Authority comes from the user.

The user remains the single source of truth.

The AI Platform remains a trusted advisor—never the owner of decisions.
