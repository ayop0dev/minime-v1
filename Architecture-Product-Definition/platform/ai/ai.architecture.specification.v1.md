# AI Platform Architecture Specification V1

## Status

**Version:** V1

**Status:** Canonical

**Layer:** Platform

**Platform Service:** AI

---

# Purpose

The AI Platform provides intelligence across the Minime platform.

Its purpose is to transform existing platform knowledge into useful understanding and actionable suggestions.

The AI Platform owns no Product Data.

It owns no business logic.

It owns no canonical truth.

It exists solely to assist.

---

# Architectural Philosophy

The AI Platform follows six architectural principles.

## 1. AI Reads. AI Does Not Own.

The AI Platform may read information from across the platform.

It never becomes the owner of that information.

Ownership always remains with the originating Product Domain or Platform Service.

---

## 2. AI Suggests. The User Decides.

Every AI output is a suggestion.

Only explicit user approval may change Product Data.

Suggestions have no authority.

Users do.

---

## 3. Platform Correctness Never Depends on AI.

Every Product Domain must remain fully functional even if every AI capability disappears.

AI enhances the platform.

It never enables the platform.

---

## 4. Intelligence Is Built on Existing Knowledge.

AI should maximize reuse of existing platform knowledge before generating new reasoning.

Existing knowledge includes:

* Product Data
* historical Events
* previous user decisions
* Analytics
* Rendering results
* Storage metadata

Reasoning begins only after available knowledge has been exhausted.

---

## 5. Deterministic Logic Before AI

AI should never replace deterministic logic.

If a deterministic solution exists and produces equivalent value, it should always be preferred.

Examples include:

* validation
* sorting
* filtering
* availability checks
* formatting
* scoring using predefined rules

AI should be reserved for problems requiring interpretation, ambiguity, creativity, or semantic understanding.

---

## 6. Intelligence Must Be Economical

AI is treated as a constrained computational resource.

Every unnecessary inference increases:

* cost
* latency
* operational complexity

The architecture therefore minimizes AI usage whenever possible.

---

# Role of the AI Platform

The AI Platform may:

* understand
* classify
* summarize
* explain
* compare
* rank
* recommend
* generate
* prioritize
* reason

The AI Platform never:

* owns entities
* executes business rules
* validates Product Data
* stores canonical state
* replaces Product Domains
* overrides user intent

---

# V1 AI Execution Model

In V1, the only AI capability is "Analyze My Profile" — an on-demand Analysis Session triggered by explicit user request.

An Analysis Session:
- Is triggered only when the user explicitly requests analysis
- Builds the Analysis Input Snapshot from current account state (profile, blocks, connected accounts, analytics summary)
- Computes an Input Hash from that snapshot
- Reuses the latest completed Analysis Session for the account when its Input Hash, `analysis_version`, and `output_schema_version` all match the current configured values (Analysis Session Reuse)
- Otherwise passes the snapshot to `AIService.analyzeProfile`, which delegates to a single `Provider.execute(request)` call
- Returns profile improvement suggestions for user review
- Does not modify any business data

Analysis Session validity depends only on whether the Input Hash, `analysis_version`, and `output_schema_version` all match. Time elapsed since the last Analysis Session never determines whether a new analysis is required. Retention policies may still use time to govern how long Analysis Sessions are kept, but time never decides analysis validity.

If the Input Hash matches but `analysis_version` or `output_schema_version` differs, the existing Analysis Session must not be reused — a new Analysis Session is created. `analysis_version` and `output_schema_version` are owned by Minime, never by the AI Provider. They are not model versions and not provider versions; changing the configured Provider or Model never changes them automatically.

The following are explicitly outside V1 scope and must not be implemented:

- Username suggestions during registration
- Social account setup suggestions
- Onboarding guidance
- Background AI analysis
- AI chat or conversational flows
- Implicit AI triggers from platform events

`AIService` is the single application entry point for AI. `Provider.execute(request)` is the single execution boundary. One Provider and one Model are active at runtime — both are selected exclusively by the configured Provider; `AIService` never selects, names, or switches models or providers.

---

# Analysis Session

The Analysis Session is an official V1 platform entity — not merely a response payload. It is a persisted record of one profile analysis execution.

Conceptual structure:

```text
AnalysisSession = {
  id
  account_id
  input_hash
  analysis_version          // semantic version string, e.g. "1.0.0"
  output_schema_version     // semantic version string, e.g. "1.0.0"
  status
  provider_key              // diagnostic metadata only
  model_key                 // diagnostic metadata only
  report
  scores
  suggestions
  recommendations
  metadata
  created_at
  completed_at
}
```

Rules:

- An Analysis Session belongs to exactly one Account.
- An Analysis Session is created only after an explicit "Analyze My Profile" user request.
- An Analysis Session represents one profile analysis execution.
- A completed Analysis Session stores one completed AI analysis result.
- Multiple Analysis Sessions may exist for one Account.
- Only completed Analysis Sessions may be reused. Failed Analysis Sessions must never be reused.
- The latest completed Analysis Session whose Input Hash, `analysis_version`, and `output_schema_version` all match the current configured values (exact string match) is the reusable result. A mismatch in any one of the three means the existing Analysis Session must not be reused.
- `analysis_version` is a semantic version string (`MAJOR.MINOR.PATCH`, e.g. `"1.0.0"`) identifying the version of Minime's own analysis business logic (prompt meaning, scoring logic, recommendation logic, structured output interpretation, analysis input rules). It changes only when that business meaning changes — never when the configured Provider or Model changes.
- `output_schema_version` is a semantic version string (`MAJOR.MINOR.PATCH`, e.g. `"1.0.0"`) identifying the structured output schema Minime uses to validate and store the AI response. It changes only when the output JSON structure changes — never when the configured Provider or Model changes.
- `provider_key` and `model_key` are diagnostic metadata only — recorded for debugging, support, monitoring, cost analysis, provider performance review, and incident investigation. They must never participate in Analysis Session Reuse, analysis validity, AI output validation, or any business decision.
- A change to the configured Provider or Model never invalidates an existing Analysis Session by itself. If a Provider or Model change must invalidate prior sessions, that intent must be expressed explicitly by incrementing `analysis_version` or `output_schema_version` — never inferred from `provider_key` or `model_key`.
- Analysis Session history is not AI learning. Analysis Sessions are stored reports only. Accepted suggestions, rejected suggestions, and learned preferences remain V2 scope.

---

# Analysis Input Snapshot

The Analysis Input Snapshot is the canonical, deterministic set of account-owned data used to generate the Input Hash and the `AIProviderRequest`. It is the only approved V1 source of AI analysis input. Full field-level definition: `implementation/11-platform-services.md`.

The snapshot must include only V1-approved analysis inputs (account identity, profile content, connected accounts, blocks, appearance, analytics summary).

The snapshot must never include:

- Transient UI state
- Runtime cache data
- Sessions
- Audit logs
- Provider output
- Previous Analysis Sessions
- Accepted or rejected AI suggestions
- Learned preferences

The snapshot is generated exclusively by `AIService`. Product Domains must not provide custom ad-hoc AI context. The snapshot must be normalized before hashing so that any change to its content — and only a change to its content — produces a different Input Hash.

---

# V1 Execution Flow

```text
User clicks "Analyze My Profile"

↓

AIService.analyzeProfile(account_id)

↓

Build Analysis Input Snapshot

↓

Generate Input Hash

↓

Find latest completed Analysis Session matching account_id, input_hash, analysis_version, output_schema_version

↓

Matching session found?

↓

YES → Reuse Analysis Session

NO  → Create requested Analysis Session

      ↓

      Build AIProviderRequest

      ↓

      Provider.execute(request)

      ↓

      Configured Provider

      ↓

      Configured Model

      ↓

      Validate structured output against current output_schema_version

      ↓

      Persist completed Analysis Session

↓

Return Analysis Session
```

---

# Failure Flow

```text
Provider failure OR structured output does not conform to current output_schema_version

↓

Mark Analysis Session as failed

↓

Return safe advisory response

↓

Never modify Product Domain data
```

---

# Knowledge Hierarchy

The AI Platform consumes knowledge in the following order.

```text
Deterministic Logic

↓

Canonical Product Data

↓

Historical Events

↓

User Decisions (V2 — no suggestion history is persisted in V1)

↓

Analytics

↓

Rendering Results

↓

Storage Metadata

↓

AI Reasoning
```

The objective is to delay AI reasoning until it genuinely adds value.

---

# Sources of Knowledge

The AI Platform may consume information from:

## Product Domains

Canonical user-controlled information.

---

## Data Platform

Current truth.

---

## Events Platform

Historical facts.

---

## Storage Platform

Asset metadata.

Never binary ownership.

---

## Analytics

Aggregated measurements.

---

## Rendering

Generated public representations.

---

## User Decisions (V2)

Accepted suggestion signals and rejection signals are outside V1 scope. No suggestion history is persisted in V1.

In V1, accepted suggestions result in ordinary Product Domain data (e.g. an accepted bio becomes `ProfileContent.bio`). The AI Platform reads that data as canonical Product Data, not as a suggestion decision record.

---

# User Decisions

User decisions are one of the highest-value inputs available to AI.

A user decision represents verified human preference.

Whenever appropriate, previous decisions should be reused before generating new reasoning.

Learning from user behavior reduces future AI consumption.

---

# Accepted Decision Ownership

When a user accepts an AI suggestion, the accepted content becomes canonical data owned by the responsible Product Domain (e.g. accepting a bio suggestion writes the bio to `ProfileContent.bio`). The Product Domain service executes the write through its own repository. No separate "accepted AI decision" record exists.

**V1 scope:** There is no `AiDecision`, `AcceptedDecision`, or `SuggestionHistory` entity in V1. The canonical data model (`implementation/03-canonical-data-model.md`) contains no such entity. Accepted suggestions are persisted as ordinary Product Domain data. Rejected suggestions are discarded — no persistence occurs.

The principle of "learning from user decisions" (reusing past decisions to avoid redundant AI calls) is **outside V1 scope**. V1 does not persist suggestion history or decision signals. This capability may be introduced in a future version with the appropriate entity definition.

The AI Platform reads Product Domain data through standard domain service interfaces. The AI Platform writes nothing to any canonical data store.

---

# AI Reasoning

Reasoning should occur only when existing knowledge cannot answer the problem.

Reasoning should produce:

* explanations
* recommendations
* alternatives
* improvements
* priorities

Reasoning should never modify platform state directly.

---

# AI Outputs

Every AI output belongs to one of several conceptual categories.

Examples include:

* suggestion
* explanation
* summary
* classification
* recommendation
* ranking
* generated content

Regardless of category:

AI output remains non-canonical until explicitly accepted by the user.

---

# AI Boundaries

The AI Platform does not own:

* Product Domains
* Data
* Storage
* Events
* Analytics
* Rendering
* Authentication
* Authorization
* Validation
* Permissions

The AI Platform remains a consumer of platform knowledge.

---

# Failure Philosophy

The platform must continue operating correctly if:

* every AI provider fails
* inference becomes unavailable
* rate limits are exceeded
* external APIs are unreachable
* AI responses are rejected

AI availability must never determine platform correctness.

---

# Cost Optimization Strategy

The AI Platform minimizes cost through architecture rather than infrastructure.

Preferred order:

```text
Reuse Existing Completed Analysis Session (Input Hash, Analysis Version, and Output Schema Version Match)

↓

Reuse Previous User Decision (V2 — no suggestion history persisted in V1)

↓

Deterministic Logic

↓

Small Model

↓

Medium Model

↓

Large Model
```

Escalation occurs only when additional intelligence justifies additional cost.

In V1, cost scales with the number of explicit "Analyze My Profile" requests that produce a new Analysis Session. Each new Analysis Session executes one `Provider.execute(request)` call. Analysis Session Reuse avoids inference entirely whenever the Input Hash, `analysis_version`, and `output_schema_version` all match.

---

# Model Independence

The architecture intentionally avoids coupling to:

* OpenAI
* Anthropic
* Google
* Meta
* Mistral
* local models
* hosted models

The platform interacts with capabilities rather than vendors.

Providers may change without affecting architecture.

---

# Prompt Independence

Prompt design is considered an implementation detail.

The architecture defines:

* what AI may know
* what AI may produce
* what AI may never control

Prompt structure remains replaceable.

---

# Relationship with Data

Data owns truth.

AI reads truth.

AI never replaces truth.

---

# Relationship with Events

Events provide historical context.

AI may discover patterns across historical Events.

AI never modifies historical facts.

---

# Relationship with Storage

Storage owns binary assets.

AI may inspect asset metadata or analyze assets when requested.

Storage ownership remains unchanged.

---

# Relationship with Analytics

Analytics measures behavior.

AI explains behavior.

Analytics provides evidence.

AI provides interpretation.

---

# Relationship with Rendering

Rendering produces presentation.

AI may analyze presentation quality.

Rendering remains independent from AI reasoning.

---

# Implementation Independence

This specification intentionally avoids defining:

* prompts
* providers
* SDKs
* inference APIs
* orchestration frameworks
* model routing
* embeddings
* vector databases
* RAG pipelines
* agent frameworks

Those belong to implementation.

---

# Architectural Non-Goals

The AI Platform is not responsible for:

* replacing Product Domains
* replacing deterministic algorithms
* executing workflows
* approving changes
* becoming the source of truth
* autonomous platform control
* permanent memory ownership — the AI Platform does not own a persistent store of learned patterns or memory. Accepted user decisions are Product Domain data owned by the Data Platform, not AI memory.

Those responsibilities belong elsewhere.

---

# Canonical Rules

The AI Platform follows these rules.

1. AI reads. AI does not own.
2. AI suggests. The user decides.
3. Platform correctness never depends on AI.
4. Deterministic logic precedes AI reasoning.
5. Existing knowledge precedes new inference.
6. User decisions are reusable intelligence.
7. AI outputs remain non-canonical until approved.
8. AI consumption should be minimized through architecture.
9. AI remains provider-independent.
10. The user remains the single source of truth.
11. Analysis Session validity depends only on Input Hash, `analysis_version`, and `output_schema_version` all matching. Time never determines whether a new analysis is required.
12. The configured Provider exclusively owns model selection. AIService never selects, names, or switches models.
13. The Analysis Input Snapshot is the only approved V1 source of AI analysis input. It is generated exclusively by AIService and must be deterministic.
14. `analysis_version` and `output_schema_version` are semantic version strings owned by Minime, never by the AI Provider, and are never changed by a Provider or Model change.
15. `provider_key` and `model_key` are diagnostic metadata only. They must never participate in Analysis Session Reuse, analysis validity, or any business decision.

---

# Canonical Principle

The AI Platform exists to increase understanding while minimizing unnecessary intelligence generation.

It consumes knowledge.

It produces suggestions.

It does not persist decision history in V1.

It never owns truth.

It never owns history.

It never owns business meaning.

The user always remains the final authority.
