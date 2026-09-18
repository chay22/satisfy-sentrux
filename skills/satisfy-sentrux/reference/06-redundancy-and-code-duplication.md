# 06 - Redundancy & Code Duplication

## Overview

Sentrux calculates a **Redundancy** score by analyzing code duplication, AST token clones, and structurally identical functions across the codebase:

$$\text{Redundancy Score} = 10,000 \times (1.0 - \text{Duplication Ratio})$$

Where `Duplication Ratio` is the proportion of duplicated AST tokens relative to total codebase tokens. To achieve an elite score of **8,500+**, Redundancy should exceed **9,600+** (duplication ratio $\le 0.04$).

---

## Clone Categories Detected by Sentrux

1. **Type 1 Clones (Exact Clones)**:
   - Copy-pasted blocks of code with identical whitespace, comments, and identifiers.
2. **Type 2 Clones (Renamed Clones)**:
   - Syntactically identical AST structures where only identifiers, parameter names, or literal values differ.
3. **Repeated Controller/Handler Boilerplate**:
   - Repeated error response mapping (e.g. converting validation errors to JSON responses).
   - Duplicate HTTP client header and retry setups.
   - Repeated date, currency, or phone number parsing logic.

---

## Deduplication Strategies Without Introducing Cycles

When extracting duplicate logic into shared functions, **ensure you do not create circular dependencies or violate layer boundaries**.

### 1. Leaf Utility Placement (`core/` or `common/`)
If a utility function (such as `parseIsoDate` or `sanitizeInput`) is needed across multiple layers (`presentation`, `application`, and `infrastructure`):
- **Never** import utilities from sibling feature modules.
- Place the function in the lowest common foundation layer (`src/core/utils/`).
- The dependency flow remains strictly downward.

---

### 2. Generic Middleware & Decorators
Instead of repeating authorization checks or input validation inside every handler:
- **TypeScript**: Use route middleware (Express/Koa) or decorators/interceptors (NestJS).
- **Python**: Use route decorators (FastAPI/Flask) or middleware functions.
- **Go**: Wrap handlers in composable middleware (`func(http.Handler) http.Handler`).

---

### 3. Centralized DTO / Schema Validation
Instead of manual field-by-field validation inside controller bodies:
- Use declarative schema validation (Zod in TS, Pydantic in Python, Go playground validator, Serde in Rust).
- Centralize validation within the data transfer object definition itself.
