# 04 - Layering, Boundaries & rules.toml Specification

## Overview

Sentrux uses a declarative configuration file, `.sentrux/rules.toml`, located at the repository root. This file establishes global architectural invariants, layer hierarchy order, and forbidden boundary crossings across any programming language.

When `sentrux check .` is executed, any violated rule fails the build and reduces the structural score.

---

## 1. `[constraints]` Table

```toml
[constraints]
max_cycles = 0          # Maximum allowable cycle edges (0 ensures a clean DAG)
max_coupling = "B"      # Maximum allowable coupling tier ("A", "B", "C")
max_cc = 15             # Hard upper limit for function cyclomatic complexity
no_god_files = true     # Forbid files exceeding maximum symbol/LOC density
min_quality = 8500      # Optional quality threshold required to pass the gate
```

---

## 2. `[[layers]]` Tables

Layers enforce the universal **Dependency Rule**: outer delivery mechanisms depend on inner domain logic, but **inner business logic must never depend on outer layers**.

```toml
[[layers]]
name = "presentation"
paths = ["src/api/**", "src/controllers/**", "src/views/**"]
order = 0

[[layers]]
name = "workers"
paths = ["src/workers/**", "src/jobs/**", "src/queues/**"]
order = 1

[[layers]]
name = "infrastructure"
paths = ["src/infra/**", "src/database/**", "src/clients/**"]
order = 2

[[layers]]
name = "application"
paths = ["src/application/**", "src/services/**", "src/usecases/**"]
order = 3

[[layers]]
name = "domain"
paths = ["src/domain/**", "src/models/**", "src/entities/**"]
order = 4

[[layers]]
name = "core"
paths = ["src/core/**", "src/common/**", "src/errors/**"]
order = 5
```

### Layer Order Mechanics
- Lower order numbers (e.g. `order = 0`) represent outer layers (entry points, HTTP/CLI controllers).
- Higher order numbers (e.g. `order = 4`) represent inner core business logic.
- An **upward violation** is flagged whenever an inner layer attempts to import code from an outer layer.

---

## 3. `[[boundaries]]` Tables

Boundaries explicitly forbid cross-module imports between specific folders, even when they reside on the same layer or across horizontal boundaries.

```toml
[[boundaries]]
from = "src/domain/**"
to = "src/presentation/**"
reason = "Domain entities must remain transport-agnostic and never import presentation logic"

[[boundaries]]
from = "src/domain/**"
to = "src/infrastructure/**"
reason = "Domain entities must never import concrete database clients or external HTTP drivers"

[[boundaries]]
from = "src/application/**"
to = "src/presentation/**"
reason = "Application use-cases must not depend on HTTP controllers"

[[boundaries]]
from = "src/cli/**"
to = "src/api/**"
reason = "CLI commands must be self-contained and not depend on HTTP API server handlers"
```

---

## Layering Best Practices Across Stacks

1. **Pure Domain / Entities**:
   - Zero framework dependencies (no Express/NestJS in TS, no Django/Flask in Python, no Gin/Echo in Go, no Axum/Tokio in Rust).
2. **Dependency Inversion**:
   - Define interfaces/ports in the `domain` or `application` layer (e.g. `UserRepository`, `EmailGateway`).
   - Implement the interface in the `infrastructure` layer (e.g. `PostgresUserRepository`, `SendgridEmailGateway`).
   - Inject the implementation at the application entrypoint (composition root).
