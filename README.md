# FlossWare Standards

This repository defines authoritative, normative standards, architecture decisions, and development conventions that apply across the FlossWare ecosystem. Its scope is broader than engineering alone.

## Core Principles

- Configuration is the source of truth.
- Defaults are minimal; capabilities are explicitly enabled.
- Components are modular and composable.
- Contracts define stable semantics; implementations realize those contracts without becoming the architectural authority.
- Repository names SHALL communicate contract, domain, and implementation-language roles where a contract family is split across repositories.
- Avoid unnecessary coupling.
- Favor automation, repeatability, and infrastructure-as-code.
- Released artifacts are derived, reproducible delivery outputs, not sources of truth.

See the core ADRs below for normative statements. AI-domain architecture decisions are maintained in [FlossWare/loom-ai](https://github.com/FlossWare/loom-ai). The knowledge corpus is maintained separately in [FlossWare/knowledge](https://github.com/FlossWare/knowledge).

## Architecture Decisions

Architecture decisions are documented as ADRs under [`adr/`](adr/).

| ADR | Topic |
| --- | --- |
| [ADR-0001](adr/ADR-0001-explicit-opt-in-cross-cutting-behavior.md) | Explicit Opt-In Cross-Cutting Behavior |
| [ADR-0005](adr/ADR-0005-event-driven-internal-bus.md) | Event-Driven Internal Bus |
| [ADR-0006](adr/ADR-0006-cross-cutting-decorators.md) | Cross-Cutting Decorators |
| [ADR-0007](adr/ADR-0007-unified-client-service-contract.md) | Unified Client-Service Contract |
| [ADR-0008](adr/ADR-0008-free-first-modular-platform.md) | Historical Free-First Platform Policy (Superseded) |
| [ADR-0009](adr/ADR-0009-core-architecture-principles.md) | Core Architecture Principles |
| [ADR-0010](adr/ADR-0010-rest-service-boundaries.md) | REST Service Boundaries and Integration |
| [ADR-0011](adr/ADR-0011-stored-procedure-database-access.md) | Stored Procedure Database Access Policy |
| [ADR-0016](adr/ADR-0016-configuration-as-source-of-truth.md) | Configuration as Source of Truth |
| [ADR-0022](adr/ADR-0022-reproducible-build-artifacts-and-distribution.md) | Reproducible Build Artifacts and Distribution |
| [ADR-0023](adr/ADR-0023-canonical-flossware-ai-state-root.md) | Canonical FlossWare AI Persistent State Root |
| [ADR-0024](adr/ADR-0024-contract-centric-repository-layering-and-naming.md) | Contract-Centric Repository Layering and Naming |
| [ADR-0025](adr/ADR-0025-ai-architecture-ownership.md) | AI Architecture Ownership |
| [ADR-0026](adr/ADR-0026-two-component-release-versioning.md) | Two-Component Release Versioning |
| [ADR-0027](adr/ADR-0027-versioning-tagging-and-releases.md) | Versioning, Tagging, and Releases |

| [ADR-0028](adr/ADR-0028-standalone-ai-components.md) | Standalone AI Components |
| [ADR-0029](adr/ADR-0029-agent-runtime-identity-and-profile-separation.md) | Agent Runtime Identity and Profile Separation |
| [ADR-0030](adr/ADR-0030-provider-credential-and-model-policy-isolation.md) | Provider Credential and Model Policy Isolation |
| [ADR-0031](adr/ADR-0031-rest-representation-contracts-and-api-evolution.md) | REST Representation Contracts and API Evolution |

New ADRs SHOULD use [`adr/TEMPLATE.md`](adr/TEMPLATE.md).

### AI-domain ADRs

AI architecture decisions are owned by FlossWare/loom-ai. This repository retains generic engineering principles and FlossWare-wide infrastructure decisions.

## CI standards

- [SonarCloud quality gate](docs/ci/sonarcloud.md) - shared GitHub Actions integration for FlossWare repositories.

## Reference architecture

- [Reference architecture diagram](docs/architecture/reference-architecture.md) (Mermaid)
- [tftp-os client contract validation](docs/architecture/reference-implementations/tftp-os-ui-contract.md)

## ADR Process

All ADRs should include:

- Status
- Date
- Context
- Scope and Non-goals (recommended; required for new ADRs)
- Decision
- Consequences (positive and negative)
- Alternatives considered
- Related ADRs

RFC 2119 keywords are used consistently:

- SHALL / SHALL NOT: mandatory requirements (MUST is treated as equivalent to SHALL)
- SHOULD / SHOULD NOT: strong recommendation
- MAY: optional behavior
