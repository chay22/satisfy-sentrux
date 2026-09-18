# 12 - Production `rules.toml` Architecture Templates

## Overview

This reference provides drop-in `.sentrux/rules.toml` templates for common architectural paradigms. Copy and adapt these configurations to establish robust structural boundaries from day one.

---

## Template 1: Clean Architecture / Hexagonal (Ports & Adapters)

Ideal for enterprise services requiring strict decoupling of domain business logic from databases, external APIs, and transport layers.

```toml
[constraints]
max_cycles = 0
max_coupling = "B"
max_cc = 10
no_god_files = true
min_quality = 8500

[[layers]]
name = "adapters-in"
paths = ["src/adapters/in/**", "src/api/**", "src/cli/**"]
order = 0

[[layers]]
name = "adapters-out"
paths = ["src/adapters/out/**", "src/infra/**"]
order = 1

[[layers]]
name = "application"
paths = ["src/application/**", "src/services/**"]
order = 2

[[layers]]
name = "domain"
paths = ["src/domain/**"]
order = 3

[[layers]]
name = "core"
paths = ["src/core/**"]
order = 4

# Domain cannot import infrastructure or transport
[[boundaries]]
from = "src/domain/**"
to = "src/adapters/**"
reason = "Domain entities must remain pure and transport-agnostic"

[[boundaries]]
from = "src/domain/**"
to = "src/infra/**"
reason = "Domain must not depend on database or external infrastructure"

# Application services cannot depend on external delivery
[[boundaries]]
from = "src/application/**"
to = "src/api/**"
reason = "Use cases must not depend on HTTP/REST controllers"
```

---

## Template 2: Standard Rust Web API (Axum / Actix-web)

Tailored for high-performance HTTP microservices with background workers and database access.

```toml
[constraints]
max_cycles = 0
max_coupling = "B"
max_cc = 8
no_god_files = true
min_quality = 8500

[[layers]]
name = "entrypoints"
paths = ["src/bin/**", "src/main.rs"]
order = 0

[[layers]]
name = "presentation"
paths = ["src/api/**", "src/routes/**", "src/handlers/**"]
order = 1

[[layers]]
name = "workers"
paths = ["src/workers/**", "src/jobs/**"]
order = 2

[[layers]]
name = "infra"
paths = ["src/infra/**", "src/db/**", "src/clients/**"]
order = 3

[[layers]]
name = "domain"
paths = ["src/domain/**", "src/models/**"]
order = 4

[[layers]]
name = "core"
paths = ["src/core/**", "src/common/**"]
order = 5

[[boundaries]]
from = "src/domain/**"
to = "src/api/**"
reason = "Domain entities must not depend on HTTP handlers"

[[boundaries]]
from = "src/domain/**"
to = "src/infra/**"
reason = "Domain must not depend on database clients"

[[boundaries]]
from = "src/infra/**"
to = "src/api/**"
reason = "Database/infrastructure must not depend on presentation handlers"
```

---

## Template 3: Command-Line Interface (CLI / TUI)

Ideal for developer tools, terminal utilities (using `clap` or `ratatui`), and control binaries.

```toml
[constraints]
max_cycles = 0
max_coupling = "A"
max_cc = 10
no_god_files = true
min_quality = 8800

[[layers]]
name = "bin"
paths = ["src/bin/**", "src/main.rs"]
order = 0

[[layers]]
name = "tui"
paths = ["src/tui/**", "src/views/**"]
order = 1

[[layers]]
name = "commands"
paths = ["src/commands/**", "src/cli/**"]
order = 1

[[layers]]
name = "logic"
paths = ["src/logic/**", "src/engine/**"]
order = 2

[[layers]]
name = "core"
paths = ["src/core/**", "src/utils/**"]
order = 3

[[boundaries]]
from = "src/logic/**"
to = "src/commands/**"
reason = "Execution engine must not depend on CLI argument parsing"

[[boundaries]]
from = "src/logic/**"
to = "src/tui/**"
reason = "Core engine must not depend on terminal UI rendering"
```
