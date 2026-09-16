[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 38. Cascade Types

## 38.1 What is it?

**Simple words:**
Cascading means **operations on a parent automatically apply to its children**. Save the order — its items are saved too. Delete the order — its items are deleted too.

**Technical wording:**
Cascade types define which `EntityManager` operations propagate from a parent entity to its associated entities along a relationship.

## 38.2 The cascade types

| Type | Propagates | Effect |
|---|---|---|
| `PERSIST` | `persist()` | Saving the parent saves new children |
| `MERGE` | `merge()` | Merging the parent merges children |
| `REMOVE` | `remove()` | Deleting the parent deletes children |
| `REFRESH` | `refresh()` | Reloading the parent reloads children |
| `DETACH` | `detach()` | Detaching the parent detaches children |
| **`ALL`** | All of the above | Convenient, but see the warning |

```java
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
private List<OrderItem> items = new ArrayList<>();

// Now:
Order order = new Order();
order.addItem(new OrderItem("Laptop", 1, new BigDecimal("50000")));
order.addItem(new OrderItem("Mouse",  2, new BigDecimal("500")));

orderRepository.save(order);
// ONE call inserts the order AND both items.
// Without cascade, we'd have to save each item separately.
```

## 38.3 `orphanRemoval` vs `CascadeType.REMOVE`

| | `CascadeType.REMOVE` | `orphanRemoval = true` |
|---|---|---|
| Deletes children when the **parent is deleted** | Yes | Yes |
| Deletes a child **removed from the collection** | **No** | **Yes** |
| Meaning | "Delete children with the parent" | "Children cannot exist without the parent" |

```java
order.getItems().remove(item);
// With CascadeType.REMOVE only → the item row survives with a null/stale FK
// With orphanRemoval = true    → DELETE FROM order_items WHERE id = ?
```

**Use `orphanRemoval = true` for true parent-child ownership** — order items, invoice lines, address books. Do **not** use it where the child has independent meaning.

## 38.4 Where cascade is right and where it is dangerous

### ✅ Correct: a genuine parent-child composition

```java
@Entity
public class Order {
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}
```
An `OrderItem` has no meaning without its `Order`. Deleting the order should delete its items. **Correct.**

### ❌ Dangerous: cascading to a shared, independent entity

```java
@Entity
public class Order {
    @ManyToOne(cascade = CascadeType.ALL)     // ❌ CATASTROPHIC
    @JoinColumn(name = "customer_id")
    private Customer customer;
}

// Consequence:
orderRepository.delete(order);
// → DELETES THE CUSTOMER, and cascades to all their other orders.
// One cancelled order wipes out a customer record.
```

**The rule to state plainly:**
> **Never cascade REMOVE (or ALL) from the `@ManyToOne` side.**
> The "many" side references a shared entity it does not own.

```java
// ✅ CORRECT — no cascade on @ManyToOne
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "customer_id", nullable = false)
private Customer customer;
```

### The ownership test

Before adding a cascade, ask: **"Does this child belong exclusively to this parent, and is it meaningless without it?"**

| Relationship | Exclusive? | Cascade? |
|---|---|---|
| Order → OrderItem | Yes | `ALL` + `orphanRemoval` |
| Invoice → InvoiceLine | Yes | `ALL` + `orphanRemoval` |
| User → UserProfile | Yes | `ALL` + `orphanRemoval` |
| Order → Customer | **No** — shared | **None** |
| OrderItem → Product | **No** — shared | **None** |
| Post → Comment | Usually yes | `ALL` + `orphanRemoval` |

## 38.5 Common Mistakes

| Mistake | Consequence |
|---|---|
| `CascadeType.ALL` on `@ManyToOne` | **Deleting a child deletes the shared parent** — data loss |
| Cascading to a shared reference entity (Product, Country) | Deleting one row removes reference data everywhere |
| No cascade where genuinely needed | `TransientObjectException` — "object references an unsaved transient instance" |
| `orphanRemoval` on a non-owned child | Unexpected deletions |
| Relying on cascade for bulk deletes | Hibernate loads every child and deletes one by one — slow |

**The `TransientObjectException` fix:**
```java
// Error: "object references an unsaved transient instance -
//         save the transient instance before flushing"
// Cause: saving a parent with a new, unsaved child and no CascadeType.PERSIST.
// Fix: add cascade = CascadeType.PERSIST (or ALL), or save the child first.
```

**Performance caution on cascaded deletes:** `CascadeType.REMOVE` loads **every** child into memory and issues one DELETE per row. Deleting an order with 10,000 items means 10,001 statements. For large volumes use a bulk `@Modifying` delete or a database-level `ON DELETE CASCADE`, accepting that lifecycle callbacks are then skipped.

## 38.6 TL Questions

**Q: What is cascading?**
A: Propagating an operation from a parent entity to its children — so saving an order also saves its items, and deleting it deletes them.

**Q: When do we use `CascadeType.ALL`?**
A: Only for true parent-child ownership where the child is meaningless without the parent — order items, invoice lines. Never from the `@ManyToOne` side to a shared entity.

**Q: What's the danger of cascade on `@ManyToOne`?**
A: The many side points at a shared entity it doesn't own. Cascading REMOVE from an order to its customer means deleting one order deletes the customer and cascades onward to all their other orders. It's a genuine data-loss scenario.

**Q: `orphanRemoval` versus `CascadeType.REMOVE`?**
A: `REMOVE` deletes children when the parent is deleted. `orphanRemoval` additionally deletes a child when it's removed from the parent's collection. We use orphan removal where the child genuinely can't exist independently.

**Q: What if we forget the cascade?**
A: Saving a parent with new unsaved children throws `TransientObjectException` — "object references an unsaved transient instance". Either add `CascadeType.PERSIST` or save the children first.

**Q: Is cascaded delete efficient?**
A: No, for large collections. Hibernate loads every child and issues one DELETE each, so deleting an order with ten thousand items is ten thousand statements. For that scale we use a bulk delete query or a database-level cascade.

## 38.7 Interview Questions

**Beginner — What does `CascadeType.PERSIST` do?**
Saving the parent also persists its new associated children.

**Intermediate — Why is `CascadeType.ALL` risky on `@ManyToOne`?**
Because it includes REMOVE, and the many side references a shared entity. Deleting one child would delete the shared parent and everything else attached to it.

**Advanced — What is `TransientObjectException` and when does it occur?**
It occurs at flush when a managed entity references an entity that has never been persisted and no `CascadeType.PERSIST` covers the association. Hibernate cannot write a foreign key to a row that doesn't exist.

**Advanced — Cascade vs database `ON DELETE CASCADE`?**
JPA cascade is application-level: Hibernate loads children and deletes them individually, so lifecycle callbacks and the persistence context stay consistent, but it's slow at volume. Database cascade is a single, fast server-side operation, but Hibernate is unaware of it, so the persistence context can hold entities for rows that no longer exist. Choose deliberately, and don't rely on both silently.

## 38.8 Quick Revision

1. Cascade propagates operations from parent to children.
2. Types: PERSIST, MERGE, REMOVE, REFRESH, DETACH, ALL.
3. **`ALL` + `orphanRemoval` only for true parent-child ownership.**
4. **Never cascade REMOVE/ALL from `@ManyToOne`.**
5. `orphanRemoval` also deletes children removed from the collection.
6. Ownership test: *is the child meaningless without this parent?*
7. Missing `PERSIST` cascade → `TransientObjectException`.
8. Cascaded deletes are row-by-row — slow at volume.

## 38.9 TL Explanation (speak this)

> "Cascading propagates operations from parent to child, so saving an order saves its items in one call. We only use `CascadeType.ALL` with `orphanRemoval` where the child genuinely can't exist without the parent — order items, invoice lines. The rule we never break is no cascade from the `@ManyToOne` side, because that side points at a shared entity: cascading remove from an order to its customer would delete the customer and all their other orders. And for very large child collections we don't rely on cascade at all, because Hibernate deletes row by row — we use a bulk delete instead."

---

[← 37. Entity Relationships](37-entity-relationships.md) | [Contents](../README.md) | [39. Lazy vs Eager Loading →](39-lazy-vs-eager-loading.md)
