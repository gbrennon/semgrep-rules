# Maintainability Rules Reference

Semgrep maintainability rules focused on reducing cognitive load, preventing
stateful mutation bugs, and maintaining manageable cyclomatic complexity.

## `python-no-mutable-default-arguments`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/maintainability/python-function-defaults.yml`

### What it Enforces
Prohibits using mutable literals or constructors (`[]`, `{}`, `set()`, `list()`,
`dict()`) as default argument values in function definitions.

### Why it Matters
In Python, default parameter expressions are evaluated once when the function is
defined, not each time it is called. A mutable default is shared across every
invocation, causing state to leak between calls.

### Non-Compliant Code
```python
def append_item(item, items=[]):
    items.append(item)
    return items

append_item(1)  # returns [1]
append_item(2)  # returns [1, 2] -- shared state!
```

### Compliant Code
```python
def append_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

## `rust-no-control-flow-nesting-above-2`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/maintainability/rust-control-flow.yml`

### What it Enforces
Disallows nesting control flow statements (`if`, `for`, `while`, `loop`,
`match`) deeper than 2 levels.

### Why it Matters
Deeply nested control structures significantly increase cognitive load, elevate
cyclomatic complexity, and make edge cases difficult to reason about. Deep
nesting should be refactored using early returns, guard clauses, or extracted
helper functions.

### Non-Compliant Code
```rust
fn process(items: &[Item]) {
    for item in items {           // Level 1
        if item.is_active() {     // Level 2
            while item.has_work() { // Level 3: FAILS rule
                item.step();
            }
        }
    }
}
```

### Compliant Code
```rust
fn process(items: &[Item]) {
    for item in items {
        if item.is_active() {
            drain_item(item);     // Extracted helper
        }
    }
}

fn drain_item(item: &Item) {
    while item.has_work() {
        item.step();
    }
}
```

## `scala-control-flow-nesting-above-2`

- **Severity:** `WARNING`
- **Languages:** Scala
- **Confidence:** `MEDIUM`
- **File:** `rules/maintainability/scala-control-flow.yml`

### What it Enforces
Flags three nested `if` blocks as a signal to apply guard clauses or extract a
cohesive helper.

### Non-Compliant Code
```scala
if (request.isValid) {
  if (user.isActive) {
    if (account.hasCredit) {
      charge(account)
    }
  }
}
```

### Compliant Code
```scala
if (!request.isValid || !user.isActive || !account.hasCredit) return
charge(account)
```

The rule is intentionally a conservative heuristic; review generated code and
formatting conventions before enabling it as an error.
