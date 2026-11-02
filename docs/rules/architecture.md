# Architecture & Boundary Rules Reference

Semgrep architecture rules enforcing Clean Architecture, Hexagonal boundaries,
and strict dependency direction across Domain, Application, and Presentation
layers in Python and Rust.

## `python-domain-no-framework-imports`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **Scope:** `**/domain/**`
- **File:** `rules/architecture/python-architecture.yml`

### What it Enforces
Prohibits modules in `domain` from importing web frameworks, ORMs, or network
libraries (e.g. `django`, `flask`, `fastapi`, `sqlalchemy`, `requests`).

### Why it Matters
The Domain layer represents the pure enterprise core. It must remain decoupled
from external frameworks and database tools so business logic is independently
testable and durable against infrastructure changes.

### Non-Compliant Code
```python
# In domain/order.py
from fastapi import HTTPException
from sqlalchemy import Column, Integer
```

### Compliant Code
```python
# In domain/order.py
from dataclasses import dataclass

@dataclass(frozen=True)
class Order:
    order_id: str
    total_amount: int
```

## `python-application-no-presentation-dependencies`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **Scope:** `**/application/**`
- **File:** `rules/architecture/python-architecture.yml`

### What it Enforces
Prohibits modules in `application` from importing presentation components,
views, or HTTP controllers.

### Why it Matters
In Clean Architecture, dependencies point strictly inward. Application use cases
orchestrate domain behavior and communicate with the outside world via ports.
They must never couple to delivery mechanisms.

### Non-Compliant Code
```python
# In application/create_order_use_case.py
from myproject.presentation.views import render_order_response
```

### Compliant Code
```python
# In application/create_order_use_case.py
from myproject.application.dto import CreateOrderResponse
```

## `python-presentation-must-be-thin-delegate`

- **Severity:** `ERROR`
- **Languages:** Python
- **Confidence:** `HIGH`
- **Scope:** `**/presentation/**`, `**/views/**`, `**/controllers/**`, `**/api/**`
- **File:** `rules/architecture/python-architecture.yml`

### What it Enforces
Disallows direct database queries or raw ORM calls in presentation controllers.

### Why it Matters
Presentation handlers should remain simple delegators: accept an HTTP request,
parse and validate the input DTO, call the application use case, and serialize
the result into a response DTO. Direct database queries bypass security rules,
transactions, and domain validations.

### Non-Compliant Code
```python
# In presentation/order_controller.py
@router.post("/orders")
def create_order(req: OrderRequest, db: Session = Depends(get_db)):
    return db.query(OrderModel).filter(OrderModel.id == req.id).first()
```

### Compliant Code
```python
# In presentation/order_controller.py
@router.post("/orders")
def create_order(req: OrderRequest, use_case: CreateOrderUseCase = Depends()):
    result = use_case.execute(req.to_dto())
    return OrderViewModel.from_dto(result)
```

## `rust-domain-no-infrastructure-dependencies`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **Scope:** `**/domain/**`
- **File:** `rules/architecture/rust-layer-boundaries.yml`

### What it Enforces
Prevents domain code from importing the `infrastructure` layer.

### Why it Matters
Technical details (database adapters, filesystem access, network protocols) must
depend on domain abstractions, never the reverse.

### Non-Compliant Code
```rust
// In domain/entity.rs
use crate::infrastructure::database::PostgresClient;
```

### Compliant Code
```rust
// In domain/ports.rs
pub trait UserRepository {
    fn find_by_id(&self, id: UserId) -> Option<User>;
}
```

## `rust-domain-no-presentation-dependencies`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **Scope:** `**/domain/**`
- **File:** `rules/architecture/rust-layer-boundaries.yml`

### What it Enforces
Prevents domain code from referencing the `presentation` layer (CLI/TUI/HTTP).

## `rust-domain-no-application-dependencies`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **Scope:** `**/domain/**`
- **File:** `rules/architecture/rust-layer-boundaries.yml`

### What it Enforces
Prevents domain code from depending on application use-case orchestration.

## `rust-application-no-presentation-dependencies`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **Scope:** `**/application/**`
- **File:** `rules/architecture/rust-layer-boundaries.yml`

### What it Enforces
Prohibits the application orchestration layer from referencing presentation
modules.

## `rust-presentation-must-delegate-no-database`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **Scope:** `**/presentation/**`
- **File:** `rules/architecture/rust-layer-boundaries.yml`

### What it Enforces
Prohibits presentation handlers from issuing raw database queries (e.g.
`sqlx::query`). Handlers must delegate to application use cases.

## `rust-domain-service-no-sibling-service-dependencies`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **Scope:** `**/domain/**/services/**`
- **File:** `rules/architecture/rust-service-boundaries.yml`

### What it Enforces
Prevents domain services from importing other domain services directly. Sibling
service coupling creates circular dependencies and tangled graphs.

## `rust-application-service-no-sibling-service-dependencies`

- **Severity:** `ERROR`
- **Languages:** Rust
- **Confidence:** `HIGH`
- **Scope:** `**/application/**/services/**`
- **File:** `rules/architecture/rust-service-boundaries.yml`

### What it Enforces
Prevents application services from depending directly on sibling application
services. Cross-service coordination must occur through domain events or
top-level composition roots.
