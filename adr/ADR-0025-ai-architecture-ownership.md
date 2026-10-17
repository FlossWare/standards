# ADR-0025: AI Architecture Ownership

## Status
Accepted

## Date
2026-09-18

## Context

FlossWare previously stored AI-specific architecture decisions in the generic standards repository. As the AI architecture has become a coherent domain with its own contracts, execution model, providers, evaluation, routing, consensus, and learning concerns, those decisions need a domain-owned architectural home.

## Decision

Generic and FlossWare-wide normative standards and infrastructure decisions SHALL remain in this repository.

AI-domain contracts and AI-specific architecture decisions SHALL be maintained by FlossWare/loom-ai.

The Loom AI ADR set includes provider abstraction, execution topology, model inventory and selection, token budgets, MCP and agent capability semantics, consensus, and durable/shared intelligence.

Standards MAY define generic principles that apply to AI systems, but SHALL NOT duplicate or redefine Loom AI domain semantics.

The loom-ai repository is authoritative for AI-domain meaning; language-specific implementations remain in the corresponding implementation repositories.

## Consequences

- FlossWare-wide standards remain reusable outside Loom.
- AI architecture evolves with the AI-domain contract rather than accumulating in a generic repository.
- Cross-repository references are explicit and ownership is unambiguous.
- Historical AI ADRs retain their substance in the Loom repository while standards no longer acts as a competing authority.

## Alternatives Considered

### Keep AI ADRs in standards
Rejected because the generic standards repository would continue accumulating domain-specific architecture.

### Duplicate AI ADRs in both repositories
Rejected because duplicated normative decisions inevitably diverge.

### Move AI ADRs to loom-ai
Chosen because loom-ai is the AI-domain contract and semantics repository.

## Related ADRs

- [ADR-0009](ADR-0009-core-architecture-principles.md) — Core Architecture Principles
- [ADR-0016](ADR-0016-configuration-as-source-of-truth.md) — Configuration as Source of Truth
- [ADR-0022](ADR-0022-reproducible-build-artifacts-and-distribution.md) — Reproducible Build Artifacts and Distribution
- [ADR-0024](ADR-0024-contract-centric-repository-layering-and-naming.md) — Contract-Centric Repository Layering and Naming

The migrated AI ADRs are maintained in FlossWare/loom-ai/docs/adr.
