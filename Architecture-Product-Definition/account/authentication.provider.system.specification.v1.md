# Authentication Provider Architecture Specification V1

**Document ID:** AUTH-PROVIDER-ARCH-V1

**Status:** Canonical

**Version:** V1

**Owner:** Authentication Domain

---

# Purpose

This document defines the canonical architecture for Authentication Providers.

Its purpose is to separate provider-specific implementation from the Authentication domain while allowing future providers to be introduced without redesigning the authentication system.

This document is the single source of truth for:

- Authentication Provider architecture
- Provider lifecycle
- Provider adapter contract
- Provider registry
- Identity normalization
- Provider extensibility
- Provider ownership boundaries

This document does not define user sessions, account lifecycle, authorization, or Connected Accounts.

---
# Authority

This document is the canonical authority for Authentication Provider architecture.

This specification defines the architecture, lifecycle, ownership boundaries, extensibility model, provider registry, provider adapter model, identity normalization, and provider relationships for all Authentication Providers.

Whenever another specification describes Authentication Provider behavior, this document takes precedence.

Other specifications may specialize provider behavior for their own domain, but they must never contradict this document.

Any architectural change affecting Authentication Providers must be introduced here first before being propagated to dependent specifications.
---

# Design Principles

The Authentication architecture must follow these principles.

## Provider Independence

The Authentication system must never be designed around a specific provider.

Google and Apple are V1 implementations.

They are not architectural concepts.

---

## Stable Core

The Authentication Core must remain unchanged when new providers are introduced.

Future provider additions must not require changes to:

- API routes
- Authentication flows
- AuthenticationIdentity schema
- Session model
- Account ownership rules
- Validation architecture
- Security architecture

---

## Provider Isolation

Every provider-specific implementation must exist behind an adapter.

The Authentication Core must never contain provider-specific business logic.

---

## Identity First

Authentication is based on provider identities.

Authentication is never based on:

- email
- username
- display name
- profile metadata

---

## Single Source of Truth

AuthenticationIdentity is the only canonical authentication identity.

Provider profile information is informational only.

---

# Architecture

```
                Client
                   │
                   ▼
        Authentication API
                   │
                   ▼
            AuthService
                   │
                   ▼
    AuthenticationProviderRegistry
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
 Google Adapter        Apple Adapter
        │                     │
        ▼                     ▼
 Google OAuth          Apple OAuth
        │                     │
        └──────────┬──────────┘
                   ▼
        Normalized AuthenticationIdentity
                   │
                   ▼
              Session Creation
```

---

# Provider Registry

The Authentication Core owns a Provider Registry.

The Provider Registry is responsible only for resolving provider implementations.

It must never implement provider logic.

Responsibilities:

- register adapters
- resolve adapters
- expose supported providers
- validate provider support

Conceptually:

```text
register(adapter)

resolve(provider)

supportedProviders()

isSupported(provider)
```

The registry must not contain Google-specific logic.

The registry must not contain Apple-specific logic.

Future providers must register themselves without changing the registry implementation.

---

# Authentication Provider

Authentication Providers represent external identity systems.

Examples include:

- Google
- Apple
- Microsoft
- GitHub
- Facebook
- Instagram
- LinkedIn
- X
- TikTok

Only Google and Apple are enabled in V1.

Other providers are architecture examples only.

Adding a provider requires:

1. implementing an adapter
2. registering the adapter
3. enabling the provider in configuration

No architectural redesign is permitted.

---

# Provider Adapter

Every Authentication Provider must implement the same contract.

Conceptually:

```ts
AuthenticationProviderAdapter

provider()

buildAuthorizationRequest()

exchangeAuthorizationCode()

validateIdentity()

normalizeIdentity()
```

The adapter owns all provider-specific behavior.

Examples include:

- OAuth endpoints
- OpenID Connect details
- JWT validation
- provider claims
- provider-specific transport
- provider secrets
- provider metadata normalization

No provider-specific logic may exist inside AuthService.

---

# Authentication Flow

The Authentication Core executes one generic flow.

```
Client

↓

Provider selected

↓

Resolve Provider Adapter

↓

Build Authorization Request

↓

External Authentication

↓

Receive Authorization Result

↓

Validate Request

↓

Exchange Authorization Code

↓

Validate Provider Identity

↓

Normalize Identity

↓

Find AuthenticationIdentity

↓

Create Session
```

The flow never changes based on provider.

Only the adapter implementation changes.

---

# Identity Normalization

Every provider returns different claims.

The adapter must normalize those claims into one canonical model.

Canonical AuthenticationIdentity:

```
auth_identity_id

account_id

provider

provider_subject

email

email_verified

provider_profile

created_at

updated_at
```

No provider may extend this schema.

Provider-specific information belongs only inside:

```
provider_profile
```

---

# Provider Profile

Provider Profile stores optional metadata returned by the provider.

Examples:

- display name
- avatar
- locale
- preferred language
- normalized claims

Provider Profile is never authoritative.

It must never be used for:

- ownership
- authorization
- identity matching
- uniqueness
- account merging

Profile Content remains user-owned.

---

# Identity Ownership

Authentication ownership is defined only by:

```
provider

+

provider_subject
```

Email never defines ownership.

Display name never defines ownership.

Provider Profile never defines ownership.

---

# Identity Merge Policy

Authentication identities never merge automatically.

Rules:

If an AuthenticationIdentity exists:

→ authenticate into the owning account.

If it does not exist:

→ continue registration.

Email equality never merges accounts.

Provider Profile never merges accounts.

Automatic account merging is prohibited.

Manual account merging is outside V1.

---

# Provider Linking

Linking requires:

- authenticated session
- explicit user action
- successful provider verification

Linking never occurs automatically.

Linking never depends on email.

Linking never depends on matching names.

The linked provider becomes another AuthenticationIdentity belonging to the same Account.

---

# Primary Provider

An Account may expose a preferred Authentication Provider.

Primary Provider exists only for user experience.

It must never affect:

- ownership
- authorization
- sessions
- permissions
- identity

Removing or changing the Primary Provider must never affect authentication.

---

# Security Ownership

The Authentication Core owns:

- flow orchestration
- validation rules
- session creation
- ownership rules
- identity rules

Each Provider Adapter owns:

- OAuth implementation
- OpenID Connect implementation
- provider endpoints
- provider secrets
- provider claims
- provider token validation
- provider transport differences

---

# Provider Secrets

Provider secrets belong only to adapters.

Examples:

Google

- Client ID
- Client Secret

Apple

- Services ID
- Team ID
- Key ID
- Private Key

Secrets must never be exposed outside backend services.

---

# API Philosophy

Authentication APIs are provider-agnostic.

Routes never contain provider names.

Provider selection is request data.

This guarantees route stability as providers evolve.

---

# Configuration

Supported providers are configuration.

They are not architectural structure.

V1 configuration:

```
google

apple
```

Future versions may enable additional providers without redesigning Authentication.

---

# Extensibility

Introducing a new provider must require only:

1. Implement Adapter

2. Register Adapter

3. Enable Configuration

Nothing else.

Specifically, adding a provider must not require changes to:

- AuthService
- Session Service
- AuthenticationIdentity
- Authentication APIs
- Validation
- Authorization
- Security Model
- Account Model

---

# Relationship with Social Accounts

Authentication Providers and Social Accounts are independent concepts.

Authentication verifies identity.

Social Accounts describe public presence.

The same platform may support both capabilities.

Example:

Instagram Authentication

and

Instagram Connected Account

are different systems.

One must never imply the other.

Authentication never creates Connected Accounts.

Connected Accounts never authenticate users.

Both domains remain completely independent.

---

# V1 Constraints

Supported Authentication Providers:

- Google
- Apple

Unsupported in V1:

- Facebook
- Instagram
- Microsoft
- GitHub
- LinkedIn
- X
- TikTok

These providers are architecture extension points only.

---

# Canonical Rules

The following rules are absolute.

- Authentication is provider-based.
- Authentication is provider-independent.
- Provider-specific logic exists only inside adapters.
- Authentication APIs are provider-agnostic.
- AuthenticationIdentity is the canonical identity.
- Email is never an identity key.
- Email never merges accounts.
- Provider Profile is informational only.
- Authentication never depends on display name.
- Provider registration extends configuration, not architecture.
- Authentication Providers and Social Accounts are permanently separated.
- Adding a provider must never require redesign of the Authentication Core.