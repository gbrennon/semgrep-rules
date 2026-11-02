# Library & Package Design Style Guide

Guidance on configuring Semgrep rules to enforce strict encapsulation, clean
exports, and defensive API boundaries in reusable libraries and SDKs. Examples
below are representative; language-specific packs cover Python, Rust, Scala, Go,
Haskell, TypeScript, Java, and Kotlin.

## Design Principles for Libraries

Reusable libraries and public SDKs require rigorous interface hygiene:

1. **Strict Encapsulation:** Private implementation details must not leak to
   consumers or across internal modules.
2. **Predictable Defaults:** Functions must avoid mutable state shared across
   invocations.
3. **Explicit Namespaces:** Avoid wildcard imports that pollute consumer
   namespaces or cause naming collisions.
4. **Resilient Error Handling:** Never swallow exceptions silently or rely on
   brittle identity checks.

## Recommended Rules for Libraries

### 1. API Encapsulation
- `python-no-private-member-access`: Enforces that internal `_private` attributes
  are not accessed outside defining modules.
- `rust-struct-fields-must-not-be-public`: Forces library structs to expose
  validated constructor methods and immutable accessors.
- `rust-no-setters`: Encourages intention-revealing methods rather than raw
  state mutations.

### 2. Import & Layout Hygiene
- `python-no-wildcard-imports`: Enforces explicit imports for clarity and
  avoids namespace pollution.
- `rust-no-inline-modules`: Enforces a one-to-one mapping between filesystem
  files and modules.

### 3. State & Complexity Management
- `python-no-mutable-default-arguments`: Prevents shared default lists/dicts
  from leaking state across invocations.
- `rust-no-control-flow-nesting-above-2`: Caps nested control structures to
  keep library algorithms readable and testable.

## Turnkey Configuration Recipe

```yaml
rules:
  # Python Rules
  - rules/design/python-design.yml
  - rules/conventions/python-imports.yml
  - rules/maintainability/python-function-defaults.yml
  - rules/correctness/python-correctness.yml
  - rules/correctness/python-exceptions.yml

  # Rust Rules
  - rules/design/rust-encapsulation.yml
  - rules/conventions/rust-module-layout.yml
  - rules/maintainability/rust-control-flow.yml
  - rules/correctness/rust-correctness.yml

  # Scala Rules
  - rules/design/scala-encapsulation.yml
  - rules/conventions/scala-imports.yml
  - rules/maintainability/scala-control-flow.yml
  - rules/correctness/scala-correctness.yml

  # Go, Haskell, TypeScript, Java, and Kotlin
  - rules/design/go-encapsulation.yml
  - rules/design/haskell-encapsulation.yml
  - rules/design/typescript-encapsulation.yml
  - rules/design/java-encapsulation.yml
  - rules/design/kotlin-encapsulation.yml
```
