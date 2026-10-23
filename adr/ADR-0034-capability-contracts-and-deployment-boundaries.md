# ADR-0034: Capability Contracts and Deployment Boundaries

## Status
Proposed

## Date
2026-10-23

## Context

FlossWare is composed of reusable capabilities. Some capabilities are substantial domain services, while others are smaller functions such as chunking, token counting, score normalization, query parsing, deduplication, embedding, or retrieval strategies.

Treating every small capability as an independently deployed microservice creates avoidable network hops, operational overhead, deployment coordination, and failure modes. Hiding reusable behavior inside a large service creates the opposite problem: unclear ownership, duplicated implementations, and contracts that cannot be reused or independently tested.

ADR-0010 states that business capabilities SHALL be exposed through services and that REST/MCP are consumer-facing protocols for capabilities. ADR-0020 separates capability implementation from protocol. The word *service* must not be interpreted as requiring a separate process, deployment, or repository for every capability.

## Scope

This decision applies to FlossWare contracts, reusable capabilities, implementation components, protocol adapters, decorators, and deployment decisions across all domains and languages.

## Non-goals

- It does not mandate a microservice per function, algorithm, class, or repository.
- It does not prohibit independently deploying a small capability when its ownership or operational requirements justify doing so.
- It does not require every internal helper to have a public or network-facing API.
- It does not prescribe a universal service framework, runtime, language, or repository layout.
- It does not turn algorithm names into service boundaries by default.

## Decision

1. **Capabilities and deployment units SHALL be modeled separately.** A capability is a coherent responsibility with defined behavior. A deployment unit is a runtime/operational packaging choice. One deployment MAY host multiple capability implementations, and a capability MAY later be extracted without changing its domain contract.

2. **Meaningful reusable capabilities SHOULD have explicit contracts.** Contracts SHALL define the relevant inputs, outputs, errors, semantic guarantees, and lifecycle/concurrency behavior. Contracts SHOULD be language-neutral when they are intended for use across languages. A capability contract may be consumed in-process or through an adapter.

3. **Capability contracts SHALL NOT imply independent deployment.** A contract is an ownership and substitutability boundary, not a promise that its implementation runs in another process. REST, MCP, events, or another protocol SHOULD be added where consumers, security, or operational requirements justify that boundary.

4. **Every proposed capability SHOULD be classified before implementation** as one of:
   - **Domain service:** owns a domain responsibility, policy, lifecycle, or data boundary.
   - **Reusable capability:** a separately specified and substitutable behavior that may be composed in-process or exposed through a service adapter.
   - **Implementation component:** an algorithm, transformation, helper, or data structure that remains internal to a capability unless independent reuse or ownership justifies a contract.
   - **Adapter or decorator:** exposes a capability through a protocol or composes behavior around an implementation while preserving its contract.

5. **A capability SHOULD be independently deployed** when one or more concrete requirements justify the boundary, such as independent scaling, availability, security/authorization, resource isolation, release lifecycle, ownership, or multiple consumers that need a stable remote interface. The design SHOULD identify the requirement rather than relying on the label “microservice.”

6. **A capability SHOULD remain in-process** when it is tightly coupled to its parent operation, has no independent ownership or operational need, and remote invocation would add latency, failure modes, or deployment overhead without meaningful reuse benefits. It MUST still be testable at the appropriate component boundary.

7. **Algorithms are not services merely because they have names.** An algorithm such as BM25 MAY be an implementation of a retrieval or scoring contract. Shared corpus statistics, indexing metadata, lexical retrieval, ranking, and reranking MAY be distinct capabilities when they have coherent responsibilities and consumers. Their separation SHALL follow data ownership, semantics, reuse, and operational requirements rather than algorithm names alone.

8. **Composition and extraction SHALL preserve semantics.** An implementation can be wrapped or composed as allowed by ADR-0033. Moving a capability from an in-process implementation to a remote service SHOULD preserve its domain contract. Transport-specific concerns belong in adapters; transport errors and deadlines must be mapped to the contract's documented behavior.

9. **Repository boundaries require separate justification.** A capability SHOULD receive a separate contract repository when language-neutral semantics need to be shared across implementations or when it is an independently governed foundation. A language-specific implementation belongs in its corresponding implementation repository under ADR-0024. A small internal component SHALL NOT receive a repository solely to make the architecture diagram look impressively crowded.

10. **Architecture inventories SHALL record the distinctions.** For each significant capability, document its responsibility, contract, consumers, data/policy ownership, implementation(s), protocol exposure (if any), and current deployment placement. Mark proposals separately from accepted boundaries. Do not claim that an unimplemented capability or service already exists.

## Illustrative retrieval classification

| Capability | Initial classification | Boundary guidance |
|---|---|---|
| Memory lifecycle and memory policy | Domain service | Owns memory-specific behavior and persistence coordination |
| Chunking | Reusable capability | Contract defines chunk derivation and provenance; implementation can run in-process |
| Embedding generation | Reusable capability | Provider/model variation is behind a stable contract |
| Vector storage and search | Reusable capability | Storage/search implementations are replaceable behind a contract |
| Lexical retrieval | Reusable capability | Owns lexical candidate retrieval, not necessarily its own deployment |
| Corpus statistics and index metadata | Candidate reusable capability | Consider a distinct contract when multiple consumers need consistent shared statistics |
| BM25 | Algorithm implementation | Strategy behind lexical retrieval/scoring; not automatically a service |
| Query parsing | Reusable capability when shared | Keep internal if only one parent capability consumes it |
| Ranking | Reusable capability | Shared ranking semantics may serve lexical, vector, and hybrid retrieval |
| Score normalization | Implementation component initially | Promote only if independently reused or governed |
| Hybrid retrieval coordination | Reusable capability or domain service | Owns orchestration of retrieval strategies; deploy separately only when justified |
| Deduplication | Reusable capability when shared | May remain an in-process pipeline stage |
| Reranking | Reusable capability | Can select local/library or remote-model implementations behind a contract |
| Token counting | Implementation component initially | Promote when shared accounting, quota, or provider-specific semantics justify it |

This table is illustrative, not a mandate to create all listed services or repositories. The capability inventory and implementation evidence determine final boundaries.

## Consequences

### Positive

- Reusable behavior has clear contracts without forcing unnecessary network boundaries.
- Teams can independently test and replace capabilities while composing them efficiently.
- Deployment and repository decisions can be revisited without rewriting domain semantics.
- Algorithm implementations remain replaceable without promoting every algorithm into a service.
- Architecture inventories distinguish existing implementations from proposed capabilities.

### Negative

- Reviewers must distinguish contracts, implementations, protocols, repositories, and deployment units.
- Some capabilities will need explicit conformance tests even when they remain in-process.
- Promotion from an internal component to a reusable capability requires evidence and deliberate contract design.
- Some service extraction decisions will remain context-dependent rather than mechanically determined.

## Alternatives Considered

### One separately deployed microservice per capability
Rejected as a universal rule. It adds operational and network complexity even when a capability has no independent lifecycle or remote consumer.

### Keep small capabilities private inside domain services
Rejected as a universal rule. It can hide reusable behavior and make implementations difficult to substitute or test independently.

### Define contracts only when extracting to a remote service
Rejected. Contract-first design allows in-process composition today and later extraction without coupling domain semantics to transport.

### Contract boundaries independent of deployment boundaries (chosen)
Provides explicit reusable capabilities while allowing in-process composition and evidence-based service extraction.

## Related ADRs

- [ADR-0001](ADR-0001-explicit-opt-in-cross-cutting-behavior.md) — Explicit Opt-In Cross-Cutting Behavior
- [ADR-0006](ADR-0006-cross-cutting-decorators.md) — Cross-Cutting Decorators
- [ADR-0007](ADR-0007-unified-client-service-contract.md) — Unified Client-Service Contract
- [ADR-0010](ADR-0010-rest-service-boundaries.md) — REST Service Boundaries and Integration
- [ADR-0020](ADR-0020-capability-protocol-separation.md) — Capability and Protocol Separation
- [ADR-0024](ADR-0024-contract-centric-repository-layering-and-naming.md) — Contract-Centric Repository Layering and Naming
- [ADR-0032](ADR-0032-language-neutral-multi-protocol-service-contracts.md) — Language-Neutral Multi-Protocol Service Contracts
- [ADR-0033](ADR-0033-composable-and-decoratable-contract-implementations.md) — Composable and Decoratable Contract Implementations

## Notes

Apply this decision when creating capabilities and when materially revisiting existing boundaries. A classification does not establish that a service exists; implementation and executable validation remain the evidence for implementation status.
