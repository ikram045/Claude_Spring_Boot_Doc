[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 36. Query Methods, JPQL, @Query and Pagination

## 36.1 `@Query` — JPQL

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // JPQL works on ENTITY names and FIELD names, not tables and columns
    @Query("SELECT o FROM Order o WHERE o.status = :status AND o.totalAmount > :amount")
    List<Order> findHighValueOrders(@Param("status") OrderStatus status,
                                    @Param("amount") BigDecimal amount);

    // Join + fetch join (solves N+1 — see Section 40)
    @Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items WHERE o.status = :status")
    List<Order> findWithItems(@Param("status") OrderStatus status);

    // DTO projection via constructor expression — selects ONLY these columns
    @Query("""
           SELECT new com.company.orderservice.dto.OrderSummaryDto(
                  o.id, o.orderNumber, o.totalAmount, o.status)
           FROM Order o WHERE o.customerId = :customerId
           """)
    List<OrderSummaryDto> findSummaries(@Param("customerId") String customerId);

    // Aggregate
    @Query("SELECT SUM(o.totalAmount) FROM Order o WHERE o.customerId = :customerId")
    BigDecimal getTotalSpend(@Param("customerId") String customerId);

    // Native SQL — when you need vendor features
    @Query(value = "SELECT * FROM orders WHERE MATCH(notes) AGAINST(:text)",
           nativeQuery = true)
    List<Order> fullTextSearch(@Param("text") String text);

    // Modifying query
    @Modifying(clearAutomatically = true, flushAutomatically = true)
    @Transactional
    @Query("UPDATE Order o SET o.status = :status WHERE o.id IN :ids")
    int bulkUpdateStatus(@Param("status") OrderStatus status, @Param("ids") List<Long> ids);
}
```

### JPQL vs SQL — the distinction that matters

| | JPQL | Native SQL |
|---|---|---|
| Operates on | **Entities and fields** | Tables and columns |
| `SELECT o FROM Order o` | `Order` = the **entity class** | `orders` = the table |
| Database portable | **Yes** | No |
| Vendor features | No | **Yes** |
| Validated at startup | **Yes** | No |
| Returns | Managed entities | Entities (with `nativeQuery`) or raw rows |

**JPQL is validated when the application starts** — a syntax error or wrong field name fails the deployment. Native SQL isn't checked until it runs. That's a meaningful argument for preferring JPQL wherever it suffices.

### `@Modifying` — three things that must be right

```java
@Modifying(clearAutomatically = true, flushAutomatically = true)
@Transactional
@Query("UPDATE Order o SET o.status = :status WHERE o.id IN :ids")
int bulkUpdateStatus(...);
```

1. **`@Modifying`** is required for UPDATE/DELETE — without it you get `QueryExecutionRequestException`.
2. **`@Transactional`** is required (better placed on the calling service method).
3. **`clearAutomatically = true`** matters: a bulk UPDATE goes **straight to the database and bypasses the persistence context**, so entities already loaded hold stale values. Clearing the context after forces a reload. `flushAutomatically = true` pushes pending changes *before* the bulk statement so they aren't lost.

**This is a genuine production bug source** — a bulk update followed by reading a previously-loaded entity returns the old value.

## 36.2 Pagination

**Why it is non-negotiable:** `findAll()` on a table with 5 million rows loads 5 million objects into heap and kills the service. Every collection endpoint must be paginated.

```java
// Repository
Page<Order> findByStatus(OrderStatus status, Pageable pageable);

// Service
@Transactional(readOnly = true)
public Page<OrderResponseDto> getOrders(OrderStatus status, Pageable pageable) {
    return orderRepository.findByStatus(status, pageable)
                          .map(orderMapper::toDto);      // Page.map preserves metadata
}

// Controller
@GetMapping
public ResponseEntity<Page<OrderResponseDto>> getOrders(
        @RequestParam(required = false) OrderStatus status,
        @PageableDefault(size = 20, sort = "createdAt",
                         direction = Sort.Direction.DESC) Pageable pageable) {
    return ResponseEntity.ok(orderService.getOrders(status, pageable));
}
// Client calls: GET /api/v1/orders?page=0&size=20&sort=createdAt,desc
```

### `Page` vs `Slice` vs `List`

| | `List` | `Slice` | `Page` |
|---|---|---|---|
| Content | All results | One page | One page |
| Knows if there's a next page | No | **Yes** | Yes |
| Total element count | No | **No** | **Yes** |
| Extra COUNT query | No | **No** | **Yes** |
| Best for | Small fixed sets | **Infinite scroll / "load more"** | **Numbered pagination** |

**Performance point worth raising:** `Page` runs a second `SELECT COUNT(*)` to compute the total. On a large table with a complex WHERE clause that count can be **slower than the data query itself**. If the UI only needs "next", use `Slice` and avoid the count entirely.

### The deep-pagination problem (Additional clarification)

`LIMIT 20 OFFSET 1000000` forces the database to scan and discard a million rows. Page 50,000 is dramatically slower than page 1. For deep pagination use **keyset (cursor) pagination**:

```java
@Query("SELECT o FROM Order o WHERE o.createdAt < :cursor ORDER BY o.createdAt DESC")
List<Order> findNextPage(@Param("cursor") LocalDateTime cursor, Pageable pageable);
// Uses the index directly — constant time regardless of depth
```

### Sorting

```java
Sort sort = Sort.by("createdAt").descending().and(Sort.by("totalAmount"));
Pageable pageable = PageRequest.of(0, 20, sort);
```
**Security caution:** never pass a raw client string into a native-SQL `ORDER BY` — it's an injection vector. With `Sort` on JPQL, Spring Data validates the property against the entity, so an invalid field throws rather than injecting.

## 36.3 Specifications — type-safe dynamic queries

```java
public class OrderSpecifications {

    public static Specification<Order> hasStatus(OrderStatus status) {
        return (root, query, cb) ->
                status == null ? null : cb.equal(root.get("status"), status);
    }

    public static Specification<Order> amountGreaterThan(BigDecimal amount) {
        return (root, query, cb) ->
                amount == null ? null : cb.greaterThan(root.get("totalAmount"), amount);
    }
}

// Repository must extend JpaSpecificationExecutor
public interface OrderRepository extends JpaRepository<Order, Long>,
                                          JpaSpecificationExecutor<Order> { }

// Usage — null specs are ignored, so filters compose cleanly
Specification<Order> spec = Specification
        .where(OrderSpecifications.hasStatus(criteria.getStatus()))
        .and(OrderSpecifications.amountGreaterThan(criteria.getMinAmount()));

Page<Order> results = orderRepository.findAll(spec, pageable);
```
**Why this beats string concatenation:** it's type-safe, composable, injection-proof, and each filter is independently testable. It is the right tool for a search screen with many optional filters.

## 36.4 Common Mistakes

| Mistake | Consequence |
|---|---|
| `findAll()` on a large table | Memory exhaustion |
| Missing `@Modifying` on UPDATE/DELETE | `QueryExecutionRequestException` |
| Missing `clearAutomatically` after a bulk update | Stale entities in the persistence context |
| `Page` when the count query is expensive | Slow endpoint — use `Slice` |
| Deep `OFFSET` pagination | Degrades badly at depth — use keyset |
| Native SQL where JPQL would do | Loses startup validation and portability |
| Client-supplied `ORDER BY` in native SQL | **SQL injection** |
| `JOIN FETCH` combined with pagination | Hibernate warns and paginates **in memory** — see Section 40 |

## 36.5 TL Questions

**Q: When do we use `@Query` instead of a derived method?**
A: When the method name would become unreadable — roughly beyond four conditions — or when we need a join fetch, an aggregate, or a DTO projection. JPQL keeps it explicit and readable.

**Q: JPQL or native SQL?**
A: JPQL by default, because it's validated at startup and portable across databases. Native SQL only when we need vendor-specific features like full-text search.

**Q: Why do bulk updates need `clearAutomatically`?**
A: A bulk UPDATE bypasses the persistence context and goes straight to the database, so any entity already loaded holds stale values. Clearing the context forces a reload on next access. Without it we get a subtle bug where the object disagrees with the database.

**Q: `Page` or `Slice`?**
A: `Page` when the UI shows numbered pages and needs the total. `Slice` for infinite scroll, because it skips the count query — and that count can be slower than the data query on a large filtered table.

**Q: How do we handle a search screen with ten optional filters?**
A: Specifications. Each filter is a small composable `Specification` that returns null when the value is absent, so we build the query dynamically without string concatenation. It's type-safe and injection-proof.

**Q: What's wrong with `page=50000`?**
A: `OFFSET` makes the database scan and discard all preceding rows, so deep pages get progressively slower. For deep pagination we'd use keyset pagination with a cursor on an indexed column, which stays constant-time.

## 36.6 Interview Questions

**Beginner — What is JPQL?**
JPA's query language, operating on entity names and field names rather than tables and columns, making it database-portable.

**Intermediate — What does `@Modifying` do?**
Tells Spring Data the query modifies data rather than selecting, so it calls `executeUpdate()` instead of `getResultList()`. Required for UPDATE and DELETE queries.

**Intermediate — Why does `Page` need two queries?**
One for the page content and a second `SELECT COUNT(*)` to compute total elements and total pages.

**Advanced — What is a constructor expression and why use it?**
`SELECT new com.company.dto.OrderSummaryDto(o.id, o.status) FROM Order o` — it instantiates DTOs directly in the query. The generated SQL selects only those columns, and no entities enter the persistence context, so there's no dirty-check overhead. It's the efficient choice for read-only list endpoints.

**Advanced — What is `JpaSpecificationExecutor`?**
An interface adding `findAll(Specification)`, `count(Specification)` and paged variants. A `Specification` wraps a Criteria API `Predicate`, so filters are composable with `and`/`or` and built type-safely at runtime.

## 36.7 Quick Revision

1. `@Query` for anything beyond a simple derived method.
2. JPQL uses **entity and field names**; it is validated at startup and portable.
3. Native SQL only for vendor-specific features.
4. `@Modifying` + `@Transactional` for UPDATE/DELETE; add `clearAutomatically`.
5. **Bulk updates bypass the persistence context** — entities go stale.
6. Always paginate collection endpoints.
7. `Page` = with count; `Slice` = no count, cheaper; use `Slice` for infinite scroll.
8. Deep `OFFSET` is slow — keyset pagination for depth.
9. Specifications = type-safe, composable dynamic filters.
10. Constructor expressions select only the columns you need.
11. Never interpolate client input into a native `ORDER BY`.

## 36.8 TL Explanation (speak this)

> "Simple lookups use derived method names, and anything more complex uses `@Query` with JPQL — which has the advantage of being validated at application startup, so a bad field name fails the deployment rather than a live request. Every collection endpoint is paginated; we use `Page` where the UI needs a total and `Slice` for infinite scroll, because `Page` runs an extra count query that can be slower than the data query itself. For search screens with many optional filters we use Specifications, which compose type-safely instead of concatenating SQL. The one thing we're careful about is bulk updates — they bypass the persistence context, so we clear it afterwards or previously-loaded entities go stale."

---

[← 35. Spring Data Repositories](35-spring-data-repositories.md) | [Contents](../README.md) | [37. Entity Relationships →](37-entity-relationships.md)
