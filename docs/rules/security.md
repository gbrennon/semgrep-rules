# Security Rules Reference

Defensive Semgrep security rules designed to prevent command injection, unsafe
deserialization, arbitrary code execution, and dangerous memory operations in
Python and Rust applications.

## `python-no-eval-exec`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/security/python-security.yml`

### What it Enforces
Disallows direct invocations of dynamic code evaluation built-ins `eval(...)`
and `exec(...)`.

### Why it Matters
Evaluating arbitrary strings as executable Python code bypasses static analysis,
defeats compiler and type invariants, and allows attackers to achieve remote
code execution (RCE) if user input reaches the evaluation context.

### Non-Compliant Code
```python
# Dangerous: user-supplied expression executed directly
result = eval(request.GET.get("formula"))
exec(f"config_{module} = True")
```

### Compliant Code
```python
import ast

# Safe: parse literals safely without execution
result = ast.literal_eval(request.GET.get("formula"))

# Safe: explicit mapping or dictionary lookup
configs[module] = True
```

## `python-subprocess-shell-true`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/security/python-security.yml`

### What it Enforces
Prohibits passing `shell=True` to any `subprocess` invocation (`run`, `Popen`,
`call`, `check_call`, `check_output`).

### Why it Matters
When `shell=True` is enabled, the command string is evaluated by the system
shell (`/bin/sh` or `cmd.exe`). If any variable in that command contains shell
metacharacters (e.g. `;`, `&&`, `|`), arbitrary commands will be executed in the
host environment.

### Non-Compliant Code
```python
import subprocess

# Vulnerable to shell command injection
subprocess.run(f"cat {user_filename}", shell=True)
```

### Compliant Code
```python
import subprocess

# Safe: arguments passed as an array without shell interpolation
subprocess.run(["cat", user_filename], check=True)
```

## `python-insecure-deserialization-pickle`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/security/python-security.yml`

### What it Enforces
Disallows unpickling byte streams using `pickle.load`, `pickle.loads`,
`_pickle.load`, or `_pickle.loads`.

### Why it Matters
The Python `pickle` protocol is not safe against malicious data. An attacker can
craft a serialized payload with a custom `__reduce__` method that spawns a
shell or executes arbitrary system commands upon unpickling.

### Non-Compliant Code
```python
import pickle

# Dangerous: arbitrary bytecode executed during unpickling
user_data = pickle.loads(raw_cookie_payload)
```

### Compliant Code
```python
import json

# Safe: structured data interchange with safe parsers
user_data = json.loads(raw_cookie_payload)
```

## `rust-no-command-raw-sh-c`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/security/rust-security.yml`

### What it Enforces
Prohibits invoking shell binaries (`sh`, `bash`, `cmd`, `cmd.exe`) with the
command-string argument (`-c` or `/c`).

### Why it Matters
Rust's `std::process::Command` by default passes arguments directly to OS
syscalls (`execve`) without invoking a shell, providing built-in immunity to
shell injection. Explicitly invoking `sh -c` re-introduces shell parsing
vulnerabilities.

### Non-Compliant Code
```rust
use std::process::Command;

// Vulnerable: shell string interpolation
Command::new("sh")
    .arg("-c")
    .arg(format!("cat {}", user_input))
    .output();
```

### Compliant Code
```rust
use std::process::Command;

// Safe: direct binary invocation with distinct arguments
Command::new("cat")
    .arg(user_input)
    .output();
```

## `rust-no-insecure-random-transmute`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/security/rust-security.yml`

### What it Enforces
Detects `std::mem::transmute` or `core::mem::transmute` used to coerce raw byte
arrays or uninitialized buffers into structured values.

### Why it Matters
`mem::transmute` is one of Rust's most dangerous primitives. It bypasses
type-system invariants and alignment requirements. Transmuting unvalidated
buffers can result in undefined behavior, invalid enum discriminants, or memory
corruption.

### Non-Compliant Code
```rust
// Dangerous: unchecked memory transmutation
let token: MySecretToken = unsafe { std::mem::transmute(random_bytes) };
```

### Compliant Code
```rust
// Safe: safe constructors and explicit decoding
let token = MySecretToken::from_bytes(&random_bytes)?;
```
