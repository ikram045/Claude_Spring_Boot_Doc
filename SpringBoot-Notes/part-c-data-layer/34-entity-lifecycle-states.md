[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 34. Entity Lifecycle States

## 34.1 The four states

Every entity is in exactly one of four states. Understanding them explains most "why didn't my change save?" problems.

```
                     ┌──────────────┐
                     │  TRANSIENT   │   new Order()
                     │              │   - not in persistence context
                     │  (new)       │   - no DB row
                     └──────┬───────┘   - no ID
                            │
                    persist() / save()
                            │
                            ▼
                     ┌──────────────┐
      find() ───────►│   MANAGED    │◄────── merge()
      query  ───────►│              │
                     │ (persistent) │   - IN the persistence context
                     │              │   - has a DB row (or will at flush)
                     │              │   - ★ DIRTY CHECKING ACTIVE ★
                     └──┬────────┬──┘
                        │        │
              detach()  │        │  remove()
              clear()   │        │
              tx ends   │        │
                        ▼        ▼
              ┌──────────────┐  ┌──────────────┐
              │   DETACHED   │  │   REMOVED    │
              │              │  │              │
              │ - has an ID  │  │ - scheduled  │
              │ - row exists │  │   for DELETE │
              │ - NOT tracked│  │ - still in   │
              │ - NO dirty   │  │   context    │
              │   checking   │  │   until flush│
              └──────────────┘  └──────────────┘
```

## 34.2 Each state explained

### 1. TRANSIENT (New)
```java
Order order = new Order();           // TRANSIENT
order.setCustomerId("CUST-1");
// Not in the persistence context. No database row. ID is null.
// If garbage collected now, nothing is lost — it was never persisted.
```

### 2. MANAGED (Persistent)
```java
@Transactional
public void demo() {
    Order order = orderRepository.save(new Order());   // TRANSIENT → MANAGED
    Order found = orderRepository.findById(1L).get();  // loaded as MANAGED

    found.setStatus(OrderStatus.SHIPPED);   // ★ dirty checking will UPDATE this
}
```
**Only managed entities get dirty checking.** This is the state that matters most.

### 3. DETACHED
```java
@Transactional
public Order load(Long id) {
    return orderRepository.findById(id).orElseThrow();   // MANAGED here
}   // ← transaction ends, persistence context closes

public void caller() {
    Order order = load(1L);            // now DETACHED
    order.setStatus(OrderStatus.SHIPPED);
    // ★ NOTHING HAPPENS. No UPDATE. The change is silently lost.
}
```
**This is the single most common JPA bug.** The object looks fine, the setter works, and nothing reaches the database.

**To persist a detached entity's changes:**
```java
@Transactional
public void update(Order detachedOrder) {
    Order managed = entityManager.merge(detachedOrder);   // returns a MANAGED copy
    // IMPORTANT: detachedOrder itself stays detached.
    // Work with 'managed' from here on.
}
```

### 4. REMOVED
```java
@Transactional
public void delete(Long id) {
    Order order = orderRepository.findById(id).orElseThrow();   // MANAGED
    orderRepository.delete(order);       // → REMOVED (still in the context)
    // DELETE is issued at flush/commit, not immediately.
}
```

## 34.3 State transition reference

| From | Operation | To |
|---|---|---|
| Transient | `persist()` / `save()` | Managed |
| Transient | `merge()` | Managed (a **copy**) |
| Managed | `remove()` | Removed |
| Managed | `detach()` / `clear()` / transaction end | Detached |
| Managed | `refresh()` | Managed (reloaded from DB) |
| Detached | `merge()` | Managed (a **copy** is returned) |
| Removed | `persist()` | Managed (undoes the delete) |

## 34.4 `persist()` vs `merge()` vs `save()`

| | `persist()` | `merge()` | `save()` (Spring Data) |
|---|---|---|---|
| Defined by | JPA | JPA | Spring Data |
| Intended for | **New** entities | **Detached** entities | Both |
| Returns | `void` | **A managed copy** | The saved entity |
| Makes the argument managed | **Yes** | **No** — returns a different instance | Depends |
| If the entity exists | Throws `EntityExistsException` | Updates it | Updates it |
| SELECT before write | No | **Yes** (to load current state) | Yes when merging |

**How `save()` actually works in Spring Data JPA:**

```java
// SimpleJpaRepository.save(), simplified
@Transactional
public <S extends T> S save(S entity) {
    if (entityInformation.isNew(entity)) {     // usually: is the ID null?
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);               // ← note: returns a DIFFERENT instance
    }
}
```

**The trap this creates:**
```java
// ❌ WRONG — for an existing entity, save() returns a different managed instance
orderRepository.save(detachedOrder);
System.out.println(detachedOrder.getVersion());   // stale — not updated

// ✅ CORRECT — always use the returned value
Order saved = orderRepository.save(detachedOrder);
System.out.println(saved.getVersion());
```
**Always use the return value of `save()`.** It costs nothing and avoids a genuinely confusing bug.

## 34.5 Real scenario — the detached-entity bug

```java
// ❌ BROKEN
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;

    public Order getOrder(Long id) {                 // NO @Transactional
        return orderRepository.findById(id).orElseThrow();
    }   // Spring Data opens a transaction for findById and closes it → DETACHED

    public void shipOrder(Long id) {                 // NO @Transactional
        Order order = getOrder(id);
        order.setStatus(OrderStatus.SHIPPED);        // detached → no dirty checking
        // NOTHING IS SAVED. No error. Status unchanged in the database.
    }
}

// ✅ FIXED
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;

    @Transactional                                    // one transaction spans the whole method
    public void shipOrder(Long id) {
        Order order = orderRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Order", id));
        order.setStatus(OrderStatus.SHIPPED);         // MANAGED → dirty checking applies
        // UPDATE issued automatically at commit. No save() needed.
    }
}
```

**The lesson, stated plainly:** `@Transactional` on the service method is what keeps entities managed across the whole unit of work. Without it, every repository call is its own micro-transaction and everything you hold afterwards is detached.

## 34.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| **Modifying a detached entity** | Change silently lost — no error |
| Ignoring `merge()`'s return value | Working with the still-detached instance |
| Ignoring `save()`'s return value | Stale ID/version |
| `persist()` on a detached entity | `EntityExistsException` |
| Accessing a lazy field after detachment | `LazyInitializationException` |
| Missing `@Transactional` on multi-step service methods | Entities detached between calls; no atomicity |

## 34.7 TL Questions

**Q: What are the entity states?**
A: Transient — a new object not known to Hibernate; Managed — tracked in the persistence context with dirty checking active; Detached — was managed but the context closed; and Removed — scheduled for deletion.

**Q: Why do changes to a detached entity get lost?**
A: Dirty checking only applies to managed entities. Once the persistence context closes, Hibernate is no longer tracking that object, so setters just change memory. There's no error, which is what makes it a hard bug to spot.

**Q: How do we reattach a detached entity?**
A: `merge()`, which copies its state into a managed instance and returns that instance. The important detail is that the original object stays detached — we must use the returned one.

**Q: What's the difference between `persist()` and `merge()`?**
A: `persist()` is for new entities and makes the passed object managed. `merge()` is for detached entities, does a SELECT to load current state, and returns a **different** managed instance. Spring Data's `save()` picks between them based on whether the ID is null.

**Q: What happens if we forget `@Transactional` on a service method?**
A: Each repository call runs in its own short transaction, so anything we hold afterwards is detached and modifications are lost. We also lose atomicity — a failure halfway through leaves earlier writes committed.

**Q: Why should we always use `save()`'s return value?**
A: Because for an existing entity it delegates to `merge()`, which returns a different managed instance. The object we passed in keeps the old ID and version, which produces confusing bugs.

## 34.8 Interview Questions

**Beginner — Name the four entity states.**
Transient, Managed (persistent), Detached, Removed.

**Beginner — Which state has dirty checking?**
Managed only.

**Intermediate — What does `merge()` return and why does it matter?**
A managed copy of the entity — a **different object** from the one passed in. The argument remains detached, so subsequent changes to it are ignored. Always use the returned reference.

**Intermediate — Can a removed entity be brought back?**
Yes — calling `persist()` on it before the flush cancels the scheduled deletion and returns it to the managed state.

**Advanced — Why does `merge()` issue a SELECT?**
It must load the current database state into the persistence context before copying the detached values in, so Hibernate can compute the correct UPDATE and enforce optimistic-locking checks. `persist()` needs no SELECT because the row doesn't exist yet.

**Advanced — How does Spring Data decide new vs existing in `save()`?**
Via `EntityInformation.isNew()`. By default it checks whether the `@Id` is null (or zero for primitives). For an entity with an assigned ID this misfires, so you implement `Persistable` and override `isNew()`, or use a `@Version` field, which Spring Data also consults.

## 34.9 Quick Revision

1. Four states: **Transient → Managed → Detached / Removed**.
2. **Only MANAGED entities get dirty checking.**
3. Detached modifications are **silently lost** — the #1 JPA bug.
4. `persist()` for new; `merge()` for detached; `save()` chooses.
5. **`merge()` returns a different instance** — use the return value.
6. **Always use `save()`'s return value.**
7. `@Transactional` on the service keeps entities managed across the unit of work.
8. Accessing lazy fields when detached → `LazyInitializationException`.
9. `remove()` marks for deletion; the DELETE runs at flush.

## 34.10 TL Explanation (speak this)

> "Entities move through four states — transient before we save them, managed while they're in the persistence context, detached once the transaction ends, and removed when scheduled for deletion. The one that causes real bugs is detached: dirty checking only works on managed entities, so setting a field on a detached object changes nothing in the database and throws no error. That's why our service methods carry `@Transactional` — it keeps the entity managed for the whole unit of work, so we can load it, modify it, and let Hibernate write the update at commit without calling `save()` at all. And when we do call `save()` or `merge()` on a detached entity we always use the returned instance, because those return a different object from the one we passed in."

---

[← 33. First Level Cache](33-first-level-cache.md) | [Contents](../README.md) | [35. Spring Data Repositories →](35-spring-data-repositories.md)
