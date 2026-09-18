# 08 - Refactoring Playbook & Step-by-Step Workflow

## Overview

This playbook outlines a universal, systematic workflow to elevate any codebase (TypeScript, Python, Go, Rust, Java) from a mediocre Sentrux score (7,000–7,500) to an elite score (**8,500–9,000+**) with zero regressions.

---

## The 6-Phase Optimization Workflow

```
┌────────────────────────────────────────────────────────┐
│ Phase 1: Baseline Assessment & Gate Validation         │
│   Run sentrux check ., verify tests pass              │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ Phase 2: Acyclicity Enforcement (Target: 10,000)       │
│   Break circular imports via leaf type extraction      │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ Phase 3: Layer & Boundary Rules (.sentrux/rules.toml)  │
│   Declare layers, enforce downward-only dependencies   │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ Phase 4: Complexity Flattening (Target: CC <= 2)       │
│   Decompose handlers into delegates; drive Gini -> 0   │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ Phase 5: Modularity Balancing                          │
│   Decentralize router and barrel export hubs           │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ Phase 6: Redundancy Optimization (Target: 9,600+)      │
│   Deduplicate utilities and parsing into leaf modules  │
└────────────────────────────────────────────────────────┘
```

---

## Phase 1: Baseline Assessment

1. Run the quality check:
   ```bash
   sentrux check .
   ```
2. Run your project's test suite and linters to verify a clean starting baseline.
3. Record the starting score.

---

## Phase 2: Acyclicity (Target: 10,000)

1. Identify files or packages involved in mutual imports ($A \to B$ and $B \to A$).
2. Apply the **Leaf Extraction Pattern**:
   - Extract shared types, interfaces, or DTOs into a dedicated leaf file (`types.ts`, `models.py`, `dto.go`).
   - Both $A$ and $B$ import the leaf file; neither imports the other.
3. Verify that cycle edges drop to **0**.

---

## Phase 3: Boundary & Layer Definition

1. Configure `.sentrux/rules.toml`:
   - Set layers in strict hierarchy order:
     - `order = 0`: Presentation / Entry points
     - `order = 1`: Background Workers
     - `order = 2`: Infrastructure / Adapters
     - `order = 3`: Application Services
     - `order = 4`: Domain Core / Entities
     - `order = 5`: Shared Core / Primitives
2. Define explicit boundaries preventing inner domain logic from importing outer infrastructure or presentation code.
3. Run `sentrux check .` to verify zero violations.

---

## Phase 4: Complexity Flattening ($\text{CC} \le 2$ Everywhere)

This phase produces the largest jump in **Complexity Equality** (routinely pushing Equality from ~6,800 to ~8,300+).

1. Search for functions with $\text{CC} \ge 3$:
   - Look for nested conditionals, `else` clauses, multi-branch `switch`/`match`, and chains of `&&`/`||`.
2. Apply refactoring patterns:
   - Convert `if ... else` to early guard returns.
   - Replace switch/match statements with dictionary/hash map lookups.
   - Replace compound boolean expressions (`a && b && c`) with list/array predicate checks (`[a, b, c].every(...)` or `all([a, b, c])`).
   - Split large controller functions into three single-responsibility delegates:
     - `validateRequest(...)` ($\text{CC} \le 2$)
     - `executeBusinessLogic(...)` ($\text{CC} \le 2$)
     - `formatResponse(...)` ($\text{CC} = 1$)
3. Ensure zero functions exceed $\text{CC} = 2$.

---

## Phase 5: Modularity Balancing

1. Check for high-degree "hub" files:
   - Monolithic routers.
   - Overloaded barrel exports (`index.ts` re-exporting everything).
2. Decompose hubs:
   - Split monolithic routers into domain-specific sub-routers.
   - Import directly from specific modules rather than through giant barrel re-exports.

---

## Phase 6: Redundancy Deduplication

1. Extract duplicated serialization, string manipulation, or validation blocks into shared leaf utility modules.
2. Place shared utilities in the lowest common foundation layer (`src/core/`) so all modules can import them without circularity.

---

## Verification & Autonomous Milestone Commits

After each discrete milestone:
1. Run test suite: Ensure 100% tests pass.
2. Run linter: Ensure 0 warnings.
3. Run `sentrux check .`: Confirm structural score improves.
4. Commit changes:
   ```bash
   git add <modified-files>
   git commit -m "refactor(<scope>): flatten complexity to CC <= 2 and modularize dependencies"
   ```
