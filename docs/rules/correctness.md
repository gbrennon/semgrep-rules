# Correctness Rules Reference

Opinionated Semgrep correctness rules for runtime bugs, logical traps, partial
operations, ignored failures, blocking calls, and language-specific pitfalls.
The detailed examples are representative. Language packs are:

- Python: `rules/correctness/python-correctness.yml`,
  `rules/correctness/python-exceptions.yml`
- Rust: `rules/correctness/rust-correctness.yml`
- Scala: `rules/correctness/scala-correctness.yml`
- Go: `rules/correctness/go-correctness.yml`
- Haskell: `rules/correctness/haskell-correctness.yml`
- TypeScript: `rules/correctness/typescript-correctness.yml`
- Java: `rules/correctness/java-correctness.yml`
- Kotlin: `rules/correctness/kotlin-correctness.yml`

## `python-no-bare-except`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/correctness/python-exceptions.yml`

### What it Enforces
Prohibits bare `except:` clauses without an explicit exception class.

### Why it Matters
A bare `except:` clause catches `BaseException`, intercepting system events such
as `KeyboardInterrupt` and `SystemExit`, preventing programs from terminating
cleanly.

### Non-Compliant Code
```python
try:
    process_data()
except:
    pass
```

### Compliant Code
```python
try:
    process_data()
except Exception as exc:
    logger.error("Processing failed: %s", exc)
```

## `python-assert-tuple-always-true`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/correctness/python-correctness.yml`

### What it Enforces
Prohibits asserting a 2-tuple `assert (expr, message)`.

### Why it Matters
In Python, assertions take the form `assert test, msg`. Enclosing both within
parentheses creates a 2-tuple. Non-empty tuples always evaluate to `True`,
meaning the assertion will never fail even if `expr` evaluates to `False`.

### Non-Compliant Code
```python
# Always evaluates to True because ('status != 200', 'Failed') is a non-empty tuple
assert (response.status == 200, "Unexpected status code")
```

### Compliant Code
```python
# Proper assert syntax: test expression followed by comma and message
assert response.status == 200, "Unexpected status code"
```

## `python-identity-vs-equality-literals`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/correctness/python-correctness.yml`

### What it Enforces
Disallows identity checks (`is` and `is not`) when comparing against string or
integer literals.

### Why it Matters
The `is` keyword checks for memory identity, not value equality. While CPython
interns small integers (-5 to 256) and some strings, this behavior is an
implementation detail. Relying on it causes sporadic comparison bugs.

### Non-Compliant Code
```python
if status is "active":
    proceed()

if count is 1000:
    limit_reached()
```

### Compliant Code
```python
if status == "active":
    proceed()

if count == 1000:
    limit_reached()

# Identity comparison remains correct for singletons
if user is None:
    authenticate()
```

## `python-silent-exception-swallow`

- **Severity:** `WARNING`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/correctness/python-correctness.yml`

### What it Enforces
Detects `try-except` blocks catching `Exception` or `BaseException` where the
body contains only `pass`.

### Why it Matters
Broadly catching all exceptions and silently ignoring them prevents bug
discovery, corrupts state, and makes production troubleshooting impossible.

### Non-Compliant Code
```python
try:
    sync_inventory()
except Exception:
    pass
```

### Compliant Code
```python
try:
    sync_inventory()
except InventoryError as err:
    logger.warning("Inventory sync failed: %s", err)
```

## `rust-no-mem-forget-on-lock`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/correctness/rust-correctness.yml`

### What it Enforces
Prohibits passing synchronization locks or guards (`MutexGuard`,
`RwLockReadGuard`) to `std::mem::forget`.

### Why it Matters
Forgetting a lock guard prevents its `Drop` implementation from executing,
permanently holding the mutex and causing deadlocks across any subsequent
acquirers.

### Non-Compliant Code
```rust
let guard = mutex.lock().unwrap();
// Dangerous: guard is forgotten; mutex will never unlock
std::mem::forget(guard);
```

### Compliant Code
```rust
{
    let guard = mutex.lock().unwrap();
    // Guard drops automatically at end of scope
}
```

## `rust-no-sleep-in-async-block`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/correctness/rust-correctness.yml`

### What it Enforces
Disallows calling `std::thread::sleep` inside async functions or async blocks.

### Why it Matters
`std::thread::sleep` blocks the OS thread assigned by the async runtime (e.g.
Tokio). Blocking a runtime worker stalls all other asynchronous tasks multiplexed
onto that worker thread.

### Non-Compliant Code
```rust
async fn poll_endpoint() {
    // Dangerous: halts the entire Tokio worker thread
    std::thread::sleep(std::time::Duration::from_millis(500));
}
```

### Compliant Code
```rust
async fn poll_endpoint() {
    // Non-blocking timer yields execution back to the runtime
    tokio::time::sleep(tokio::time::Duration::from_millis(500)).await;
}
```

## `scala-unsafe-result-get`

- **Severity:** `WARNING`
- **Languages:** Scala
- **Confidence:** `MEDIUM`
- **File:** `rules/correctness/scala-correctness.yml`

### What it Enforces
Flags unsafe `get` calls on optional or failure-carrying results, including
`Option.get` and `Try.get`.

### Non-Compliant Code
```scala
val customer = customerRepository.find(id).get
val response = Try(callRemoteService()).get
```

### Compliant Code
```scala
val customer = customerRepository.find(id).fold(defaultCustomer)(identity)
val response = Try(callRemoteService()).fold(handleFailure, identity)
```

## `scala-blocking-await-result`

- **Severity:** `WARNING`
- **Languages:** Scala
- **Confidence:** `HIGH`
- **File:** `rules/correctness/scala-correctness.yml`

### What it Enforces
Flags `Await.result` calls that synchronously block while a Future completes.

### Non-Compliant Code
```scala
val result = Await.result(service.fetch(), 5.seconds)
```

### Compliant Code
```scala
service.fetch().map(render)
```

## `scala-blocking-thread-sleep`

- **Severity:** `WARNING`
- **Languages:** Scala
- **Confidence:** `HIGH`
- **File:** `rules/correctness/scala-correctness.yml`

### What it Enforces
Flags `Thread.sleep`, which blocks the current execution thread.

### Non-Compliant Code
```scala
Thread.sleep(1000)
```

### Compliant Code
```scala
scheduler.scheduleOnce(1.second)(continueWork)
```
