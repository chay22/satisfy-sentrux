# 08 - Refactoring Playbook & Step-by-Step Workflow

## Overview

This playbook provides a systematic, step-by-step methodology to elevate a Rust codebase from a mediocre Sentrux score (7,000–7,500) to an elite score (**8,500–9,000+**) while maintaining 100% test pass rates and zero Clippy warnings.

---

## The 6-Phase Optimization Workflow

```
┌────────────────────────────────────────────────────────┐
│ Phase 1: Baseline Assessment & Gate Validation         │
│   Run sentrux check ., record 5 sub-metrics            │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ Phase 2: Acyclicity Enforcement (Target: 10,000)       │
│   Break all cyclic imports via leaf type extraction    │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ Phase 3: Boundary & Layer Definition (.sentrux/rules)  │
│   Declare layers, enforce downward-only dependencies   │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ Phase 4: Complexity Flattening (Target: CC <= 2)       │
│   Decompose all handlers/functions; drive Gini -> 0   │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ Phase 5: Modularity Balancing                          │
│   Decentralize router hubs into sub-route modules      │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ Phase 6: Redundancy Optimization (Target: 9,600+)      │
│   Deduplicate parsers, HTTP utilities, shared logic   │
└────────────────────────────────────────────────────────┘
```

---

## Phase 1: Baseline Assessment

1. Run the quality gate:
   ```bash
   sentrux check .
   ```
2. Run test and lint baselines:
   ```bash
   cargo test
   cargo clippy --all-targets -- -D warnings
   ```
3. Record current score and identify the lowest metric among:
   - Acyclicity
   - Modularity
   - Equality
   - Redundancy
   - Hierarchy Depth

---

## Phase 2: Acyclicity (Pushing to 10,000)

1. Check for circular imports or mutual references between files in the same or adjacent directories.
2. For any file $A$ and file $B$ where $A \to B$ and $B \to A$:
   - Identify shared structs, enums, or traits.
   - Create a leaf file (e.g. `types.rs`, `models.rs`, or `ports.rs`).
   - Move shared definitions to the leaf file.
   - Update both $A$ and $B$ to import from the leaf file.
3. Verify that cycle edges drop to **0**.

---

## Phase 3: Layer & Boundary Rules

1. Define or refine `.sentrux/rules.toml`:
   - Set layers in strict order:
     - `order = 0`: UI/Presentation (`cli`, `api`)
     - `order = 1`: Background Workers (`workers`)
     - `order = 2`: Infrastructure (`infra`)
     - `order = 3`: Domain Core (`domain`)
     - `order = 4`: Primitives & Shared Errors (`core`)
2. Define explicit boundaries preventing `domain` from importing `api`, `infra`, or `ctl`.
3. Run `sentrux check .` to ensure 0 boundary violations.

---

## Phase 4: Complexity Flattening ($\text{CC} \le 2$ Everywhere)

This phase yields the largest gain in **Complexity Equality** (can jump from 6,800 to 8,300+).

1. Scan for functions with $\text{CC} \ge 3$:
   - Grep for `if ... else`, multi-arm `match`, and `&&` / `||`.
2. Apply idiomatic refactoring patterns (no metric deception):
   - Replace `if cond { return ...; } else { ... }` with guard clauses and early returns.
   - Replace complex nested conditionals with domain predicate methods (`user.is_authorized()`). *Never use array iterator hacks (`[a, b].into_iter().all(...)`) to game the metric.*
   - Split large handlers into:
     - `validate_input(...) -> Result<Clean, AppError>` ($\text{CC} \le 2$)
     - `query_or_persist(...) -> Result<Output, AppError>` ($\text{CC} \le 2$)
     - `format_response(...) -> Payload` ($\text{CC} = 1$)
3. Verify that zero functions exceed $\text{CC} = 2$.

---

## Phase 5: Modularity Balancing

1. Check the main router file (`src/api/handlers/router.rs` or equivalent).
2. If the router directly imports 8+ handler files:
   - Group routes by business context (e.g., `auth_routes`, `finance_routes`, `system_routes`).
   - Move route registration into sub-router modules.
   - Have the root router merge the sub-routers using `.merge(...)`.
3. This reduces degree concentration ($k_c$) and increases Newman's $Q$.

---

## Phase 6: Redundancy Deduplication

1. Search for repeated string parsing, date manipulation, or validation logic.
2. Place shared utility functions into `src/core/` (leaf layer) so any module can import them without circularity or boundary violations.
3. Replace manual loop conversions with standard trait implementations (`FromStr`, `TryFrom`, `Display`).

---

## Verification & Autonomous Commit Discipline

After completing each discrete refactoring milestone:
1. Verify tests pass:
   ```bash
   cargo test
   ```
2. Verify lints pass:
   ```bash
   cargo clippy --all-targets -- -D warnings
   ```
3. Verify Sentrux rules pass:
   ```bash
   sentrux check .
   ```
4. Create a clean git commit:
   ```bash
   git add <modified-files>
   git commit -m "refactor(<scope>): <concise description of refactoring to CC <= 2 and modularity>"
   ```
