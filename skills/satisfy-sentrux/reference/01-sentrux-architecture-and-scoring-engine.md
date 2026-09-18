# 01 - Sentrux Architecture & Scoring Engine

## Overview

Sentrux is a language-agnostic static analysis engine and structural architecture quality gate. It evaluates codebases using graph theory and abstract syntax tree (AST) inspection, computing a compound **Quality Signal** score capped at **10,000**.

Because Sentrux relies on Tree-sitter grammars for AST parsing, its structural scoring operates with consistent mathematical rigor across **TypeScript/JavaScript, Python, Go, Rust, Java, and C#**.

---

## The Compound Quality Signal Formula

Sentrux computes the overall codebase quality as the **geometric mean** of five core structural metrics:

$$\text{Quality Signal} = \sqrt[5]{\text{Modularity} \times \text{Acyclicity} \times \text{Depth} \times \text{Equality} \times \text{Redundancy}}$$

Because the geometric mean is multiplicative, **any single weak metric severely suppresses the entire score**. A codebase with four perfect 10,000 scores and one 5,000 score will drop from 10,000 to 8,705.

### The 5 Metric Pillars

| Metric | Target | Formula / Normalized Derivation | Impact |
| :--- | :--- | :--- | :--- |
| **Acyclicity** | **10,000** | $10,000 \times (1.0 - \frac{\text{Cycle Edges}}{\text{Total Edges}})$ | Catastrophic if cycles exist. 0 cycles = 10,000. |
| **Redundancy** | **9,500+** | $10,000 \times (1.0 - \text{Duplication Ratio})$ | Penalizes clone groups and duplicated AST blocks. |
| **Hierarchy Depth** | **8,800+** | Depth-penalty function based on layer distance | Encourages balanced DAG depth (not too flat, not too deep). |
| **Complexity Equality** | **8,500+** | $10,000 \times (1.0 - \text{Gini Index})$ | High CC outliers cause Gini inequality, dragging score down. |
| **Modularity ($Q$)** | **7,500+** | $10,000 \times \frac{Q + 0.5}{1.5}$ (where $Q$ is Newman's modularity) | Penalizes high-degree import hubs and cross-module entanglement. |

---

## Sentrux Analysis Pipeline

```
┌────────────────────────────────────────────────────────┐
│ 1. Git & File Discovery                                │
│    git ls-files filtering, project map indexing       │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ 2. AST Extraction & Language Analysis                  │
│    Tree-sitter parser: imports, functions, calls, AST  │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ 3. Graph Construction                                  │
│    - File & module nodes                               │
│    - Import dependency edges                           │
│    - Call dependency edges                             │
│    - Layer containment edges                           │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ 4. Metric Computation                                  │
│    - Tarjan SCC for Acyclicity                         │
│    - Newman modularity Q for community clustering      │
│    - Complexity Gini for equality                      │
│    - Token clone detection for redundancy              │
│    - Layer hierarchy DAG depth                         │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│ 5. Rules & Gate Enforcement                            │
│    .sentrux/rules.toml: constraints, layers, boundaries│
└────────────────────────────────────────────────────────┘
```

---

## Quality Score Benchmarks

- **< 7,000**: Monolithic structure, entangled circular dependencies, frequent god functions ($\text{CC} \ge 15$).
- **7,000 – 7,999**: Basic layering, lingering hub files (barrel exports), wide variation in function complexity ($\text{Gini} > 0.30$).
- **8,000 – 8,499**: Clean acyclic DAG, architectural boundary rules passing, no functions with $\text{CC} > 5$.
- **8,500 – 8,999**: High structural purity: zero functions with $\text{CC} \ge 3$, modular router delegates, deduplicated utilities, perfect acyclicity ($10,000$).
- **9,000+**: Ultra-balanced community structure, near-zero complexity variance ($\text{Gini} \le 0.10$), zero clone groups, optimal modularity ($Q > 0.70$).
