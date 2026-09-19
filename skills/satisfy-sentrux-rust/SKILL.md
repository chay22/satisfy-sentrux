---
name: satisfy-sentrux-rust
description: "Use when analyzing, refactoring, or optimizing Rust codebases to satisfy Sentrux architectural quality gates while maintaining idiomatic Rust design, or when balancing Sentrux scores with Rust coding guidelines. Keywords: sentrux, architecture score, quality signal, cyclomatic complexity, gini, modularity, acyclicity, rules.toml, dependency cycle, rust, rust-skills, actionbook, coding-guidelines, idiomatic rust, anti-pattern, ownership"
---

# Satisfy Sentrux (Rust Edition): Architecture & Idiomatic Quality Guide

## Overview

Sentrux evaluates codebase structural integrity through graph-theoretic static analysis, computing a compound **Quality Signal** (capped at **10,000**) as the geometric mean of 5 core metrics:

$$\text{Quality Signal} = \sqrt[5]{\text{Modularity} \times \text{Acyclicity} \times \text{Depth} \times \text{Equality} \times \text{Redundancy}}$$

Because the geometric mean multiplies all five factors, **a deficit in any single metric drags down the overall score**.

> [!CRITICAL]
> ### The Dual-Gate Mandate: Never Deceive the Metric
> **Optimizing Sentrux scores by degrading Rust code quality is strictly forbidden.**  
> Contorting code with unidiomatic hacks—such as hiding `&&` with iterator arrays, abusing branchless combinators where guard clauses are clearer, or introducing leaky `Arc<Mutex<T>>` wrappers just to silence cycle detectors—is an architectural anti-pattern.
>
> True excellence balances **high Sentrux structural scores** with **idiomatic, zero-cost, type-safe Rust**.

---

## Companion Skills: Rust Action Book (`actionbook/rust-skills`)

To prevent metric-gaming and maintain idiomatic Rust quality, this skill pairs directly with **[Rust Action Book Skills](https://github.com/actionbook/rust-skills)**.

### 1. Harness & User Prerequisite Check
When this skill is loaded, the agent **MUST check** if `actionbook/rust-skills` (or its components: `rust-router`, `coding-guidelines`, `m01-ownership`) is installed in the workspace or agent environment (`~/.claude/skills/`, `~/.agents/skills/`, or project skills).

- **If NOT installed**, the agent MUST prompt the user to install the companion skills:
  ```bash
  npx skills add actionbook/rust-skills
  ```
- **Cognitive Fallback (Self-Guiding Mode)**: If external skills cannot be installed immediately, the agent **MUST internally adopt the Meta-Cognition Framework and Rust Guidelines**:
  - **Layer 3: Domain Constraints (WHY)**: Align with business rules, data lifecycles, and audit invariants.
  - **Layer 2: Design Choices (WHAT)**: Select idiomatic architectures (typestates, RAII, zero-cost abstractions).
  - **Layer 1: Language Mechanics (HOW)**: Honor borrow checker semantics, error propagation (`?`), and type safety.
  - **Never skip directly to AST tweaking** without tracing through Layers 3 and 2.

### 2. Companion Skills Catalog

| Group | Skill | Core Focus | Trigger / Role |
| :--- | :--- | :--- | :--- |
| **Core** | `rust-router` | Master router for Rust architectural questions | Invoked first for cross-layer reasoning |
| **Core** | `rust-learner` | Crate version & API lookup | Dependencies & version-specific features |
| **Core** | `coding-guidelines`| Rust coding conventions & style lookup | Idiomacy checks, naming, doc conventions |
| **Mechanics** | `m01-ownership` | Data ownership & lifetime management | `E0382`, `E0597`, move vs borrow |
| **Mechanics** | `m02-resource` | Allocation & smart pointer selection | `Box`, `Rc`, `Arc`, `RefCell` |
| **Mechanics** | `m03-mutability`| Mutation boundaries & interior mutability | `mut`, `Cell`, `E0596`, `E0499` |
| **Mechanics** | `m04-zero-cost` | Compile-time vs runtime polymorphism | Generics, trait objects (`dyn`), `E0277` |
| **Mechanics** | `m05-type-driven`| Preventing invalid states with types | Newtypes, typestates, `PhantomData` |
| **Mechanics** | `m06-error-handling` | Domain errors vs bugs | `Result`, `Error`, `panic!`, `?` operator |
| **Mechanics** | `m07-concurrency` | Concurrency & asynchronous execution | `Send`, `Sync`, `tokio`, threads |
| **Design** | `m09-domain` | Domain modeling & entity boundaries | DDD, value objects, boundaries |
| **Design** | `m10-performance`| Hot paths, allocations & profiling | Benchmarking, cache locality |
| **Design** | `m11-ecosystem` | Crate selection & dependency governance | Vetting third-party crates |
| **Design** | `m12-lifecycle` | Resource cleanup & RAII | `Drop`, lazy initialization |
| **Design** | `m13-domain-error` | Fault tolerance & error domains | Retries, fallback, circuit breaker |
| **Design** | `m14-mental-model`| Idiomatic Rust reasoning & core concepts | Understanding Rust semantics |
| **Design** | `m15-anti-pattern`| Identifying & purging code smells | Metric deception, clone abuse, bad abstractions |

---

## Metric Deception vs. Genuine Structural Refactoring

| Target Metric | ❌ Deceptive Metric Hack (FORBIDDEN) | ✅ Idiomatic Structural Refactoring (REQUIRED) |
| :--- | :--- | :--- |
| **Complexity ($\text{CC} \le 2$)** | Hiding `&&`/`||` with `[a, b].into_iter().all(...)`. Loses short-circuiting and harms readability. | Extract cohesive predicate functions (`user.is_authorized()`), or use natural early returns. |
| **Complexity ($\text{CC} \le 2$)** | Over-complicating logic with forced branchless chains `(!cond).then_some(...).ok_or_else(...)`. | Guard clauses with early `return Err(...)` or the standard `?` operator. |
| **Acyclicity ($10,000$)** | Slapping `Arc<Mutex<T>>` or `Box<dyn Any>` everywhere to bypass borrow/cycle checks at runtime. | Leaf extraction: Move shared DTOs/types/traits to a dedicated leaf module (`types.rs` or `ports.rs`). |
| **Equality (Gini)** | Shoving nested branches into complex declarative macros just because Tree-sitter skips macro bodies. | Type-driven state machine (`m05-type-driven`) or trait polymorphism (`m04-zero-cost`). |
| **Modularity ($Q$)** | Splitting modules arbitrarily into 1-line files without semantic boundaries. | Logical domain clustering: grouping related operations around entities and sub-routers. |

---

## The 5 Pillars & Targets

| Metric | Target | Normalized Derivation | Primary Idiomatic Action | Reference |
| :--- | :--- | :--- | :--- | :--- |
| **Acyclicity** | **10,000** | $10,000 \times (1 - \frac{\text{Cycle Edges}}{\text{Total Edges}})$ | Extract leaf DTOs & port traits; break circular references cleanly | [05-acyclicity](reference/05-acyclicity-and-dependency-cycles.md) |
| **Complexity Equality** | **8,500+** | $10,000 \times (1 - \text{Gini Index})$ | Decompose functions to single responsibilities ($\text{CC} \le 2$) | [02-complexity-gini](reference/02-cyclomatic-complexity-and-equality-gini.md) |
| **Redundancy** | **9,600+** | $10,000 \times (1 - \text{Duplication Ratio})$ | Deduplicate shared parsers & helpers into leaf crates/modules | [06-redundancy](reference/06-redundancy-and-code-duplication.md) |
| **Hierarchy Depth** | **8,800+** | Layer DAG depth penalty function | Maintain balanced 4–5 layer DAG (`api` $\to$ `domain` $\to$ `core`) | [04-layering-rules](reference/04-layering-boundaries-and-rules-toml.md) |
| **Modularity ($Q$)** | **7,500+** | $10,000 \times \frac{Q + 0.5}{1.5}$ | Decentralize monolithic router hubs into sub-route modules | [03-modularity-q](reference/03-modularity-q-and-community-graph.md) |

---

## Quick Reference: Idiomatic Refactoring for $\text{CC} \le 2$

Sentrux calculates function cyclomatic complexity in Rust via Tree-sitter AST nodes:
$$\text{CC} = 1 + \sum (\text{branch nodes}) + \sum (\text{logic operators})$$
Branch nodes: `if_expression`, `else_clause`, `for_expression`, `while_expression`, `loop_expression`, `match_arm`.  
Logic nodes: `binary_expression` with `&&` or `||`.

### 1. Guard Clauses & Early Returns (Eliminating `else_clause`)
- ❌ **CC = 3 (Nested)**: `if cond { return Err(...); } else { Ok(...) }`
- ✅ **CC = 2 (Idiomatic)**:
  ```rust
  if !is_valid {
      return Err(AppError::InvalidInput);
  }
  Ok(process_data())
  ```

### 2. Domain Predicates (Handling Multi-Condition Logic)
Never use iterator array hacks to hide boolean operators. Instead, encapsulate business rules into readable, testable methods or predicate functions:
- ❌ **CC = 1 (Metric Deception)**: `if [is_valid, is_admin].into_iter().all(std::convert::identity)`
- ✅ **CC = 2 (Clean & Idiomatic)**:
  ```rust
  if !user.can_access_resource(resource) {
      return Err(AppError::Unauthorized);
  }
  ```

### 3. Dispatch Tables & Typestates (Replacing Multi-Arm `match`)
- ❌ **CC = 6**: 5+ arm `match` block inside a business handler.
- ✅ **CC = 1**: Command dispatch table or typestate transitions:
  ```rust
  const HANDLERS: &[(&str, fn(&Context) -> Result<(), AppError>)] = &[
      ("start", handle_start),
      ("stop", handle_stop),
  ];
  ```

### 4. Decomposing Handlers into 3 Single-Responsibility Delegates
Split monolithic presentation/API handlers into three clean, independently testable functions ($\text{CC} \le 2$ each):
1. `validate_request_payload(req) -> Result<Validated, AppError>` ($\text{CC} \le 2$)
2. `execute_business_operation(ctx, validated) -> Result<Entity, AppError>` ($\text{CC} \le 2$)
3. `format_response_json(entity) -> Json<Response>` ($\text{CC} = 1$)

Main entry point:
```rust
pub async fn handle_request(
    State(ctx): State<AppContext>,
    Json(payload): Json<CreateRequest>,
) -> Result<Json<CreateResponse>, AppError> {
    let validated = validate_request_payload(payload)?;
    let entity = execute_business_operation(&ctx, validated).await?;
    Ok(format_response_json(entity))
}
```

---

## Complete Reference Catalog

Detailed architectural deep dives, proofs, templates, and case studies are located in [`reference/`](reference/):

| Ref # | Document | Scope & Key Takeaways |
| :--- | :--- | :--- |
| **01** | [**Architecture & Scoring Engine**](reference/01-sentrux-architecture-and-scoring-engine.md) | Geometric mean formula, AST parsing pipeline, graph construction, and scoring tiers. |
| **02** | [**Complexity & Equality (Gini)**](reference/02-cyclomatic-complexity-and-equality-gini.md) | Tree-sitter AST nodes, Gini inequality derivation, and idiomatic Rust flattening patterns without metric deception. |
| **03** | [**Modularity ($Q$) & Community Graph**](reference/03-modularity-q-and-community-graph.md) | Newman's $Q$ modularity, quadratic penalty of hub files, and hierarchical route delegation. |
| **04** | [**Layering, Boundaries & rules.toml**](reference/04-layering-boundaries-and-rules-toml.md) | Full specification for `.sentrux/rules.toml` (`[constraints]`, `[[layers]]`, `[[boundaries]]`). |
| **05** | [**Acyclicity & Dependency Cycles**](reference/05-acyclicity-and-dependency-cycles.md) | Tarjan's SCC algorithm, breaking mutual references via leaf extraction, and cycle prevention. |
| **06** | [**Redundancy & Code Duplication**](reference/06-redundancy-and-code-duplication.md) | Token clone detection, leaf utility placement in `core/`, and trait-based deserialization. |
| **07** | [**Sentrux CLI & MCP Cheatsheet**](reference/07-sentrux-cli-and-mcp-cheatsheet.md) | Commands (`check`, `gate`, `mcp`, `scan`), MCP tools, and metric inspection techniques. |
| **08** | [**Refactoring Playbook & Workflow**](reference/08-refactoring-playbook-and-step-by-step-workflow.md) | The battle-tested 6-phase optimization workflow from baseline to 8,764+ with dual-gate verification. |
| **09** | [**Case Study: 6,500 to 8,764**](reference/09-case-study-from-6500-to-8764.md) | Real-world empirical case study of an async Rust backend showing exact metric deltas. |
| **10** | [**CI/CD & GitHub Actions Gating**](reference/10-ci-cd-and-github-actions.md) | Automating `sentrux check` and `sentrux gate` in GitHub Actions to block PR regressions. |
| **11** | [**Rust Quirks & AST Edge Cases**](reference/11-rust-quirks-and-edge-cases.md) | Re-exports (`pub use`), macro parsing, dynamic dispatch (`dyn Trait`), and module layouts. |
| **12** | [**Production rules.toml Templates**](reference/12-rules-toml-architecture-templates.md) | Drop-in templates for Clean Architecture, Web APIs (Axum/Actix), and CLI tools. |
| **13** | [**Cross-Language Equivalency Matrix**](reference/13-cross-language-equivalency-matrix.md) | Tree-sitter AST and refactoring patterns for TypeScript/JavaScript, Python, and Go. |
| **14** | [**Rust Skills Coexistence & Idiomacy**](reference/14-rust-skills-coexistence-and-idiomacy-guide.md) | In-depth integration guide for pairing Sentrux with `actionbook/rust-skills` to prevent metric deception. |

---

## Continuous Dual-Gate Verification Discipline

Before declaring any architectural refactoring complete, run the **Dual-Gate verification**:

```bash
# Gate 1: Idiomatic Rust, Correctness, and Compiler Discipline
cargo test
cargo clippy --all-targets -- -D warnings

# Gate 2: Sentrux Structural Architecture Gate
sentrux check .
```

### Self-Audit Question
> *"Did this refactoring make the code genuinely cleaner, or did it just trick the Tree-sitter node count?"*  
> If any change reduced readability, broke short-circuit semantics, or introduced unnecessary runtime wrappers, **revert it and re-architect using type-driven design (`m05-type-driven`) or leaf extraction (`m01-ownership`)**.
