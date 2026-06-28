# Minime Platform Storage

The Storage Platform Service manages the complete lifecycle of binary assets in Minime V1.

Storage is responsible for binary assets only.

It does not own structured information.

It does not own Product Domain behavior.

It does not own rendering.

Its responsibility begins when a binary asset is created and ends when that asset is safely removed.

---

# Purpose

The purpose of the Storage Platform Service is to provide one shared system for managing binary assets across every Product Domain.

Storage ensures that assets are:

* safely managed
* efficiently stored
* reusable
* inexpensive to operate
* easy to deliver
* easy to replace
* easy to remove

Storage exists to support the product.

It is not the product.

---

# Responsibilities

The Storage Platform Service is responsible for:

* binary asset lifecycle
* asset validation
* asset processing
* persistent storage
* asset replacement
* asset delivery preparation
* orphan cleanup
* storage efficiency
* storage cost optimization

The Storage Platform Service is **not** responsible for:

* structured data
* Product Domain logic
* rendering
* analytics
* AI behavior
* public profile behavior
* business decisions

---

# Core Philosophy

Storage owns binary assets.

Data owns structured references.

Product Domains decide why an asset exists.

Storage decides how that asset is managed.

Rendering consumes prepared assets.

Public Profile delivers them.

Each architectural area owns exactly one responsibility.

---

# Document Reading Order

Read the Storage documents in the following order.

---

## 1. Storage Architecture

```text
storage.architecture.specification.v1.md
```

Defines the purpose, responsibilities, and boundaries of the Storage Platform Service.

Read this first.

---

## 2. Asset Lifecycle

```text
asset.lifecycle.specification.v1.md
```

Defines how binary assets move from creation to permanent removal.

---

## 3. Storage Cost Optimization

```text
storage.cost.optimization.specification.v1.md
```

Defines the architectural principles that minimize storage cost while preserving simplicity and scalability.

---

## 4. Asset Delivery

```text
asset.delivery.specification.v1.md
```

Defines the architectural contract between Storage and every consumer of binary assets.

---

# Relationships

The Storage Platform Service cooperates with the rest of the Platform.

```text
Product Domains

↓

Data

↓

Storage

↓

Rendering

↓

Public Profile
```

Data owns structured references.

Storage owns binary assets.

Rendering consumes binary assets.

Public Profile delivers visitor experiences.

Events observe Storage activity.

AI may consume approved assets where Product Domain rules allow.

---

# Design Principles

The Storage Platform Service follows these principles.

* Binary assets only.
* One asset, one lifecycle.
* References before copies.
* Replace instead of mutate.
* Process once, reuse many times.
* Remove orphaned assets.
* Keep Product Domains storage-independent.
* Prefer architectural simplicity over infrastructure complexity.
* Optimize storage before scaling infrastructure.

---

# Non-Goals

The Storage Platform Service intentionally avoids defining:

* storage providers
* cloud vendors
* CDN providers
* filesystem layouts
* upload APIs
* image processing libraries
* deployment architecture
* infrastructure decisions

Those belong to implementation.

---

# Summary

The Storage Platform Service is the binary asset manager of Minime.

It guarantees that every binary asset follows a predictable lifecycle, remains independent from Product Domains, and is delivered efficiently without exposing storage implementation.

This separation keeps Minime simple, maintainable, scalable, and cost-efficient regardless of future storage technology.
