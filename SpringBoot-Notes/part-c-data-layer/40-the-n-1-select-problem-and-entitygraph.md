[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 40. The N+1 Select Problem and EntityGraph

## 40.1 What is it?

**Simple words:**
You run **1** query to get a list of orders. Then, for each order, Hibernate runs **1 more query** to get its items.

10 orders → **11 queries**. 1,000 orders → **1,001 queries**.

That's the N+1 problem: **1** query for the parents plus **N** queries for the children.

**Technical wording:**
The N+1 select problem occurs when an initial query retrieves N entities and each associated collection or `ToOne` reference is subsequently initialised with its own query, resulting in N+1 statements where a single join would suffice.

## 40.2 Seeing it happen

```java
@Transactional(readOnly = true)
public List<OrderResponseDto> getAllOrders() {

    List<Order> orders = orderRepository.findAll();
    // QUERY 1:  SELECT * FROM orders                      ← returns 100 orders

    return orders.stream()
            .map(order -> {
                // On EACH iteration, the lazy collection initialises:
                // QUERY 2..101:  SELECT * FROM order_items WHERE order_id = ?
                int itemCount = order.getItems().size();
                return new OrderResponseDto(order.getId(), itemCount);
            })
            .toList();
}
// TOTAL: 101 queries instead of 1 or 2.
```

**Why it's so dangerous:** it **works perfectly in development**. With 10 test rows it's 11 fast queries — nobody notices. In production with 10,000 rows it's 10,001 queries, each with network round-trip latency. A 50ms endpoint becomes 30 seconds. **It is the single most common JPA performance failure.**

### Detecting it

```properties
# Development only
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG

# Best tool: count queries per request and fail the build if a threshold is exceeded
# (datasource-proxy or p6spy)
```
**A practice worth proposing to a TL:** add an integration test that asserts the query count for key endpoints. It turns N+1 from a production surprise into a build failure.

## 40.3 Solution 1 — JOIN FETCH

```java
@Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items WHERE o.status = :status")
List<Order> findByStatusWithItems(@Param("status") OrderStatus status);
```
```sql
-- ONE query
SELECT o.*, i.* FROM orders o
LEFT JOIN order_items i ON i.order_id = o.id
WHERE o.status = 'PENDING';
```

**Why `DISTINCT`:** the join produces one row per item, so an order with 3 items appears 3 times and Hibernate would return the same `Order` object 3 times in the list. `DISTINCT` de-duplicates the object list. (In Hibernate 6 this is automatic for entity queries; in Hibernate 5 you also wanted `hibernate.query.passDistinctThrough=false` so `DISTINCT` wasn't sent to the database unnecessarily.)

**Limitations:** cannot fetch two `List` collections (`MultipleBagFetchException`), and combined with pagination it paginates in memory.

## 40.4 Solution 2 — @EntityGraph (declarative, preferred)

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // Fetch items eagerly for THIS query only
    @EntityGraph(attributePaths = {"items"})
    List<Order> findByStatus(OrderStatus status);

    // Multiple, including nested paths
    @EntityGraph(attributePaths = {"items", "items.product", "customer"})
    Optional<Order> findById(Long id);
}
```

**Why `@EntityGraph` is usually the better choice:**

| | `JOIN FETCH` | `@EntityGraph` |
|---|---|---|
| Written in | The JPQL string | An annotation |
| Works on derived query methods | No | **Yes** |
| Works on inherited `findById` | No | **Yes** |
| Nested paths | Verbose | `"items.product"` |
| Reusable across queries | No | **Yes** (named graphs) |
| Combines with pagination | Poorly | Better |

**Named entity graphs — define once, reuse:**

```java
@Entity
@NamedEntityGraph(
    name = "Order.withItemsAndCustomer",
    attributeNodes = {
        @NamedAttributeNode("customer"),
        @NamedAttributeNode(value = "items", subgraph = "items-subgraph")
    },
    subgraphs = @NamedSubgraph(name = "items-subgraph",
                               attributeNodes = @NamedAttributeNode("product"))
)
public class Order { }

public interface OrderRepository extends JpaRepository<Order, Long> {
    @EntityGraph(value = "Order.withItemsAndCustomer")
    List<Order> findByStatus(OrderStatus status);
}
```

**`FETCH` vs `LOAD` graph type:**
- `EntityGraphType.FETCH` (default): listed attributes are EAGER, **everything else is LAZY**.
- `EntityGraphType.LOAD`: listed attributes are EAGER, everything else keeps its **mapped** fetch type.

`FETCH` gives tighter control and is usually what you want.

## 40.5 Solution 3 — @BatchSize

```java
@Entity
public class Order {
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    @BatchSize(size = 20)                  // Hibernate-specific
    private List<OrderItem> items;
}
```

Instead of one query per order, Hibernate batches IDs:
```sql
SELECT * FROM order_items WHERE order_id IN (1,2,3,...,20);
SELECT * FROM order_items WHERE order_id IN (21,...,40);
```
**100 orders: 101 queries → 6 queries.** Not one, but a massive improvement with **zero query changes**.

```properties
# Apply globally
spring.jpa.properties.hibernate.default_batch_fetch_size=20
```

**Why `@BatchSize` deserves attention:** it is the only solution that **works with pagination**, fixes N+1 across *all* queries at once, and requires no code change. Setting `default_batch_fetch_size` globally is a genuinely high-value one-line configuration — a strong suggestion to bring to a TL.

## 40.6 Solution 4 — DTO projection (best for read-only lists)

```java
@Query("""
       SELECT new com.company.orderservice.dto.OrderSummaryDto(
              o.id, o.orderNumber, o.totalAmount, COUNT(i))
       FROM Order o LEFT JOIN o.items i
       WHERE o.status = :status
       GROUP BY o.id, o.orderNumber, o.totalAmount
       """)
List<OrderSummaryDto> findSummaries(@Param("status") OrderStatus status);
```
One query, only the needed columns, no entities in the persistence context, no dirty-check snapshots. **For a list screen this is the fastest option available.**

## 40.7 Choosing between them

| Situation | Best solution |
|---|---|
| Need the full entity graph, no pagination | `JOIN FETCH` or `@EntityGraph` |
| Derived query method or inherited `findById` | **`@EntityGraph`** |
| **With pagination** | **`@BatchSize`** or `@EntityGraph` (two-query) |
| Read-only list/summary screen | **DTO projection** |
| Global safety net | **`default_batch_fetch_size`** |
| Multiple `List` collections | `Set` + fetch, or `@BatchSize` |

## 40.8 Common Mistakes

| Mistake | Consequence |
|---|---|
| Not noticing N+1 in development | Discovered in production under load |
| "Fixing" it with EAGER | Makes it worse — now *every* query over-fetches |
| `JOIN FETCH` with pagination | In-memory pagination, OOM risk |
| Two `List` fetch joins | `MultipleBagFetchException` |
| Forgetting `DISTINCT` (Hibernate 5) | Duplicate parent objects in results |
| Not enabling `default_batch_fetch_size` | Missing a free, global improvement |

**Why EAGER is not a fix — state this clearly:**
```java
@OneToMany(fetch = FetchType.EAGER)     // ❌ "solves" N+1 by making it universal
```
EAGER still fires a separate query per parent for collections (or one huge cartesian join), and now **every** query loading an `Order` pays the cost, including ones that never touch `items`. It converts a per-query problem into a permanent one.

## 40.9 TL Questions

**Q: What is the N+1 problem?**
A: One query fetches N parents, then each parent's lazy association fires its own query — so N+1 statements where one or two would do. A hundred orders becomes a hundred and one queries.

**Q: Why is it so dangerous?**
A: Because it doesn't show up in development. With ten test rows it's eleven fast queries and nobody notices; with ten thousand production rows it's ten thousand queries and the endpoint times out.

**Q: How do we fix it?**
A: Depends on the case. For a specific query we use `@EntityGraph` or a join fetch. For read-only list screens we use a DTO projection so only the needed columns are selected. And globally we set `default_batch_fetch_size`, which batches the lazy loads and turns a hundred and one queries into about six with no code change.

**Q: Why not just use EAGER?**
A: Because that makes it worse. EAGER still issues the extra queries, and now every query that loads an order pays the cost even if it never touches the items. Lazy plus a targeted fetch keeps the cost where it's actually needed.

**Q: How do we detect it?**
A: SQL logging in development, and ideally an integration test that counts queries per endpoint and fails the build past a threshold — so it's caught in CI rather than production.

**Q: What's the catch with join fetch and pagination?**
A: Hibernate can't apply SQL `LIMIT` when the collection join multiplies rows, so it loads the whole result set and paginates in memory. For paginated endpoints we use `@BatchSize` or an entity graph instead.

**Q: What is `@EntityGraph` and why prefer it over join fetch?**
A: It's a declarative way to say which associations to fetch for a given query. It works on derived query methods and on inherited methods like `findById`, where we can't edit a JPQL string, and it handles nested paths cleanly.

## 40.10 Interview Questions

**Beginner — What causes N+1?**
Lazy associations being initialised one by one after a query returns multiple parent entities.

**Intermediate — Why is `DISTINCT` needed with a collection join fetch?**
The join produces one row per child, so the parent appears multiple times in the result list. `DISTINCT` de-duplicates at the object level. Hibernate 6 does this automatically for entity queries.

**Intermediate — How does `@BatchSize` help?**
Instead of one query per parent, Hibernate collects up to N parent IDs and loads their collections with a single `IN` query, reducing N+1 to roughly N/batchSize + 1 queries.

**Advanced — `@EntityGraph` FETCH vs LOAD?**
`FETCH` makes listed attributes eager and treats all others as lazy regardless of their mapping. `LOAD` makes listed attributes eager while others keep their mapped fetch type. `FETCH` gives more predictable control.

**Advanced — Why can't `JOIN FETCH` be paginated safely?**
SQL `LIMIT` applies to result rows, but a collection join makes rows a multiple of entities, so limiting rows would truncate a parent's children arbitrarily. Hibernate therefore fetches everything and paginates in memory, which defeats the purpose and risks memory exhaustion.

## 40.11 Quick Revision

1. N+1 = 1 parent query + N child queries.
2. **Works fine in dev, fails in production** — always test with realistic data.
3. Fixes: `JOIN FETCH`, `@EntityGraph`, `@BatchSize`, DTO projection.
4. **`@EntityGraph` works on derived and inherited methods** — prefer it.
5. `@BatchSize` / `default_batch_fetch_size` is the **global safety net** and works with pagination.
6. DTO projections are fastest for read-only lists.
7. **EAGER is not a fix** — it makes the cost permanent and universal.
8. `DISTINCT` needed with collection fetch joins (Hibernate 5).
9. Join fetch + pagination → in-memory pagination.
10. Detect with SQL logging; enforce with a query-count test.

## 40.12 TL Explanation (speak this)

> "N+1 is where one query loads a hundred orders and then a hundred more queries load each order's items. The reason it bites is that it looks fine in development with ten rows and only becomes a timeout in production. We fix it per-query with `@EntityGraph` or a join fetch, and for read-only list screens we use a DTO projection so the SQL selects only the columns we actually display. We also set `default_batch_fetch_size` globally, which batches lazy loads into `IN` queries and turns a hundred and one queries into about six with no code change. What we deliberately don't do is switch to EAGER — that just makes every query pay the cost permanently."

---

[← 39. Lazy vs Eager Loading](39-lazy-vs-eager-loading.md) | [Contents](../README.md) | [41. Transactions and @Transactional →](41-transactions-and-transactional.md)
