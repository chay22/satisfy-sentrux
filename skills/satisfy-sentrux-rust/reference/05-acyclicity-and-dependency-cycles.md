# 05 - Acyclicity & Dependency Cycles

## Overview

Acyclicity measures the absence of circular dependencies in the codebase's import and call graph. Sentrux uses **Tarjan's Strongly Connected Components (SCC)** algorithm to detect cycles.

$$\text{Acyclicity Score} = 10,000 \times \left(1.0 - \frac{\text{Cycle Edges}}{\text{Total Edges}}\right)$$

If the codebase has **0 cycle edges**, the Acyclicity score is **10,000**. Even a single cycle edge can cause the score to drop precipitously to 8,000 or lower, which heavily drags down the overall geometric mean.

---

## How Sentrux Detects Cycles

1. Constructs a directed graph $G = (V, E)$ where:
   - $V$ is the set of all Rust source files.
   - $E$ contains intra-crate import edges (`use crate::...`) and intra-crate function call edges.
2. Executes `sentrux_core::metrics::arch::graph::compute_sccs`.
3. Any strongly connected component with $|V| > 1$ (or a self-loop) contains cycle edges.
4. Total cycle edges are counted against total graph edges.

---

## Common Cycle Sources in Rust & How to Break Them

### 1. The Mutual Model-Helper Cycle

**The Problem**:
- `src/domain/models.rs` needs a validation or formatting helper from `src/domain/utils.rs`.
- `src/domain/utils.rs` accepts a model struct from `src/domain/models.rs` as an argument.
- Result: `models.rs <──> utils.rs` forms a 2-node cycle!

**The Solution (Dependency Inversion / Leaf Extraction)**:
- Extract raw types or traits into a dedicated leaf file: `src/domain/types.rs`.
- `models.rs` and `utils.rs` both import `types.rs`.
- Neither imports the other.
```
models.rs ──┐
            ├──> types.rs
utils.rs  ──┘
```

---

### 2. Error Definition vs Domain Modules

**The Problem**:
- `src/core/errors.rs` imports specific error types or domain entities from `src/domain/wallet.rs` to construct custom error variants.
- `src/domain/wallet.rs` imports `AppError` from `src/core/errors.rs` to return results.
- Result: `core <──> domain` cycle, violating both acyclicity and layer boundaries!

**The Solution**:
- Define errors generically in `core::error::AppError` (e.g., accepting `String` or generic context).
- Domain errors should either be self-contained in `src/domain/error.rs` or implement standard `std::error::Error` without depending on higher-level web errors.

---

### 3. Background Worker / Infrastructure Client Mutual Dependency

**The Problem**:
- `src/workers/payment_sync/worker.rs` calls methods on `src/infra/gateway_client.rs`.
- `src/infra/gateway_client.rs` imports data models or response handlers declared in `src/workers/payment_sync/`.

**The Solution**:
- Move shared data transfer objects (DTOs) into `src/domain/payment.rs` or `src/infra/dto.rs`.
- The dependency becomes strictly one-way: `workers -> infra`.
