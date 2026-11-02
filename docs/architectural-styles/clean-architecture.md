# Clean Architecture Style Guide

A complete guide to enforcing Clean Architecture and Hexagonal (Ports &
Adapters) boundaries using the portfolio's language-specific Semgrep packs.
The examples below focus on the original Python and Rust rules; the same profile
is available for Scala, Go, Haskell, TypeScript, Java, and Kotlin.

## Architectural Principles

Clean Architecture organizes software into concentric circles with one
inviolable rule:

> **The Dependency Rule:** Source code dependencies must point only inward,
> toward higher-level policies. Nothing in an inner circle can know anything at
> all about something in an outer circle.

```
       +---------------------------------------------+
       | Presentation / UI / CLI (Outer Adapters)   |
       |   +-------------------------------------+   |
       |   | Application / Use Cases (Services)  |   |
       |   |   +-----------------------------+   |   |
       |   |   | Domain Core (Entities, VOs) |   |   |
       |   |   +-----------------------------+   |   |
       |   +-------------------------------------+   |
       | Infrastructure / DB / HTTP (Outer Adapters) |
       +---------------------------------------------+
```

## Layer Mapping & Rule Enforcement

### 1. Domain Layer (`**/domain/**`)
- **Responsibility:** Pure business entities, value objects, and domain rules.
- **Enforced Rules:**
  - `python-domain-no-framework-imports`: Prohibits importing web frameworks,
    ORMs, or HTTP clients.
  - `rust-domain-no-infrastructure-dependencies`: Prohibits importing
    infrastructure adapters.
  - `rust-domain-no-presentation-dependencies`: Prohibits referencing
    presentation or UI code.
  - `rust-domain-no-application-dependencies`: Prohibits coupling to application
    use case orchestration.

### 2. Application Layer (`**/application/**`)
- **Responsibility:** Use case orchestration, inbound ports, and outbound port
  interfaces.
- **Enforced Rules:**
  - `python-application-no-presentation-dependencies`: Prohibits coupling to
    views, controllers, or HTTP routes.
  - `rust-application-no-presentation-dependencies`: Prohibits importing
    presentation modules.
  - `rust-application-service-no-sibling-service-dependencies`: Prevents tight
    coupling between sibling application use cases.

### 3. Presentation Layer (`**/presentation/**`, `**/views/**`)
- **Responsibility:** Thin HTTP controllers, CLI handlers, or view presenters.
- **Enforced Rules:**
  - `python-presentation-must-be-thin-delegate`: Disallows direct database
    queries (`db.execute`, `session.query`). Controllers must parse input DTOs,
    delegate to use cases, and serialize responses.
  - `rust-presentation-must-delegate-no-database`: Disallows direct SQL queries
    in presentation handlers.

### 4. Structural Design & Encapsulation
- **Enforced Rules:**
  - `python-class-max-5-public-methods`: Enforces focused classes with minimal
    public surfaces.
  - `python-class-max-3-private-methods`: Prevents classes from hiding multiple
    internal algorithms.
  - `rust-struct-fields-must-not-be-public`: Guarantees encapsulation and domain
    invariant protection.
  - `rust-no-setters`: Enforces intention-revealing domain behavior instead of
    anemic field mutation.

## Turnkey `.semgrep.yml` Configuration

Drop this configuration into your repository root to enforce Clean Architecture
on every commit:

```yaml
rules:
  # Python Rules
  - rules/architecture/python-architecture.yml
  - rules/design/python-design.yml
  - rules/conventions/python-imports.yml
  - rules/maintainability/python-function-defaults.yml

  # Rust Rules
  - rules/architecture/rust-layer-boundaries.yml
  - rules/architecture/rust-service-boundaries.yml
  - rules/design/rust-encapsulation.yml
  - rules/maintainability/rust-control-flow.yml

  # Scala Rules
  - rules/architecture/scala-layer-boundaries.yml
  - rules/design/scala-encapsulation.yml
  - rules/conventions/scala-imports.yml
  - rules/maintainability/scala-control-flow.yml

  # Go, Haskell, TypeScript, Java, and Kotlin
  - rules/architecture/go-layer-boundaries.yml
  - rules/architecture/haskell-layer-boundaries.yml
  - rules/architecture/typescript-layer-boundaries.yml
  - rules/architecture/java-layer-boundaries.yml
  - rules/architecture/kotlin-layer-boundaries.yml
```
