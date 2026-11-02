# Conventions Rules Reference

Semgrep convention rules standardizing module layouts, import hygiene, and
source structure in Python and Rust.

## `python-no-wildcard-imports`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/conventions/python-imports.yml`

### What it Enforces
Prohibits wildcard imports of the form `from module import *`.

### Why it Matters
Wildcard imports pollute the local namespace, obscure the origin of functions
and classes, defeat static analysis tools, and risk silent variable shadowing
when upstream modules introduce new exports.

### Non-Compliant Code
```python
from math import *

def calculate(x):
    return sin(x) + cos(x)
```

### Compliant Code
```python
import math
from math import cos, sin

def calculate(x):
    return sin(x) + cos(x)
```

## `rust-no-inline-modules`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/conventions/rust-module-layout.yml`

### What it Enforces
Disallows defining inline modules `mod foo { ... }` in production code,
exempting test modules named `test` or `tests`.

### Why it Matters
Inline modules encourage monolithic files with nested scopes. Rust module
declarations should map directly to the filesystem (`mod foo;` in a file with
`foo.rs` or `foo/mod.rs`), improving navigability and maintainability.

### Non-Compliant Code
```rust
// In src/lib.rs
mod helpers {
    pub fn calculate() {}
}
```

### Compliant Code
```rust
// In src/lib.rs
mod helpers;

// In src/helpers.rs
pub fn calculate() {}

// Test modules remain allowed inline
#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn test_calc() {}
}
```
