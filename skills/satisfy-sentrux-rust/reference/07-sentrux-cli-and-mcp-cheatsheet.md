# 07 - Sentrux CLI & MCP Cheatsheet

## Overview

Sentrux provides both a command-line interface (`sentrux`) and a Model Context Protocol (MCP) server for automated architectural validation and continuous quality gating.

---

## Sentrux CLI Reference

```bash
# 1. Enforce architectural rules in .sentrux/rules.toml
sentrux check .

# 2. Local regression gate — compare against a local checkpoint
sentrux gate .

# 3. Save current state as local ephemeral checkpoint before refactoring (keep baseline.json gitignored)
sentrux gate . --save

# 4. Start the MCP server for AI pair programming
sentrux mcp

# 5. Open the GUI visualization pre-loaded with current directory
sentrux scan .

# 6. Check installed plugins and language support
sentrux plugin list

# 7. Verify version
sentrux --version
```

---

## Sentrux MCP Server & Tool Reference

When configuring Sentrux as an MCP server, the following tools become available:

| Tool | Purpose | Key Parameters |
| :--- | :--- | :--- |
| `scan` | Initiates an architectural scan of a workspace | `path: string` |
| `rescan` | Refreshes graph analysis after code changes | - |
| `health` | Fetches overall structural health metrics | - |
| `check_rules` | Evaluates rules in `.sentrux/rules.toml` and returns violations | `path?: string` |
| `dsm` | Returns the Dependency Structure Matrix | - |
| `test_gaps` | Analyzes untested modules and dependency blast radius | - |
| `git_stats` | Analyzes churn, author coupling, and hotspot risk | - |

---

## Debugging & Metric Introspection Techniques

When `sentrux check .` outputs only the top-level score (`Quality: 8764`), you can inspect the internal 5-metric breakdown using debugger or binary analysis tools:

### 1. Inspecting Sub-metrics with GDB

```bash
gdb -batch \
  -ex "file /usr/local/bin/sentrux" \
  -ex "b sentrux_core::metrics::rules::checks::check_min_quality" \
  -ex "run check ." \
  -ex "p *(double*)($rdx + 0x1f0)" \
  /usr/local/bin/sentrux
```

### 2. Inspecting Graph Node & Edge Counts

From standard `sentrux check .` output:
```
[build_graphs] 61 files | maps 0.5ms, imports 14.5ms, calls+inherit 1.0ms, total 16.0ms | 60 import, 47 call, 0 inherit edges
```
- **Files**: Total nodes in the structural graph.
- **Import Edges**: Number of internal intra-crate `use crate::...` connections.
- **Call Edges**: Invocations detected across file boundaries.
- **Inherit Edges**: Trait implementations / polymorphism links.

### 3. Locating CC Outliers via Tree-sitter or Ripgrep

To quickly find functions that exceed $\text{CC} \ge 3$:
```bash
# Find functions with nested ifs or match statements
rg "(\bif\b|\bmatch\b|\bfor\b|\bwhile\b)" src/ -c | sort -t: -k2 -nr | head -n 20
```
