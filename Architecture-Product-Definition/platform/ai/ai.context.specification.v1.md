# AI Context Specification V1

## Status

**Version:** V1

**Status:** Canonical

**Layer:** Platform

**Platform Service:** AI

---

# Purpose

This specification defines how the AI Platform builds contextual understanding before performing any reasoning.

The quality of AI output depends primarily on the quality of context rather than the size of the language model.

Minime therefore treats context construction as a first-class architectural responsibility.

---

# Architectural Philosophy

The AI Context model follows six principles.

## Context Before Reasoning

Reasoning should never begin until sufficient context has been collected.

Incomplete context produces unreliable suggestions.

Better context is preferred over larger models.

---

## Read Existing Knowledge First

The AI Platform should maximize existing platform knowledge before generating new intelligence.

Existing knowledge is significantly cheaper than repeated inference.

---

## Context Is Read-Only

The AI Platform builds context by reading platform information.

Context construction never changes:

* Product Data
* Events
* Storage
* Analytics
* Rendering

Context is observational.

---

## Context Is Disposable

Context exists only for understanding.

It is a temporary working view.

It is not canonical.

It is not permanent platform knowledge.

---

## Context Should Be Minimal

Only information required to answer the current task should be included.

Unnecessary context increases:

* latency
* token consumption
* cost
* reasoning complexity

Smaller context is preferred whenever possible.

---

## Context Is Independent From Models

Context construction belongs to the platform.

Reasoning belongs to the selected model.

Changing models should not require rebuilding platform knowledge.

---

# Context Sources

The AI Platform may construct context from multiple sources.

## Product Data

Canonical user-controlled information.

Examples:

* profile
* bio
* buttons
* social accounts
* appearance
* settings

Product Data is the highest-confidence knowledge source.

---

## Events

Historical facts describing platform activity.

Examples:

* profile updated
* block added
* connected account added
* asset uploaded

Events explain how the platform reached its current state.

**V2 scope:** Suggestion accepted and suggestion rejected events are outside V1 scope. V1 emits no AI-specific events. Accepted suggestions produce ordinary Product Domain events (e.g. `profile.updated`).

---

## User Decisions (V2)

**V2 scope:** Persistent user decision context (accepted/rejected suggestion signals) is outside V1 scope. No suggestion history is stored in V1. The following describes intended future behavior.

User decisions represent verified human preference.

Examples:

* accepted suggestion
* rejected recommendation
* manual edit
* preferred ordering

Previous user decisions should be reused whenever appropriate.

They are often more valuable than generating new reasoning.

---

## Analytics

Analytics provide aggregated behavioral information.

The AI Platform may only consume analytics data that Minime V1 actually collects. The canonical definition of what Minime collects is owned by the Analytics Privacy Policy.

Examples:

* most clicked links
* device category breakdown (Desktop, Mobile, Tablet)
* engagement trends

Analytics describe measured behavior rather than user intent.

---

## Rendering

Rendered output provides visual context.

AI may inspect rendered pages to evaluate:

* readability
* hierarchy
* consistency
* presentation quality

Rendering remains independent from AI.

---

## Storage Metadata

Storage contributes metadata only.

Examples:

* image dimensions
* format
* aspect ratio
* file size

Binary ownership remains with the Storage Platform.

---

# Context Construction

Conceptually every AI task follows the same process.

```text
Task

↓

Determine Required Context

↓

Read Existing Knowledge

↓

Remove Irrelevant Information

↓

Construct Working Context

↓

Reason
```

Reasoning always comes last.

---

# Context Scope

Every AI task defines its own scope.

Examples:

## Bio Improvement

Requires:

* current bio
* display name
* previous accepted bio suggestions (V2 — not available in V1; use current bio only)

Does not require:

* analytics
* QR codes
* rendering history

---

## Link Prioritization

Requires:

* buttons
* analytics
* previous ordering decisions (V2 — not available in V1; use current block order)

Does not require:

* avatar metadata
* storage details

---

## Profile Review

Requires:

* profile data
* rendered output
* selected analytics
* user goals

Scope should remain as narrow as possible.

---

# Context Reuse

Previously constructed context may be reused when still valid.

Context reuse reduces:

* repeated reads
* repeated reasoning
* AI cost
* response latency

Reusable context is preferred over rebuilding identical context.

---

# Context Freshness

Context should reflect current platform knowledge.

If relevant information changes:

* Product Data
* Events
* User Decisions (V2 — no suggestion history in V1; Product Data changes are the primary freshness trigger)

the previous context should be considered stale.

In V1, context freshness is determined by whether the Analysis Input Snapshot (account state: profile, blocks, connected accounts, analytics summary) has changed since the last Analysis Session. The Input Hash captures this snapshot; a changed Input Hash means no matching completed Analysis Session exists, so a new `Provider.execute(request)` call is made. An unchanged Input Hash, combined with a matching `analysis_version` and `output_schema_version`, means the latest completed Analysis Session is reused (Analysis Session Reuse).

The Analysis Input Snapshot is the only approved V1 context object for an Analysis Session. It is constructed exclusively by `AIService`; Product Domains must not supply custom ad-hoc AI context.

Freshness is determined by platform state, never by elapsed time.

---

# Context Confidence

Different context sources provide different levels of confidence.

General priority:

```text
User Decision (V2 — no suggestion history in V1)

↓

Canonical Product Data

↓

Historical Events

↓

Analytics

↓

Rendering

↓

Storage Metadata

↓

Generated AI Content
```

When multiple sources disagree, higher-confidence sources should take precedence.

In V1, the highest-confidence context source available is Canonical Product Data — the current profile, blocks, and connected accounts state.

---

# Context Compression

Before reasoning, context should be reduced to only information that contributes to the current task.

Compression should preserve meaning while reducing unnecessary tokens.

Architecture should optimize understanding rather than volume.

---

# Relationship with Data

Data provides canonical truth.

Context reads Data.

Context never changes Data.

---

# Relationship with Events

Events provide historical explanation.

Context may use historical facts to understand why current state exists.

---

# Relationship with Analytics

Analytics provide measurable behavior.

Context may combine behavior with Product Data to produce better recommendations.

---

# Relationship with Rendering

Rendering provides presentation context.

Context may inspect presentation without becoming dependent on rendering implementation.

---

# Relationship with Suggestions

Constructed context becomes the input for AI suggestions.

Suggestions never become part of the context until the user explicitly accepts or rejects them.

Only the user's decision becomes reusable knowledge.

---

# Cost Optimization

The architecture minimizes AI cost through efficient context engineering.

Preferred order:

```text
Reuse Existing Context

↓

Update Changed Information Only

↓

Compress Context

↓

Reason Once

↓

Reuse Results
```

Repeated reasoning over identical context should be avoided whenever possible.

---

# Implementation Independence

This specification intentionally avoids defining:

* prompt templates
* token limits
* embeddings
* vector databases
* retrieval algorithms
* cache technologies
* model providers

Those belong to implementation.

The architecture defines only the principles governing context construction.

---

# Canonical Rules

The AI Context model follows these rules.

1. Context precedes reasoning.
2. Context is read-only.
3. Context is temporary.
4. Context should remain minimal.
5. Existing knowledge should be reused before generating new reasoning.
6. User decisions are the highest-value contextual signal (V2 — no suggestion history in V1; Canonical Product Data is the highest-confidence V1 source).
7. Freshness depends on platform state.
8. Compression should preserve meaning while reducing cost.
9. Context construction remains model-independent.
10. Better context is preferred over larger models.

---

# Canonical Principle

The AI Platform should spend more effort understanding the problem than generating the answer.

Well-constructed context produces better intelligence, lower cost, lower latency, and greater consistency than repeated large-model reasoning.

The platform's accumulated knowledge is the primary source of intelligence.

Language models amplify that knowledge—they do not replace it.
