# Authentication Policy V1

## Status

Approved

---

# Purpose

This document defines the authentication and verification rules used by Minime Account Claim System V1.

Authentication is responsible for verifying that a user controls the identity used during account creation.

Successful authentication converts a Username Reservation Record into an Approved User Account.

---

# Scope

This policy applies to:

* Account registration
* Account verification
* Initial account ownership
* Account login
* Identity verification

This policy does not define:

* Account security upgrades
* Two-factor authentication
* Recovery procedures
* Advanced account linking
* Risk analysis
* Fraud detection

Those systems may be introduced in future versions.

---

# Core Principles

## Provider-Based Authentication Only

Minime V1 authentication is provider-based only.

Minime never authenticates users directly.

Minime accepts successful authentication assertions from supported external identity providers only.

---

## Verification Before Account Creation

An account must never be created before successful authentication.

The sequence is:

```text
Username Reservation
↓
Provider Authentication
↓
Approved User Account
```

---

## One Verified Identity Required

Only one verified provider identity is required to create an account.

Either of the following is sufficient:

```text
Verified Google Identity
```

or

```text
Verified Apple Identity
```

---

# Supported Authentication Methods

## External Identity Providers

Minime V1 supports authentication exclusively through trusted external identity providers.

V1 supported providers:

```text
Google
Apple
```

Both providers are first-class and equal. Neither provider is preferred or ranked above the other.

---

## Google Sign-In

Users may register or sign in using Google Sign-In.

Successful Google authentication is considered verified immediately.

No additional verification step is required.

**Google Provider Adapter requirements** (see "Provider Adapter Architecture" below for the general adapter contract):

* The Google adapter must validate the `id_token` server-side, never trusting a client-supplied claim.
* The verified `sub` claim must be normalized into `provider_subject`.
* The verified `email` claim, if present, may be normalized into optional provider metadata only.
* Google `email` must never be the canonical identity key. Identity is always `(provider, provider_subject)`.

---

## Apple Sign-In

Users may register or sign in using Apple Sign-In.

Successful Apple authentication is considered verified immediately.

No additional verification step is required.

**Apple Provider Adapter requirements** (see "Provider Adapter Architecture" below for the general adapter contract):

* Apple authentication requires an Apple Developer Program account for production use.
* Apple web authentication uses a Services ID as the OAuth `client_id`.
* The Apple `client_secret` is a JWT generated server-side, signed with the Apple private key (`ES256`); it is never a static secret. This secret is adapter-owned configuration — see "Provider Adapter Architecture" and `08-security-model.md`.
* The Apple private key must never be exposed to the frontend and must never leave the backend execution environment.
* Apple may return `name` and `email` only on the first authorization for a given user; the adapter must capture this data at that moment (as optional provider metadata), because it will not be returned again on subsequent sign-ins.
* Apple may return a private relay email address. A relay email is valid optional provider metadata only — it is never treated differently from a direct email for storage purposes, and never used for identity matching.
* The verified `sub` claim must be normalized into `provider_subject`.
* The system must never depend on Apple `email` for identity matching, account lookup, or duplicate detection. Identity is always `(provider, provider_subject)`.
* Apple's `response_mode=form_post` callback delivery is an Apple-adapter-specific transport detail (see "Provider Adapter Architecture" — Callback Transport Normalization). It never changes the core, provider-agnostic callback contract.

---

# Core Authentication Flow

The authentication flow is provider-agnostic. `provider` is request data passed into the flow, never part of the API route structure, and the flow never branches on `provider` outside of adapter dispatch.

```text
provider (request data)
↓
resolve Provider Adapter for provider
↓
build authorization URL (adapter)
↓
validate callback state (core, provider-agnostic)
↓
exchange authorization code server-side (adapter)
↓
validate provider token/assertion (adapter)
↓
normalize identity (adapter)
↓
create / link / find AuthenticationIdentity (core)
↓
create session, or issue registration handoff (core)
```

Rules that apply to every provider's flow, regardless of which provider:

* The flow must use HTTPS only.
* The flow must use an unguessable, single-use `state` parameter, validated by the backend on callback, to protect against CSRF.
* The flow must use a `nonce`, bound into the requested `id_token` and validated by the backend, to protect against replay.
* The callback must validate that the `provider` supplied in the callback request data matches the provider bound to the stored `state` context at flow-start time. A mismatch must be rejected before any token exchange occurs.
* The callback must reject any `provider` value outside the V1 allowlist (`google`, `apple`) before resolving an adapter or making any provider call.
* The authorization code must be exchanged for tokens by the backend only, inside the resolved adapter. The frontend never performs token exchange and never sees a client secret.
* The returned `id_token` (or equivalent provider assertion) must be validated server-side, inside the resolved adapter: signature (against the provider's published keys), issuer, audience, and expiry must all be checked.
* A token that fails any of these checks must be rejected; the request must not proceed.
* `provider_subject` is extracted only after successful validation, via the adapter's identity normalization step.
* Duplicate identity detection (see "Identity Merge Policy") executes immediately after `provider_subject` extraction, before any Account is created, linked, or session is issued.
* Provider access tokens and refresh tokens are not persisted unless a future version introduces a capability that explicitly requires them and the security model is updated accordingly. V1 has no such capability.

This document defines the policy. Implementation-level method and endpoint contracts are defined in `implementation/04-service-contracts.md`, `implementation/05-api-contracts.md`, and `implementation/08-security-model.md`.

---

# Provider Adapter Architecture

Authentication provider handling is modeled through a Provider Adapter concept. Provider-specific logic belongs only inside provider adapters; core authentication services contain no Google-specific or Apple-specific branching beyond resolving which adapter to use for a given `provider` value.

Conceptually, every provider adapter implements:

```text
AuthenticationProviderAdapter = {
  provider                    — the AuthenticationProvider value this adapter handles
  buildAuthorizationUrl(...)  — constructs the provider's authorization URL from state, nonce, and redirect target
  exchangeCodeForTokens(...)  — performs the backend-to-provider authorization-code exchange
  validateIdToken(...)        — validates the provider's id_token / assertion (signature, issuer, audience, expiry)
  normalizeIdentity(...)      — maps validated provider data into a provider-agnostic identity shape
}
```

The full implementation-level type contract is defined in `implementation/04-service-contracts.md`.

## Adapter Boundary Rules

* Core authentication services (`AuthService`) call adapters only through this contract. They never branch on `provider` for any reason other than selecting which adapter to invoke.
* Each adapter owns its own secret configuration (client ID, client secret or signing key, scopes, endpoints). Secrets never cross adapter boundaries and never reach the frontend.
* `normalizeIdentity` is the only place provider-specific response shapes are translated into the common `NormalizedAuthenticationIdentity` shape (`provider`, `provider_subject`, `email`, `email_verified`, `provider_profile`). Nothing outside the adapter ever reads a provider's raw response shape.
* Adding a new provider in a future version means adding a new adapter and adding that provider's value to the allowlist. It must never require changes to API route structure, core service method signatures, or the `AuthenticationIdentity` data model shape.

## Callback Transport Normalization

Providers may differ in how they deliver the OAuth callback at the transport level (for example, Apple's `response_mode=form_post` delivers the callback as a form-encoded POST rather than a query-string redirect). This is an adapter/transport-layer concern only. Regardless of transport, every provider's callback is normalized into the same provider-agnostic `{provider, code, state}` input before it reaches the core authentication flow. The core flow, and the public API contract, never differ by provider.

## V1 Allowlist

V1 supports exactly two configured adapters:

```text
google
apple
```

No other `provider` value is accepted in V1. A request naming any other provider value must be rejected before any adapter is resolved.

## Future Extensibility (Not Enabled in V1)

The adapter model exists specifically so that future providers can be added without changing the core authentication architecture, the data model shape, the service contracts, or the API route structure — only by adding a new adapter and an allowlist entry.

Examples of providers that may be added in a future version, **none of which are enabled, configured, or supported in V1**:

```text
Instagram
Facebook
Microsoft
GitHub
LinkedIn
X
TikTok
```

These are listed only to illustrate the extension point. Naming them here does not authorize, schedule, or imply their implementation in V1.

---

# Unsupported Authentication Methods

The following methods are not supported in V1:

```text
Email OTP
Phone Number Signup
Phone OTP
SMS Verification
WhatsApp Verification
Password Authentication
Facebook Login
X Login
LinkedIn Login
```

These methods are outside the scope of V1.

---

# Authentication Identity

Each account must have at least one authentication identity.

Authentication identities are external provider identities, not email addresses or passwords.

## Identity Fields

Each authentication identity record contains:

```text
auth_identity_id  — permanent internal identifier for this identity record
account_id        — the owning Account
provider          — the identity provider: "google" or "apple" in V1
provider_subject  — the stable unique identifier issued by the provider (Google sub or Apple sub); immutable
email             — the email address reported by the provider, if any; optional; informational only; not the uniqueness key
email_verified    — whether the provider reported the email as verified; informational only
provider_profile  — generic, optional, non-authoritative provider-returned metadata (e.g. display name, avatar URL, locale, or normalized raw claims); never used for ownership, authorization, uniqueness, or identity matching
created_at        — record creation timestamp
updated_at        — record update timestamp
```

There is no `display_name` field on this entity. Provider-reported display name, if any, lives inside `provider_profile` as one possible key among others — it is never promoted to a dedicated top-level field. `provider_profile` never replaces Profile Content: Profile Content remains user-owned and authoritative for everything shown on the public profile.

## Uniqueness Rule

Authentication identities are unique by `(provider, provider_subject)`. `provider_subject` is the only authentication identity key.

`email` is informational only. It is optional, it is never unique, and it is never used as the login identity or the uniqueness key. It may change or be absent over the lifetime of a provider identity. `email` must never be used for automatic account merge — see "Identity Merge Policy" below.

---

# Account Uniqueness

Authentication identities may not be duplicated across accounts.

```text
Google Account A (sub: google_sub_123)
```

cannot create multiple Minime accounts.

```text
Apple Account A (sub: apple_sub_456)
```

cannot create multiple Minime accounts.

---

# Identity Merge Policy

This is the canonical V1 policy governing how authenticated identities relate to Accounts.

## Core Rule

`provider_subject` is the only authentication identity key. `email` must never be used for automatic account merge.

## Authentication Outcomes

* If a logged-out user authenticates with a `(provider, provider_subject)` that already exists, the system logs into the owning Account.
* If a logged-out user authenticates with a `(provider, provider_subject)` that does not exist, the system continues registration (or creates a new Account) according to the Account Claim flow.
* If the same email appears from a different provider than one already on file, the system must not automatically merge the two into one Account.
* If Google and Apple return the same email, they still remain separate identities unless explicitly linked by a logged-in Account owner. Email match alone never proves account ownership.

## Linking Requires an Authenticated Owner

If a user wants to add another provider to the same Account, they must already be logged in and must explicitly start provider linking. Linking is never inferred, suggested-and-auto-applied, or triggered by an unauthenticated flow.

Linking requires successful OAuth verification for the new provider identity — the same backend token exchange and `id_token` validation used for sign-in.

Duplicate provider identity linking must be rejected if that `(provider, provider_subject)` is already linked to another Account.

## Email Is Informational Only

Email match must never prove account ownership and must never be used as an automatic-merge signal. Email match may be shown to the user as purely informational (e.g. "this Google account uses the same email as your linked Apple account") only if a future product decision chooses to surface it — it must never drive an automatic action.

## No Merge in V1

In V1, no automatic account merge exists. In V1, no manual account merge exists, since none is documented elsewhere in the frozen architecture. A future version may introduce explicit, user-initiated account merge as a distinct, separately specified capability; it does not exist today.

---

# Provider Linking and Unlinking

A signed-in user may link an additional supported provider to their Account, and may unlink a provider, subject to the following rules. See "Identity Merge Policy" above for the rules governing when linking is and is not permitted.

## Minimum Requirement

Every Account must have at least one authentication provider linked at all times.

The last remaining authentication provider must never be removed. Unlinking must be rejected if it would leave the Account with zero linked providers.

## Linking

Linking requires the user to complete a full authentication flow with the new provider while signed in. The resulting `(provider, provider_subject)` pair is checked for duplicates exactly as during registration: if it is already bound to a different Account, linking must be rejected.

Linking never replaces an existing identity. An Account may have one Google identity and one Apple identity linked simultaneously, and, in a future version, identities from additional adapters without any change to this rule.

## Unlinking

Unlinking removes the authentication identity record identified by `auth_identity_id`. Unlinking never deletes the Account, never deletes Product Data, and never affects any other linked identity.

## Primary Provider

If both Google and Apple are linked to the same Account, either may be designated `primary_provider` — a UI/default-display preference only.

* `primary_provider` must never determine ownership.
* `primary_provider` must never determine authorization.
* Ownership and authorization are always determined by `account_id` + the linked `AuthenticationIdentity` binding (`provider` + `provider_subject`), never by which provider is marked primary.
* Removing or changing `primary_provider` must never affect authentication validity for any linked identity.
* The last remaining linked provider can never be removed, regardless of whether it is marked primary.

---

# Authentication Result

Authentication may return:

## success

Authentication completed successfully.

---

## failed

Authentication failed.

---

## expired

Authentication token expired.

---

## blocked

Authentication request blocked by system protections.

The `blocked` result is distinct from `failed`. A `failed` result indicates an incorrect credential. A `blocked` result indicates that the authentication request cannot proceed regardless of credential correctness.

The `blocked` state is triggered when the number of authentication attempts for a given identity exceeds the permitted threshold within a defined evaluation window. The Authentication Policy defines that a threshold and evaluation window must exist; their specific values are supplied by implementation configuration.

A blocked authentication context expires automatically. Re-authentication may be attempted only after the blocking period expires.

The Authentication Policy defines that the blocking period must not be indefinite and that every blocked state must have a defined expiry. The specific expiry duration is supplied by implementation configuration.

The authentication system owns enforcement of the blocked state. No Product Domain may override or bypass a blocked authentication result.

---

# Reservation Relationship

Authentication operates on an existing Username Reservation Record.

Authentication never creates usernames.

Authentication never selects usernames.

Authentication only verifies identity ownership.

---

# Successful Registration Flow

## Google Flow

```text
Username Available
↓
Create Reservation
↓
Google Sign-In
↓
Create Approved User Account
↓
Create Public Profile
```

---

## Apple Flow

```text
Username Available
↓
Create Reservation
↓
Apple Sign-In
↓
Create Approved User Account
↓
Create Public Profile
```

---

# Failed Registration Flow

```text
Username Available
↓
Create Reservation
↓
Authentication Not Completed
↓
Reservation Expires
↓
Username Returns To Available
```

---

# Relationship with Social Accounts

Authentication providers are not Social Accounts. Social Accounts are not Authentication providers. This boundary holds for V1's providers (Google, Apple) and for every future adapter equally.

* Social Accounts are user-declared identifiers (e.g. an Instagram handle) and are never authentication providers, regardless of whether that same platform is ever added as a future Authentication Provider Adapter.
* Linking a Google or Apple authentication identity never verifies, implies, or grants ownership of any social account.
* If a future version adds an Authentication Provider Adapter for a platform that is also a Social Accounts platform (e.g. Instagram, Facebook), that addition must never imply, by itself: social account ownership verification, Connected Account creation, profile link creation, or social account import. Those remain entirely separate flows unless a future architecture version explicitly and separately connects them.

---

# Future Versions

Future versions may introduce:

* Phone verification
* SMS OTP
* WhatsApp verification
* Multi-factor authentication
* Account recovery
* Security levels
* Device trust systems
* Explicit, user-initiated account merge (distinct from the automatic merge that is permanently excluded — see "Identity Merge Policy")
* Additional identity provider adapters (e.g. Instagram, Facebook, Microsoft, GitHub, LinkedIn, X, TikTok), added via the Provider Adapter Architecture without changing core routes, service contracts, or the data model shape

These features are outside the scope of V1. Provider linking/unlinking between Google and Apple is already a V1 capability (see "Provider Linking and Unlinking") — it is not future scope.
