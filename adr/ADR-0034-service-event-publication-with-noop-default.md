# ADR-0034: Service Event Publication with a No-Op Default

## Status
Proposed

## Date
2026-10-23

## Context

FlossWare services need to support future event-driven consumers without requiring those consumers to repeatedly enumerate and query all stored data. A service should be able to announce meaningful state changes, such as a memory record being created, updated, or deleted, so downstream components can react incrementally.

This does not mean deploying a broker or requiring a standalone network messaging service now. Services SHOULD use the shared FlossWare/messaging contract for publication and envelopes rather than inventing service-specific publisher interfaces. Publication is fire-and-forget: the producer does not wait for consumers or downstream processing. The shared contract must offer a no-op implementation so services invoke publication consistently even when no transport is configured.

This decision complements ADR-0005, which describes event-driven integration and delivery guarantees, and ADR-0001, which requires cross-cutting behavior to be explicitly opted into.

## Scope

Applies to FlossWare service contracts and service implementations that create, update, delete, or otherwise change domain state. It applies across languages and protocols.

## Non-goals

- Does not require deploying or operating a messaging broker or standalone network messaging service now.
- Does not define a universal broker, wire protocol, transport, or consumer framework.
- Does not make event delivery a replacement for resource APIs, full enumeration, export, or reconciliation.
- Does not require every internal computation or read operation to emit an event.
- Does not promise durable delivery, replay, ordering, or exactly-once processing unless a concrete publisher contract explicitly provides those guarantees.

## Decision

- Every FlossWare service that changes domain state SHALL define meaningful domain events and publish them through the shared FlossWare/messaging contract after the relevant state transition succeeds.
- The shared messaging contract SHALL provide a no-op publisher. An in-process implementation MAY be used where useful; neither requires a broker or standalone network service.
- Services SHALL NOT invent incompatible local event envelopes or publisher interfaces when the shared contract can express the need. Domain code SHALL publish through the contract even when its configured implementation is no-op, and SHALL NOT contain broker-specific conditionals.
- Events SHOULD describe facts that have already occurred, using past-tense names such as `memory.record.created`, `memory.record.updated`, and `memory.record.deleted`. Event names and payloads SHALL be domain-specific, documented, and evolvable.
- Event envelopes SHOULD include a unique event ID, event type, envelope version, occurrence time, producer/service identity, aggregate or resource ID, and a schema-defined payload. They SHOULD carry enough information for consumers to identify and fetch the changed resource, while avoiding unnecessary duplication of large or sensitive content.
- Events SHALL identify the changed resource and its revision/version where the domain supports versioning. Consumers SHALL be able to distinguish repeated delivery of the same event from distinct changes.
- A successful domain operation SHALL NOT be reported as failed solely because the default no-op publisher did nothing. A real publisher's failure and the relationship between persistence and publication SHALL be documented by that service; implementations MUST NOT claim atomic persistence-and-publication unless they actually provide it.
- Where loss of an event would cause unacceptable inconsistency, implementations SHOULD use an outbox or another mechanism that durably couples change records with event delivery. This is a future implementation choice, not a requirement to build an outbox now.
- Event publication SHALL be explicit and configurable in accordance with ADR-0001. Enabling publication to a real external system is an opt-in deployment/configuration decision; the no-op implementation does not require a separate opt-in to ordinary domain operations.
- Services SHALL retain direct resource retrieval, enumeration, and export APIs. Events are a change-notification path, not the authoritative data store. Consumers SHOULD be designed to recover from missed notifications through reconciliation or a snapshot/export path.
- Contracts SHOULD define events for meaningful lifecycle changes, including create, update, and delete where those operations exist. Events SHOULD avoid exposing implementation details such as database table names or backend-specific identifiers.
- Conformance tests SHOULD verify that successful state changes invoke the publication port with the expected event, that failed state changes do not announce a successful change, and that the no-op publisher preserves normal service behavior.

## Consequences

### Positive
- Services are ready for incremental downstream processing without requiring immediate messaging infrastructure.
- New consumers can react to changes rather than continuously scan all records.
- Domain operations remain independent of broker technology.
- The no-op default keeps deployments and tests simple.

### Negative
- Event schemas become part of the service's public compatibility surface.
- Consumers must handle duplicates, missed events, and eventual consistency.
- A no-op publisher provides no delivery or notification guarantee; it only establishes the call boundary. Fire-and-forget likewise does not imply durable handoff or successful receipt.
- Durable publication may later require an outbox, retries, monitoring, and operational ownership.

## Alternatives Considered

### Require a messaging broker for every service
Rejected because it introduces operational infrastructure before there is a consumer or deployment need.

### Let each consumer poll and scan service data
Rejected as the only integration model because it creates repeated work and delays notification, though enumeration and reconciliation remain supported.

### Add events only when a real consumer appears
Rejected because it makes publication an invasive retrofit across service implementations and misses the opportunity to establish a consistent contract early.

## Related ADRs
- [ADR-0001](ADR-0001-explicit-opt-in-cross-cutting-behavior.md) — Explicit Opt-In Cross-Cutting Behavior
- [ADR-0005](ADR-0005-event-driven-internal-bus.md) — Event Driven Internal Bus
- [FlossWare/messaging](https://github.com/FlossWare/messaging) — Shared messaging contract and pluggable transports
- [ADR-0006](ADR-0006-cross-cutting-decorators.md) — Cross-Cutting Decorators
- [ADR-0010](ADR-0010-rest-service-boundaries.md) — REST Service Boundaries and Integration
- [ADR-0031](ADR-0031-rest-representation-contracts-and-api-evolution.md) — REST Representation Contracts and API Evolution
- [ADR-0033](ADR-0033-composable-and-decoratable-contract-implementations.md) — Composable and Decoratable Contract Implementations
