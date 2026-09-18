# 03 - Modularity ($Q$) & Community Graph

## Overview

Sentrux measures structural modularity using **Newman's Modularity ($Q$)**, a network science metric that quantifies how well a network is partitioned into cohesive, loosely coupled communities.

$$\text{Modularity Score} = 10,000 \times \frac{Q + 0.5}{1.5}$$

where $Q \in [-0.5, 1.0]$. A modularity score of **7,200+** requires $Q \ge 0.58$, and **8,000+** requires $Q \ge 0.70$.

---

## Newman's $Q$ Formula in Sentrux

Let $m$ be the total number of deduplicated internal dependency edges (imports + calls), and $k_c$ be the total degree of community $c$:

$$Q = \sum_{c} \left[ \frac{e_{cc}}{m} - \left( \frac{k_c}{2m} \right)^2 \right]$$

- **$e_{cc}$**: Intra-community edges (connections entirely within module/layer $c$).
- **$k_c$**: Total degree (in-edges + out-edges) of community $c$.
- **$\left( \frac{k_c}{2m} \right)^2$**: The quadratic degree penalty.

### The Danger of "Hub" Files (Degree Inflation)

When a single file (such as a router, dispatcher, or re-export barrel file) imports 15 different modules:
1. Community $c$ accumulates an outsized degree $k_c$.
2. The penalty term $\left( \frac{k_c}{2m} \right)^2$ expands quadratically.
3. This severely degrades $Q$, capping the Modularity score between 5,500 and 7,200.

---

## Common Hub Antipatterns Across Languages

| Language | Antipattern | Description | Architectural Solution |
| :--- | :--- | :--- | :--- |
| **TypeScript / JS** | **Barrel Files (`index.ts`)** | `index.ts` re-exporting 20+ components creates a giant artificial hub. | Import directly from feature submodules (`import { x } from './auth/service'`). |
| **Python** | **Monolithic `__init__.py`** | Exposing all package internals through a single package init. | Keep `__init__.py` minimal; import specific modules. |
| **Go** | **God Package / Flat Struct** | Putting 30 source files inside one flat root package. | Split into bounded internal sub-packages (`internal/auth`, `internal/billing`). |
| **Any Web API** | **Monolithic Root Router** | A single `router` file importing every controller in the system. | Hierarchical route delegation: Root router mounts feature sub-routers. |

---

## Architectural Remedy: Hierarchical Route & Service Delegation

### Monolithic Hub (Poor Modularity)
```
router ──┬──> authController
         ├──> userController
         ├──> orderController
         ├──> billingController
         ├──> reportController
         ├──> notificationController
         └──> adminController
```

### Hierarchical Delegation (High Modularity)
```
router ──┬──> authRouter    ──> authController
         ├──> commerceRouter ──┬──> orderController
         │                     └──> billingController
         └──> adminRouter   ──┬──> reportController
                               └──> adminController
```
By grouping related routes into domain sub-routers, degrees are distributed evenly across balanced communities, maximizing $Q$.
