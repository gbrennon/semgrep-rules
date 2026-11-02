# Layered Architecture Style Guide

Guidance on configuring Semgrep rules to enforce classic N-Tier and MVC layered
architectures across the portfolio's supported language packs. The examples are
representative rather than exhaustive.

## Architectural Principles

Traditional Layered Architecture organizes applications into sequential tiers:

1. **Presentation Layer:** User interface, API routers, and serialization.
2. **Business / Service Layer:** Domain logic, workflow coordination, business
   rules.
3. **Data Access / Persistence Layer:** Repositories, database access, ORM
   entities.

### Core Constraints
- Tiers must not be bypassed: Presentation cannot execute database queries
  directly without going through the Service layer.
- Dependencies flow downward: Presentation -> Service -> Data Access. Lower
  layers must never import higher layers.

## Enforcing Layered Boundaries

### 1. Presentation Thin Delegation
Presentation controllers must never query databases directly:
- `python-presentation-must-be-thin-delegate`
- `rust-presentation-must-delegate-no-database`

### 2. Service Cohesion & Method Limits
Services in the business tier must maintain single responsibilities:
- `python-class-max-5-public-methods`
- `python-class-max-3-private-methods`
- `rust-impl-max-5-public-methods`

### 3. Defensive Security
Shielding data ingestion and API endpoints from injection:
- `python-no-eval-exec`
- `python-subprocess-shell-true`
- `python-insecure-deserialization-pickle`

## Turnkey Configuration Recipe

```yaml
rules:
  # Python Rules
  - rules/architecture/python-architecture.yml
  - rules/design/python-design.yml
  - rules/security/python-security.yml
  - rules/correctness/python-correctness.yml

  # Rust Rules
  - rules/architecture/rust-layer-boundaries.yml
  - rules/design/rust-encapsulation.yml
  - rules/security/rust-security.yml
  - rules/correctness/rust-correctness.yml

  # Scala Rules
  - rules/architecture/scala-layer-boundaries.yml
  - rules/security/scala-security.yml
  - rules/correctness/scala-correctness.yml

  # Go, Haskell, TypeScript, Java, and Kotlin
  - rules/architecture/go-layer-boundaries.yml
  - rules/architecture/haskell-layer-boundaries.yml
  - rules/architecture/typescript-layer-boundaries.yml
  - rules/architecture/java-layer-boundaries.yml
  - rules/architecture/kotlin-layer-boundaries.yml
```
