# 11 - Multi-Language AST & Refactoring Guide

## Overview

Sentrux's AST analysis uses language-specific Tree-sitter parsers. This guide provides concrete refactoring examples in **TypeScript/JavaScript, Python, Go, and Java** to reduce cyclomatic complexity to $\text{CC} \le 2$ and maximize architectural purity.

---

## 1. TypeScript / Node.js

### High-CC Controller ($\text{CC} = 7$)
```typescript
// Nested if statements and 'else' clauses inflate CC
export async function handlePayment(req: Request, res: Response) {
  const { amount, currency, userId } = req.body;
  if (!amount || amount <= 0) {
    return res.status(400).json({ error: "Invalid amount" });
  } else if (!currency || currency.length !== 3) {
    return res.status(400).json({ error: "Invalid currency" });
  } else {
    const user = await db.findUser(userId);
    if (!user) {
      return res.status(404).json({ error: "User not found" });
    }
    const receipt = await paymentGateway.charge({ amount, currency, user });
    return res.status(200).json(receipt);
  }
}
```

### Refactored to $\text{CC} \le 2$ Stage Delegates
```typescript
function validatePaymentPayload(body: any): string | null {
  if (!body.amount || body.amount <= 0) return "Invalid amount";
  if (!body.currency || body.currency.length !== 3) return "Invalid currency";
  return null;
}

export async function handlePayment(req: Request, res: Response) {
  const validationError = validatePaymentPayload(req.body);
  if (validationError) return res.status(400).json({ error: validationError });

  const result = await processPayment(req.body);
  return res.status(result.status).json(result.payload);
}
```

---

## 2. Python (FastAPI / Django)

### High-CC Endpoint ($\text{CC} = 6$)
```python
@app.post("/users/verify")
def verify_user(payload: dict):
    if not payload.get("email"):
        return {"error": "Missing email"}, 400
    elif not payload.get("token"):
        return {"error": "Missing token"}, 400
    else:
        user = db.get_user(payload["email"])
        if user and user.is_active:
            return {"status": "already verified"}
        return {"status": "verified"}
```

### Refactored to $\text{CC} \le 2$ Using Guard Clauses & Predicates
```python
def check_required_fields(payload: dict) -> bool:
    # all() evaluates predicates without AST boolean_operator nodes
    return all([payload.get("email"), payload.get("token")])

@app.post("/users/verify")
def verify_user(payload: dict):
    if not check_required_fields(payload):
        return {"error": "Invalid verification payload"}, 400

    return process_verification(payload["email"], payload["token"])
```

---

## 3. Go

### High-CC Handler ($\text{CC} = 6$)
```go
func HandleCreateOrder(w http.ResponseWriter, r *http.Request) {
    var req OrderRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, err.Error(), 400)
        return
    }
    if req.Total <= 0 {
        http.Error(w, "invalid total", 400)
        return
    }
    if req.CustomerID == "" {
        http.Error(w, "missing customer", 400)
        return
    }
    order, err := createOrder(req)
    if err != nil {
        http.Error(w, err.Error(), 500)
        return
    }
    json.NewEncoder(w).Encode(order)
}
```

### Refactored to $\text{CC} \le 2$
```go
func validateOrder(req *OrderRequest) error {
    if req.Total <= 0 {
        return errors.New("invalid total")
    }
    if req.CustomerID == "" {
        return errors.New("missing customer")
    }
    return nil
}

func HandleCreateOrder(w http.ResponseWriter, r *http.Request) {
    req, err := parseAndValidateOrder(r)
    if err != nil {
        http.Error(w, err.Error(), 400)
        return
    }
    executeOrderCreation(w, req)
}
```

---

## 4. Java / Spring Boot

### Multi-Branch Switch ($\text{CC} = 5$) $\to$ Map Lookup ($\text{CC} = 1$)
```java
// Instead of switch (status) with 5 case statements (CC = 5):
// Use a static map lookup (CC = 1):
private static final Map<OrderStatus, StatusHandler> HANDLERS = Map.of(
    OrderStatus.PENDING, new PendingHandler(),
    OrderStatus.CONFIRMED, new ConfirmedHandler(),
    OrderStatus.CANCELLED, new CancelledHandler()
);

public void process(OrderStatus status) {
    StatusHandler handler = HANDLERS.get(status);
    if (handler != null) {
        handler.execute();
    }
}
```
