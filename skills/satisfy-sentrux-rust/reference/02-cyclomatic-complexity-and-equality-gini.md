# 02 - Cyclomatic Complexity & Complexity Equality (Gini)

## Overview

Sentrux does not only measure the maximum Cyclomatic Complexity ($\text{CC}$) of functions; it computes the **Gini Inequality Index** of the complexity distribution across all functions in the codebase.

$$\text{Complexity Equality Score} = 10,000 \times (1.0 - \text{Gini Index})$$

A codebase where most functions have $\text{CC} = 1$ but several handlers have $\text{CC} = 15$ will have a severe Gini penalty ($\text{Gini} \approx 0.35 \implies \text{Equality} \approx 6,500$). To maximize Equality, function complexity must be uniformly low ($\text{CC} \le 2$).

---

## Sentrux Tree-sitter CC Calculation Engine

Sentrux calculates function complexity using Tree-sitter AST nodes:

$$\text{CC} = 1 + \sum (\text{Branch Nodes}) + \sum (\text{Logic Operators})$$

### Branch Nodes Counted
- `if_expression`
- `else_clause` (Note: an `else if` or `else` adds an additional branch point!)
- `for_expression`
- `while_expression`
- `loop_expression`
- `match_arm`

### Logic Operators Counted
- `binary_expression` where operator is `&&` or `||`

---

## The Pareto Effect on Gini

If a codebase contains 300 functions:
- If 290 functions have $\text{CC} = 1$ and 10 functions have $\text{CC} = 10$:
  - Mean complexity $\approx 1.3$
  - Gini Index $\approx 0.28$
  - Equality score $\approx 7,200$
- If all 300 functions have $\text{CC} \in \{1, 2\}$:
  - Mean complexity $\approx 1.5$
  - Gini Index $\approx 0.17$
  - Equality score $\approx 8,300$
- If 90% of functions have $\text{CC} = 1$ and 10% have $\text{CC} = 2$:
  - Gini Index $\le 0.08$
  - Equality score $\ge 9,200$

---

## Systematic Rust Refactoring Patterns for $\text{CC} \le 2$

### Pattern 1: Eliminating `else_clause` via Early Returns

**High CC ($\text{CC} = 3$):**
```rust
// 1 base + 1 if + 1 else = CC 3
pub fn validate_amount(amount: i64) -> Result<(), AppError> {
    if amount <= 0 {
        Err(AppError::BadRequest("Amount must be positive".into()))
    } else {
        Ok(())
    }
}
```

**Low CC ($\text{CC} = 2$):**
```rust
// 1 base + 1 if (no else) = CC 2
pub fn validate_amount(amount: i64) -> Result<(), AppError> {
    if amount <= 0 {
        return Err(AppError::BadRequest("Amount must be positive".into()));
    }
    Ok(())
}
```

**Branchless ($\text{CC} = 1$):**
```rust
// 1 base + 0 branches = CC 1
pub fn validate_amount(amount: i64) -> Result<(), AppError> {
    (amount > 0)
        .then_some(())
        .ok_or_else(|| AppError::BadRequest("Amount must be positive".into()))
}
```

---

### Pattern 2: Domain Predicates vs. Boolean Complexity

Tree-sitter counts each `&&` and `||` in a `binary_expression` as a complexity node. However, **never game the metric** using artificial iterator arrays:

❌ **Metric Deception Anti-Pattern (DO NOT DO THIS):**
```rust
// Tricking AST count with iterator method call: loses short-circuit evaluation,
// allocates array, evaluates all terms eagerly, and harms readability!
if [role == "admin", is_active].into_iter().all(std::convert::identity) {
    grant_access();
}
```

✅ **Idiomatic Structural Pattern (Domain Predicate):**
Encapsulate multi-condition business rules into a dedicated predicate method on the domain model or an authorization helper:
```rust
impl User {
    #[inline]
    pub fn is_active_admin(&self) -> bool {
        self.role == Role::Admin && self.is_active
    }
}

// In handler: CC = 2 (or CC = 1 if inverted with guard clause)
if user.is_active_admin() {
    grant_access();
}
```
This keeps the caller clean ($\text{CC} \le 2$), maintains short-circuit evaluation, enables isolated unit testing, and expresses clear domain intent (`m09-domain`, `m05-type-driven`).

---

### Pattern 3: Decomposing Multi-Arm `match` Statements

Every match arm adds $+1$ to $\text{CC}$. A `match` with 6 arms has $\text{CC} = 7$.

**High CC ($\text{CC} = 7$):**
```rust
pub fn parse_command(cmd: &str) -> Option<Action> {
    match cmd {
        "start" => Some(Action::Start),
        "stop" => Some(Action::Stop),
        "pause" => Some(Action::Pause),
        "resume" => Some(Action::Resume),
        "reload" => Some(Action::Reload),
        _ => None,
    }
}
```

**Branchless Lookup ($\text{CC} = 1$):**
```rust
pub fn parse_command(cmd: &str) -> Option<Action> {
    const ACTIONS: &[(&str, Action)] = &[
        ("start", Action::Start),
        ("stop", Action::Stop),
        ("pause", Action::Pause),
        ("resume", Action::Resume),
        ("reload", Action::Reload),
    ];
    ACTIONS.iter().find(|(k, _)| *k == cmd).map(|(_, v)| *v)
}
```

---

### Pattern 4: Handler Decomposition into Stage Delegates

Long API handlers often mix validation, database query, authorization, and serialization, pushing $\text{CC} > 10$.

**Refactoring Strategy**:
Decompose into single-purpose stage functions with $\text{CC} \le 2$:
1. `extract_and_validate(...) -> Result<CleanInput, AppError>` ($\text{CC} \le 2$)
2. `execute_query(pool, input) -> Result<Model, AppError>` ($\text{CC} \le 2$)
3. `format_response(model) -> Json<ResponsePayload>` ($\text{CC} = 1$)
4. Main handler chains them with `?`:
```rust
pub async fn handle_create(
    State(ctx): State<AppContext>,
    Json(payload): Json<CreateRequest>,
) -> Result<Json<CreateResponse>, AppError> {
    let clean = validate_create_request(payload)?;
    let entity = insert_entity(&ctx.pool, clean).await?;
    Ok(Json(format_create_response(entity)))
}
```
All four functions have $\text{CC} \le 2$, driving the Gini variance toward zero.
