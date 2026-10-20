# ADR-0033: Composable and Decoratable Contract Implementations

## Status
Proposed

## Date
2026-10-20

## Context

FlossWare contracts may have multiple implementations with different trade-offs, such as in-memory, filesystem-backed, database-backed, remote-service, cached, instrumented, or resilient implementations. Consumers should depend on the contract rather than on a concrete implementation, and it should be possible to combine implementations without changing the contract or duplicating domain behavior.

For example, a filesystem-backed store could be wrapped by an in-memory cache; a remote implementation could be wrapped with metrics, retry, or resilience behavior; and decorators could themselves be composed into a stack. If composability is not considered when a contract is designed, implementation-specific assumptions can leak into the contract and make later composition awkward or unsafe.

ADR-0006 addresses decorators as a mechanism for opted-in cross-cutting concerns. This ADR addresses the broader design question: whether a contract and its implementations support substitutable, stackable implementations, including decorators that add behavior or delegate to another implementation.

## Scope

This decision applies to FlossWare service and library contracts where multiple implementations or layered behavior may be useful, regardless of language, runtime, storage backend, or protocol.

## Non-goals

- It does not require every contract to have multiple implementations or a decorator implementation immediately.
- It does not prescribe a universal decorator framework, base class, language feature, or package layout.
- It does not make decorators a substitute for a clear contract or justify arbitrary layers of indirection.
- It does not permit decorators to silently change the contract's documented semantics.
- It does not change the explicit opt-in requirement for cross-cutting side effects established by ADR-0001 and ADR-0006.

## Decision

- When defining a FlossWare contract, designers SHALL consider whether implementations should be composable through a decorator or equivalent wrapper pattern. The decision and any reason not to support it SHOULD be recorded in the contract's design notes or implementation plan.
- Where composition is appropriate, the contract SHALL be expressed independently of concrete implementations, and a decorator SHALL implement the same public contract as the component it wraps.
- A decorator SHALL delegate to a wrapped implementation for behavior it does not replace. It MAY add behavior, transform implementation strategy, or coordinate with another implementation, provided it preserves the contract's documented guarantees.
- Decorators and implementations MAY be stacked in an explicit, configurable order. Each layer SHALL have a clear responsibility, and ordering SHALL be documented when it can affect observable behavior.
- A stack SHALL preserve the contract's specified input/output semantics, errors, resource lifecycle, cancellation, deadlines, concurrency guarantees, and consistency guarantees as applicable. Any intentional semantic change SHALL be represented explicitly in the contract or in a distinct contract/API.
- Composition SHALL NOT require consumers to know which concrete implementation or decorator stack is active. Configuration and diagnostics SHOULD make the selected stack inspectable.
- Implementations SHOULD support useful combinations without requiring every possible combination to be implemented as a separate concrete class.
- Decorators that introduce side effects or cross-cutting behavior, such as persistence, retries, caching, auditing, metrics, or policy enforcement, SHALL follow the explicit opt-in policy in ADR-0001 and the mechanism guidance in ADR-0006.
- Contracts SHOULD include conformance tests that can be run against each implementation and relevant decorator stacks. Tests SHOULD cover interactions between layers where order or combined behavior matters.
- A simple direct implementation MAY be provided as the default when no composition is needed. The pattern SHOULD remain proportionate to the contract's actual needs.

### Example

A storage contract might have a filesystem implementation and an in-memory implementation. A cache decorator can wrap the filesystem implementation:

`CacheStore(FileSystemStore(...))`

A stack can add further behavior without changing the consumer-facing contract:

`MetricsStore(CacheStore(FileSystemStore(...)))`

The same contract may also be implemented directly by an in-memory store. The examples are illustrative; the contract must define the semantics needed to make each implementation substitutable.

## Consequences

### Positive

- Implementations can be combined instead of multiplying bespoke implementation classes.
- Consumers remain independent of storage, caching, instrumentation, and resilience choices.
- New behavior can often be added without modifying existing implementations.
- Lightweight in-memory or filesystem implementations can coexist with richer backends under the same contract.
- Contract conformance tests can validate both individual implementations and meaningful compositions.

### Negative

- Layer ordering can affect correctness, latency, resource use, and failure behavior.
- Excessive or opaque stacks can make debugging and performance analysis harder.
- Some capabilities cannot be safely decorated without explicit semantics, especially transactions, caching, retries, and concurrent operations.
- Supporting a common contract across implementations may require additional conformance testing and documentation.

## Alternatives Considered

### Require one implementation per contract

Rejected as a general rule. It prevents useful combinations and encourages implementation-specific options to leak into contracts.

### Mandate decorators for every contract

Rejected. Some contracts have one simple implementation or semantics that make wrapping unnecessary. The decision is to consider composability by default, not to add abstractions without a use case.

### Use inheritance or implementation-specific subclasses as the primary composition mechanism

Rejected as the default. Inheritance can couple behavior to a class hierarchy and tends to multiply combinations. It MAY be appropriate where the language or domain makes it the clearer solution.

### Use only global interceptors or framework middleware

Rejected as the universal approach. These can be useful for cross-cutting concerns, but explicit decorators also support domain-level implementation composition and remain usable outside a specific framework.

## Related ADRs

- [ADR-0001](ADR-0001-explicit-opt-in-cross-cutting-behavior.md) — Explicit Opt-In Cross-Cutting Behavior
- [ADR-0006](ADR-0006-cross-cutting-decorators.md) — Cross-Cutting Decorators
- [ADR-0007](ADR-0007-unified-client-service-contract.md) — Unified Client-Service Contract
- [ADR-0024](ADR-0024-contract-centric-repository-layering-and-naming.md) — Contract-Centric Repository Layering and Naming
- [ADR-0032](ADR-0032-language-neutral-multi-protocol-service-contracts.md) — Language-Neutral Multi-Protocol Service Contracts

## Notes

This ADR establishes a design consideration and composability policy. It does not require implementation work across all repositories in one migration. Apply it when creating contracts and when materially extending existing ones.
