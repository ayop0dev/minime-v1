# architecture.status.field.specification.v1.md

# Architecture Status Field Specification V1

**Version:** 1.0
**Status:** Canonical
**Domain:** Governance

---

## Architecture Status Field

### Purpose

The optional `Architecture Status` field may appear in selected architectural specifications.

It is **not** a document approval status.

It is **not** a lifecycle state.

It is **not** part of the authority hierarchy.

Instead, it indicates whether the architectural concept described by that specification belongs to the frozen architectural baseline of the current major version.

---

## Meaning

When present:

```text
Architecture Status: Frozen
```

means:

* the architectural concept has completed architecture review;
* its architectural responsibilities are considered stable for the current major version;
* implementation may proceed without reopening architectural design;
* future modifications require an architecture revision rather than ordinary specification editing.

This field does **not** change document authority.

It does **not** replace the normal document status.

It does **not** indicate implementation completion.

---

## Relationship to Status

The repository distinguishes two independent concepts.

### Document Status

Example:

```text
Status: Approved
```

This describes the approval state of the document itself.

---

### Architecture Status

Example:

```text
Architecture Status: Frozen
```

This describes the stability of the architectural concept documented by the specification.

The two fields serve different purposes and shall never be interpreted as equivalent.

---

## Usage Rules

`Architecture Status` is optional.

When used, it shall appear only for specifications that describe architectural concepts explicitly covered by the Repository Freeze Boundary.

It shall never appear:

* on policies;
* on implementation planning documents;
* on reports;
* on audit documents;
* on review documents;
* on execution reports;
* on temporary specifications.

---

## Authority

The Architecture Governance Canon remains the sole authority defining when architecture becomes frozen.

The `Architecture Status` field is informational only.

It never grants authority.

It never changes governance.

It never overrides canonical architecture documents.
