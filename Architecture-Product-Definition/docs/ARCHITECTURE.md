# Minime — Architecture

## Purpose

Minime follows a deliberate architectural reading order.

Every engineer, reviewer, and AI working with this repository must understand the architecture before proposing changes to it.

Reading order is mandatory. Understanding architecture precedes implementation.

---

## Mandatory Reading Order

```
Architecture Governance Canon
        ↓
Product Architecture
        ↓
Platform Architecture
        ↓
Domain READMEs
        ↓
Domain Specifications
        ↓
Implementation Planning
        ↓
Source Code
```

---

## Architectural Rules

- Do not propose architecture before understanding it.
- Do not redesign Product Architecture during implementation.
- Do not redesign Platform Architecture without governance approval.
- Read canonical documents before advisory reports.
- Architecture governs implementation.
- Audits advise. Governance decides.

---

## Repository Philosophy

Minime is built on ownership first, implementation independence, and deliberate simplicity.

Every architectural decision is evaluated by its long-term value: low operational cost, clear responsibility, and stable evolution — not by file count, framework preference, or theoretical elegance.

---

## AI Instructions

Before producing recommendations, complete the following in order:

1. Read the Governance Canon.
2. Read Product Architecture.
3. Read Platform Architecture.
4. Read the affected Domain specification.
5. Identify canonical ownership for every entity involved.
6. Verify all domain and platform boundaries.
7. Only then produce recommendations.

Skipping this process is considered an incomplete architectural review.
