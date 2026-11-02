# Semgrep Rules Portfolio

An opinionated portfolio of Semgrep rules for enforcing security, correctness,
architecture, design, conventions, and maintainability across multiple
programming languages.

The rules encode the repository owner's engineering preferences. They are
organized as focused, reviewable policies and marked opt-in where they depend on
project architecture or a deliberate design choice.

## Supported languages

| Language | Semgrep mode | Rule files |
|---|---|---:|
| Python | Native | 6 |
| Rust | Native | 5 |
| Scala | Native | 6 |
| Go | Native | 6 |
| TypeScript | Native | 6 |
| Java | Native | 6 |
| Kotlin | Native | 6 |
| Haskell | Generic regex fallback | 6 |

C# is not part of the portfolio.

## Rule categories

- **Security** — command execution, unsafe deserialization, weak randomness,
  injection, and dangerous APIs.
- **Correctness** — partial operations, ignored failures, blocking calls, and
  language-specific runtime traps.
- **Architecture** — opt-in dependency-direction and layer-boundary checks.
- **Design** — encapsulation, mutable state, setters, and interface hygiene.
- **Conventions** — import and module-layout consistency.
- **Maintainability** — nesting and complexity heuristics.

Each category has a language-specific YAML file under `rules/<category>/`.
The detailed index and guides live in [`docs/README.md`](docs/README.md).

## Quick start

Validate every rule definition:

```bash
semgrep --validate --config rules
```

Scan a project with the complete portfolio:

```bash
semgrep scan --config /path/to/semgrep-rules/rules --error .
```

Run a single language/category pack:

```bash
semgrep scan \
  --config /path/to/semgrep-rules/rules/security/go-security.yml \
  --error .
```

Run the repository self-scan, excluding the rule source itself:

```bash
semgrep scan --config rules --error --exclude rules .
```

## Architecture profiles

The repository includes configuration guidance for:

- [Clean Architecture](docs/architectural-styles/clean-architecture.md)
- [Layered Architecture](docs/architectural-styles/layered-architecture.md)
- [Library & Package Design](docs/architectural-styles/library-package-design.md)

Architecture rules use conventional paths such as `domain`, `application`, and
`presentation`. Adapt or disable those opt-in rules when a project uses a
different layout.

## Rule metadata

Every rule declares its category, technology, confidence, scope, policy,
rationale, and references. `policy: opt-in` identifies rules that encode a
project or architectural preference rather than a language-wide defect.

## Scope notes

- Haskell rules use Semgrep's `generic` mode with Haskell metadata because the
  installed Semgrep engine does not provide a Haskell parser.
- Regex-based architecture and maintainability rules are heuristics; review
  findings against the project's conventions before enabling them as errors.
- The portfolio does not replace type checking, compilation, tests, or security
  review.
