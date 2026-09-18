# 04 - Layering, Boundaries & rules.toml Configuration

## Overview

Sentrux allows declarative architecture enforcement using `.sentrux/rules.toml`. This file defines global constraints, layered hierarchy order, and forbidden cross-boundary dependencies. When `sentrux check .` is run, any boundary or layering violation fails the check and penalizes the architecture score.

---

## Anatomy of `.sentrux/rules.toml`

### 1. `[constraints]`

Global constraints enforce codebase-wide invariants:

```toml
[constraints]
max_cycles = 0          # Zero tolerance for dependency cycles (Tarjan SCC)
max_coupling = "B"      # Maximum allowable coupling tier ("A", "B", "C")
max_cc = 20             # Maximum allowed cyclomatic complexity for any single function
no_god_files = true     # Forbid files exceeding maximum symbol/LOC density
min_quality = 8500      # Optional quality floor required to pass gate
```

- **`max_cycles = 0`**: Ensures the codebase remains a Directed Acyclic Graph (DAG).
- **`max_cc`**: Hard ceiling. While max_cc can be 20, high Sentrux Equality requires most functions to have $\text{CC} \le 2$.
- **`no_god_files`**: Triggers if a file has too many exports, responsibilities, or lines of code.

---

### 2. `[[layers]]`

Layers enforce the **Dependency Rule**: modules in outer/higher layers can depend on inner/lower layers, but **inner layers must never depend on outer layers**.

```toml
[[layers]]
name = "cli"
paths = ["src/bin/cli.rs", "src/cli/**"]
order = 0

[[layers]]
name = "api"
paths = ["src/bin/server.rs", "src/api/**"]
order = 0

[[layers]]
name = "workers"
paths = ["src/workers/**"]
order = 1

[[layers]]
name = "infra"
paths = ["src/infra/**"]
order = 2

[[layers]]
name = "domain"
paths = ["src/domain/**"]
order = 3

[[layers]]
name = "core"
paths = ["src/core/**"]
order = 4
```

#### Layer Order Mechanics
- In Sentrux, the order number establishes the hierarchy depth.
- Higher-order layers represent inner/core abstractions (e.g. `domain` and `core`).
- Lower-order layers represent delivery mechanisms or outer adapters (e.g. `cli`, `api`).
- An upward violation occurs if an inner layer attempts to import an outer layer.

---

### 3. `[[boundaries]]`

Boundaries explicitly forbid cross-module imports even between layers of the same order or horizontally related modules.

```toml
[[boundaries]]
from = "src/domain/**"
to = "src/api/**"
reason = "Domain logic must remain pure and never depend on HTTP presentation/routing"

[[boundaries]]
from = "src/domain/**"
to = "src/infra/**"
reason = "Domain logic must not depend on infrastructure adapters"

[[boundaries]]
from = "src/infra/**"
to = "src/api/**"
reason = "Infrastructure must not depend on HTTP API handlers"

[[boundaries]]
from = "src/api/**"
to = "src/ctl/**"
reason = "API server must never depend on CLI commands"

[[boundaries]]
from = "src/ctl/**"
to = "src/api/**"
reason = "CLI utility must be standalone and never depend on HTTP API handlers"
```

---

## Best Practices for Layering

1. **Keep `core` and `domain` completely pure**:
   - Only depend on `std`, serializing crates (`serde`), or pure algorithmic libraries.
   - Zero dependencies on Axum, Tokio, SQLx, or CLI frameworks (`clap`).
2. **Use Inversion of Control (Dependency Inversion)**:
   - If `domain` needs an email sender or database persistence, define the trait in `domain/ports.rs` (or `domain/repositories.rs`).
   - Implement the trait in `src/infra/email.rs` or `src/infra/db.rs`.
   - Pass the implementation via Axum `State` or function arguments.
3. **Prevent Horizontal Leakage**:
   - `src/ctl` and `src/api` should never import each other. Both are entry points that orchestrate lower layers.
