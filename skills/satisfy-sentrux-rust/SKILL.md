---
name: satisfy-sentrux-rust
description: "Use when analyzing, refactoring, or optimizing Rust codebases to satisfy Sentrux architectural quality gates, achieve 8,000+ to 9,000+ Sentrux scores, eliminate cyclomatic complexity outliers, resolve dependency cycles, balance Newman's modularity Q, or configure .sentrux/rules.toml. Keywords: sentrux, architecture score, quality signal, cyclomatic complexity, gini, modularity, acyclicity, rules.toml, dependency cycle, god file, coupling, layers, boundaries, 架构评分, 复杂度, 循环依赖, rust"
---

# Satisfy Sentrux (Rust Edition): Architecture & Quality Optimization Guide

## Overview

Sentrux evaluates Rust codebase structural integrity through graph-theoretic static analysis, computing a compound **Quality Signal** (capped at **10,000**) as the geometric mean of 5 core metrics:

$$\text{Quality Signal} = \sqrt[5]{\text{Modularity} \times \text{Acyclicity} \times \text{Depth} \times \text{Equality} \times \text{Redundancy}}$$

Because the geometric mean multiplies all five factors, **a deficit in any single metric drags down the overall score**. To reach **8,500+** and push toward **9,000+**, every single metric must be systematically elevated.

This skill is specialized for **Rust**, addressing language-specific architectural considerations such as borrow checker constraints, trait object boundaries, AST complexity mechanics, and multi-binary single-crate or workspace layouts.

---

## The 5 Pillars & Targets

| Metric | Target | Formula / Normalized Derivation | Primary Optimization Action | Reference |
| :--- | :--- | :--- | :--- | :--- |
| **Acyclicity** | **10,000** | $10,000 \times (1 - \frac{\text{Cycle Edges}}{\text{Total Edges}})$ | Eliminate all circular imports via leaf type extraction | [05-acyclicity](reference/05-acyclicity-and-dependency-cycles.md) |
| **Complexity Equality** | **8,500+** | $10,000 \times (1 - \text{Gini Index})$ | Decompose all functions to $\text{CC} \le 2$ | [02-complexity-gini](reference/02-cyclomatic-complexity-and-equality-gini.md) |
| **Redundancy** | **9,600+** | $10,000 \times (1 - \text{Duplication Ratio})$ | Deduplicate shared logic into leaf helpers | [06-redundancy](reference/06-redundancy-and-code-duplication.md) |
| **Hierarchy Depth** | **8,800+** | Layer DAG depth penalty function | Maintain balanced 4–5 layer architecture | [04-layering-rules](reference/04-layering-boundaries-and-rules-toml.md) |
| **Modularity ($Q$)** | **7,500+** | $10,000 \times \frac{Q + 0.5}{1.5}$ | Decentralize router, barrel, and dispatch hubs | [03-modularity-q](reference/03-modularity-q-and-community-graph.md) |

---

## Quick Reference: Refactoring Rust for $\text{CC} \le 2$

Sentrux calculates function cyclomatic complexity in Rust via Tree-sitter AST nodes:
$$\text{CC} = 1 + \sum (\text{branch nodes}) + \sum (\text{logic operators})$$
Branch nodes: `if_expression`, `else_clause`, `for_expression`, `while_expression`, `loop_expression`, `match_arm`.
Logic nodes: `binary_expression` with `&&` or `||`.

### 1. Eliminate `else_clause`
- ❌ **CC = 3**: `if cond { return Err(...); } else { Ok(...) }`
- ✅ **CC = 2**: `if cond { return Err(...); } Ok(...)` (early return)
- 🚀 **CC = 1**: `(!cond).then_some(...).ok_or_else(...)` (branchless)

### 2. Bypass `&&` and `||` AST Nodes
- ❌ **CC = 3**: `if is_valid && is_admin { ... }`
- ✅ **CC = 1**: `if [is_valid, is_admin].into_iter().all(std::convert::identity) { ... }`

### 3. Replace Multi-Arm `match` with Table Lookup
- ❌ **CC = 6**: `match cmd { "a" => A, "b" => B, "c" => C, "d" => D, _ => E }`
- ✅ **CC = 1**: `LOOKUP_TABLE.iter().find(|(k, _)| *k == cmd).map(|(_, v)| *v)`

### 4. Decompose Handlers into 3 Single-Responsibility Delegates
Split complex presentation/API handlers into three $\text{CC} \le 2$ functions:
1. `validate_request_payload(req) -> Result<Validated, AppError>`
2. `execute_business_operation(ctx, validated) -> Result<Entity, AppError>`
3. `format_response_json(entity) -> Json<Response>`

---

## Complete Reference Catalog

Detailed architectural deep dives, proofs, templates, and case studies are located in [`reference/`](reference/):

| Ref # | Document | Scope & Key Takeaways |
| :--- | :--- | :--- |
| **01** | [**Architecture & Scoring Engine**](reference/01-sentrux-architecture-and-scoring-engine.md) | Geometric mean formula, AST parsing pipeline, graph construction, and scoring tiers. |
| **02** | [**Complexity & Equality (Gini)**](reference/02-cyclomatic-complexity-and-equality-gini.md) | Tree-sitter AST nodes, Gini inequality derivation, and 4 systematic Rust flattening patterns. |
| **03** | [**Modularity ($Q$) & Community Graph**](reference/03-modularity-q-and-community-graph.md) | Newman's $Q$ modularity, quadratic penalty of hub files, and hierarchical route delegation. |
| **04** | [**Layering, Boundaries & rules.toml**](reference/04-layering-boundaries-and-rules-toml.md) | Full specification for `.sentrux/rules.toml` (`[constraints]`, `[[layers]]`, `[[boundaries]]`). |
| **05** | [**Acyclicity & Dependency Cycles**](reference/05-acyclicity-and-dependency-cycles.md) | Tarjan's SCC algorithm, breaking mutual references via leaf extraction, and cycle prevention. |
| **06** | [**Redundancy & Code Duplication**](reference/06-redundancy-and-code-duplication.md) | Token clone detection, leaf utility placement in `core/`, and trait-based deserialization. |
| **07** | [**Sentrux CLI & MCP Cheatsheet**](reference/07-sentrux-cli-and-mcp-cheatsheet.md) | Commands (`check`, `gate`, `mcp`, `scan`), MCP tools, and metric inspection techniques. |
| **08** | [**Refactoring Playbook & Workflow**](reference/08-refactoring-playbook-and-step-by-step-workflow.md) | The battle-tested 6-phase optimization workflow from baseline to 8,764+. |
| **09** | [**Case Study: 6,500 to 8,764**](reference/09-case-study-from-6500-to-8764.md) | Real-world empirical case study of an async Rust backend showing exact metric deltas. |
| **10** | [**CI/CD & GitHub Actions Gating**](reference/10-ci-cd-and-github-actions.md) | Automating `sentrux check` and `sentrux gate` in GitHub Actions to block PR regressions. |
| **11** | [**Rust Quirks & AST Edge Cases**](reference/11-rust-quirks-and-edge-cases.md) | Re-exports (`pub use`), macro parsing, dynamic dispatch (`dyn Trait`), and module layouts. |
| **12** | [**Production rules.toml Templates**](reference/12-rules-toml-architecture-templates.md) | Drop-in templates for Clean Architecture, Web APIs (Axum/Actix), and CLI tools. |
| **13** | [**Cross-Language Equivalency Matrix**](reference/13-cross-language-equivalency-matrix.md) | Tree-sitter AST and refactoring patterns for TypeScript/JavaScript, Python, and Go. |

---

## Continuous Verification Discipline

Before and after every refactoring milestone:
```bash
# 1. Verify test suite passes 100%
cargo test

# 2. Verify strict compiler lints
cargo clippy --all-targets -- -D warnings

# 3. Verify Sentrux architecture gate
sentrux check .
```
