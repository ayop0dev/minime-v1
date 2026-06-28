# Events Platform Service

The Events Platform Service defines how Minime records, describes, and retains system and product facts over time.

Events are used to describe that something happened.

Events do not own business logic, do not replace Product Data, and do not become the source of truth for any Product Domain.

---

## Purpose

The purpose of the Events Platform Service is to provide a shared architectural foundation for recording meaningful facts across Minime.

Events help the platform understand:

* when something happened
* which owner it belongs to
* which Product Domain produced it
* which entity it relates to
* how long it may be retained
* whether it may be used by Analytics, AI, or operational systems

Events exist to support observation, analysis, automation, debugging, and future intelligence.

They do not define product behavior.

---

## Platform Role

The Events Platform Service is a shared Platform Service.

It supports Product Domains, but it does not own Product Domain meaning.

Product Domains decide:

* what actions are meaningful
* when an event should be emitted
* what the event represents
* which entity the event relates to

The Events Platform defines:

* the structure of an event
* the lifecycle of an event
* delivery expectations
* retention principles
* event boundaries
* platform-level guarantees

---

## Architectural Position

The Events Platform sits beside other Platform Services:

```text
platform/
│
├── data/
├── storage/
├── events/
└── ai/
```

The Events Platform depends on Product Domains for business meaning.

It may reference Data entities, Storage asset references, Rendering activity, Analytics activity, and AI activity, but it does not own any of them.

---

## Core Principles

The Events Platform follows these principles:

1. Events describe facts.
2. Events do not execute business logic.
3. Events are not canonical Product Data.
4. Events never replace user-approved Data.
5. Events are emitted by Product Domains or Platform Services.
6. Events should be append-only whenever possible.
7. Events should be immutable after creation.
8. Events must have a clear owner.
9. Events must have a clear source.
10. Events may support Analytics, AI, and operational observability.

---

## Source of Truth

The user remains the single source of truth for Minime.

Events may describe user actions, system actions, or platform observations, but they do not override user-controlled Product Data.

For example:

* if a user updates a bio, Product Data owns the new bio
* an event may record that the bio was updated
* the event does not become the bio
* the event cannot replace the canonical bio value

Events describe change.

Data owns state.

---

## Relationship with Data

Data owns canonical structured records.

Events may reference Data records.

Events may describe changes to Data records.

Events may support historical analysis of Data changes.

However, Events do not own canonical Data.

An event may say:

```text
button_link_updated
```

But the Button Product Domain still owns the button link.

---

## Relationship with Storage

Storage owns binary assets.

Events may describe Storage-related facts, such as:

* asset uploaded
* asset replaced
* asset deleted
* asset processing completed
* orphan cleanup completed

Events may reference asset identifiers or structured asset references.

Events must not contain binary asset payloads.

---

## Relationship with Rendering

Rendering produces public-facing read models and output views.

Events may describe Rendering-related facts, such as:

* profile render requested
* render object generated
* render cache invalidated
* public page viewed

Rendering output does not become canonical because of an event.

Events may help explain rendering activity, but they do not own rendered state.

---

## Relationship with Analytics

Analytics may consume Events.

Events may become one source of Analytics input.

Analytics may aggregate Events into reports, counters, trends, or insights.

However:

* Events are raw facts
* Analytics are interpreted measurements
* Analytics must not mutate Product Data directly
* Analytics must not become the source of truth

---

## Relationship with AI

AI may consume Events to understand behavior, context, and usage patterns.

AI may produce suggestions based on Events.

However:

* AI Suggestions are not canonical
* Events do not approve AI Suggestions
* AI must not mutate Product Data without user approval
* Events may explain AI activity, but they do not authorize AI decisions

---

## What Counts as an Event

An Event is a recorded fact that something meaningful happened in the system.

Examples:

```text
account_created
profile_published
button_created
button_reordered
asset_uploaded
asset_replaced
public_profile_viewed
out_link_clicked
ai_suggestion_generated
ai_suggestion_approved
render_object_generated
```

Events should describe completed facts, not intentions.

Preferred:

```text
profile_published
```

Avoid:

```text
publish_profile_requested
```

unless the request itself is important as an operational fact.

---

## What Does Not Count as an Event

The following are not Events:

* canonical Product Data
* binary assets
* temporary UI state
* drafts that are not saved
* business rules
* validation logic
* permissions
* rendering templates
* AI prompts
* analytics reports
* background jobs themselves

Events may describe these things happening, but they do not become those things.

---

## Immutability

Events should be treated as immutable once created.

If a fact changes later, a new event should be emitted.

An existing event should not be edited to represent a different fact.

This protects historical accuracy and makes Events useful for debugging, analytics, and future intelligence.

---

## Ownership

Every Event must have a clear ownership context.

For V1, Events should usually be owned by an Account.

Where relevant, an Event may also reference:

* user actor
* account owner
* product domain
* source entity
* target entity
* platform service
* public visitor context

Ownership does not mean the Event is Product Data.

Ownership means the Event belongs to a clear account or operational context.

---

## Event Categories

The Events Platform may support several categories of events:

```text
Product Events
Platform Events
Analytics Events
Rendering Events
Storage Events
AI Events
Operational Events
```

These categories exist for clarity.

They do not create separate business ownership models.

---

## V1 Scope

The V1 Events Platform defines architecture only.

It does not require a specific implementation technology.

The implementation may use:

* database tables
* queues
* logs
* event streams
* background workers
* analytics pipelines

The architecture should remain valid regardless of implementation choice.

---

## V1 Non-Goals

The Events Platform does not define:

* event-sourcing as the Product Data model
* complex distributed messaging
* cross-service orchestration
* business workflow engines
* permanent audit logs for every field
* real-time infrastructure requirements
* vendor-specific event technology
* AI automation approval rules

Minime V1 favors simple, understandable event recording over complex event-driven architecture.

---

## Files in This Directory

This directory contains the Events Platform Service specifications:

```text
platform/events/
│
├── README.md
├── events.architecture.specification.v1.md
├── event.lifecycle.specification.v1.md
├── event.delivery.specification.v1.md
└── event.retention.policy.v1.md
```

---

## Related Platform Services

Events must remain consistent with:

```text
platform/platform.architecture.specification.v1.md
platform/data/
platform/storage/
platform/ai/
```

Events depend on Data and Product Domains for meaning.

Events may support AI, Analytics, Rendering, and operations.

Events do not replace any of them.

---

## Canonical Rule

Events describe what happened.

Data owns what is true now.

Storage owns binary assets.

Rendering owns generated public views.

AI owns suggestions and interpretation.

Product Domains own business meaning.

The user remains the source of truth.
