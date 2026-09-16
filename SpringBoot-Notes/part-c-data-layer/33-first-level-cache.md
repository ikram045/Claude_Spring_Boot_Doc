[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 33. First Level Cache

## 33.1 What is it?

**Simple words:**
The first-level cache **is** the persistence context, viewed from the caching angle. Within one transaction, if you ask for the same entity twice, the second request comes **from memory** — no second SELECT.

**Technical wording:**
The first-level cache is a mandatory, transaction-scoped cache maintained by the persistence context, keyed by entity type and identifier, guaranteeing object identity and eliminating redundant primary-key lookups within its scope.

## 33.2 Demonstration

```java
@Transactional
public void firstLevelCacheDemo() {

    Order o1 = orderRepository.findById(1L).orElseThrow();
    // SQL:  SELECT * FROM orders WHERE id = 1     ← database hit

    Order o2 = orderRepository.findById(1L).orElseThrow();
    // NO SQL — served from the first-level cache

    System.out.println(o1 == o2);          // true — the SAME object instance
}
```

**Two guarantees here:**
1. **No redundant SELECT** for the same ID in the same transaction.
2. **Object identity** — both references point to the same instance. Change one and the other "sees" it, because they *are* the same object.

## 33.3 Key characteristics

| Property | First Level Cache |
|---|---|
| Scope | **Transaction / persistence context** |
| Enabled by default | **Yes — cannot be disabled** |
| Shared between transactions | **No** |
| Shared between users/threads | **No** |
| Keyed by | Entity class + primary key |
| Cleared when | Transaction ends, or `clear()`/`detach()` |

## 33.4 Important limitation — queries bypass the cache

```java
@Transactional
public void limitation() {

    Order o1 = orderRepository.findById(1L).orElseThrow();   // SELECT — cached

    // A JPQL query ALWAYS goes to the database
    List<Order> orders = orderRepository.findByStatus(OrderStatus.PENDING);
    // SELECT * FROM orders WHERE status = 'PENDING'  ← executes even if id=1 matches

    // BUT: if the result contains id=1, Hibernate returns the CACHED instance
    // rather than a new object — identity is preserved, the query is not avoided.
}
```

**Two-part rule to remember:**
- The first-level cache only avoids a database round trip for **primary-key lookups** (`find`/`getReference`).
- For query results, the **query always runs**, but returned rows already present in the context are **mapped back to the existing instances**, preserving identity.

A subtle consequence: if a row changed in the database since it was cached, a query returning that row gives you the **cached (stale) version**, not the database version — because identity wins. Use `refresh()` if you genuinely need the current database state.

## 33.5 First vs Second Level Cache

| | First Level (L1) | Second Level (L2) |
|---|---|---|
| Scope | Persistence context / transaction | **`SessionFactory` / application-wide** |
| Default | **Always on** | **Off** — must be configured |
| Shared across transactions | No | **Yes** |
| Shared across users | No | **Yes** |
| Provider | Hibernate built-in | Ehcache, Hazelcast, Infinispan, Redis |
| Cleared on | Transaction end | TTL, eviction policy, explicit invalidation |
| Risk | None | **Stale data** in a clustered deployment |

```java
// Enabling L2 (only when justified)
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Country { }
```
```properties
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
```

**When L2 is appropriate:** read-mostly reference data — countries, currencies, product categories, configuration. **When it is dangerous:** frequently updated transactional data, especially across multiple application instances, where one node's update leaves other nodes serving stale data unless you run a distributed cache.

**The honest recommendation to give a TL:** *"L2 adds an invalidation problem. For reference data it's a clear win; for transactional data I'd reach for a query-level cache or an explicit Redis cache with a deliberate TTL, so the staleness window is a conscious decision rather than a side effect."*

## 33.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Expecting L1 to span transactions | It doesn't — new transaction, new empty context |
| Expecting queries to be served from L1 | Only PK lookups skip the database |
| Loading huge result sets in one transaction | L1 grows unbounded → `OutOfMemoryError` |
| Enabling L2 on volatile data in a cluster | Stale reads across nodes |
| Forgetting `clear()` in a long batch loop | Memory grows with every entity processed |

**The batch-processing pattern that matters:**

```java
@Transactional
public void processLargeBatch(List<Long> orderIds) {
    int count = 0;
    for (Long id : orderIds) {
        Order order = orderRepository.findById(id).orElseThrow();
        order.setStatus(OrderStatus.PROCESSED);

        if (++count % 50 == 0) {
            entityManager.flush();    // push pending SQL to the DB
            entityManager.clear();    // EMPTY the persistence context → free memory
        }
    }
}
```
Without `flush()` + `clear()`, processing 100,000 orders keeps 100,000 entities **plus 100,000 snapshots** in memory and the JVM runs out of heap. **This is the standard answer to "how do you handle bulk updates with JPA?"**

## 33.7 TL Questions

**Q: What is the first-level cache?**
A: The persistence context acting as a cache. Within a transaction, repeated lookups of the same entity by ID come from memory instead of the database, and both references point to the same object instance.

**Q: Can we disable it?**
A: No — it's mandatory and fundamental to how JPA works. Dirty checking and the identity guarantee both depend on it.

**Q: Does it help with queries?**
A: Only for primary-key lookups. A JPQL query always hits the database, though any returned rows already in the context are mapped back to the existing instances so identity is preserved.

**Q: What's the risk in batch processing?**
A: The persistence context grows with every entity loaded, each with a change-detection snapshot, so a large batch exhausts heap. We flush and clear every fifty records or so to keep it bounded.

**Q: Should we enable the second-level cache?**
A: Only for read-mostly reference data like countries and currencies. For transactional data across multiple instances it introduces stale reads, so we'd prefer an explicit Redis cache with a deliberate TTL where the staleness window is a conscious choice.

## 33.8 Interview Questions

**Beginner — What is the scope of the first-level cache?**
The persistence context, which in Spring equals the transaction.

**Intermediate — Is it shared between two users?**
No. Each transaction has its own persistence context, so there's no sharing between threads or users.

**Intermediate — How do you clear it?**
`entityManager.clear()` detaches everything; `detach(entity)` removes one. It also clears automatically when the transaction ends.

**Advanced — Why does a query return a cached instance rather than fresh data?**
Because the persistence context guarantees one instance per row. When a query result row matches an entity already in the context, Hibernate returns the existing instance rather than overwriting it, to avoid discarding pending in-memory changes. Use `refresh()` to force a reload.

**Advanced — What is the query cache and how does it relate to L2?**
An optional cache of **query result identifiers**, enabled separately with `hibernate.cache.use_query_cache`. It stores IDs, not entities, so it requires L2 to be enabled to resolve those IDs; without L2 it causes N additional lookups and makes performance worse.

## 33.9 Quick Revision

1. L1 cache = the persistence context; scoped to the transaction.
2. **Always on, cannot be disabled.**
3. Avoids repeat SELECTs for **PK lookups only**.
4. Guarantees one object instance per row (`==` holds).
5. Queries always hit the database; results are mapped to existing instances.
6. Not shared across transactions, threads or users.
7. **Batch processing: `flush()` + `clear()` every ~50 records.**
8. L2 is application-wide, off by default, good for reference data.
9. L2 in a cluster risks stale reads — use deliberately.

## 33.10 TL Explanation (speak this)

> "The first-level cache is just the persistence context seen as a cache — within a transaction, looking up the same entity by ID twice hits the database once, and both references are literally the same object. It's always on and can't be disabled, because dirty checking depends on it. The limitation worth knowing is that it only short-circuits primary-key lookups; a JPQL query always runs, though its results map back to instances already in the context. The practical rule is in batch jobs: we flush and clear every fifty records, otherwise the context keeps every entity plus its snapshot and we run out of heap."

---

[← 32. EntityManager and Persistence Context](32-entitymanager-and-persistence-context.md) | [Contents](../README.md) | [34. Entity Lifecycle States →](34-entity-lifecycle-states.md)
