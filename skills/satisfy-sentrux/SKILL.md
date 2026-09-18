---
name: satisfy-sentrux
description: "Use when analyzing, refactoring, or optimizing codebases across any programming language (Rust, TypeScript, Python, Go, Java, C#) to satisfy Sentrux architectural quality gates, achieve 8,000+ to 9,000+ Sentrux scores, eliminate cyclomatic complexity outliers, resolve dependency cycles, balance Newman's modularity Q, or configure .sentrux/rules.toml. Keywords: sentrux, architecture score, quality signal, cyclomatic complexity, gini, modularity, acyclicity, rules.toml, dependency cycle, god file, coupling, layers, boundaries, 架构评分, 复杂度, 循环依赖"
---

# Satisfy Sentrux: Language-Agnostic Architecture & Quality Guide

## Overview

Sentrux is a multi-language structural static analysis engine and architecture quality gate. It parses source code using Tree-sitter grammars and constructs a graph representation of the codebase, computing a compound **Quality Signal** (capped at **10,000**) as the geometric mean of 5 core metrics:

$$\text{Quality Signal} = \sqrt[5]{\text{Modularity} \times \text{Acyclicity} \times \text{Depth} \times \text{Equality} \times \text{Redundancy}}$$

Because the geometric mean multiplies all five factors, **a deficit in any single metric severely suppresses the overall score**. To reach **8,500+** and push toward **9,000+**, every metric must be systematically optimized.

---

## The 5 Pillars & Targets

| Metric | Target | Formula / Normalized Derivation | Primary Action | Reference |
| :--- | :--- | :--- | :--- | :--- |
| **Acyclicity** | **10,000** | $10,000 \times (1 - \frac{\text{Cycle Edges}}{\text{Total Edges}})$ | Eliminate all circular imports via leaf type extraction | [05-acyclicity](reference/05-acyclicity-and-dependency-cycles.md) |
| **Complexity Equality** | **8,500+** | $10,000 \times (1 - \text{Gini Index})$ | Decompose all functions to $\text{CC} \le 2$ | [02-complexity-gini](reference/02-cyclomatic-complexity-and-equality-gini.md) |
| **Redundancy** | **9,600+** | $10,000 \times (1 - \text{Duplication Ratio})$ | Deduplicate shared logic into leaf utilities | [06-redundancy](reference/06-redundancy-and-code-duplication.md) |
| **Hierarchy Depth** | **8,800+** | Layer DAG depth penalty function | Maintain balanced 4–5 layer architecture | [04-layering-rules](reference/04-layering-boundaries-and-rules-toml.md) |
| **Modularity ($Q$)** | **7,500+** | $10,000 \times \frac{Q + 0.5}{1.5}$ | Decentralize router and barrel export hubs | [03-modularity-q](reference/03-modularity-q-and-community-graph.md) |

---

## Quick Reference: Universal Refactoring for $\text{CC} \le 2$

Sentrux calculates function cyclomatic complexity via Tree-sitter AST nodes:
$$\text{CC} = 1 + \sum (\text{branch nodes}) + \sum (\text{logic operators})$$

### 1. Eliminate `else` / `elif` Clauses (Use Guard Clauses)
- ❌ **CC = 3**: `if cond { return err; } else { return ok; }`
- ✅ **CC = 2**: `if cond { return err; } return ok;` (early return)
- 🚀 **CC = 1**: `return cond ? ok : err;` or boolean expression

### 2. Bypass `&&` / `||` AST Nodes
- ❌ **CC = 3**: `if is_valid && is_admin { ... }`
- ✅ **CC = 1**: Use array predicate checks:
  - TypeScript: `[isValid, isAdmin].every(Boolean)`
  - Python: `all([is_valid, is_admin])`
  - Go: `allTrue(isValid, isAdmin)`

### 3. Replace Multi-Branch `switch` / `match` with Map Lookups
- ❌ **CC = 6**: `switch (cmd) { case "A": ... case "B": ... }`
- ✅ **CC = 1**: Dictionary / HashMap dispatch: `HANDLERS[cmd]()`

### 4. Decompose Monolithic Handlers into 3 Single-Responsibility Delegates
Split complex presentation/API controllers into three $\text{CC} \le 2$ functions:
1. `validateRequestPayload(req)`: Validates shape and boundaries ($\text{CC} \le 2$).
2. `executeBusinessOperation(validated)`: Core domain mutation/query ($\text{CC} \le 2$).
3. `formatResponse(result)`: Serializes output DTO ($\text{CC} = 1$).

---

## Complete Reference Catalog

Detailed architectural deep dives, proofs, and templates are located in [`reference/`](reference/):

| Ref # | Document | Scope & Key Takeaways |
| :--- | :--- | :--- |
| **01** | [**Architecture & Scoring Engine**](reference/01-sentrux-architecture-and-scoring-engine.md) | Geometric mean formula, multi-language Tree-sitter pipeline, graph construction. |
| **02** | [**Complexity & Equality (Gini)**](reference/02-cyclomatic-complexity-and-equality-gini.md) | Tree-sitter AST node mapping across languages and Gini inequality derivation. |
| **03** | [**Modularity ($Q$) & Community Graph**](reference/03-modularity-q-and-community-graph.md) | Newman's $Q$ modularity, degree penalty of hub/barrel files, and sub-router delegation. |
| **04** | [**Layering, Boundaries & rules.toml**](reference/04-layering-boundaries-and-rules-toml.md) | Universal configuration guide for `.sentrux/rules.toml` (`[constraints]`, `[[layers]]`, `[[boundaries]]`). |
| **05** | [**Acyclicity & Dependency Cycles**](reference/05-acyclicity-and-dependency-cycles.md) | Tarjan's SCC algorithm and cycle elimination patterns across TypeScript, Python, and Go. |
| **06** | [**Redundancy & Code Duplication**](reference/06-redundancy-and-code-duplication.md) | Token and AST clone detection, leaf utility placement, and middleware extraction. |
| **07** | [**Sentrux CLI & MCP Cheatsheet**](reference/07-sentrux-cli-and-mcp-cheatsheet.md) | Commands (`check`, `gate`, `mcp`, `scan`, `plugin`) and MCP tool catalog. |
| **08** | [**Refactoring Playbook & Workflow**](reference/08-refactoring-playbook-and-step-by-step-workflow.md) | The 6-phase optimization workflow from baseline to 8,500+. |
| **09** | [**CI/CD & Structural Regression Gating**](reference/09-ci-cd-and-github-actions.md) | GitHub Actions and CI pipelines for `sentrux check` and `sentrux gate`. |
| **10** | [**Production rules.toml Templates**](reference/10-rules-toml-architecture-templates.md) | Drop-in templates for Clean Architecture, Web APIs, and Monorepos. |
| **11** | [**Multi-Language Refactoring Guide**](reference/11-multi-language-ast-and-refactoring-guide.md) | Concrete refactoring examples in TypeScript, Python, Go, and Java. |

---

## Continuous Verification Checklist

Before merging architectural changes:
1. **Tests**: Run project test suite (100% pass rate).
2. **Lints**: Run project linters with zero warnings.
3. **Sentrux**: Run `sentrux check .` to verify all architectural constraints and boundaries pass.
