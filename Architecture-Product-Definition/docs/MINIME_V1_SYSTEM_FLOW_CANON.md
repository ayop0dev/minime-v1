# Minime V1 — System Flow Canon

**Status:** Canonical
**Version:** V1
**Repository State:** Canonical Repository State V1.0
**Type:** Responsibility Handoff Map (not a runtime pipeline)

---

## 1. Purpose

This file exists to keep the parts of Minime V1 from blurring into each other.

As the repository grows, it becomes easy to confuse who owns what: a Product Domain might be mistaken for a storage engine, Rendering might be asked to make product decisions, Public Profile might be treated as the owner of profile data, Analytics might be expected to change behavior, or AI might drift from suggesting into deciding.

This canon prevents that confusion. It states, in plain terms, the main moving parts of Minime V1, what each part owns, and how responsibility passes cleanly from one part to the next — so that every future specification has a shared, simple mental model to build on.

---

## 2. What This File Is

* It is a **system flow canon**.
* It is a **responsibility handoff map** — it shows where one part's job ends and the next part's job begins.
* It explains how the **already-approved** parts of Minime V1 work together.
* It **prepares** the repository for the minimal Platform specifications (Persistence, Storage, Events, AI Suggestions) without writing them yet.

---

## 3. What This File Is Not

* It is **not** a redesign of any Product Domain.
* It is **not** backend implementation planning.
* It is **not** a technology selection document — no databases, vendors, queues, or frameworks are chosen here.
* It is **not** a large, layered system blueprint.
* It **does not** add new Product Domains.

---

## 4. The Simple System Flow

```text
User
↓
Product Domains
↓
Platform Capabilities
↓
Renderer
↓
Public Profile
↓
Events
↓
Analytics / Feedback
```

Read this top to bottom as **responsibility flowing forward**, not as a mandatory sequence that every single action must run through.

Most real operations touch only part of this flow. Editing a bio touches the User, a Product Domain, and Platform Persistence — and stops there. A visitor opening a profile touches Rendering, Public Profile, and may produce an Event. The diagram describes **how responsibility is arranged**, so each part knows where its job ends. It is a map of ownership, not a runtime script.

---

## 5. Main Responsibility Map

| Part | Owns | Does Not Own |
|---|---|---|
| User | Truth, approval, decisions | System automation |
| Product Domains | Product meaning and rules | Storage engines, databases, infrastructure |
| Platform Capabilities | Persistence, storage, events, AI suggestion support | Product decisions |
| Renderer | Read-safe output for visitors | Editing, ownership, analytics decisions |
| Public Profile | Serving the visitor experience | Source data mutation |
| Events | What happened | Reports or product decisions |
| Analytics | Reading and reporting events | Changing product behavior |
| AI Suggestions | Suggestions and explanations | Truth, publishing, silent edits |

---

## 6. Product Domains

Product Domains define **what Minime means as a product**. They own the business meaning, the rules, the validation, and the user-facing concepts. When a question is "what should happen, and is it allowed?", the answer belongs to a Product Domain.

The stable V1 Product Domains include:

* Account
* Authentication
* Username
* Social Accounts
* Connected Accounts
* Profile
* Rendering
* Public Profile
* Out Links
* Analytics

The Account domain owns account management, settings, and the QR code entry point. The Profile domain owns the profile fields, blocks, and design (appearance) as one business concept.

Product Domains own meaning. They **do not** own binary storage, persistence lifecycle, event transport, or AI decision-making. When a domain needs to keep a record, store a file, emit an event, or request a suggestion, it hands that responsibility down to a Platform Capability. The domain still decides *what* the data means; it just does not run the machinery that keeps or moves it.

---

## 7. Platform Capabilities

Platform is **not** a large, layered system. For V1 it means only a small set of shared capabilities that multiple Product Domains already need — proven by the existing repository, not added on speculation.

The approved V1 Platform Capabilities are exactly four:

```text
Persistence
Storage
Events
AI Suggestions
```

In plain language:

* **Persistence** keeps structured records and metadata — the durable representation of what the user provided or confirmed.
* **Storage** keeps binary assets and the media that is served to visitors (avatars, images, generated assets).
* **Events** carry a description of what happened, from the systems that produce them to the systems that read them.
* **AI Suggestions** supports generating suggestions and managing their lifecycle, so domains can offer help without owning that machinery.

Platform keeps, moves, and supports data. It **never** makes a product decision. It does not decide what a profile means, what is valid, or what becomes public — those are Product Domain decisions.

Do **not** add Cache, Search, Queues, Notifications, Feature Flags, Monitoring, or a generic Security service as V1 Platform Capabilities. If any of these is needed at all in V1, it remains an implementation detail beneath these four capabilities — not a new architectural layer.

---

## 8. Rendering

Rendering prepares the **visitor-safe** version of the profile — what a visitor is allowed to see, in the shape they can see it.

* It **consumes** prepared product and platform data.
* It **must not** mutate source data.
* It **must not** reach into raw domain internals in a way that violates the Render Object canon. It works from prepared, read-safe values, not from raw records, raw theme data, or database rows.
* It **must not** become a Product Domain owner. Rendering presents; it does not decide product meaning.

Rendering is the point where owned, approved data turns into something a visitor can view — and nothing more.

---

## 9. Public Profile Serving

Public Profile **serves the rendered experience** to visitors.

It should be fast, safe, cache-aware, and — from the visitor's perspective — read-only. A visitor consumes the profile; a visitor never edits it.

Public Profile **must not** become the owner of profile content, account data, styling rules, storage, or analytics interpretation. It is the delivery surface, not the source. The source of truth stays with the user and the Product Domains; Public Profile only delivers the prepared result.

---

## 10. Events and Analytics

* **Public Profile and Out Links may emit events** — for example, a profile view or a link click.
* **Events describe what happened.** They are factual records of an occurrence, nothing more.
* **Analytics consumes events.** Analytics is downstream; it reads events that source systems produced.
* **Analytics reports observations.** It aggregates and presents what the events show.
* **Analytics must not control** rendering, user decisions, AI decisions, or product behavior in V1. It observes and reports; it never steers.

Events flow one way: from the systems that produce them, into Analytics, and out as reports. Analytics reading an event never loops back to change how the product behaves.

---

## 11. AI Suggestions

AI in Minime V1 is a **suggestion layer, not a decision layer**.

AI may:

* suggest usernames,
* suggest profile improvements,
* suggest bio improvements,
* suggest link improvements,
* assist with onboarding,

— all within approved V1 scope, and always as suggestions the user can accept, edit, or reject.

AI must **not**:

* publish anything,
* silently edit user content,
* confirm external accounts as true,
* scrape, or reintroduce the retired Discovery Engine in any form,
* override the user's styling or profile decisions.

**User approval is required before any AI output becomes user-owned content.** Until the user approves, a suggestion is just a suggestion. This keeps the user as the source of truth and keeps AI firmly in a helping role.

---

## 12. Responsibility Handoff Rules

```text
User input becomes Product Domain data only after validation.
Product Domain data is persisted through Platform Persistence.
Uploaded files go through Platform Storage.
Visitor-safe output is prepared by Rendering.
Visitor actions may become Events.
Analytics reads Events.
AI Suggestions may read allowed context and suggest improvements, but user approval is required.
```

Each line is a clean handoff: one part finishes its job and passes a well-defined result to the next. No part reaches across the flow to do another part's job.

---

## 13. Simplicity Rules

* Do not introduce a new layer unless **multiple domains already need it**.
* Do not create a Platform Capability for a **single** use case.
* Do not convert implementation details into architecture.
* Do not introduce vendors, technologies, queues, workers, or infrastructure choices.
* Prefer **one clear handoff** over many abstract layers.
* Keep V1 small.

---

## 14. Explicit Non-Goals

This canon does **not**:

* redesign any Product Domain,
* add new user-facing features,
* make backend technology decisions,
* introduce a large, multi-layered system map,
* bring back the Discovery Engine in any form,
* draw a generic microservice map,
* add an advanced automation layer,
* grant AI any autonomy,
* expand V1 scope in any way.

---

## 15. Final Canon Statement

Minime V1 is made of **user-approved product data**, supported by **minimal platform capabilities**, prepared by **rendering**, served through the **public profile**, observed through **events and analytics**, and improved only through **user-approved suggestions**.

```text
Domains decide what the product means.
Platform keeps, moves, and supports the data.
Renderer prepares what visitors can see.
Public Profile serves the rendered experience.
Events describe what happened.
Analytics reads events but does not control behavior.
AI suggests but never decides.
The user remains the source of truth.
```
