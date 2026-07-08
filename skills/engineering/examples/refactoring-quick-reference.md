# Refactoring Pattern Catalog

One before/after per pattern, best language for each.

## Code Smell Fixes

### Extract Method (Long Function)
```python
# BEFORE: 200-line function doing 5 things
def process_order(order):
    # validate (10 lines) ... calculate totals (15 lines)
    # apply discounts (20 lines) ... save to db (10 lines) ... send email (10 lines)

# AFTER: main function reads like a recipe
def process_order(order):
    validate_order(order)
    totals = calculate_order_totals(order)
    totals = apply_discounts(totals, order.customer)
    save_order(order, totals)
    send_confirmation_email(order)
```

### Guard Clauses (Deep Nesting)
```go
// BEFORE: 4 levels of nesting
func processPayment(p Payment) error {
    if p.Amount > 0 {
        if p.Account != nil {
            if p.Account.Balance >= p.Amount {
                // main logic here
            } else { return errors.New("insufficient funds") }
        } else { return errors.New("no account") }
    } else { return errors.New("invalid amount") }
}

// AFTER: flat, linear flow
func processPayment(p Payment) error {
    if p.Amount <= 0        { return errors.New("invalid amount") }
    if p.Account == nil      { return errors.New("no account") }
    if p.Account.Balance < p.Amount { return errors.New("insufficient funds") }
    // main logic here - no nesting!
}
```

### Domain Types (Primitive Obsession)
```rust
// BEFORE: raw strings passed everywhere
fn validate_email(email: &str) -> bool { ... }
fn send_to(email: &str) { ... }  // could pass any string!

// AFTER: type-safe wrapper ensures validity at construction
struct Email(String);
impl Email {
    fn new(s: String) -> Result<Email, ValidationError> {
        if is_valid_email(&s) { Ok(Email(s)) } else { Err(ValidationError) }
    }
}
fn send_to(email: &Email) { ... }  // can only receive validated email
```

### Polymorphism (Switch on Type Code)
```typescript
// BEFORE: switch repeated in multiple places
function calculateBonus(emp: Employee): number {
    switch(emp.type) {
        case 'engineer': return emp.salary * 0.1;
        case 'manager':  return emp.salary * 0.2 + emp.teamSize * 1000;
    }
}

// AFTER: each type handles its own logic
abstract class Employee { abstract calculateBonus(): number; }
class Engineer extends Employee {
    calculateBonus() { return this.salary * 0.1; }
}
class Manager extends Employee {
    calculateBonus() { return this.salary * 0.2 + this.teamSize * 1000; }
}
```

### Extract Predicate (Comments Explaining Code)
```c
// BEFORE: comment needed to explain condition
// Check if customer is eligible for discount
if (c.orders > 10 && c.total > 1000 && c.years > 2) { ... }

// AFTER: self-documenting function name
if (isEligibleForLoyaltyDiscount(customer)) { ... }

bool isEligibleForLoyaltyDiscount(Customer c) {
    return c.orders > 10 && c.total > 1000 && c.years > 2;
}
```

### Parameter Object (Data Clumps)
```go
// BEFORE: same group of params repeated
func drawLine(x1, y1, x2, y2 int)
func distance(x1, y1, x2, y2 int) float64

// AFTER: extract domain concept
type Point struct { X, Y int }
func drawLine(start, end Point)
func distance(start, end Point) float64
```

## Performance Patterns

### Data Layout (AoS → SoA)
```cpp
// BEFORE: Array of Structs - loads unused fields into cache
struct Particle { Vector3 pos, vel, accel; float mass, lifetime; Color color; bool active; };
std::vector<Particle> particles;  // cache loads entire struct per access

// AFTER: Structure of Arrays - only load what you need
std::vector<Vector3> positions;   // hot: touched every frame
std::vector<Vector3> velocities;  // hot: touched every frame
std::vector<float> lifetimes;     // cold: checked occasionally
// 10-100x faster: sequential access, SIMD-friendly, no wasted cache loads
```

### Batch Operations (N+1 Query)
```javascript
// BEFORE: N queries (one per item)
for (const user of users) {
    await db.update('users', { lastSeen: now }, { id: user.id });
}

// AFTER: 1 query
const ids = users.map(u => u.id);
await db.update('users', { lastSeen: now }, { id: { $in: ids } });
```

### Buffer Reuse (Allocation Storm)
```cpp
// BEFORE: allocates string every iteration
for (int i = 0; i < 1000000; i++) {
    std::string msg = "Processing: " + std::to_string(i);
    process(msg);
}

// AFTER: reuse buffer
std::string buffer;
buffer.reserve(256);
for (int i = 0; i < 1000000; i++) {
    buffer = "Processing: ";
    buffer += std::to_string(i);
    process(buffer);
}
```

## Architecture Patterns

### Dependency Injection
```java
// BEFORE: tight coupling, untestable
class UserService {
    private MySQLDatabase db = new MySQLDatabase();
}

// AFTER: injectable, testable
class UserService {
    private Database db;
    UserService(Database db) { this.db = db; }
}
```

### Async/Await (Callback Hell)
```typescript
// BEFORE: nested callbacks
fetchUser(id, (err, user) => {
    fetchPosts(user.id, (err, posts) => {
        fetchComments(posts[0].id, (err, comments) => {
            callback(null, { user, posts, comments });
        });
    });
});

// AFTER: linear flow, parallel where possible
const user = await fetchUser(id);
const posts = await fetchUserPosts(user.id);
const comments = await fetchComments(posts[0].id);
await Promise.all([updateAnalytics(user.id), sendNotification(user.email)]);
return { user, posts, comments };
```
