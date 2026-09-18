# 09 - CI/CD Integration & Quality Gating

## Overview

Integrating Sentrux into your CI/CD pipeline ensures that structural regressions, circular dependencies, layering violations, and complexity spikes are caught automatically before pull requests are merged.

---

## 1. Golden Rule: Keep `baseline.json` Gitignored

A common question is whether `.sentrux/baseline.json` should be committed to git.

### **No — `.sentrux/baseline.json` should remain `.gitignored`**.

Here is why:
1. **Source of Truth is `rules.toml`**: The definitive architectural rules live in `.sentrux/rules.toml` (which *is* committed to git). It enforces hard ceilings (`max_cycles = 0`, `max_cc = 10`, `[[layers]]`, `[[boundaries]]`, and `min_quality = 8500`).
2. **Git Churn & Merge Conflicts**: Committing JSON snapshots on every milestone creates noisy diffs and frequent merge conflicts when multiple pull requests are in flight.
3. **Stale Baseline Risk**: If developers improve code quality but forget to regenerate `baseline.json`, CI gates against an outdated, lower standard.
4. **Intended Role of `baseline.json`**:
   - `sentrux gate . --save` is designed as an **ephemeral local scratchpad** for developers during refactoring sessions.
   - You run `--save` before editing, check `sentrux gate .` while refactoring to verify you haven't regressed from where you started, and discard it when finished.

---

## 2. Recommended CI/CD Approach: Declarative Quality Gates

The standard, robust way to run Sentrux in CI is running `sentrux check .` backed by a `min_quality` constraint in `.sentrux/rules.toml`:

```toml
[constraints]
max_cycles = 0
max_coupling = "B"
max_cc = 10
no_god_files = true
min_quality = 8500   # Fails CI if score drops below 8,500
```

---

## 3. GitHub Actions Workflow

Add this to `.github/workflows/sentrux.yml`:

```yaml
name: Architecture Quality Gate

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  architecture:
    name: Sentrux Quality Gate
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Full git history for churn and temporal coupling analysis

      - name: Install Sentrux
        run: curl -sSL https://raw.githubusercontent.com/sentrux/sentrux/main/install.sh | sudo bash

      - name: Verify Sentrux Installation
        run: sentrux --version

      - name: Architecture Quality Check
        run: sentrux check .
```

`sentrux check .` will evaluate all layers, boundaries, cyclomatic complexity limits, and the `min_quality` score. If any rule fails, the workflow fails immediately with detailed diagnostics.

---

## 4. Optional: Ephemeral PR Regression Gating (Without Committing Files)

If your team wants dynamic regression gating (ensuring a PR branch never has a lower score than `main`) **without committing baseline files to git**, generate the baseline dynamically inside the CI runner:

```yaml
      - name: Ephemeral Regression Gate
        if: github.event_name == 'pull_request'
        run: |
          # 1. Fetch main and generate ephemeral baseline on the fly
          git checkout main
          sentrux gate . --save # Local to this runner only

          # 2. Switch back to PR branch and compare
          git checkout -
          sentrux gate . # Fails if PR is worse than main
```
This guarantees strict forward progression while keeping the repository clean and 100% free of committed JSON snapshot files.
