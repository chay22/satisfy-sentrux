# Satisfy Sentrux Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/skills.sh-compatible-success)](https://skills.sh)

**Satisfy Sentrux skills guide AI coding agents to systematically satisfy Sentrux architectural quality gates, eliminate dependency cycles, reduce cyclomatic complexity, and achieve 8,500+ to 9,000+ quality scores.**

---

## ⚠️ Important: Pick Only ONE Skill

This repository provides **two distinct skills**. They share the same core architectural goal, but are customized for different environments:

| Skill | Focus | Best For |
| :--- | :--- | :--- |
| **`satisfy-sentrux`** | **Language-Agnostic** | Projects written in TypeScript, Python, Go, Java, C#, or mixed polyglot repositories. |
| **`satisfy-sentrux-rust`** | **Rust-Specialized** | Pure Rust codebases requiring Rust-specific AST rules, borrow checker navigation, and Cargo workspace layouts. |

> [!WARNING]
> **Do NOT install or enable both skills at the same time.**  
> They are mutually exclusive alternatives designed for the same architectural gate. Install **only one skill** that matches your project stack.

---

## Installation

You can install either skill directly into your project using `npx skills@latest add`:

### Option A: Interactive Selection
Run the command and select either `satisfy-sentrux` or `satisfy-sentrux-rust` from the prompt:

```bash
npx skills@latest add chay22/satisfy-sentrux
```

### Option B: Install Directly by Name

**For language-agnostic / general projects:**
```bash
npx skills@latest add chay22/satisfy-sentrux --skill satisfy-sentrux
```

**For Rust-focused projects:**
```bash
npx skills@latest add chay22/satisfy-sentrux --skill satisfy-sentrux-rust
```

> [!TIP]
> To install globally across all your coding agent workspaces, append the `-g` flag:
> ```bash
> npx skills@latest add chay22/satisfy-sentrux --skill satisfy-sentrux -g
> ```

---

## What These Skills Do

[Sentrux](https://github.com/chay22/satisfy-sentrux) uses structural static analysis to evaluate codebase health, calculating an overall **Quality Signal** (up to 10,000) using the geometric mean of 5 pillars:

$$\text{Quality Signal} = \sqrt[5]{\text{Modularity} \times \text{Acyclicity} \times \text{Depth} \times \text{Equality} \times \text{Redundancy}}$$

These skills teach coding agents how to:
1. **Eliminate Dependency Cycles (Acyclicity $\to$ 10,000):** Extract leaf types and break circular dependencies via Tarjan's SCC analysis.
2. **Flatten Cyclomatic Complexity (Equality Gini $\to$ 8,500+):** Refactor functions to maintain $\text{CC} \le 2$ using early returns, lookup tables, and single-responsibility handlers.
3. **Decentralize Hubs (Modularity $Q \to$ 7,500+):** Remove monolithic barrel files and central routers that trigger Newman's modularity penalties.
4. **Enforce Clean Boundaries (Hierarchy Depth $\to$ 8,800+):** Organize code into balanced 4–5 layer DAGs and configure `.sentrux/rules.toml`.
5. **Deduplicate Logic (Redundancy $\to$ 9,600+):** Consolidate shared utilities into zero-dependency leaf modules.

---

## References

- [**satisfy-sentrux**](https://github.com/chay22/satisfy-sentrux/blob/main/skills/satisfy-sentrux/SKILL.md)  
  *Language-Agnostic Edition.* Universal architectural guidelines and AST refactoring rules for TypeScript, Python, Go, Java, C#, and polyglot stacks to pass `sentrux check` quality gates.

- [**satisfy-sentrux-rust**](https://github.com/chay22/satisfy-sentrux/blob/main/skills/satisfy-sentrux-rust/SKILL.md)  
  *Rust Edition.* Dedicated Rust optimization guide tailored for Rust Tree-sitter AST nodes (`match_arm`, `if_expression`), branchless combinators, borrow checker safety, dynamic dispatch (`dyn Trait`), and Cargo workspace structures.

---

## License

This project is licensed under the [MIT License](LICENSE).
