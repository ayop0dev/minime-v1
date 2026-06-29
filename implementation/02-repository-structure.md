# Minime V1 — Repository Structure

## Status

Implementation specification.

This document defines the canonical repository structure for Minime V1 implementation.

The architecture is already frozen under:

```text
v1.0-architecture
```

This document does not redesign the architecture.

It translates the frozen architecture into a practical code repository layout.

---

# 1. Repository Model

Minime V1 uses a monorepo.

```text
Monorepo: Turborepo
Package Manager: pnpm
Language: TypeScript
```

The monorepo exists to keep the frontend, backend, shared contracts, validation, and tooling in one controlled implementation workspace.

The monorepo must not be used to introduce unnecessary abstraction or premature microservices.

---

# 2. Top-Level Structure

```text
minime-v1/
│
├── apps/
│   ├── web/
│   └── api/
│
├── packages/
│   ├── shared/
│   ├── validation/
│   ├── config/
│   └── ui/
│
├── implementation/
│
├── docs/
│
├── scripts/
│
├── package.json
├── pnpm-workspace.yaml
├── turbo.json
├── tsconfig.base.json
├── .env.example
├── .gitignore
└── README.md
```

---

# 3. Apps

## 3.1 `apps/web`

The `web` app contains the Next.js frontend.

Responsibilities:

* authenticated dashboard UI
* public profile rendering UI
* onboarding UI
* profile editing UI
* analytics UI
* account/settings UI

The web app may call API endpoints.

The web app must not own business meaning.

The web app must not directly access the database.

The web app must not contain duplicated backend validation rules where shared validation exists.

---

## 3.2 `apps/api`

The `api` app contains the NestJS backend.

Responsibilities:

* authentication
* account management
* username management
* profile management
* social accounts
* connected accounts
* rendering data APIs
* out link redirects
* analytics ingestion
* storage coordination
* event publishing
* AI request orchestration

The API app owns server-side execution.

The API app may access the database through repositories.

The API app may use Redis, object storage, background jobs, and internal events.

---

# 4. Packages

Packages contain shared implementation assets.

Packages must remain small.

A package must only exist when at least two apps or modules need the same code.

Do not create packages for speculative future reuse.

---

## 4.1 `packages/shared`

Shared TypeScript types and non-business utilities.

Allowed:

* shared DTO types
* shared constants
* simple utility functions
* shared enums
* shared response shapes

Not allowed:

* database access
* API clients with hidden side effects
* business services
* domain ownership logic
* framework-specific code

---

## 4.2 `packages/validation`

Shared validation schemas.

Primary tool:

```text
Zod
```

Responsibilities:

* request validation schemas
* shared DTO validation
* input constraints shared by frontend and backend
* form validation rules where appropriate

Rules:

* Backend remains the final validation authority.
* Frontend validation is only a user experience layer.
* Shared validation must not replace permission checks.
* Shared validation must not contain database queries.

---

## 4.3 `packages/config`

Shared static configuration.

Allowed:

* TypeScript config helpers
* environment variable schema definitions
* shared constants
* app-level config types

Not allowed:

* secrets
* runtime credentials
* user-specific config
* business configuration owned by Product Domains

---

## 4.4 `packages/ui`

Shared UI components used by the web app.

Allowed:

* buttons
* inputs
* layout primitives
* dashboard components
* shared design tokens

Not allowed:

* data fetching logic
* business decisions
* authentication decisions
* direct API mutation logic

---

# 5. API App Structure

```text
apps/api/src/
│
├── main.ts
├── app.module.ts
│
├── common/
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   ├── pipes/
│   ├── errors/
│   └── utils/
│
├── config/
│
├── database/
│   ├── prisma/
│   ├── migrations/
│   └── seed/
│
├── platform/
│   ├── data/
│   ├── storage/
│   ├── events/
│   └── ai/
│
├── modules/
│   ├── account/
│   ├── auth/
│   ├── username/
│   ├── social-accounts/
│   ├── connected-accounts/
│   ├── profile/
│   ├── rendering/
│   ├── out-links/
│   └── analytics/
│
├── jobs/
│
└── tests/
```

---

# 6. Product Module Structure

Each Product Domain inside `apps/api/src/modules/` should follow the same structure where useful.

Example:

```text
apps/api/src/modules/profile/
│
├── profile.module.ts
├── profile.controller.ts
├── profile.service.ts
│
├── dto/
├── validation/
├── repositories/
├── policies/
├── events/
├── errors/
└── tests/
```

Rules:

* Controllers expose HTTP endpoints.
* Services coordinate use cases.
* Repositories handle persistence.
* Policies handle authorization and permission decisions.
* DTOs define input/output shapes.
* Validation defines module-specific validation.
* Events define events emitted by the module.
* Errors define module-specific domain errors.

Do not create empty folders.

Only create folders when the module actually needs them.

---

# 7. Platform Service Structure

Platform Services live under:

```text
apps/api/src/platform/
```

Canonical Platform Services:

```text
data
storage
events
ai
```

No additional Platform Services may be added for V1 unless implementation reveals a genuine architectural gap.

Each Platform Service may contain:

```text
contracts/
services/
adapters/
errors/
tests/
```

Rules:

* Platform Services provide technical capability.
* Platform Services do not own business meaning.
* Platform Services do not decide Product Domain behavior.
* Product Domains call Platform Services when they need shared technical capability.

---

# 8. Web App Structure

```text
apps/web/src/
│
├── app/
│
├── components/
│
├── features/
│   ├── account/
│   ├── auth/
│   ├── username/
│   ├── social-accounts/
│   ├── connected-accounts/
│   ├── profile/
│   ├── rendering/
│   ├── out-links/
│   └── analytics/
│
├── lib/
├── hooks/
├── styles/
└── tests/
```

Rules:

* `app/` owns Next.js routing.
* `features/` mirrors Product Domains where useful.
* `components/` contains reusable UI components.
* `lib/` contains frontend-only utilities.
* Frontend code must not duplicate backend ownership logic.
* Frontend code must not access the database.

---

# 9. Import Boundaries

Allowed:

```text
apps/web        → packages/*
apps/api        → packages/*
apps/api        → apps/api/src/platform/*
apps/api/modules → apps/api/platform/*
```

Not allowed:

```text
apps/web → apps/api/src/*
apps/api → apps/web/src/*
packages → apps/*
packages/shared → packages/ui
packages/config → apps/*
```

Rules:

* Apps may depend on packages.
* Packages must not depend on apps.
* Backend may depend on Platform Services.
* Platform Services must not depend on Product Domain modules.
* Product Domains may depend on Platform Services.
* Product Domains must not directly depend on unrelated Product Domains unless explicitly required by an implementation contract.

---

# 10. Naming Rules

Use kebab-case for folders:

```text
social-accounts
connected-accounts
out-links
```

Use PascalCase for classes:

```text
ProfileService
UsernameRepository
AnalyticsController
```

Use camelCase for variables and functions:

```text
createProfile
updateUsername
recordOutLinkClick
```

Use UPPER_SNAKE_CASE for environment variables:

```text
DATABASE_URL
REDIS_URL
JWT_ACCESS_SECRET
S3_BUCKET_NAME
```

---

# 11. Environment Files

Required:

```text
.env.example
```

Local files:

```text
.env
.env.local
```

Rules:

* `.env.example` must document every required variable.
* Real secrets must never be committed.
* Environment validation must happen at application startup.
* Missing required variables must fail fast.

---

# 12. Testing Structure

Tests should live close to the code they verify when practical.

Allowed:

```text
module/tests/
*.spec.ts
*.e2e-spec.ts
```

Testing levels:

* unit tests for pure logic
* service tests for use cases
* repository tests for persistence behavior
* API tests for endpoint contracts
* e2e tests for critical user flows

Critical V1 flows must have tests before production release.

---

# 13. Scripts

Repository-level scripts live in:

```text
scripts/
```

Allowed:

* setup scripts
* database reset scripts
* seed scripts
* deployment helpers
* maintenance helpers

Scripts must be explicit and safe.

Dangerous scripts must require clear confirmation or environment gating.

---

# 14. Documentation Placement

Implementation documentation lives in:

```text
implementation/
```

Architecture documentation remains separate and frozen.

Do not create new architecture documents during implementation unless a real architectural gap is discovered.

---

# 15. Repository Rules

The repository must stay boring, explicit, and easy to navigate.

Rules:

* Do not create empty abstraction layers.
* Do not create packages before they are needed.
* Do not introduce microservices in V1.
* Do not hide business behavior in shared utilities.
* Do not duplicate validation between frontend and backend when shared validation exists.
* Do not allow Platform Services to own Product Domain meaning.
* Do not allow frontend code to become the source of truth.
* Do not add new top-level folders without a clear implementation reason.

---

# 16. Final Decision

Minime V1 uses a Turborepo monorepo with two primary apps:

```text
apps/web
apps/api
```

Shared code lives in small, explicit packages:

```text
packages/shared
packages/validation
packages/config
packages/ui
```

The backend implementation mirrors the frozen Product Domains and Platform Services.

The repository structure is approved for V1 implementation.
