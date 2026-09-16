[← Back to Contents](../README.md) · Part F — Master Flows and Final Revision

---

# 49. Complete JPA/Hibernate Flow

```
 SERVICE METHOD (@Transactional)
        │
 ═══════▼══════════════════════════════════════════════════
 1. TRANSACTION BEGINS
      PlatformTransactionManager → HikariCP connection
      → setAutoCommit(false)
      → EntityManager + PERSISTENCE CONTEXT created
      → bound to the thread (TransactionSynchronizationManager)
 ═══════┬══════════════════════════════════════════════════
        │
 ┌──────▼───────────────────────────────────────────────────┐
 │ 2. READ:  orderRepository.findById(1L)                    │
 │      a. check the PERSISTENCE CONTEXT (L1 cache)          │
 │           present → return the SAME instance, NO SQL      │
 │      b. absent → SELECT * FROM orders WHERE id = 1        │
 │      c. build the entity, state = MANAGED                 │
 │      d. ★ take a SNAPSHOT of loaded values (dirty check) ★│
 │      e. lazy associations → proxies, not loaded yet       │
 └──────┬───────────────────────────────────────────────────┘
        │
 ┌──────▼───────────────────────────────────────────────────┐
 │ 3. MODIFY:  order.setStatus(SHIPPED)                      │
 │      in-memory only. NO SQL. NO save() needed.            │
 └──────┬───────────────────────────────────────────────────┘
        │
 ┌──────▼───────────────────────────────────────────────────┐
 │ 4. CREATE:  repository.save(newOrder)                     │
 │      id == null → persist() → MANAGED, INSERT QUEUED      │
 │      id != null → merge() → returns a DIFFERENT instance  │
 │      cascade propagates to children                       │
 └──────┬───────────────────────────────────────────────────┘
        │
 ┌──────▼───────────────────────────────────────────────────┐
 │ 5. LAZY ACCESS:  order.getItems().size()                  │
 │      proxy initialises → SELECT ... WHERE order_id = ?    │
 │      ★ in a loop over N parents → N+1 PROBLEM ★           │
 └──────┬───────────────────────────────────────────────────┘
        │
 ┌──────▼───────────────────────────────────────────────────┐
 │ 6. FLUSH  (before commit, or before an affected query)    │
 │      compare every managed entity to its snapshot         │
 │      generate SQL in fixed order:                         │
 │        INSERT → UPDATE → collection deletes →             │
 │        collection inserts → DELETE                        │
 │      @Version → optimistic lock check in the WHERE clause │
 └──────┬───────────────────────────────────────────────────┘
        │
 ═══════▼══════════════════════════════════════════════════
 7. COMMIT                       │  7'. ROLLBACK (RuntimeException)
      connection.commit()        │       connection.rollback()
      connection → pool          │       connection → pool
      context closes             │       context closes
      entities → DETACHED        │       entities → DETACHED
 ═══════════════════════════════════════════════════════════
        │
        ▼
  ⚠ AFTER THIS POINT:
     - entities are DETACHED → setters do nothing
     - touching a lazy field → LazyInitializationException
     → which is exactly why we map to DTOs INSIDE the transaction
```

**The five facts this diagram encodes:**
1. L1 cache is checked before any SELECT by ID.
2. Dirty checking replaces explicit `save()` calls.
3. Lazy access inside a loop is where N+1 is born.
4. Flush order is fixed — INSERTs before DELETEs.
5. After commit everything is detached — map to DTOs before that.

---

[← 48. Complete REST API Request/Response Flow](48-complete-rest-api-request-response-flow.md) | [Contents](../README.md) | [50. Complete Spring Security / JWT Flow →](50-complete-spring-security-jwt-flow.md)
