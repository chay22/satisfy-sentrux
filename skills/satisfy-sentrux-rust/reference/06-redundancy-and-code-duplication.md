# 06 - Redundancy & Code Duplication

## Overview

Sentrux calculates a **Redundancy** score by detecting duplicated code blocks, similar AST structures, and cloned functions across the project:

$$\text{Redundancy Score} = 10,000 \times (1.0 - \text{Duplication Ratio})$$

Where `Duplication Ratio` is the ratio of duplicated AST tokens/nodes to total code tokens. In a pristine codebase, Redundancy should exceed **9,600+** (duplication ratio $\le 0.04$).

---

## What Sentrux Detects as Redundancy

1. **Type 1 Clones (Exact Duplicates)**:
   - Copy-pasted helper functions across different handler or worker files (e.g., date parsing, currency formatting, hash generation).
2. **Type 2 Clones (Renamed Variables / Identical AST)**:
   - Functions with identical control flow and statements, differing only in identifier names.
3. **Repeated Boilerplate**:
   - Repeated Axum extractor error response mappings.
   - Repeated SQL transaction commit/rollback wrappers.
   - Identical HTTP client header construction logic.

---

## Deduplication Strategies Without Violating Layering

When deduplicating code, **be extremely careful not to introduce cycle edges or boundary violations**. 

### 1. Leaf Utility Placement (`src/core/utils/`)

If a utility is needed by multiple layers (e.g. `domain`, `infra`, and `workers`), it must live in the lowest common layer (`src/core/`):

- **Bad**: `src/workers/payment_sync/bank_worker.rs` imports a date parser from `src/infra/scrapers/web_scraper.rs`.
  - *Violation*: Worker directly depends on scraper infrastructure.
- **Good**: Move `parse_dmy_date(s: &str) -> Result<NaiveDate, ...>` to `src/core/date_utils.rs`.
  - Both `bank_worker.rs` and `web_scraper.rs` import from `src/core/date_utils.rs`.
  - Layer flow is clean: `workers -> core` and `infra -> core`.

---

### 2. Macro vs Generic Function Deduplication

For repeated error-handling blocks or JSON responses, prefer small generic helper functions over giant macros:

```rust
// In src/api/handlers/common.rs
pub fn json_ok<T: serde::Serialize>(data: T) -> (axum::http::StatusCode, axum::Json<serde_json::Value>) {
    (
        axum::http::StatusCode::OK,
        axum::Json(serde_json::json!({
            "status": "success",
            "data": data
        })),
    )
}
```
This centralizes the serialization format while keeping cyclomatic complexity at $\text{CC} = 1$.

---

### 3. Shared Deserializers & Converters

Instead of writing manual string-to-number or string-to-date loops inside multiple functions, implement standard Rust traits:

- Implement `std::str::FromStr` for domain types.
- Implement `serde::Deserialize` or use custom `#[serde(deserialize_with = "...")]` functions.
- Centralize parsing in the type definition itself.
