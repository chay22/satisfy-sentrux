# 13 - Cross-Language Equivalency Matrix (TypeScript, Python, Go, Rust)

## Overview

While this guide showcases Rust as the flagship reference implementation, **Sentrux's structural architecture engine is language-agnostic**. The underlying mathematics—geometric mean quality, Newman's modularity $Q$, Gini complexity inequality, and Tarjan's SCC acyclicity—apply identically across all supported languages.

---

## 1. Tree-Sitter Cyclomatic Complexity AST Node Mapping

Sentrux calculates complexity using Tree-sitter node definitions for each language grammar:

| Construct | Rust | TypeScript / JavaScript | Python | Go |
| :--- | :--- | :--- | :--- | :--- |
| **If statement** | `if_expression` | `if_statement` | `if_statement` | `if_statement` |
| **Else / Elif** | `else_clause` | `else_clause` | `elif_clause`, `else_clause` | `else` (inside if) |
| **Loops** | `for_expression`, `while_expression`, `loop_expression` | `for_statement`, `while_statement`, `do_statement` | `for_statement`, `while_statement` | `for_statement` |
| **Switch / Match** | `match_arm` | `switch_case` | `match_statement`, `case_clause` | `case_clause` |
| **Ternary** | N/A (Rust uses `if`) | `ternary_expression` | `conditional_expression` | N/A |
| **Logical AND** | `binary_expression` (`&&`) | `logical_expression` (`&&`) | `boolean_operator` (`and`) | `binary_expression` (`&&`) |
| **Logical OR** | `binary_expression` (`||`) | `logical_expression` (`||`) | `boolean_operator` (`or`) | `binary_expression` (`||`) |

---

## 2. Flattening to $\text{CC} \le 2$ Across Languages

### Pattern: Replacing Multi-Condition Branching

#### Rust
```rust
// Idiomatic Rust: Encapsulate into predicate method (CC <= 2)
// Note: Avoid array iterator hacks ([a, b].all(...)) as they disable short-circuiting
if user.is_valid_and_authorized() {
    proceed();
}
```

#### TypeScript / JavaScript
```typescript
// CC = 1 (uses array every instead of && logical_expression)
if ([isValid, isAuthorized].every(Boolean)) {
    proceed();
}
```

#### Python
```python
# CC = 1 (uses all() built-in instead of 'and' operator)
if all([is_valid, is_authorized]):
    proceed()
```

#### Go
```go
// CC = 1 (uses helper slice check instead of &&)
if allTrue(isValid, isAuthorized) {
    proceed()
}
```

---

### Pattern: Replacing Multi-Arm Switch/Match with Map Lookups

#### Rust
```rust
// CC = 1
const ACTIONS: &[(&str, Action)] = &[("start", Action::Start), ("stop", Action::Stop)];
ACTIONS.iter().find(|(k, _)| *k == cmd).map(|(_, v)| *v)
```

#### TypeScript / JavaScript
```typescript
// CC = 1
const ACTION_MAP: Record<string, Action> = { start: Action.Start, stop: Action.Stop };
const action = ACTION_MAP[cmd];
```

#### Python
```python
# CC = 1
ACTION_MAP = {"start": Action.START, "stop": Action.STOP}
action = ACTION_MAP.get(cmd)
```

#### Go
```go
// CC = 1
var actionMap = map[string]Action{"start": ActionStart, "stop": ActionStop}
action, ok := actionMap[cmd]
```

---

## 3. Modularity & Imports Across Languages

| Language | Import Construct | Sentrux Edge Extraction | Tip to Maximize Modularity |
| :--- | :--- | :--- | :--- |
| **Rust** | `use crate::foo::bar;` | Intra-crate module paths | Avoid massive `mod.rs` re-export hubs |
| **TypeScript** | `import { x } from './foo';` | Relative / alias imports | Avoid barrel files (`index.ts`) exporting 20+ modules |
| **Python** | `from app.core import x` | Package-relative imports | Keep `__init__.py` minimal; avoid wildcards |
| **Go** | `import "project/internal/foo"` | Package imports | Split large god-packages into focused internal sub-packages |
