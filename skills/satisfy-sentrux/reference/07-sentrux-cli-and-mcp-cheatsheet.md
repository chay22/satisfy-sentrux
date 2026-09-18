# 07 - Sentrux CLI & MCP Cheatsheet

## Overview

Sentrux provides both a command-line interface (`sentrux`) and a Model Context Protocol (MCP) server for automated architectural validation, structural regression gating, and live graph exploration.

---

## Sentrux CLI Reference

```bash
# 1. Enforce architectural rules defined in .sentrux/rules.toml
sentrux check .

# 2. Local regression gate — compare against a local checkpoint
sentrux gate .

# 3. Save current state as local ephemeral checkpoint before refactoring (keep baseline.json gitignored)
sentrux gate . --save

# 4. Start the MCP server for AI coding assistants
sentrux mcp

# 5. Open the interactive GUI visualizer pre-loaded with target directory
sentrux scan .

# 6. Manage and inspect language plugins (Tree-sitter grammars)
sentrux plugin list

# 7. Check version
sentrux --version
```

---

## Sentrux MCP Server Reference

When integrating Sentrux with AI agents via Model Context Protocol (MCP):

| Tool | Purpose | Key Parameters |
| :--- | :--- | :--- |
| `scan` | Triggers a full architectural scan of a workspace directory | `path: string` |
| `rescan` | Refreshes graph analysis after modifying files | - |
| `health` | Retrieves high-level structural health metrics (Quality, Acyclicity, Modularity) | - |
| `check_rules` | Evaluates rules in `.sentrux/rules.toml` and returns exact violation locations | `path?: string` |
| `dsm` | Returns the Dependency Structure Matrix | - |
| `test_gaps` | Analyzes untested modules and dependency blast radius | - |
| `git_stats` | Analyzes code churn, author coupling, and temporal hotspots | - |

---

## Interpreting Output

A typical `sentrux check .` run outputs:
```
Scanning ....
[scan] git ls-files: 85 total, 84 kept, 1 dropped
[build_project_map] 84 files, 22 unique dirs
[resolve] 84 resolved, 0 unresolved
[build_graphs] 84 files | 120 import, 65 call edges
sentrux check — 15 rules checked

Quality: 8840

✓ All rules pass
```
- **Files**: Total indexed source files matching supported language extensions.
- **Import Edges**: Directed references between files.
- **Call Edges**: Invocations detected across file boundaries.
- **Quality**: The geometric mean of the 5 architectural metrics.
