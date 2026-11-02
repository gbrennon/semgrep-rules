# Semgrep Rules Portfolio

This repository is an opinionated portfolio of Semgrep rules for eight
languages and six quality categories. The rules encode the owner's engineering
preferences as focused, reviewable policies rather than pretending to be a
language-neutral standard.

## Language matrix

| Language | Semgrep mode | Coverage |
|---|---|---|
| Python | Native | All six categories |
| Rust | Native | All six categories |
| Scala | Native | All six categories |
| Go | Native | All six categories |
| TypeScript | Native | All six categories |
| Java | Native | All six categories |
| Kotlin | Native | All six categories |
| Haskell | Generic regex fallback | All six categories |

C# is intentionally not included. Haskell uses `generic` matching because the
installed Semgrep engine does not provide a Haskell parser.

## Taxonomies

- [Security](rules/security.md) — dangerous execution, deserialization,
  injection, and weak randomness.
- [Correctness](rules/correctness.md) — partial operations, ignored failures,
  blocking calls, and language-specific runtime traps.
- [Design](rules/design.md) — encapsulation, mutation, setters, and interface
  hygiene.
- [Architecture](rules/architecture.md) — dependency direction and thin
  delivery boundaries.
- [Conventions](rules/conventions.md) — import and module-layout hygiene.
- [Maintainability](rules/maintainability.md) — nesting and complexity signals.

## Repository layout

```text
rules/<category>/<language>-*.yml
  Language-specific Semgrep definitions.

docs/rules/*.md
  Category references, rationale, and remediation examples.

docs/architectural-styles/*.md
  Profile-oriented rule-selection guidance.
```

Each rule declares `category`, `technology`, `confidence`, `scope`, `policy`,
`rationale`, and `references` metadata. Architecture rules are generally
`conventional-hexagonal` and `opt-in`; they depend on project paths such as
`domain`, `application`, and `presentation`.

## Validation and usage

Validate all definitions:

```bash
semgrep --validate --config rules
```

Scan a project:

```bash
semgrep scan --config /path/to/semgrep-rules/rules --error .
```

Scan one language/category pack:

```bash
semgrep scan \
  --config /path/to/semgrep-rules/rules/security/go-security.yml \
  --error .
```

Run the repository self-scan:

```bash
semgrep scan --config rules --error --exclude rules .
```

Use the architecture profiles for curated combinations:

- [Clean Architecture](architectural-styles/clean-architecture.md)
- [Layered Architecture](architectural-styles/layered-architecture.md)
- [Library & Package Design](architectural-styles/library-package-design.md)

## Interpreting findings

A finding is a review signal, not proof of a defect. Confirm data flow, project
conventions, and intended architecture before changing code or promoting an
opt-in rule to an error policy. Regex-based rules can miss unusual formatting
and can require project-specific tuning.
