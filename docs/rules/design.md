# Design & Encapsulation Rules Reference

Opinionated Semgrep design rules enforcing structural encapsulation, single
responsibility, controlled mutation, and cohesive boundaries. The detailed
examples are representative. Language packs are:

- Python: `rules/design/python-design.yml`
- Rust: `rules/design/rust-encapsulation.yml`
- Scala: `rules/design/scala-encapsulation.yml`
- Go: `rules/design/go-encapsulation.yml`
- Haskell: `rules/design/haskell-encapsulation.yml`
- TypeScript: `rules/design/typescript-encapsulation.yml`
- Java: `rules/design/java-encapsulation.yml`
- Kotlin: `rules/design/kotlin-encapsulation.yml`

## `python-class-max-5-public-methods`

- **Severity:** `WARNING`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/design/python-design.yml`

### What it Enforces
Flags classes declaring more than 5 distinct public methods.

### Why it Matters
Classes with extensive public interfaces tend to accumulate disparate
responsibilities, violating the Single Responsibility Principle (SRP). Capping
public methods encourages decomposing monoliths into focused, composable
collaborators.

### Non-Compliant Code
```python
class UserManager:
    def create_user(self): ...
    def update_user(self): ...
    def delete_user(self): ...
    def send_welcome_email(self): ...
    def generate_billing_pdf(self): ...
    def export_audit_log(self): ...  # >5 public methods
```

### Compliant Code
```python
class UserService:
    def create_user(self): ...
    def update_user(self): ...
    def delete_user(self): ...

class UserNotificationService:
    def send_welcome_email(self): ...

class UserBillingService:
    def generate_billing_pdf(self): ...
```

## `python-class-max-3-private-methods`

- **Severity:** `WARNING`
- **Languages:** Python
- **Confidence:** `HIGH`
- **File:** `rules/design/python-design.yml`

### What it Enforces
Flags classes declaring more than 3 distinct private helper methods.

### Why it Matters
When a class requires four or more private helpers, it typically harbors a
hidden abstraction or complex sub-routine that belongs in its own dedicated
collaborator or strategy object.

### Non-Compliant Code
```python
class OrderProcessor:
    def process(self): ...
    def _validate_items(self): ...
    def _calculate_taxes(self): ...
    def _apply_coupons(self): ...
    def _format_receipt(self): ...  # >3 private helpers
```

### Compliant Code
```python
class TaxCalculator:
    def calculate(self, items): ...

class OrderProcessor:
    def __init__(self, tax_calculator: TaxCalculator):
        self._tax_calculator = tax_calculator
    def process(self): ...
    def _validate_items(self): ...
```

## `python-no-private-member-access`

- **Severity:** `WARNING`
- **Languages:** Python
- **Confidence:** `MEDIUM`
- **File:** `rules/design/python-design.yml`

### What it Enforces
Flags external callers reading or modifying attributes prefixed with a single
underscore (e.g. `obj._secret`).

### Why it Matters
Python relies on the single-underscore convention to designate implementation
internals. External callers accessing private members bind themselves to
unstable implementation details, breaking encapsulation.

### Non-Compliant Code
```python
account = BankAccount()
account._balance += 500  # Bypassing deposit validation
```

### Compliant Code
```python
account = BankAccount()
account.deposit(500)
```

## `rust-impl-max-5-public-methods`

- **Severity:** `WARNING`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/design/rust-encapsulation.yml`

### What it Enforces
Flags `impl` blocks defining more than 5 distinct `pub fn` methods.

### Why it Matters
Encourages small, decoupled structs and trait segregation in Rust, preventing
broad struct APIs that become maintenance bottlenecks.

### Non-Compliant Code
```rust
impl OrderService {
    pub fn create(&self) {}
    pub fn update(&self) {}
    pub fn cancel(&self) {}
    pub fn refund(&self) {}
    pub fn notify(&self) {}
    pub fn export(&self) {}  // >5 public methods
}
```

### Compliant Code
```rust
impl OrderService {
    pub fn create(&self) {}
    pub fn update(&self) {}
    pub fn cancel(&self) {}
}

impl OrderAccountingService {
    pub fn refund(&self) {}
}
```

## `rust-impl-max-3-private-methods`

- **Severity:** `WARNING`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/design/rust-encapsulation.yml`

### What it Enforces
Flags `impl` blocks defining more than 3 private helper methods.

### Why it Matters
Numerous private functions in an `impl` block indicate that the struct is
orchestrating multiple distinct algorithms that should be encapsulated into
focused internal collaborators.

## `rust-struct-fields-must-not-be-public`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/design/rust-encapsulation.yml`

### What it Enforces
Requires struct fields to remain private (`pub` fields are rejected).

### Why it Matters
Exposing public fields allows external callers to mutate state directly,
bypassing constructors, validation routines, and domain invariants.

### Non-Compliant Code
```rust
pub struct User {
    pub id: u64,
    pub email: String,
}
```

### Compliant Code
```rust
pub struct User {
    id: u64,
    email: String,
}

impl User {
    pub fn new(id: u64, email: String) -> Result<Self, UserError> { ... }
    pub fn email(&self) -> &str { &self.email }
}
```

## `rust-avoid-public-tuple-struct-fields`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/design/rust-encapsulation.yml`

### What it Enforces
Disallows public inner fields in tuple structs (e.g. `pub struct UserId(pub u64)`).

### Why it Matters
Tuple structs represent domain value objects. Exposing their inner value
permits callers to instantiate arbitrary, unvalidated values without going
through domain validation constructors.

### Non-Compliant Code
```rust
pub struct EmailAddress(pub String);
```

### Compliant Code
```rust
pub struct EmailAddress(String);

impl EmailAddress {
    pub fn parse(raw: String) -> Result<Self, EmailError> { ... }
    pub fn as_str(&self) -> &str { &self.0 }
}
```

## `rust-no-setters`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **File:** `rules/design/rust-encapsulation.yml`

### What it Enforces
Prohibits methods named with the `set_` prefix.

### Why it Matters
Generic setters encourage anemic domain models where objects are treated as
passive data structures. Domain logic should be driven by intent-revealing
operations rather than raw field updates.

### Non-Compliant Code
```rust
impl Order {
    pub fn set_status(&mut self, status: Status) {
        self.status = status;
    }
}
```

### Compliant Code
```rust
impl Order {
    pub fn mark_as_shipped(&mut self, tracking_number: TrackingNumber) {
        self.status = Status::Shipped(tracking_number);
    }
}
```

## Scala Encapsulation Rules

### `scala-public-mutable-class-members`

- **Severity:** `WARNING`
- **Languages:** Scala
- **Confidence:** `MEDIUM`
- **File:** `rules/design/scala-encapsulation.yml`

Flags `var` members declared inside classes. Prefer private state and methods
that enforce valid state transitions.

```scala
// Non-compliant
class Account {
  var balance: BigDecimal = 0
}

// Compliant
final class Account private (private var balance: BigDecimal) {
  def deposit(amount: BigDecimal): Unit =
    require(amount > 0)
    balance += amount
}
```

### `scala-no-setter-methods`

- **Severity:** `WARNING`
- **Languages:** Scala
- **Confidence:** `HIGH`
- **File:** `rules/design/scala-encapsulation.yml`

Flags conventional `setX` methods. Prefer operations that express domain intent
and validate transitions.
