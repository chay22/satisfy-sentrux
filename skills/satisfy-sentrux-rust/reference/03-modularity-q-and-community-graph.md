# 03 - Modularity ($Q$) & Community Graph

## Overview

Sentrux measures modularity using **Newman's Modularity ($Q$)**, a network science metric that quantifies the degree to which a network is partitioned into dense modules with sparse cross-module connections.

$$\text{Modularity Score} = 10,000 \times \frac{Q + 0.5}{1.5}$$

where $Q \in [-0.5, 1.0]$. A modularity score of **7,200+** requires $Q \ge 0.58$, and **8,000+** requires $Q \ge 0.70$.

---

## Newman's $Q$ Formula in Sentrux

Let $m$ be the total number of deduplicated internal dependency edges (imports + calls), and $k_c$ be the degree (edge count) of community $c$:

$$Q = \sum_{c} \left[ \frac{e_{cc}}{m} - \left( \frac{k_c}{2m} \right)^2 \right]$$

In Sentrux's internal graph:
- **Nodes**: Source files (`.rs`).
- **Edges**: Intra-crate imports (`use crate::...`) and call invocations.
- **Communities**: Grouped by directory clusters and layer boundaries (e.g. `api`, `cli`, `infra`, `domain`, `workers`).

### The Danger of "Hub" Files (Degree Inflation)

If a single file (such as `src/api/handlers/router.rs` or a monolithic `mod.rs`) imports 12 different modules:
- Community $c$ gets an outsized degree $k_c$.
- The penalty term $\left( \frac{k_c}{2m} \right)^2$ grows quadratically!
- This significantly reduces $Q$, capping the modularity score around 6,000 – 7,200.

---

## Architectural Strategies to Boost Modularity

### 1. Decentralized Sub-Routers over Centralized Monolithic Routers

**Poor Modularity (Hub Pattern):**
```
router.rs ──┬──> auth.rs
            ├──> users.rs
            ├──> wallets.rs
            ├──> transactions.rs
            ├──> reports.rs
            ├──> notifications.rs
            ├──> system.rs
            └──> sync.rs
```
Here, `router.rs` accumulates a massive degree, destroying community balance.

**Optimal Modularity (Hierarchical Delegation):**
```
router.rs ──┬──> auth_routes.rs   ──> auth_handlers.rs
            ├──> finance_routes.rs ──┬──> wallets.rs
            │                        └──> transactions.rs
            └──> system_routes.rs  ──> system.rs
```
By grouping related routes into domain-level sub-routers, the edge degree is partitioned evenly across cohesive sub-communities.

---

### 2. Eliminating Hidden Cross-Module Imports

Ensure modules communicate through explicit public interfaces rather than reaching deep into neighboring submodules.

- **Bad**: `src/workers/sync_worker.rs` directly importing private helper items from `src/infra/scrapers/web_feed.rs`.
- **Good**: `src/infra/scrapers` exposes a unified facade in `src/infra/scrapers/mod.rs`, and workers depend only on the high-level trait/function.

---

### 3. Balancing Community Sizes

To maximize $Q$, keep community sizes and inter-community connections balanced:
- Keep the number of edges within each module ($e_{cc}$) high.
- Keep the cross-boundary edges low.
- Keep the layer flow strictly downward (e.g., `api -> domain -> core`).
