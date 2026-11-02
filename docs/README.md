# Semgrep Rules Portfolio

A curated collection of production-grade Semgrep rules for **Python** and
**Rust**, engineered to enforce clean software design, defensive security,
runtime correctness, and architectural boundaries.

## Overview & Taxonomies

The repository organizes rules into fundamental quality categories:

| Category | Languages | Focus | Guide |
|---|---|---|---|
| **Security** | Python, Rust | Injection & memory safety | [Doc](rules/security.md) |
| **Correctness** | Python, Rust | Logic bugs & lock safety | [Doc](rules/correctness.md) |
| **Design** | Python, Rust | Encapsulation & method caps | [Doc](rules/design.md) |
| **Architecture** | Python, Rust | Clean Architecture | [Doc](rules/architecture.md) |
| **Conventions** | Python, Rust | Layout & imports | [Doc](rules/conventions.md) |
| **Maintainability**| Python, Rust | Complexity & defaults | [Doc](rules/maintainability.md) |

## Architectural Style Guides

Curated packs tailored for specific architecture paradigms:

- **[Clean Architecture Guide](architectural-styles/clean-architecture.md):**
  Enforces inward dependency rules, domain purity, application use case
  independence, and thin presentation controllers.
- **[Layered Architecture Guide](architectural-styles/layered-architecture.md):**
  Enforces strict 3-tier / N-tier separation, preventing presentation shortcuts
  directly to database layers.
- **[Library & Package Design Guide](architectural-styles/library-package-design.md):**
  Enforces encapsulation, avoids mutable parameter defaults, and eliminates
  wildcard exports for reusable packages.

## Quick-Start Usage

### Validating Rules
Validate all rule definitions across the entire collection:
```bash
semgrep --validate --config rules
```

### Running on a Project
Scan your repository using the shared rule suite:
```bash
semgrep scan --config /path/to/semgrep-rules/rules --error .
```
