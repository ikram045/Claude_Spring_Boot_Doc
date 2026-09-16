[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 39. Lazy vs Eager Loading

## 39.1 What is it?

**Simple words:**
When you load an `Order`, should Hibernate also load all its `OrderItem`s immediately?

- **EAGER** = yes, load everything now
- **LAZY** = no, load them only if the code actually asks for them

**Technical wording:**
`FetchType` determines whether an association is initialised at load time (EAGER) or deferred until first access via a proxy or collection wrapper (LAZY).

## 39.2 The defaults (memorise this table)

| Relationship | Default fetch | Is the default sensible? |
|---|---|---|
| `@OneToOne` | **EAGER** | ❌ No — override to LAZY |
| `@ManyToOne` | **EAGER** | ❌ No — override to LAZY |
| `@OneToMany` | **LAZY** | ✅ Yes |
| `@ManyToMany` | **LAZY** | ✅ Yes |

**The memory aid:** the annotations ending in **`ToOne` default to EAGER**; those ending in **`ToMany` default to LAZY**.

**The rule to apply everywhere:**
```java
@ManyToOne(fetch = FetchType.LAZY)      // ALWAYS write this explicitly
@OneToOne(fetch = FetchType.LAZY)       // ALWAYS write this explicitly
```

## 39.3 Why EAGER is harmful

```java
@Entity
public class OrderItem {
    @ManyToOne                    // EAGER by default!
    private Order order;
}

@Entity
public class Order {
    @ManyToOne                    // EAGER by default!
    private Customer customer;
}

@Entity
public class Customer {
    @ManyToOne                    // EAGER by default!
    private Address address;
}
```

Loading **one** `OrderItem` now triggers:
```sql
SELECT * FROM order_items WHERE id = 1;
SELECT * FROM orders      WHERE id = ?;    -- because of EAGER
SELECT * FROM customers   WHERE id = ?;    -- because of EAGER
SELECT * FROM addresses   WHERE id = ?;    -- because of EAGER
```
**Four queries to read one row** — and you may have needed only the product name.

**Worse:** EAGER is applied even when the query explicitly doesn't need it. `findAll()` on 100 order items would fire hundreds of extra queries. **EAGER cannot be turned off per query; LAZY can always be overridden with a fetch join.** That asymmetry is the core argument.

**State it this way to a TL:** *"LAZY is the safe default because we can always fetch more when we need it. EAGER is a decision baked into the mapping that every query pays for, whether it needs the data or not."*

## 39.4 How lazy loading works internally

```java
Order order = orderRepository.findById(1L).get();
// SELECT * FROM orders WHERE id = 1
// order.items is NOT a real ArrayList — it's a PersistentBag proxy (uninitialised)

System.out.println(order.getItems().size());   // ← FIRST ACCESS
// NOW: SELECT * FROM order_items WHERE order_id = 1
```

For `@ManyToOne`, Hibernate creates a **proxy subclass** of the target entity with all fields uninitialised except the ID. Touching any other field triggers the SELECT.

```java
OrderItem item = repo.findById(1L).get();
Order order = item.getOrder();      // a proxy — NO query yet
Long id = order.getId();            // still NO query — the ID is already known
String num = order.getOrderNumber();// ← query fires HERE
```
**Useful consequence:** you can set a foreign key without loading the parent row:
```java
item.setOrder(entityManager.getReference(Order.class, orderId));   // no SELECT
```

## 39.5 `LazyInitializationException` — the classic failure

```java
@Service
public class OrderService {

    public Order getOrder(Long id) {          // ❌ no @Transactional
        return orderRepository.findById(id).orElseThrow();
    }   // ← transaction ends, session closes, entity is DETACHED
}

@RestController
public class OrderController {
    @GetMapping("/{id}")
    public Order get(@PathVariable Long id) {
        Order order = orderService.getOrder(id);
        return order;     // Jackson serializes → touches order.getItems()
    }                     // ❌ LazyInitializationException:
}                         //    "could not initialize proxy - no Session"
```

**The cause, in one line:** the lazy proxy needs an open Hibernate session to run its query, and the session closed when the transaction ended.

### The four fixes, ranked

```java
// ✅ FIX 1 — JOIN FETCH (BEST: one query, explicit, works with DTOs)
@Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items WHERE o.id = :id")
Optional<Order> findByIdWithItems(@Param("id") Long id);

// ✅ FIX 2 — @EntityGraph (declarative, reusable)
@EntityGraph(attributePaths = {"items", "customer"})
Optional<Order> findById(Long id);

// ✅ FIX 3 — map to a DTO inside the transaction (our standard approach)
@Transactional(readOnly = true)
public OrderResponseDto getOrder(Long id) {
    Order order = orderRepository.findById(id).orElseThrow();
    return orderMapper.toDto(order);     // lazy fields initialised while session is open
}

// ❌ FIX 4 — open-in-view (the default, and the one to DISABLE)
// spring.jpa.open-in-view=true keeps the session open for the whole HTTP request
```

### Why `open-in-view` should be disabled

Spring Boot enables `spring.jpa.open-in-view=true` **by default**, which keeps the Hibernate session open for the entire request — so lazy loading "just works" in the controller and during serialization.

**Why that is a problem:**

| Issue | Explanation |
|---|---|
| **Hidden N+1** | Serialization silently triggers queries; you never see them in the service |
| **Connection held too long** | The DB connection is held through view rendering, starving the pool |
| **No transaction** | Those queries run outside a transaction — each in auto-commit |
| **Unpredictable performance** | Query count depends on what Jackson touches |
| **Hides real bugs** | Masks missing fetch joins until production load reveals them |

```properties
spring.jpa.open-in-view=false      # set this in EVERY project
```

**What happens when you disable it:** you immediately get `LazyInitializationException` wherever a fetch was missing. That feels like a regression but is actually the point — **it surfaces exactly the places that were doing hidden queries.** Fix each with a fetch join, an entity graph, or DTO mapping inside the transaction.

Boot even logs a warning about this at startup, which most teams ignore. Turning it off is one of the highest-value single-line changes you can propose.

## 39.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Leaving `@ManyToOne` EAGER | Extra queries on every load |
| Relying on `open-in-view` | Hidden N+1, connections held, unpredictable latency |
| Accessing lazy fields outside a transaction | `LazyInitializationException` |
| `JOIN FETCH` on two collections at once | `MultipleBagFetchException` |
| `JOIN FETCH` with pagination | **Hibernate paginates in memory** — loads everything first |
| Serializing entities directly | Triggers lazy loads during serialization |

**`MultipleBagFetchException` — worth knowing the fix:**
```java
// ❌ Fails: cannot fetch two List collections in one query
@Query("SELECT o FROM Order o JOIN FETCH o.items JOIN FETCH o.payments")

// ✅ Fix 1: use Set instead of List for the collections
// ✅ Fix 2: two separate queries — the second reuses the persistence context
// ✅ Fix 3: @BatchSize(size = 20) on the collection
```

**`JOIN FETCH` + pagination — the silent killer:**
Hibernate logs `HHH000104: firstResult/maxResults specified with collection fetch; applying in memory` and then **loads the entire result set into memory** before paginating. On a large table this is an out-of-memory incident. Use `@EntityGraph` with a two-query approach, or `@BatchSize`, when you need both.

## 39.7 TL Questions

**Q: What's the difference between lazy and eager?**
A: Eager loads the association immediately with the parent; lazy defers it until the code actually accesses it, using a proxy.

**Q: What are the defaults and do we keep them?**
A: The `ToOne` associations default to EAGER and the `ToMany` ones to LAZY. We override every `ToOne` to LAZY explicitly, because eager fetching is baked into the mapping and every query pays for it, whereas lazy can always be upgraded with a fetch join when we need the data.

**Q: What causes `LazyInitializationException`?**
A: Touching a lazy association after the transaction has closed. The proxy needs an open session to run its query and there isn't one. We fix it by fetching what we need inside the transaction — a join fetch, an entity graph, or mapping to a DTO before returning.

**Q: Why do we disable `open-in-view`?**
A: Because it keeps the session open for the whole request, so lazy loads happen silently during JSON serialization. That hides N+1 problems, holds a database connection far longer than needed, and runs those queries outside any transaction. Disabling it makes the missing fetches fail loudly in development instead of quietly degrading production.

**Q: What happens internally when we access a lazy field?**
A: For a collection, Hibernate holds a `PersistentBag` proxy that issues a SELECT on first access. For a `ToOne`, it holds a proxy subclass with only the ID populated; touching any other property triggers the query.

**Q: What's the catch with `JOIN FETCH` and pagination?**
A: Hibernate can't apply SQL `LIMIT` correctly when a collection join multiplies the rows, so it loads the entire result set and paginates in memory. It logs a warning most people miss. For paginated queries with collections we use an entity graph or `@BatchSize` instead.

## 39.8 Interview Questions

**Beginner — What's the default fetch type for `@OneToMany`?**
LAZY. `@ManyToOne` and `@OneToOne` default to EAGER.

**Intermediate — How does Hibernate implement lazy loading?**
Through proxies — a CGLIB/ByteBuddy subclass for `ToOne` associations and a `PersistentCollection` wrapper for collections. The query fires on first access to any non-identifier property.

**Intermediate — Why can you set a foreign key without loading the parent?**
Because the proxy already holds the identifier. `getReference()` returns a proxy and calling `getId()` doesn't trigger a query, so you can assign the association without a SELECT.

**Advanced — What is `MultipleBagFetchException`?**
Hibernate cannot fetch two `List`-mapped collections (bags) in a single query, because the cartesian product makes row-to-element attribution ambiguous. Fix by using `Set`, splitting into two queries, or using `@BatchSize`.

**Advanced — Why is in-memory pagination with collection fetch dangerous?**
Because Hibernate must fetch the entire result set before it can paginate, so memory usage scales with the whole table rather than the page size. It's a latent out-of-memory failure that only appears as the data grows.

## 39.9 Quick Revision

1. **`ToOne` defaults EAGER; `ToMany` defaults LAZY.**
2. **Always override `ToOne` to LAZY explicitly.**
3. LAZY can be upgraded per query; EAGER cannot be switched off.
4. Lazy uses proxies; the query fires on first non-ID access.
5. `LazyInitializationException` = lazy access after the session closed.
6. Fixes: `JOIN FETCH`, `@EntityGraph`, DTO mapping inside the transaction.
7. **Set `spring.jpa.open-in-view=false` in every project.**
8. `JOIN FETCH` + pagination → in-memory pagination. Avoid.
9. Two `List` fetch joins → `MultipleBagFetchException`; use `Set` or `@BatchSize`.
10. `getReference()` sets an FK without a SELECT.

## 39.10 TL Explanation (speak this)

> "We set every `ToOne` association to LAZY explicitly, because JPA defaults those to EAGER and that cost is baked into the mapping — every query pays for it whether it needs the data or not. Lazy is the safe default since we can always fetch more with a join fetch or entity graph when a specific use case needs it. We also disable `open-in-view`, which Boot enables by default: it keeps the Hibernate session open through JSON serialization, so lazy loads happen silently and we get hidden N+1 queries and connections held far longer than necessary. Turning it off makes missing fetches fail loudly in development, which is exactly where we want to find them."

---

[← 38. Cascade Types](38-cascade-types.md) | [Contents](../README.md) | [40. The N+1 Select Problem and EntityGraph →](40-the-n-1-select-problem-and-entitygraph.md)
