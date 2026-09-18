# 05 - Acyclicity & Dependency Cycles

## Overview

Acyclicity measures the absence of circular references within the codebase's import and invocation graph. Sentrux uses **Tarjan's Strongly Connected Components (SCC)** algorithm to detect cycles across all supported languages.

$$\text{Acyclicity Score} = 10,000 \times \left(1.0 - \frac{\text{Cycle Edges}}{\text{Total Edges}}\right)$$

If the codebase has **0 cycle edges**, the Acyclicity score is a perfect **10,000**. Even a single circular dependency edge can cause the score to plunge to 8,000 or lower, which heavily degrades the overall geometric mean.

---

## How Circular Dependencies Arise Across Languages

1. **TypeScript / JavaScript**:
   - `userService.ts` imports `orderService.ts` to calculate total spend.
   - `orderService.ts` imports `userService.ts` to fetch customer details.
   - *Result*: Circular module reference. May cause runtime `undefined` during module evaluation or memory leaks.

2. **Python**:
   - `models/user.py` imports `utils/formatters.py`.
   - `utils/formatters.py` imports `models/user.py` for type annotations.
   - *Result*: `ImportError: cannot import name ... from partially initialized module` or cycle edges in static graph.

3. **Go**:
   - `package auth` imports `package user`.
   - `package user` imports `package auth`.
   - *Result*: Go compiler forbids circular package imports (`import cycle not allowed`), but circular dependencies between internal struct methods or callback interfaces can still trigger call-graph cycles.

---

## Strategies to Eliminate Cycle Edges

### 1. Leaf Type / DTO Extraction
Extract the mutually shared data models into a separate leaf file or package that depends on nothing else:

```
Before (Cycle):
  serviceA.ts <──> serviceB.ts

After (Acyclic DAG):
  serviceA.ts ──┐
                ├──> types.ts (Leaf)
  serviceB.ts ──┘
```

---

### 2. Dependency Inversion Principle (DIP)
Instead of module $A$ depending directly on concrete module $B$ and $B$ depending on $A$:
1. Module $A$ defines an abstract interface (e.g. `UserLookupPort`).
2. Module $B$ implements `UserLookupPort`.
3. The dependency points one-way: $B \to A$.

---

### 3. Event-Driven / Observer Decoupling
If Service $A$ needs to trigger actions in Service $B$ upon completion:
- Service $A$ emits an event (e.g. `UserRegisteredEvent`).
- Service $B$ subscribes to the event.
- Service $A$ has zero knowledge or imports of Service $B$.
