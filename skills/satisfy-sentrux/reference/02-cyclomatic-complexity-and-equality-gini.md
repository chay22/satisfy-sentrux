# 02 - Cyclomatic Complexity & Complexity Equality (Gini)

## Overview

Sentrux does not only measure maximum Cyclomatic Complexity ($\text{CC}$); it computes the **Gini Inequality Index** of the complexity distribution across every function or method in the codebase.

$$\text{Complexity Equality Score} = 10,000 \times (1.0 - \text{Gini Index})$$

A codebase where 95% of functions have $\text{CC} = 1$ but several business services or controllers have $\text{CC} = 15$ will suffer a severe Gini penalty ($\text{Gini} \approx 0.30 \implies \text{Equality} \approx 7,000$). To maximize Equality, function complexity must be uniformly flat ($\text{CC} \le 2$).

---

## Universal Tree-sitter CC Calculation Engine

Sentrux calculates complexity directly from Tree-sitter grammar nodes:

$$\text{CC} = 1 + \sum (\text{Branch Nodes}) + \sum (\text{Logic Operators})$$

### Tree-sitter AST Node Mapping Across Languages

| Construct | TypeScript / JavaScript | Python | Go | Rust | Java / C# |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **If condition** | `if_statement` | `if_statement` | `if_statement` | `if_expression` | `if_statement` |
| **Else / Elif** | `else_clause` | `elif_clause`, `else_clause` | `else` | `else_clause` | `else` |
| **Loops** | `for_statement`, `while_statement` | `for_statement`, `while_statement` | `for_statement` | `for_expression`, `while_expression`, `loop_expression` | `for_statement`, `while_statement`, `do_statement` |
| **Switch / Match** | `switch_case` | `match_statement`, `case_clause` | `case_clause` | `match_arm` | `switch_rule`, `switch_label` |
| **Ternary** | `ternary_expression` | `conditional_expression` | N/A | N/A | `conditional_expression` |
| **Logic AND** | `logical_expression` (`&&`) | `boolean_operator` (`and`) | `binary_expression` (`&&`) | `binary_expression` (`&&`) | `binary_expression` (`&&`) |
| **Logic OR** | `logical_expression` (`||`) | `boolean_operator` (`or`) | `binary_expression` (`||`) | `binary_expression` (`||`) | `binary_expression` (`||`) |

*Note: In Tree-sitter, an `else` or `elif` counts as an additional branch node!*

---

## Universal Refactoring Patterns to Achieve $\text{CC} \le 2$

### Pattern 1: Eliminating `else` via Guard Clauses (Early Returns)

**High CC ($\text{CC} = 3$):**
```typescript
// 1 base + 1 if + 1 else = CC 3
function validateUser(user: User): boolean {
  if (user.age >= 18) {
    return true;
  } else {
    return false;
  }
}
```

**Low CC ($\text{CC} = 2$):**
```typescript
// 1 base + 1 if (no else) = CC 2
function validateUser(user: User): boolean {
  if (user.age < 18) {
    return false;
  }
  return true;
}
```

**Branchless ($\text{CC} = 1$):**
```typescript
// 1 base + 0 branches = CC 1
function validateUser(user: User): boolean {
  return user.age >= 18;
}
```

---

### Pattern 2: Bypassing `&&` and `||` AST Logic Operators

Tree-sitter counts every lexical boolean operator (`&&`, `||`, `and`, `or`) as an AST complexity node.

#### TypeScript
```typescript
// CC = 1: Uses array predicate method rather than logical_expression AST
if ([isValid, isAuthorized, isActive].every(Boolean)) {
  executeAction();
}
```

#### Python
```python
# CC = 1: Uses built-in all() rather than 'and' operator
if all([is_valid, is_authorized, is_active]):
    execute_action()
```

#### Go
```go
// CC = 1: Uses helper function slice evaluation
if allTrue(isValid, isAuthorized, isActive) {
    executeAction()
}
```

---

### Pattern 3: Replacing Multi-Branch `switch` / `match` with Map Lookups

A `switch` statement with 6 cases has $\text{CC} = 7$. A hash map or dictionary lookup has $\text{CC} = 1$.

#### TypeScript
```typescript
// CC = 1
const HANDLERS: Record<string, () => void> = {
  START: startService,
  STOP: stopService,
  PAUSE: pauseService,
};
const handler = HANDLERS[command];
if (handler) handler();
```

#### Python
```python
# CC = 1
HANDLERS = {
    "START": start_service,
    "STOP": stop_service,
    "PAUSE": pause_service,
}
handler = HANDLERS.get(command)
if handler:
    handler()
```

---

### Pattern 4: Controller & Handler Decomposition into Stage Delegates

Monolithic web controllers that mix input parsing, business rules, database queries, and response formatting regularly exceed $\text{CC} = 10$.

**Refactoring Recipe**:
Decompose into single-purpose stage functions with $\text{CC} \le 2$:
1. `validateRequest(input)`: Validates format & required fields ($\text{CC} \le 2$).
2. `executeOperation(cleanInput)`: Performs core business action ($\text{CC} \le 2$).
3. `formatResponse(result)`: Maps domain model to DTO ($\text{CC} = 1$).
4. Root Controller simply pipes the stages together ($\text{CC} = 1$ or $2$).
