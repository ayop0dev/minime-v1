# Google Tag Manager Provider Specification V1

## Purpose

This specification defines how Minime integrates with Google Tag Manager (GTM).

The objective is to allow users to connect their public profiles to Google's tagging ecosystem without exposing the platform to arbitrary code execution or unnecessary configuration complexity.

Minime integrates with GTM.

It does not manage GTM.

---

# Provider Identity

Provider Name:

```text
Google Tag Manager
```

Provider Type:

```text
Tracking Container
```

Supported Version:

```text
V1
```

---

# Responsibilities

The Google Tag Manager provider owns:

* GTM container configuration
* container validation
* provider lifecycle
* provider loading requirements

The provider does not own:

* tags
* triggers
* variables
* pixels
* analytics configuration
* marketing logic

Those remain entirely inside the user's GTM workspace.

---

# Configuration Model

V1 supports a single configuration value:

```text
Google Tag Manager Container ID
```

Example:

```text
GTM-XXXXXXX
```

No additional configuration is required.

---

# Source of Truth

The GTM Container ID entered by the user is the only source of truth for this provider.

Minime never stores or manages:

* GTM workspaces
* GTM tags
* GTM triggers
* GTM variables
* GTM templates
* GTM environments

Google Tag Manager remains the owner of those resources.

---

# Validation

Before a provider becomes active, the Container ID must pass basic validation.

Validation should verify:

* expected identifier format
* supported provider type
* non-empty value

Validation must remain lightweight.

The platform should not attempt to inspect or authenticate the user's GTM account.

---

# Loading Eligibility

The provider may load only when:

* a valid Container ID exists
* the profile is publicly rendered
* integrations are enabled

If any required condition fails, the provider must remain inactive.

---

# Public Profiles Only

The GTM provider should operate only on public pages.

Administrative interfaces and authenticated management areas must remain unaffected unless explicitly supported in future versions.

---

# Failure Behavior

Invalid or unavailable GTM configurations must never interrupt profile rendering.

If the provider cannot load:

* the profile remains available
* SEO remains unaffected
* Analytics remains operational
* publishing remains operational

Provider failures must remain isolated.

---

# Security Principles

**Honest threat model:** A validated GTM Container ID is not, by itself, a script-injection-safe input. Once a container is configured, the account owner's own GTM workspace — which Minime explicitly does not manage or inspect (see "Responsibilities") — can add any tag, including a Custom HTML tag containing arbitrary JavaScript. Format-validating the Container ID does not and cannot prevent this; claiming otherwise would be inaccurate. The actual security boundary is containment, defined below, not input validation.

**Containment boundary:** The GTM snippet is never injected directly into the public profile document's own DOM context. It is loaded inside a sandboxed `<iframe>` with `sandbox="allow-scripts"` only — `allow-same-origin`, `allow-top-navigation`, and `allow-popups` are never granted. A sandboxed iframe without `allow-same-origin` executes in a unique, opaque origin: scripts running inside it cannot read or write the parent page's DOM, cannot access the parent page's (or any Minime origin's) cookies or storage, and cannot navigate the visitor away from the page. Whatever the account owner's GTM workspace does, it is confined to that opaque origin. This is the mechanism that satisfies "Minime never executes arbitrary user code against its own origin or user sessions" — GTM tags may still execute arbitrary JavaScript, but only inside a context with no access to Minime data.

The provider must never accept, from the user, anything other than the Container ID string itself (no raw JavaScript, HTML, CSS, script URLs, or inline scripts as configuration inputs) — this remains true and is enforced at the settings-write boundary. It is distinct from, and does not substitute for, the iframe containment boundary above.

---

# Privacy

The GTM provider must never expose:

* internal identifiers
* unpublished information
* private profile data
* administrative metadata

The provider only exposes information already available through the public page.

---

# Ownership Boundaries

Google Tag Manager owns:

* tracking implementation
* tags
* triggers
* variables
* external integrations

Minime owns:

* provider configuration
* loading lifecycle
* provider validation
* provider security

Ownership must never overlap.

---

# Future Compatibility

Future provider capabilities may include:

* multiple GTM containers
* environment selection
* provider health diagnostics

These capabilities are intentionally excluded from V1.

---

# V1 Constraints

V1 intentionally excludes:

* custom scripts
* custom HTML
* custom JavaScript
* GTM API integration
* workspace synchronization
* tag management
* trigger management
* variable management
* provider analytics
* provider debugging

The provider should remain intentionally simple.

---

# Success Criteria

This specification is considered successful when:

* Users can connect a valid GTM container in seconds.
* Minime never executes arbitrary user code.
* GTM remains the owner of all tracking logic.
* Invalid configurations never affect profile availability.
* The provider remains lightweight, secure, and independent from the core platform.
