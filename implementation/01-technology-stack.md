# Minime V1 — Technology Stack

## Status

Approved.

---

# Purpose

This document defines the canonical technology stack for Minime V1.

Every technology listed in this document is approved for implementation.

Implementation must follow these decisions.

Implementation must not introduce additional frameworks, runtimes, databases, or infrastructure without an approved implementation specification.

This document does not redefine architecture.

This document defines how the frozen architecture is implemented.

---

# Engineering Decision

## Runtime

### Decision

Node.js LTS.

### Purpose

Provide the runtime environment for all backend services.

### Implementation Rules

* All backend execution must use the current Node.js LTS release.
* All production environments must use the same major Node.js version.
* Development and CI environments must match the production major version.

### Do

* Use Node.js LTS.
* Keep all environments synchronized.
* Upgrade only after validation.

### Don't

* Do not use non-LTS releases.
* Do not run different major versions across environments.
* Do not depend on experimental runtime features.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Programming Language

### Decision

TypeScript.

### Purpose

Provide a single programming language across the repository.

### Implementation Rules

* All application code must use TypeScript.
* Shared packages must use TypeScript.
* Backend modules must use TypeScript.
* Frontend modules must use TypeScript.
* Shared contracts must use TypeScript.

### Do

* Enable strict type checking.
* Export explicit types.
* Keep public interfaces strongly typed.

### Don't

* Do not write production JavaScript files.
* Do not disable strict typing globally.
* Do not use the `any` type unless no alternative exists and the usage is documented.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Package Manager

### Decision

pnpm.

### Purpose

Provide deterministic dependency management across the monorepo.

### Implementation Rules

* All dependency management must use pnpm.
* Workspace configuration must use pnpm workspaces.
* Lock files must be committed.

### Do

* Use a single lock file.
* Install dependencies through pnpm.
* Keep workspace packages synchronized.

### Don't

* Do not use npm.
* Do not use Yarn.
* Do not commit multiple lock files.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Repository Model

### Decision

Turborepo Monorepo.

### Purpose

Provide a single repository for all production applications and shared packages.

### Implementation Rules

* The repository must use Turborepo.
* Applications must be placed under `apps/`.
* Shared packages must be placed under `packages/`.
* Shared implementation must exist only when used by multiple applications.

### Do

* Share contracts through packages.
* Share validation through packages.
* Keep application ownership independent.

### Don't

* Do not duplicate shared code.
* Do not create packages for speculative reuse.
* Do not introduce multiple repositories for V1.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Frontend Framework

### Decision

Next.js.

### Purpose

Implement the public experience and authenticated dashboard.

### Implementation Rules

* The frontend application must use Next.js.
* The App Router must be used.
* Server Components may be used where appropriate.
* Client Components must only be used when required.

### Do

* Use Next.js routing.
* Keep rendering responsibilities inside the frontend.
* Keep business decisions in backend services.

### Don't

* Do not move business logic into frontend components.
* Do not access the database from the frontend.
* Do not duplicate backend ownership rules.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Backend Framework

### Decision

NestJS.

### Purpose

Implement all backend business services.

### Implementation Rules

* All backend modules must be implemented using NestJS.
* Product Domains must be implemented as NestJS modules.
* Platform Services must be implemented inside the NestJS application.

### Do

* Use dependency injection.
* Keep controllers thin.
* Keep business logic inside services.

### Don't

* Do not implement standalone HTTP servers.
* Do not place business logic inside controllers.
* Do not bypass dependency injection.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## HTTP Engine

### Decision

Fastify Adapter.

### Purpose

Provide HTTP request processing for the backend application.

### Implementation Rules

* NestJS must use the Fastify adapter.
* All HTTP endpoints must execute through Fastify.

### Do

* Use Fastify for HTTP execution.
* Keep adapter configuration centralized.

### Don't

* Do not use the Express adapter.
* Do not run multiple HTTP engines.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## API Architecture

### Decision

REST.

### Purpose

Provide the canonical communication protocol between frontend and backend.

### Implementation Rules

* All public application APIs must use REST.
* All endpoints must use HTTPS.
* JSON must be used for request and response bodies unless binary transfer is required.
* API versioning must be implemented through URL versioning.

### Do

* Use resource-oriented endpoints.
* Use standard HTTP methods.
* Use standard HTTP status codes.
* Keep request and response contracts explicit.

### Don't

* Do not implement GraphQL.
* Do not expose RPC-style endpoints.
* Do not return inconsistent response structures.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Database

### Decision

PostgreSQL.

### Purpose

Provide persistent storage for all business data.

### Implementation Rules

* PostgreSQL is the only relational database for Minime V1.
* All Product Domain business data must be stored in PostgreSQL.
* Database schema changes must use migrations.
* Database access must occur through repositories.

### Do

* Normalize schemas unless a documented implementation decision requires otherwise.
* Use transactions for multi-step writes.
* Define indexes for production queries.

### Don't

* Do not introduce additional relational databases.
* Do not access PostgreSQL directly from controllers.
* Do not place business logic inside SQL migrations.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## ORM

### Decision

Prisma.

### Purpose

Provide database access for the backend application.

### Implementation Rules

* Prisma is the only approved ORM.
* Prisma schema is the canonical database schema definition.
* All migrations must be generated through Prisma.

### Do

* Use Prisma Client.
* Use typed queries.
* Keep database access inside repositories.

### Don't

* Do not mix multiple ORMs.
* Do not bypass Prisma for business persistence.
* Do not duplicate schema definitions.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Cache

### Decision

Redis.

### Purpose

Provide distributed caching and shared runtime state.

### Implementation Rules

Redis must be used only for:

* response caching
* background job queues
* rate limiting
* distributed locks
* temporary application state

Redis must not become the source of truth.

Persistent business data must remain in PostgreSQL.

### Do

* Define expiration for cached values.
* Invalidate cache after business data changes.
* Keep cache disposable.

### Don't

* Do not store canonical business data.
* Do not depend on cache persistence.
* Do not implement business ownership in Redis.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Object Storage

### Decision

S3 Compatible Object Storage.

### Purpose

Store user-uploaded files and generated assets.

### Implementation Rules

Object Storage must store:

* profile images
* uploaded media
* generated files
* future static assets

Business metadata must remain in PostgreSQL.

### Do

* Store object metadata inside PostgreSQL.
* Generate unique object keys.
* Support signed upload and download URLs.

### Don't

* Do not store uploaded files inside PostgreSQL.
* Do not expose storage provider implementation.
* Do not use local disk as permanent storage.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Authentication

### Decision

JWT Access Tokens with Refresh Tokens.

### Purpose

Authenticate users and authorize API access.

### Implementation Rules

* Access tokens must be short-lived.
* Refresh tokens must be revocable.
* Authentication must execute in the backend.
* Authorization must execute before business operations.

### Do

* Validate every protected request.
* Rotate refresh tokens.
* Revoke compromised sessions.

### Don't

* Do not trust frontend authentication state.
* Do not store secrets in client code.
* Do not bypass authentication middleware.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Validation

### Decision

Zod.

### Purpose

Provide runtime input validation.

### Implementation Rules

* Shared validation schemas must use Zod.
* Backend validation is the final authority.
* Frontend validation exists only for user experience.

### Do

* Validate every external input.
* Reuse shared validation schemas.
* Reject invalid requests before business execution.

### Don't

* Do not trust client input.
* Do not duplicate shared validation rules.
* Do not replace authorization with validation.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Background Jobs

### Decision

BullMQ.

### Purpose

Execute asynchronous background work.

### Implementation Rules

BullMQ must be used for:

* media processing
* scheduled jobs
* retryable tasks
* long-running operations

Background jobs must execute outside the HTTP request lifecycle.

### Do

* Queue non-immediate work.
* Configure retry policies.
* Record job failures.
* Make jobs idempotent.

### Don't

* Do not execute long-running work inside HTTP requests.
* Do not depend on job execution order unless explicitly required.
* Do not store business data inside job queues.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Event Bus

### Decision

NestJS EventEmitter.

### Purpose

Provide internal application event communication.

### Implementation Rules

* Internal application events must use NestJS EventEmitter.
* Events must remain inside the backend application.
* Events must not cross process boundaries.

### Do

* Publish events after successful business operations.
* Keep event payloads immutable.
* Keep event handlers independent.

### Don't

* Do not use events as the source of truth.
* Do not depend on handler execution order.
* Do not implement distributed messaging in V1.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Logging

### Decision

Pino.

### Purpose

Provide structured application logging.

### Implementation Rules

* All application logs must use Pino.
* Logs must be structured JSON.
* Every request must have a request identifier.

### Do

* Log unexpected failures.
* Log application startup.
* Log background job failures.
* Log security events.

### Don't

* Do not log passwords.
* Do not log access tokens.
* Do not log refresh tokens.
* Do not log secrets.
* Do not log sensitive personal information.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Testing

### Decision

Vitest and Playwright.

### Purpose

Verify implementation correctness.

### Implementation Rules

* Unit tests must use Vitest.
* End-to-end tests must use Playwright.
* Critical business flows must have automated tests.

### Do

* Test business services.
* Test repositories.
* Test API contracts.
* Test authentication flows.
* Test rendering flows.

### Don't

* Do not depend on manual verification.
* Do not merge failing tests.
* Do not skip critical production scenarios.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Reverse Proxy

### Decision

Nginx.

### Purpose

Handle inbound HTTP traffic.

### Implementation Rules

Nginx must manage:

* TLS termination
* request forwarding
* compression
* static asset delivery
* security headers

### Do

* Enable HTTP compression.
* Forward requests to the backend.
* Configure security headers.

### Don't

* Do not expose backend services directly.
* Do not terminate TLS inside the application.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Deployment

### Decision

Docker.

### Purpose

Provide a consistent runtime environment.

### Implementation Rules

* Every production service must run inside a Docker container.
* Development and production environments must use the same container definitions.

### Do

* Build immutable images.
* Use multi-stage builds.
* Keep container images minimal.

### Don't

* Do not deploy directly from source code.
* Do not modify running containers manually.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Continuous Integration

### Decision

GitHub Actions.

### Purpose

Automate validation before deployment.

### Implementation Rules

Every pull request must execute:

* dependency installation
* type checking
* linting
* unit tests
* build validation

Deployment workflows must execute only after successful validation.

### Do

* Fail fast.
* Keep workflows reproducible.
* Validate every merge.

### Don't

* Do not bypass automated validation.
* Do not deploy failing builds.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Environment Configuration

### Decision

Environment Variables.

### Purpose

Provide runtime configuration.

### Implementation Rules

* Runtime configuration must use environment variables.
* Every required variable must exist in `.env.example`.
* Missing required variables must prevent application startup.

### Do

* Validate environment variables during startup.
* Keep secrets outside the repository.

### Don't

* Do not hardcode secrets.
* Do not commit production credentials.
* Do not access undefined environment variables.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## Versioning

### Decision

Semantic Versioning.

### Purpose

Provide consistent application versioning.

### Implementation Rules

Application releases must follow Semantic Versioning.

Version format:

```text
MAJOR.MINOR.PATCH
```

### Do

* Increment MAJOR for incompatible changes.
* Increment MINOR for backward-compatible features.
* Increment PATCH for backward-compatible fixes.

### Don't

* Do not change released versions.
* Do not publish unversioned releases.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Engineering Decision

## AI Provider

### Decision

Configurable AI provider via environment variables. V1 implementation defaults are defined in `.env.example`.

### Purpose

Execute on-demand profile Analysis Sessions when the user explicitly requests profile analysis ("Analyze My Profile").

### Implementation Rules

* Provider is selected via `AI_PROVIDER` environment variable. No specific vendor is architecturally required.
* Model is configured via `AI_MODEL_ID` environment variable. No specific model is architecturally required.
* API key is configured via `AI_API_KEY` environment variable.
* If `AI_API_KEY` is absent: AI is disabled; `AIService.analyzeProfile` returns empty suggestions.
* AI calls must time out after `AI_TIMEOUT_MS` milliseconds (default: 10000).
* Analysis Session results are cached by input hash for `AI_CACHE_TTL_SECONDS` (default: 300). Identical context within TTL returns cached result without inference.
* Prompt templates live in `packages/config/ai-prompts.ts` — not in the database.
* Replacing the provider requires only a compatible adapter implementation and env var update. No domain service code changes.
* V1 AI capability is limited to "Analyze My Profile" on-demand Analysis Sessions. Username suggestions, social setup suggestions, and onboarding guidance are not V1 AI capabilities.

### Do

* Return empty suggestions on any AI failure; never throw to callers.
* Configure everything through environment variables.
* Keep prompt templates in application code.

### Don't

* Do not store AI responses in the database.
* Do not apply AI suggestions without explicit user confirmation.
* Do not block product domain operations on AI failure.
* Do not introduce multi-provider routing in V1.
* Do not invoke `AIService` from background jobs, event handlers, or scheduled tasks.

### Decision Authority

Minime V1 Canonical Engineering Decision.

### Status

Approved.

---

# Final Technology Stack

| Category        | Technology                    |
| --------------- | ----------------------------- |
| Runtime         | Node.js LTS                   |
| Language        | TypeScript                    |
| Package Manager | pnpm                          |
| Monorepo        | Turborepo                     |
| Frontend        | Next.js                       |
| Backend         | NestJS                        |
| HTTP Engine     | Fastify Adapter               |
| API             | REST                          |
| Database        | PostgreSQL                    |
| ORM             | Prisma                        |
| Cache           | Redis                         |
| Object Storage  | S3 Compatible                 |
| Authentication  | JWT + Refresh Tokens          |
| Validation      | Zod                           |
| Background Jobs | BullMQ                        |
| Event Bus       | NestJS EventEmitter           |
| AI Provider     | Configurable via env vars; V1 defaults in `.env.example`     |
| Logging         | Pino                          |
| Testing         | Vitest + Playwright           |
| Reverse Proxy   | Nginx                         |
| Deployment      | Docker                        |
| CI              | GitHub Actions                |
| Configuration   | Environment Variables         |
| Versioning      | Semantic Versioning           |

---

# Implementation Rule

All implementation work must use the technology stack defined in this document.

No technology may be replaced or supplemented unless a newer approved implementation specification explicitly supersedes this document.
