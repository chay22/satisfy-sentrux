# 14 - Rust Skills Coexistence & Idiomatic Architecture Guide

## Overview: The Dual-Gate Philosophy

Sentrux provides an objective structural measure of codebase health based on graph theory:
- **Acyclicity** (cycle-free DAG)
- **Modularity $Q$** (community clustering)
- **Complexity Equality** (uniform low cyclomatic complexity, Gini $\le 0.15$)
- **Hierarchy Depth** (layered boundaries)
- **Redundancy** (deduplication)

However, when developers or AI coding agents optimize solely for AST node counts, they risk falling victim to **Goodhart's Law**: *"When a measure becomes a target, it ceases to be a good measure."*

Artificially hacking Tree-sitter AST nodes—such as using `[a, b].into_iter().all(...)` to hide `&&`, wrapping branches in unreadable declarative macros, or using `Arc<Mutex<T>>` to bypass borrow-checker architectural signals—yields high Sentrux numbers on paper while degrading actual Rust code quality.

The **Dual-Gate Philosophy** demands both:
1. **Gate 1 (Rust Action Book)**: Idiomatic, type-driven, zero-cost, borrow-checker-sound Rust design.
2. **Gate 2 (Sentrux Quality Signal)**: High structural score (8,500–9,000+) via clean architectural decomposition.

---

## Companion Framework: `actionbook/rust-skills`

To enforce Gate 1, this skill pairs directly with **[Rust Action Book Skills](https://github.com/actionbook/rust-skills)**.

### Installation

```bash
# Add rust-skills package via npx skills
npx skills add actionbook/rust-skills
```

### The Three-Layer Meta-Cognition Model

When addressing architectural refactoring or compiler errors, never jump directly to mechanical syntax edits. Always trace through the cognitive layers:

```
Layer 3: Domain Constraints (WHY)
 └── Business rules, auditability, data lifecycle, concurrency model
      ↓
Layer 2: Design Choices (WHAT)
 └── Design patterns, DDD entities, typestates, error domains
      ↓
Layer 1: Language Mechanics (HOW)
 └── Ownership, borrowing, lifetimes, traits, zero-cost abstractions
```

### Complete Skills Mapping

#### Core Skills
- `rust-router`: Master router for all Rust queries. Directs analysis through Layer 3 $\to$ Layer 2 $\to$ Layer 1.
- `rust-learner`: Fetches crate and version information to ensure modern idiomacy.
- `coding-guidelines`: Enforces official Rust API guidelines, naming conventions, and doc style.

#### Layer 1: Language Mechanics (m01–m07)
- `m01-ownership`: "Who should own this data?" Solves `E0382`, `E0597`, move vs borrow cleanly without spurious `.clone()`.
- `m02-resource`: "What ownership pattern fits?" Selects `Box`, `Rc`, `Arc`, or stack allocations without leaking resources.
- `m03-mutability`: "Why does this data need to change?" Defines explicit mutation boundaries; prevents uncontrolled interior mutability.
- `m04-zero-cost`: "Compile-time or runtime polymorphism?" Selects generics vs `dyn Trait` to maintain zero-cost abstractions.
- `m05-type-driven`: "How can types prevent invalid states?" Replaces complex conditional validation with typestates and newtypes.
- `m06-error-handling`: "Expected failure or bug?" Idiomatic `Result`, domain error enums, and `?` propagation.
- `m07-concurrency`: "CPU-bound or I/O-bound?" Models thread safety (`Send`/`Sync`) and async runtimes cleanly.

#### Layer 2: Design Choices (m09–m15)
- `m09-domain`: Domain-driven boundaries, entities, and value objects.
- `m10-performance`: Bottleneck identification and allocation reduction.
- `m11-ecosystem`: Vetting crates to avoid bloat and unsafe dependencies.
- `m12-lifecycle`: RAII, `Drop`, and deterministic resource cleanup.
- `m13-domain-error`: Fault tolerance, retries, and error resilience.
- `m14-mental-model`: Core Rust reasoning models.
- `m15-anti-pattern`: Detecting and eliminating code smells (including metric-gaming hacks!).

---

## Balancing the 5 Sentrux Pillars with Rust Guidelines

### 1. Acyclicity ($10,000$) vs. Clean Ownership (`m01-ownership`, `m02-resource`)

- ❌ **Metric Deception Hack**: Wrapping circular dependencies in `Arc<Mutex<T>>` or `Rc<RefCell<T>>` to make the borrow checker happy at runtime while maintaining an architectural cycle.
- ✅ **Idiomatic Solution**: 
  1. Identify shared data models or contracts.
  2. Extract them downward into a leaf module (e.g., `ports.rs` or `types.rs`).
  3. Reverse the dependency arrow using trait interfaces (`m04-zero-cost`).

### 2. Complexity Equality ($\text{Gini} \le 0.15$) vs. Readability (`m05-type-driven`, `m15-anti-pattern`)

- ❌ **Metric Deception Hack**:
  - `[cond1, cond2].into_iter().all(std::convert::identity)` to hide `&&`.
  - Nested `then_some(...).ok_or_else(...)` combinator gymnastics for straightforward branching.
  - Hiding complex match statements inside declarative macros.
- ✅ **Idiomatic Solution**:
  - **Guard Clauses**: Early returns (`if !cond { return Err(...); }`) naturally achieve $\text{CC} = 2$ with maximum readability.
  - **Type-Driven Enums & Typestates**: Eliminate multi-branch validation by encoding states directly in the type system.
  - **Stage Delegates**: Break monolithic handlers into single-purpose functions (`validate`, `execute`, `format`).

### 3. Modularity ($Q \ge 7,500$) vs. Cohesion (`m09-domain`)

- ❌ **Metric Deception Hack**: Splitting modules randomly into arbitrary files to artificially boost community partition counts.
- ✅ **Idiomatic Solution**: Decentralize routers by grouping routes hierarchically around domain aggregates (e.g., `user_routes()`, `billing_routes()`).

### 4. Redundancy ($9,600+$) vs. DRY Prudence (`m15-anti-pattern`)

- ❌ **Metric Deception Hack**: Merging two completely different domain entities into a single generic struct just to avoid duplicate field definitions.
- ✅ **Idiomatic Solution**: Deduplicate shared infrastructure code (HTTP client wrappers, database error mappers, validation helpers) into leaf utility crates while keeping domain models decoupled.

---

## Dual-Gate Checklist for PRs & Code Review

Run before merging any architectural change:

```bash
# 1. Zero compiler errors & warnings
cargo clippy --all-targets -- -D warnings

# 2. 100% test pass rate
cargo test

# 3. Sentrux quality gate pass
sentrux check .
```

### The 4 Red-Flag Questions:
1. *Did I introduce any array/iterator trick just to dodge `&&` or `||`?*
2. *Did I wrap an unidiomatic `.clone()` or `Arc<Mutex<T>>` to avoid fixing an ownership model?*
3. *Is the code readable to a standard Rust developer without understanding Tree-sitter AST rules?*
4. *Did I verify with `m15-anti-pattern` that no code smell was created?*

If the answer to any red-flag question is "Yes", **revert and re-architect**.
