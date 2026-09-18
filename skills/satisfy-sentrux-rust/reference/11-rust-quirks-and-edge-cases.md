# 11 - Rust-Specific Quirks & Tree-Sitter AST Edge Cases

## Overview

Sentrux analyzes Rust codebases using Tree-sitter's Rust grammar. While Tree-sitter provides fast syntactic parsing, it differs from `rustc` because it performs static AST matching without full type resolution or macro expansion. Understanding these nuances allows you to structure Rust code for optimal architectural scores.

---

## 1. How Re-Exports (`pub use`) Affect the Import Graph

When you write:
```rust
// in src/domain/mod.rs
pub use self::wallet::Wallet;
```
Sentrux registers an import edge from `src/domain/mod.rs` to `src/domain/wallet.rs`.

### The Facade Re-export Trap
If an outer layer imports through a re-export facade:
- `src/api/handlers/wallets.rs` uses `crate::domain::Wallet`.
- If `crate::domain` re-exports everything from 10 submodules, Sentrux's suffix resolver may map the import edge to `src/domain/mod.rs` rather than `src/domain/wallet.rs`.
- **Architectural Impact**: `domain/mod.rs` accumulates an artificial hub degree, reducing Newman's $Q$.
- **Best Practice**: Direct imports (`use crate::domain::wallet::Wallet;`) produce precise, one-to-one dependency edges that reflect actual module communication.

---

## 2. Macros & Declarative Macro Expansions

Because Tree-sitter parses the raw unexpanded syntax tree:
1. **`macro_rules!` Blocks**:
   - Conditionals or loops inside a `macro_rules!` invocation may not be counted towards the caller function's cyclomatic complexity if the macro invocation is parsed as a generic `macro_invocation` AST node.
   - However, repetitive boilerplate inside macros can inflate token duplication in clone detection.
2. **Derive Macros (`#[derive(...)]`)**:
   - Do not increment cyclomatic complexity.
   - Safe to use extensively without affecting the Complexity Equality score.
3. **Format & Query Macros (`format!`, `sqlx::query!`)**:
   - `format!` does not generate branch nodes.
   - `sqlx::query_as!` generates minimal AST tokens in the outer function, keeping $\text{CC}$ low.

---

## 3. Dynamic Dispatch (`dyn Trait`) vs Static Generics (`impl Trait`)

Sentrux extracts both **import edges** and **call edges**:
- **Static Dispatch (`T: Trait` / `impl Trait`)**:
  - Sentrux directly links the caller file to the trait definition module.
- **Dynamic Dispatch (`Box<dyn Trait>` / `&dyn Trait`)**:
  - The call edge points only to the interface definition in the port/domain module.
  - The concrete infrastructure adapter is fully decoupled from the caller, achieving 100% boundary isolation.

---

## 4. Single-File vs Directory Modules in Rust 2021/2024

Rust supports two module file layouts:
1. **Directory layout**: `src/foo/mod.rs` + `src/foo/bar.rs`
2. **Sibling layout**: `src/foo.rs` + `src/foo/bar.rs`

Sentrux indexes both cleanly via `[build_project_map]`. However, for glob patterns in `.sentrux/rules.toml`:
- Use `src/foo/**` to match all submodules and siblings.
- Keep layer ordering aligned with the module directory hierarchy.
