# ADR-0031: REST Representation Contracts and API Evolution

## Status
Proposed

## Date
2026-10-09

## Context

FlossWare uses REST/HTTP for synchronous service contracts ([ADR-0010](ADR-0010-rest-service-boundaries.md)). Some endpoints encode a version in their path (for example, `/api/v1/...`) while relying on generic JSON media types for representations.

URL versioning is simple and operationally visible, but it can conflate resource identity with a particular contract revision. Media types provide an HTTP-native way to identify representations and negotiate supported formats. Neither mechanism alone defines resource semantics, backward compatibility, or a migration policy.

FlossWare needs a common policy that supports independently implemented services and clients without requiring every repository to invent its own API evolution strategy.

## Scope

This ADR applies to FlossWare-owned REST/HTTP APIs and their clients, including APIs implemented in different languages or frameworks. It covers resource URLs, representation media types, compatibility, error representations, and migration of existing endpoints.

## Non-goals

- It does not require every capability or repository to expose HTTP.
- It does not require all existing `/v1/` paths to be removed.
- It does not require an immediate migration or a breaking change to existing clients.
- It does not mandate one REST framework, programming language, or service topology.
- It does not claim that media types alone make an API RESTful.
- It does not define AI-domain semantics; those remain with Loom and loom-ai where applicable.

## Decision

### Contract and resource identity

- REST APIs SHALL define and document their externally observable contract, including resource semantics, request/response schemas, status codes, and compatibility expectations.
- Resource URLs SHOULD identify resources or resource collections rather than encode representation revisions when stable resource identity is practical.
- URL versioning MAY be retained when it provides a useful operational or compatibility boundary. Existing versioned URLs SHALL NOT be removed solely to conform to a stylistic preference.
- Repository release versions, resource identifiers, representation-contract versions, and implementation versions SHALL remain distinct.

### Media types and negotiation

- APIs SHOULD use standard media types where they adequately express the contract, such as `application/json` for generic JSON or `application/problem+json` for RFC 9457 problem details.
- APIs with a distinct, durable domain representation SHOULD consider a documented vendor media type (for example, `application/vnd.flossware.<contract>+json`) when it provides meaningful contract identification.
- Clients and servers SHOULD use `Accept` and `Content-Type` consistently for representation negotiation and request-body format declaration.
- A server SHALL NOT silently return a representation incompatible with the client's declared supported media type.
- A server SHOULD return `406 Not Acceptable` when it cannot provide an acceptable response representation, and `415 Unsupported Media Type` when it cannot process the request representation.
- Media-type parameters MAY identify a contract version when their semantics and negotiation behavior are documented. FlossWare APIs SHALL NOT introduce arbitrary version parameters without specifying compatibility and fallback behavior.
- A new media type or media-type version SHOULD be introduced for materially incompatible representation semantics, not for every additive field or implementation release.

### Compatibility and evolution

- Additive changes SHOULD preserve existing client behavior. The compatibility policy MUST account for strict schema validators and other clients for which an apparently additive change can still be incompatible.
- A client requesting a specific contract version SHALL receive that contract's documented semantics or an explicit negotiation error; the server SHALL NOT silently reinterpret the request under an incompatible contract.
- Breaking contract changes SHALL be documented with migration guidance and a compatibility/deprecation policy when practical.
- Existing API versions SHOULD remain supported for a documented transition period when consumers depend on them. Removal requires an explicit migration/deprecation decision, not merely the existence of a newer contract.
- Contract schemas and compatibility tests SHOULD be maintained independently of framework-specific implementation details.

### Errors and discoverability

- APIs SHOULD use RFC 9457 Problem Details (`application/problem+json`) for machine-readable HTTP errors where appropriate.
- APIs SHOULD document capabilities, supported representations, authentication requirements, and error behavior.
- Hypermedia links and capability discovery MAY be used where they materially reduce client coupling. FlossWare SHALL prefer established formats and documented semantics over ad hoc conventions.

### Adoption

- Repositories exposing REST endpoints SHALL inventory their current routes, media types, schemas, clients, and compatibility expectations.
- Each affected repository SHALL track implementation work in its own issue, referencing this ADR and identifying existing-client constraints.
- Migration plans SHALL be incremental and SHALL preserve current behavior unless a separately documented breaking change is approved.
- This ADR is a proposed organization-wide standard until accepted through the repository's ADR process.

## Adoption Tracking (2026-10-09)

This section records implementation status; it does not change the decision's **Proposed** status.

### Current sequence

1. **Claude Ensemble first:** validate the representation-contract approach against a real gateway and its clients before generalizing it.
2. **Loom second:** apply the lessons to HTTP bindings only. Loom's core contracts remain transport-neutral.
3. **Other FlossWare APIs later:** inventory and adopt incrementally as they are audited or changed.

### Initial Claude Ensemble slice

- Tracking issue: [claude-ensemble#334](https://github.com/FlossWare/claude-ensemble/issues/334)
- Initial implementation PR: [claude-ensemble#335](https://github.com/FlossWare/claude-ensemble/pull/335)
- The PR introduces opt-in RFC 9457 Problem Details for gateway-generated errors when a client includes `application/problem+json` in `Accept`.
- Existing `/api/v1/` paths, success representations, and default JSON error envelopes are preserved.
- This is an initial compatibility experiment, **not** evidence that all CE gateway routes or independently hosted services conform yet. The PR must pass checks and review before merge; forwarded-response handling and the Graph/Memory service boundaries still require separate audits.

### Loom follow-on

- Tracking issue: [FlossWare/loom#25](https://github.com/FlossWare/loom/issues/25)
- The intended outcome is a language-neutral conformance specification for HTTP bindings, not HTTP-specific requirements in Loom core.

### Ecosystem inventory

- Tracking issue: [engineering-standards#50](https://github.com/FlossWare/engineering-standards/issues/50)
- The inventory is deliberately broader than the immediate implementation sequence. CE remains first; Loom follows after the CE lessons are reviewed.

## Consequences

### Positive

- Consistent contract-evolution rules across FlossWare.
- Resource identity can remain stable while representations evolve.
- Better interoperability between language-independent contracts and multiple implementations.
- Explicit handling of negotiation failures, errors, and deprecation.
- No blanket migration that needlessly breaks existing clients.

### Negative

- Vendor media types and negotiation add documentation and testing work.
- Some clients, proxies, caches, or gateways may require explicit configuration for negotiated representations.
- Existing APIs may retain URL versioning, so FlossWare will temporarily support more than one evolution pattern.
- Contract compatibility requires tests and maintenance discipline.

## Alternatives Considered

### URL versioning only

Rejected as the only organization-wide strategy. It is operationally simple and remains permitted, but it does not itself describe representation semantics or negotiation.

### Media-type versioning only

Rejected as an absolute mandate. Media types are useful for representation contracts, but URL versioning remains legitimate where operational constraints or existing clients make it preferable.

### No explicit contract evolution policy

Rejected. It leaves compatibility and migration decisions inconsistent across repositories.

### Replace every existing versioned URL immediately

Rejected. A broad path migration creates client risk without guaranteeing a corresponding architectural benefit.

## Related ADRs

- [ADR-0007](ADR-0007-unified-client-service-contract.md) — Unified Client-Service Contract
- [ADR-0010](ADR-0010-rest-service-boundaries.md) — REST Service Boundaries and Integration
- [ADR-0024](ADR-0024-contract-centric-repository-layering-and-naming.md) — Contract-Centric Repository Layering and Naming
- [ADR-0027](ADR-0027-versioning-tagging-and-releases.md) — Versioning, Tagging, and Releases

## References

- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457)
- [RFC 6906: The `profile` Link Relation Type](https://www.rfc-editor.org/rfc/rfc6906)
