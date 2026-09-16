[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 32. EntityManager and Persistence Context

## 32.1 What is the Persistence Context?

**Simple words:**
The persistence context is a **workspace in memory where Hibernate keeps the entities it is currently managing**, for the duration of a transaction.

Think of it as Hibernate's **notepad**: it writes down every entity you load or save, remembers what each looked like originally, and at the end compares them to decide what SQL to run.

This one concept explains **first-level cache, dirty checking, lazy loading and entity states** — all four are consequences of the persistence context.

**Technical wording:**
The persistence context is a set of managed entity instances with unique persistent identity, maintained by an `EntityManager`. It acts as a first-level cache, tracks state changes for automatic dirty checking, and guarantees that a given database row maps to exactly one object instance within its scope.

## 32.2 EntityManager

**Simple words:** the `EntityManager` is your **handle to the persistence context** — the API you use to find, save and delete entities.

```java
@Repository
public class OrderRepositoryImpl {

    @PersistenceContext          // NOT @Autowired — see explanation below
    private EntityManager entityManager;
}
```

| Method | What it does |
|---|---|
| `persist(entity)` | Make a new entity managed → INSERT at flush |
| `merge(entity)` | Copy a detached entity's state into a managed one → UPDATE |
| `find(Class, id)` | Look up by PK — **checks the persistence context first** |
| `getReference(Class, id)` | Return a lazy **proxy** without hitting the DB |
| `remove(entity)` | Mark for deletion → DELETE at flush |
| `flush()` | Force pending SQL to the database **now** |
| `clear()` | Detach **all** entities from the context |
| `detach(entity)` | Detach one entity |
| `refresh(entity)` | Reload from the database, discarding in-memory changes |
| `createQuery(jpql)` | Create a JPQL query |

### Why `@PersistenceContext` and not `@Autowired`?

This is an excellent interview question.

`EntityManager` is **not thread-safe** and is bound to a transaction. If Spring injected a single shared instance into your singleton repository, concurrent requests would share one persistence context — a severe correctness bug.

`@PersistenceContext` injects a **thread-safe proxy**. On each call it looks up the `EntityManager` bound to the **current thread's transaction** and delegates to it. So every request transparently gets its own persistence context.

```
Singleton Repository
      │ holds
      ▼
 EntityManager PROXY  (shared, thread-safe)
      │ at call time resolves
      ▼
 The real EntityManager for THIS thread's transaction
```

**In practice with Spring Data JPA you rarely touch either** — but understanding this explains why persistence context scope equals transaction scope.

## 32.3 Persistence context scope = transaction scope

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;

    @Transactional
    public void updateOrderStatus(Long orderId, OrderStatus newStatus) {
        // ─── PERSISTENCE CONTEXT OPENS HERE (transaction begins) ───

        Order order = orderRepository.findById(orderId)     // SELECT; entity now MANAGED
                .orElseThrow(() -> new ResourceNotFoundException("Order", orderId));

        order.setStatus(newStatus);       // just a setter — no SQL yet

        // NO save() CALL NEEDED — see dirty checking below

        // ─── TRANSACTION COMMITS ───
        // Hibernate flushes: compares current state to the snapshot,
        // detects status changed, issues UPDATE, then commits.
        // PERSISTENCE CONTEXT CLOSES — all entities become DETACHED.
    }
}
```

**The most surprising thing for newcomers: you don't call `save()`.** Because the entity is managed, Hibernate detects the change automatically.

## 32.4 Dirty Checking

**Simple words:** when Hibernate loads an entity it takes a **snapshot** of the loaded values. At flush time it compares the current values against that snapshot. Any difference produces an UPDATE automatically.

```
1. findById(1)
        │
        ▼
   SELECT ... FROM orders WHERE id = 1
        │
        ▼
   Entity created and stored in the persistence context
   PLUS a SNAPSHOT of the loaded values:
        snapshot = { status: "PENDING", amount: 500 }
        │
        ▼
2. order.setStatus(SHIPPED)          ← in-memory only
        current  = { status: "SHIPPED", amount: 500 }
        │
        ▼
3. Transaction commit → FLUSH
        compare current vs snapshot → 'status' differs
        │
        ▼
   UPDATE orders SET status = 'SHIPPED', version = version + 1
   WHERE id = 1 AND version = 3
```

**Memory cost note:** that snapshot doubles the memory per managed entity. Loading 100,000 entities in one transaction means 200,000 objects. For bulk reads use a projection or `@Transactional(readOnly = true)`, which lets Hibernate skip snapshots entirely.

## 32.5 Flush modes and ordering

**Flush** = sending pending SQL to the database. It does **not** commit.

Hibernate flushes automatically:
1. Before the transaction commits
2. Before a query that might be affected by pending changes
3. When you call `flush()` explicitly

```java
@Transactional
public void demo() {
    Order order = new Order();
    orderRepository.save(order);        // persist() — NO INSERT yet, just queued

    // Hibernate flushes here, because the query could be affected by the pending insert
    List<Order> all = orderRepository.findAll();   // INSERT runs, then SELECT
}
```

**Why defer writes at all?** Batching (fewer round trips), correct ordering (parent before child), and the chance to cancel work that becomes unnecessary.

**Operation order at flush (fixed, and occasionally surprising):**
```
1. INSERTs      2. UPDATEs      3. Collection deletions
4. Collection updates/inserts   5. Entity DELETEs
```
This is why a "delete then insert with the same unique key" in one transaction fails with a constraint violation — the INSERT runs *before* the DELETE. The fix is an explicit `flush()` between them.

## 32.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Expecting changes to persist without a transaction | Entity is detached; no dirty checking; **changes silently lost** |
| `@Autowired EntityManager` | Thread-safety violation |
| Loading thousands of entities in one transaction | Memory blowout from snapshots |
| Using a detached entity and expecting dirty checking | No updates |
| Calling `save()` on an already-managed entity | Harmless but unnecessary — dirty checking already handles it |
| Delete-then-insert with the same key, no `flush()` | Constraint violation from flush ordering |

## 32.7 TL Questions

**Q: What is the persistence context?**
A: Hibernate's in-memory workspace for the entities it currently manages, scoped to the transaction. It acts as the first-level cache, tracks changes for dirty checking, and guarantees one object instance per database row within its scope.

**Q: Why don't we call `save()` after changing a managed entity?**
A: Because dirty checking handles it. Hibernate snapshots the entity when it's loaded and compares at flush time, issuing an UPDATE for whatever changed. Calling `save()` is harmless but redundant.

**Q: Why `@PersistenceContext` instead of `@Autowired` for `EntityManager`?**
A: `EntityManager` isn't thread-safe and is bound to a transaction. `@PersistenceContext` injects a proxy that resolves the right `EntityManager` for the current thread's transaction, so concurrent requests each get their own persistence context.

**Q: What happens if we modify an entity outside a transaction?**
A: The entity is detached, so there's no dirty checking and the change is silently lost. This is a common bug — no error, just a change that never reaches the database.

**Q: What is flushing and when does it happen?**
A: Flushing sends the pending SQL to the database without committing. Hibernate flushes before commit, before a query whose results could be affected by pending changes, and when we call `flush()` explicitly.

**Q: Why does Hibernate defer writes instead of issuing SQL immediately?**
A: To batch statements into fewer round trips, to order them correctly — parents before children — and to avoid work that later becomes unnecessary within the same transaction.

**Q: What's the memory impact of the persistence context?**
A: Each managed entity costs roughly double, because of the snapshot used for dirty checking. That's why we use `readOnly = true` or projections for large reads — Hibernate can then skip snapshots.

## 32.8 Interview Questions

**Beginner — What is `EntityManager`?**
The JPA interface for interacting with the persistence context — finding, persisting, merging and removing entities, and creating queries.

**Intermediate — `persist()` vs `merge()`?**
`persist()` makes a **new** entity managed and throws if it's already persistent; it makes the passed instance managed. `merge()` copies the state of a **detached** instance into a managed one and **returns the managed instance** — the passed object stays detached. Forgetting to use the returned value from `merge()` is a classic bug.

**Intermediate — `find()` vs `getReference()`?**
`find()` hits the database immediately (unless it's already in the context) and returns the entity or null. `getReference()` returns a lazy **proxy** without a query; the database is hit on first property access, and it throws `EntityNotFoundException` then if the row is missing. `getReference()` is useful for setting a foreign-key association without loading the whole entity.

**Advanced — What guarantee does the persistence context give about identity?**
Within one persistence context, a given database row is represented by exactly **one** object instance, so `==` reference equality holds for two lookups of the same ID. This is the *repeatable read at the object level* guarantee.

**Advanced — Transaction-scoped vs extended persistence context?**
Transaction-scoped (the Spring default) opens and closes with the transaction. An extended persistence context spans multiple transactions, keeping entities managed between them — used in stateful Jakarta EE conversations, and rare in Spring applications.

## 32.9 Quick Revision

1. Persistence context = in-memory workspace of managed entities, **scoped to the transaction**.
2. `EntityManager` is the API handle to it.
3. Use **`@PersistenceContext`**, never `@Autowired` — thread safety.
4. **Dirty checking**: snapshot at load, compare at flush, auto-UPDATE.
5. **No `save()` needed** for an already-managed entity.
6. Flush ≠ commit. Flush sends SQL; commit ends the transaction.
7. Flush triggers: before commit, before an affected query, explicit call.
8. Flush order: INSERT → UPDATE → collection ops → DELETE.
9. One row = one object instance within a context (identity guarantee).
10. Snapshots double memory — use `readOnly` or projections for large reads.
11. Changes outside a transaction are **silently lost**.

## 32.10 TL Explanation (speak this)

> "The persistence context is Hibernate's in-memory workspace for the entities in the current transaction, and almost everything else follows from it. It snapshots each entity when loaded, so at commit time it compares and issues an UPDATE for whatever changed — that's dirty checking, and it's why we don't call `save()` on an entity we loaded inside a transaction. It's also the first-level cache, so repeated lookups of the same ID in one transaction don't re-query. The practical consequences are that modifying an entity outside a transaction silently does nothing, and that loading tens of thousands of entities is expensive because of those snapshots — so bulk reads use `readOnly` or projections."

---

[← 31. Entity and Mapping Annotations](31-entity-and-mapping-annotations.md) | [Contents](../README.md) | [33. First Level Cache →](33-first-level-cache.md)
