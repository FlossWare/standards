# ADR-0024: Contract-Centric Repository Layering and Naming

## Status
Accepted

## Date
2026-09-17

## Context

FlossWare contains language-neutral contracts, domain-specific contracts, and concrete implementations in one or more programming languages. Without a consistent repository structure, the repository name does not reliably communicate whether a project defines a contract, implements a contract, specializes a contract for a domain, or implements a domain contract in a particular language.

This ambiguity is especially problematic when a language-specific implementation is created first. The implementation can accidentally become the perceived architectural authority, even when the contract is intended to remain language-neutral.

FlossWare also needs a reusable convention that applies beyond Loom. The convention must make language implementations peers rather than making one implementation language canonical.

## Scope

This decision applies to FlossWare repositories whose boundaries are represented by a contract, a domain specialization of that contract, or an implementation of either.

It applies across programming languages and domains.

## Non-goals

- This decision does not require every FlossWare repository to be part of a contract family.
- This decision does not prescribe a programming language.
- This decision does not require a particular build system, deployment model, transport, or runtime.
- This decision does not require a one-to-one mapping between contracts and repositories when a different repository boundary is explicitly justified.
- This decision does not define runtime dependency graphs beyond the semantic ownership relationships described here.

## Decision

FlossWare SHALL use the following repository naming and layering convention when a contract family is split across repositories:

```text
{contract}
{contract}-{language}
{contract}-{domain}
{contract}-{domain}-{language}
```

The names have these meanings:

| Pattern | Meaning |
| --- | --- |
| `{contract}` | Foundational, language-neutral contract or protocol |
| `{contract}-{language}` | Implementation of the foundational contract in a specific language |
| `{contract}-{domain}` | Domain-specific contracts and semantics built on the foundational contract |
| `{contract}-{domain}-{language}` | Implementation of the domain contracts in a specific language |

### Contract repository

`{contract}` SHALL be the architectural authority for the foundational contract's language-neutral semantics.

It SHALL NOT depend on a particular implementation language.

### Language implementation repository

`{contract}-{language}` SHALL contain the implementation of the foundational contract for that language.

The language suffix identifies the implementation technology. It does not make that language architecturally canonical.

Additional language implementations SHALL be peers:

```text
{contract}-python
{contract}-java
{contract}-erlang
```

### Domain contract repository

`{contract}-{domain}` SHALL contain domain-specific contracts and semantics that build on the foundational contract.

The domain repository SHALL remain implementation-language neutral unless an exception is explicitly documented.

A domain repository MAY extend the foundational contract with domain-specific semantics, but SHALL NOT redefine, contradict, or replace foundational contract semantics. The foundational `{contract}` repository remains authoritative for those semantics.

### Domain implementation repository

`{contract}-{domain}-{language}` SHALL contain the implementation of the domain contracts for the specified language.

It SHALL remain distinguishable from the domain contract repository so that domain semantics do not become coupled to one implementation language.

### Loom example

Loom follows the convention:

```text
loom
loom-python
loom-ai
loom-ai-python
```

Here:

- `loom` is the language-neutral Loom protocol and semantic contract.
- `loom-python` is a Python implementation of Loom.
- `loom-ai` contains AI-domain contracts and semantics built on Loom.
- `loom-ai-python` contains the Python implementation of those AI-domain contracts.

Future implementations may include:

```text
loom-java
loom-erlang
loom-ai-java
loom-ai-erlang
```

Python is therefore not special. It is one implementation language among peers.

### Naming

A two-segment name of the form `{contract}-{language}` is reserved for a language implementation. Domain names SHOULD avoid colliding with language identifiers. If a domain identifier would create an ambiguous repository name, the repository SHALL use an explicit domain-qualified naming form rather than relying on positional guesswork.

Repository names SHALL continue to follow the FlossWare lowercase kebab-case convention.

Repository names and distribution/import identifiers serve different purposes and MUST NOT be assumed to match. A repository rename does not automatically rename a published package, Maven artifact, Python import package, or public API. Those identifiers SHOULD align when practical, but any change must account for compatibility, release, and migration requirements. Documentation must state the canonical repository name and accurately identify any intentionally retained legacy artifact/import names.

The suffixes in this ADR encode architectural role. They are not arbitrary project tags.

When a repository name cannot clearly communicate its role under this convention, the repository boundary SHOULD be reconsidered before introducing a new naming pattern.

## Consequences

### Positive

- Repository names communicate architectural role.
- Contract authority remains separate from implementation language.
- Python, Java, Erlang, and other implementations can evolve as peers.
- Domain contracts remain reusable across languages.
- New contract families can follow the same convention without inventing another repository taxonomy.
- Documentation, dependency review, and repository discovery become easier.

### Negative

- A single conceptual capability may span multiple repositories.
- Cross-repository changes require explicit coordination.
- Small contract families may incur repository overhead.
- Existing repositories may need restructuring or documentation updates as their current responsibilities are clarified.

## Alternatives Considered

### Put the implementation language in the base repository name

Rejected. A name such as `loom-python` cannot serve as the architectural authority for a language-neutral Loom contract because it makes the implementation language appear foundational.

### Keep the domain contract and implementation in one repository

Rejected as the general rule. This couples domain semantics to an implementation language when the domain contract should remain reusable and independently specified.

### Use one monorepo for contracts and all implementations

Rejected as the general FlossWare convention. A monorepo can be appropriate for a particular system, but it should not be required to express contract and implementation boundaries.

### Invent a naming convention independently for each project

Rejected. The purpose of this decision is to establish one reusable FlossWare convention rather than repeat the same architectural decision for every contract family.

## Related ADRs

- [ADR-0009](ADR-0009-core-architecture-principles.md) — Core Architecture Principles
- [ADR-0010](ADR-0010-rest-service-boundaries.md) — REST Service Boundaries and Integration

## Notes

This ADR defines repository architecture and naming. Individual contract repositories remain authoritative for their own normative contract semantics, while implementation repositories remain authoritative for their language-specific realization.
