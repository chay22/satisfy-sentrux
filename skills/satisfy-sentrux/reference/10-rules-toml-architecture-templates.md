# 10 - Production `rules.toml` Architecture Templates

## Overview

This reference provides drop-in `.sentrux/rules.toml` configurations for popular software architecture styles across any programming language.

---

## Template 1: Clean Architecture / Hexagonal (Ports & Adapters)

Applicable to TypeScript, Python, Go, Java, and C# services requiring strict decoupling of domain business logic from databases and transport frameworks.

```toml
[constraints]
max_cycles = 0
max_coupling = "B"
max_cc = 10
no_god_files = true
min_quality = 8500

[[layers]]
name = "adapters-in"
paths = ["src/adapters/in/**", "src/api/**", "src/controllers/**"]
order = 0

[[layers]]
name = "adapters-out"
paths = ["src/adapters/out/**", "src/infra/**", "src/gateways/**"]
order = 1

[[layers]]
name = "application"
paths = ["src/application/**", "src/services/**", "src/usecases/**"]
order = 2

[[layers]]
name = "domain"
paths = ["src/domain/**", "src/entities/**"]
order = 3

[[layers]]
name = "core"
paths = ["src/core/**", "src/common/**"]
order = 4

# Domain cannot import infrastructure or delivery
[[boundaries]]
from = "src/domain/**"
to = "src/adapters/**"
reason = "Domain entities must remain transport- and database-agnostic"

[[boundaries]]
from = "src/domain/**"
to = "src/infra/**"
reason = "Domain entities must not depend on concrete infrastructure"

# Application services cannot depend on external delivery
[[boundaries]]
from = "src/application/**"
to = "src/api/**"
reason = "Use cases must not depend on HTTP/REST controllers"
```

---

## Template 2: Modern Web API (Node/TS, Python FastAPI, Go)

Tailored for standard REST/GraphQL microservices with controllers, services, database models, and background workers.

```toml
[constraints]
max_cycles = 0
max_coupling = "B"
max_cc = 10
no_god_files = true
min_quality = 8500

[[layers]]
name = "entrypoints"
paths = ["src/server.*", "src/app.*", "src/main.*"]
order = 0

[[layers]]
name = "presentation"
paths = ["src/routes/**", "src/controllers/**", "src/handlers/**"]
order = 1

[[layers]]
name = "workers"
paths = ["src/workers/**", "src/jobs/**", "src/queues/**"]
order = 2

[[layers]]
name = "services"
paths = ["src/services/**", "src/logic/**"]
order = 3

[[layers]]
name = "infrastructure"
paths = ["src/infra/**", "src/db/**", "src/repositories/**"]
order = 4

[[layers]]
name = "domain"
paths = ["src/domain/**", "src/models/**"]
order = 5

[[layers]]
name = "common"
paths = ["src/common/**", "src/utils/**", "src/errors/**"]
order = 6

[[boundaries]]
from = "src/domain/**"
to = "src/presentation/**"
reason = "Domain models must not import HTTP controllers"

[[boundaries]]
from = "src/services/**"
to = "src/presentation/**"
reason = "Business services must not depend on HTTP presentation"
```

---

## Template 3: Monorepo / Multi-Package Layout

For monorepos (e.g. Nx, Turborepo, Go workspaces) containing multiple packages:

```toml
[constraints]
max_cycles = 0
max_coupling = "A"
max_cc = 12
no_god_files = true
min_quality = 8800

[[layers]]
name = "apps"
paths = ["apps/**"]
order = 0

[[layers]]
name = "features"
paths = ["packages/feature-*/**"]
order = 1

[[layers]]
name = "ui"
paths = ["packages/ui/**"]
order = 2

[[layers]]
name = "data"
paths = ["packages/data-access/**", "packages/api-client/**"]
order = 3

[[layers]]
name = "shared"
paths = ["packages/shared/**", "packages/core/**"]
order = 4

[[boundaries]]
from = "packages/shared/**"
to = "packages/data-access/**"
reason = "Core shared primitives must not depend on data-access packages"

[[boundaries]]
from = "packages/ui/**"
to = "apps/**"
reason = "UI components must not import applications"
```
