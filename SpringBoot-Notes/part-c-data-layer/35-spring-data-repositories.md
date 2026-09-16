[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 35. Spring Data Repositories

## 35.1 What is it?

**Simple words:**
You write an **interface** with no implementation, and Spring Data **creates the implementation for you at runtime**. You never write a repository class.

**Technical wording:**
Spring Data JPA generates proxy implementations of repository interfaces at runtime, deriving queries from method names and delegating to `SimpleJpaRepository`, which wraps the `EntityManager`.

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByCustomerId(String customerId);
}
// No implementation class. Spring writes it.
```

## 35.2 The repository hierarchy

```
        Repository<T, ID>                    ← marker interface, no methods
                │
        CrudRepository<T, ID>                ← save, findById, findAll, delete, count
                │
   PagingAndSortingRepository<T, ID>         ← + findAll(Pageable), findAll(Sort)
                │
     ┌──────────┴──────────┐
     │                      │
ListCrudRepository     JpaRepository<T, ID>  ← + flush, saveAndFlush, deleteInBatch,
  (Boot 3+)                                     getReferenceById, JPA-specific methods
```

| Interface | Adds |
|---|---|
| `Repository` | Nothing — just marks the interface for scanning |
| `CrudRepository` | `save`, `saveAll`, `findById`, `existsById`, `findAll`, `count`, `deleteById`, `delete` |
| `PagingAndSortingRepository` | `findAll(Pageable)`, `findAll(Sort)` |
| **`JpaRepository`** | `flush()`, `saveAndFlush()`, `deleteAllInBatch()`, `getReferenceById()`, and returns `List` instead of `Iterable` |

### `JpaRepository` vs `CrudRepository` — common interview question

| Aspect | `CrudRepository` | `JpaRepository` |
|---|---|---|
| Technology | Any Spring Data store (Mongo, Redis, JPA…) | **JPA-specific** |
| `findAll()` returns | `Iterable<T>` | **`List<T>`** (more convenient) |
| Paging/sorting | No | Yes (inherits it) |
| Batch delete | No | `deleteAllInBatch()` |
| Flush control | No | `flush()`, `saveAndFlush()` |
| Portability | Higher | Lower (tied to JPA) |
| **Use when** | Store-agnostic code | **Standard choice for JPA projects** |

**Practical answer:** use `JpaRepository` unless you have a specific reason to stay store-agnostic. The convenience of `List` returns and batch operations outweighs a portability that almost never gets exercised.

**Note on `deleteAllInBatch()`:** it issues a single `DELETE FROM orders` rather than loading entities and deleting one by one — far faster, but it **skips cascade and lifecycle callbacks**, so use it knowingly.

## 35.3 How Spring Data generates the implementation (internal)

```
1. @EnableJpaRepositories (auto-enabled by Boot) scans for Repository sub-interfaces
              │
              ▼
2. For each, JpaRepositoryFactoryBean creates a PROXY
              │
              ▼
3. For every method, decide how to handle it:
              │
    ┌─────────┼─────────────────┬───────────────────────┐
    ▼         ▼                 ▼                       ▼
 Inherited  Has @Query?    Name parseable?        Custom impl
 CRUD       use that       DERIVE the query       (XxxRepositoryImpl)
    │       JPQL/SQL             │                      │
    ▼                            ▼                      ▼
 delegate to             PartTree parses:         delegate to
 SimpleJpaRepository     findBy|CustomerId|And|   your class
                         Status|OrderBy...
              │
              ▼
4. If a method name cannot be parsed → STARTUP FAILURE
   "No property 'custmerId' found for type Order"
```

**Why startup failure is a good thing:** a typo in a derived method name breaks the deployment rather than a live request. Fail fast, again.

## 35.4 Derived query methods

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // ---- Simple ----
    List<Order> findByCustomerId(String customerId);
    Optional<Order> findByOrderNumber(String orderNumber);

    // ---- Multiple conditions ----
    List<Order> findByCustomerIdAndStatus(String customerId, OrderStatus status);
    List<Order> findByStatusOrStatus(OrderStatus s1, OrderStatus s2);

    // ---- Comparison ----
    List<Order> findByTotalAmountGreaterThan(BigDecimal amount);
    List<Order> findByTotalAmountBetween(BigDecimal min, BigDecimal max);
    List<Order> findByCreatedAtAfter(LocalDateTime date);

    // ---- String matching ----
    List<Order> findByCustomerIdContaining(String part);
    List<Order> findByCustomerIdStartingWith(String prefix);
    List<Order> findByCustomerIdIgnoreCase(String customerId);

    // ---- Null checks ----
    List<Order> findByCancelledAtIsNull();
    List<Order> findByCancelledAtIsNotNull();

    // ---- Collections ----
    List<Order> findByStatusIn(List<OrderStatus> statuses);

    // ---- Sorting and limiting ----
    List<Order> findByStatusOrderByCreatedAtDesc(OrderStatus status);
    List<Order> findTop10ByStatusOrderByTotalAmountDesc(OrderStatus status);
    Optional<Order> findFirstByCustomerIdOrderByCreatedAtDesc(String customerId);

    // ---- Counting / existence / deletion ----
    long countByStatus(OrderStatus status);
    boolean existsByOrderNumber(String orderNumber);
    void deleteByStatus(OrderStatus status);

    // ---- Nested property traversal (note the underscore) ----
    List<Order> findByCustomer_Address_City(String city);

    // ---- Paging ----
    Page<Order> findByStatus(OrderStatus status, Pageable pageable);
    Slice<Order> findByCustomerId(String customerId, Pageable pageable);
}
```

**Keyword reference:**

| Keyword | SQL equivalent |
|---|---|
| `And`, `Or` | `AND`, `OR` |
| `Is`, `Equals` | `=` |
| `Between` | `BETWEEN` |
| `LessThan`, `GreaterThan`, `...Equal` | `<`, `>`, `<=`, `>=` |
| `After`, `Before` | `>`, `<` (dates) |
| `IsNull`, `IsNotNull` | `IS NULL`, `IS NOT NULL` |
| `Like`, `NotLike` | `LIKE` |
| `StartingWith`, `EndingWith`, `Containing` | `LIKE 'x%'`, `'%x'`, `'%x%'` |
| `In`, `NotIn` | `IN`, `NOT IN` |
| `True`, `False` | `= true`, `= false` |
| `IgnoreCase` | `UPPER(x) = UPPER(?)` |
| `OrderBy...Asc/Desc` | `ORDER BY` |
| `Top`, `First` | `LIMIT` |
| `Distinct` | `DISTINCT` |

**When to stop using derived queries:** once a method name exceeds ~4 conditions, it becomes unreadable:
```java
// ❌ Technically valid, practically terrible
List<Order> findByCustomerIdAndStatusAndTotalAmountGreaterThanAndCreatedAtBetweenOrderByCreatedAtDesc(...);
```
Switch to `@Query` or a Specification. **Readability is the deciding factor** — a good point to make in a code review discussion.

## 35.5 Custom repository implementation

For logic that can't be expressed declaratively:

```java
// 1. The custom interface
public interface OrderRepositoryCustom {
    List<Order> searchWithDynamicFilters(OrderSearchCriteria criteria);
}

// 2. The implementation — name MUST end with "Impl"
@RequiredArgsConstructor
public class OrderRepositoryCustomImpl implements OrderRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<Order> searchWithDynamicFilters(OrderSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Order> query = cb.createQuery(Order.class);
        Root<Order> root = query.from(Order.class);

        List<Predicate> predicates = new ArrayList<>();
        if (criteria.getStatus() != null) {
            predicates.add(cb.equal(root.get("status"), criteria.getStatus()));
        }
        if (criteria.getMinAmount() != null) {
            predicates.add(cb.greaterThanOrEqualTo(root.get("totalAmount"),
                                                   criteria.getMinAmount()));
        }
        query.where(predicates.toArray(new Predicate[0]))
             .orderBy(cb.desc(root.get("createdAt")));

        return entityManager.createQuery(query).getResultList();
    }
}

// 3. Extend BOTH interfaces
public interface OrderRepository extends JpaRepository<Order, Long>, OrderRepositoryCustom { }
```
**The `Impl` suffix is mandatory** — Spring Data looks for `<CustomInterfaceName>Impl`. Getting the name wrong gives a confusing "no bean found" style failure at startup.

## 35.6 TL Questions

**Q: How does Spring Data create the implementation?**
A: At startup it scans for repository interfaces and creates a proxy for each. Inherited CRUD methods delegate to `SimpleJpaRepository`; methods with `@Query` use that query; and other method names are parsed into queries. If a name can't be parsed, startup fails.

**Q: `JpaRepository` or `CrudRepository`?**
A: `JpaRepository` for our JPA projects — it returns `List` instead of `Iterable`, includes paging and sorting, and adds `flush` and batch deletes. `CrudRepository` only matters if we want the code to be portable across Spring Data stores, which we don't in practice.

**Q: What happens if we misspell a property in a method name?**
A: The application fails to start with a message naming the unknown property. That's deliberate — it catches the typo at deploy time instead of on a live request.

**Q: When do we stop using derived queries?**
A: Around four conditions. Beyond that the method name becomes unreadable, so we switch to `@Query` with JPQL, or a Specification for genuinely dynamic filters.

**Q: How do we write a query that can't be expressed by a method name?**
A: Either `@Query` with JPQL or native SQL, or a custom repository fragment — an interface plus an `Impl` class using the `EntityManager` and the Criteria API, which our main repository interface also extends.

## 35.7 Interview Questions

**Beginner — Do we write repository implementations?**
No — Spring Data generates them at runtime from the interface.

**Intermediate — What is `SimpleJpaRepository`?**
The default implementation backing all standard CRUD methods. It wraps the `EntityManager` and is annotated `@Transactional(readOnly = true)` at class level, with write methods overriding that.

**Intermediate — What does `deleteAllInBatch()` do differently?**
It issues one bulk `DELETE` statement instead of loading each entity and deleting it individually. Much faster, but it bypasses cascading and lifecycle callbacks and doesn't update the persistence context.

**Advanced — How are repository methods transactional by default?**
`SimpleJpaRepository` is annotated `@Transactional(readOnly = true)`, and mutating methods like `save` and `delete` are annotated `@Transactional`. So a single repository call always runs in a transaction — which is exactly why an entity returned from a repository call outside a service transaction comes back detached.

**Advanced — What is the fragment/`Impl` mechanism?**
Spring Data composes a repository from fragments. For each extended interface it looks for a class named `<Interface>Impl` and wires it into the proxy, letting you mix generated methods with hand-written ones in a single repository.

## 35.8 Quick Revision

1. Declare an interface; Spring Data generates the implementation.
2. Hierarchy: `Repository` → `CrudRepository` → `PagingAndSortingRepository` → `JpaRepository`.
3. Use **`JpaRepository`** in JPA projects.
4. Query sources: inherited CRUD, derived names, `@Query`, custom `Impl`.
5. Unparseable method names **fail at startup**.
6. Keep derived names to ~4 conditions; beyond that use `@Query`.
7. Custom fragment class **must** be named `<Interface>Impl`.
8. `deleteAllInBatch()` is fast but skips cascades and callbacks.
9. Repository methods are transactional by default (`readOnly` for reads).

## 35.9 TL Explanation (speak this)

> "We only declare repository interfaces — Spring Data generates the implementations at startup. Standard CRUD comes from `JpaRepository`, and anything else is either a derived query from the method name or an explicit `@Query`. A nice property is that a typo in a derived method name fails the startup rather than a live request. We keep derived names short, about four conditions, and move to `@Query` or a Specification beyond that, because the generated names become unreadable. For genuinely dynamic filtering we add a custom fragment with the Criteria API."

---

[← 34. Entity Lifecycle States](34-entity-lifecycle-states.md) | [Contents](../README.md) | [36. Query Methods, JPQL, @Query and Pagination →](36-query-methods-jpql-query-and-pagination.md)
