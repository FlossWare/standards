# ADR-0032: Language-Neutral Multi-Protocol Service Contracts

## Status
Proposed

## Date
2026-10-19

## Context

FlossWare is building reusable capabilities that include Memory, Learning, genetic optimization (GA), messaging, alerts, caching, budgeting, observability, and services reused by Claude Ensemble (CE). These capabilities must be independently implementable and consumable across languages, including a pure Java implementation without a Python runtime or Python-specific packaging.

REST/HTTP is already the default synchronous service contract ([ADR-0010](ADR-0010-rest-service-boundaries.md)). FlossWare also needs a consistent way to make capabilities available to AI clients through MCP, to support typed RPC through gRPC where useful, and to integrate asynchronous work and events. Without a shared policy, each repository may implement its own protocol stack, duplicate business logic, or require changes to a central REST host whenever a capability is added.

The platform needs to standardize service identity, discovery, protocol bindings, and behavior while allowing each capability to own its domain contract and implementation.

## Scope

This ADR applies to FlossWare service capabilities and their consumers, including CE integrations, regardless of implementation language, runtime, framework, or deployment topology. It governs protocol exposure strategy, shared behavior across interfaces, service discovery metadata, and service URL organization.

## Non-goals

- It does not require every service to run a separate listener or process for every protocol.
- It does not require every operation to be exposed over every protocol when the operation's semantics, security, or consumer needs make a binding inappropriate.
- It does not mandate a programming language, web framework, RPC framework, message broker, or service mesh.
- It does not make MCP the canonical business API or replace REST with MCP.
- It does not require gRPC for every service regardless of need.
- It does not prescribe a single database, persistence backend, or deployment model.

## Decision

### Language-neutral service contracts

- Service contracts SHALL be language-neutral and SHALL define externally observable behavior independently of implementation frameworks, package names, and runtime-specific interfaces.
- A conforming service SHALL be implementable in Java or another supported language without requiring Python or importing implementation code from another language.
- Service descriptors, schemas, discovery metadata, and protocol bindings SHALL use documented interoperable formats and protocols.
- The service implementation SHALL remain the source of truth for domain behavior. Protocol adapters SHALL NOT maintain competing implementations of business rules.

### Protocol exposure

- REST over HTTP SHALL be the universal baseline interface for externally accessible FlossWare services where request/response semantics are appropriate.
- FlossWare capabilities SHALL be designed for multi-protocol exposure from the beginning. Protocol exposure SHALL be selected according to the capability's semantics and supported consumers rather than embedded as assumptions in domain logic.
- MCP SHALL be a first-class integration interface for AI clients. Externally available capabilities suitable for tool or resource interaction SHOULD be discoverable and invocable through MCP, either through a service-native MCP interface or a shared/platform MCP adapter. Exceptions for security, suitability, or operational reasons SHALL be explicit.
- An MCP implementation MAY call an existing REST API or invoke the same application/service layer directly. Both approaches SHALL reuse the same domain behavior, validation, authorization rules, and audit requirements.
- gRPC SHALL be a supported platform binding and SHOULD be used when typed RPC, streaming, throughput, or other concrete requirements justify it. Services are not required to expose every operation through gRPC without such a need.
- Asynchronous messaging and versioned events SHOULD be used for durable background work, notifications, event distribution, or decoupled workflows where request/response is insufficient. The contract SHALL document delivery, ordering, retry, idempotency, and durability semantics where applicable.
- Streaming transports, including HTTP streaming or WebSockets, MAY be used when continuous results, bidirectional communication, or progress updates are required.
- A capability SHALL NOT be required to duplicate its business logic merely to support another protocol.

### Shared behavior and authorization

- All protocol bindings for a capability SHALL preserve its domain invariants and apply equivalent authorization policy for equivalent operations.
- Adapters SHALL authenticate their clients and propagate or translate identity and authorization context through documented rules. Connecting to an adapter SHALL NOT grant unrestricted access to the underlying service.
- Protocol-specific semantics SHALL be documented. FlossWare SHALL NOT claim that REST request/response, MCP tools/resources, gRPC streams, and asynchronous events are interchangeable when their behavior differs.
- Error mapping, request correlation, tracing, metrics, and audit provenance SHOULD be consistent across bindings where the protocol supports them.
- A failure in optional telemetry SHALL NOT silently change the business result or bypass required authorization and budget enforcement.

### Service discovery and extension

- Services SHALL publish a language-neutral descriptor containing at minimum a stable service ID, contract/API version information, health/readiness information, and the available protocol bindings and their locations.
- Where applicable, descriptors SHOULD reference OpenAPI for REST, MCP connection/configuration metadata, protobuf service definitions for gRPC, and AsyncAPI or an equivalent documented event contract.
- The service catalog and routing layer SHALL discover and route to registered services without hardcoded service-specific branches in the generic REST entry point.
- Adding a compliant service SHOULD require implementing its service contract and registering/configuring its descriptor, not modifying the routing host's core code.
- Registration, discovery, authorization, and external exposure SHALL remain distinct decisions. Discovery SHALL NOT automatically make a service publicly accessible.
- The catalog MAY group services into families for navigation and filtering. Family names SHALL be metadata and SHALL NOT be included in service URL paths solely for organizational categorization.
- REST URL paths SHOULD use a stable service-oriented namespace such as `/v1/{service}/...`. Families SHOULD NOT be inserted into the path unless they represent a real API or security boundary.

### Implementation and conformance

- Implementations MAY use in-process adapters or separately deployed adapters/gateways. The deployment choice SHALL NOT alter the language-neutral wire contract.
- Protocol adapters SHOULD be thin and delegate to the canonical service implementation or its public API rather than duplicate domain logic.
- Each advertised binding SHALL have automated contract/conformance tests covering schemas, errors, authorization, and relevant operation semantics.
- Cross-language conformance tests SHOULD verify that implementations in different languages behave consistently at the protocol boundary.
- A protocol binding SHALL be advertised as available only when it is configured and usable; registration alone SHALL NOT imply readiness.

## Consequences

### Positive

- Java, Python, and other implementations can participate without sharing a runtime.
- REST remains a universal, inspectable service baseline while MCP and other bindings serve their intended consumers.
- Capabilities can be added through discovery and registration without service-specific changes to the generic host.
- One domain implementation can support multiple interfaces without duplicating business rules.
- CE and other clients can reuse FlossWare services through stable contracts.
- Service family reorganizations do not require URL changes.

### Negative

- Multiple protocol bindings increase contract, authorization, documentation, and conformance-testing work.
- A shared catalog and adapters become platform components that must be operated and secured.
- Some protocol concepts do not map one-to-one; each binding requires explicit semantic decisions.
- Services may need different exposure profiles rather than a simplistic rule that every operation must be available everywhere.

## Alternatives Considered

### REST only

Rejected as the complete platform strategy. REST remains mandatory as the baseline, but MCP supports AI-native discovery and invocation, while gRPC and asynchronous messaging address other integration needs.

### Require every service to implement every protocol natively

Rejected. This would duplicate adapter work, force unnecessary listeners, and couple each service to protocols it may not need. Native implementation remains an option when appropriate.

### Require a separate MCP server for every service

Rejected. A service MAY expose MCP natively, but a shared MCP server can expose selected capabilities by consuming their REST APIs or invoking shared service logic.

### Make MCP the canonical service interface

Rejected. MCP is an AI integration protocol, not a replacement for general-purpose service APIs and event contracts.

### Hardcode each service in the central REST host

Rejected. It couples the host to capability-specific knowledge and makes adding services require host changes.

### Put service families in public URL paths

Rejected as a default. Families are catalog metadata; service-oriented paths are more stable when organizational classifications change.

## Related ADRs

- [ADR-0005](ADR-0005-event-driven-internal-bus.md) — Event-Driven Internal Bus
- [ADR-0007](ADR-0007-unified-client-service-contract.md) — Unified Client-Service Contract
- [ADR-0010](ADR-0010-rest-service-boundaries.md) — REST Service Boundaries and Integration Model
- [ADR-0016](ADR-0016-configuration-as-source-of-truth.md) — Configuration as Source of Truth
- [ADR-0024](ADR-0024-contract-centric-repository-layering-and-naming.md) — Contract-Centric Repository Layering and Naming
- [ADR-0031](ADR-0031-rest-representation-contracts-and-api-evolution.md) — REST Representation Contracts and API Evolution

## Notes

This ADR defines the architectural policy. Detailed descriptor schemas, catalog API, MCP exposure policy, gRPC conventions, and asynchronous message envelopes should be specified separately and adopted incrementally. Existing services should migrate through tracked issues and conformance tests rather than a blanket rewrite.
