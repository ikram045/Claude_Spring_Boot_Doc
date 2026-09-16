[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 37. Entity Relationships

## 37.1 The four relationship types

| Annotation | Meaning | Example |
|---|---|---|
| `@OneToOne` | One row relates to exactly one row | User ↔ UserProfile |
| `@OneToMany` | One row relates to many rows | Order → OrderItems |
| `@ManyToOne` | Many rows relate to one row | OrderItem → Order |
| `@ManyToMany` | Many relate to many | Student ↔ Course |

**Key concept — the owning side:**
In a relational database, the **foreign key lives in one table**. The entity holding that foreign key is the **owning side**. The other side is the **inverse side**, marked with `mappedBy`.

> **Only changes to the owning side are written to the database.** Setting the inverse side alone changes nothing — a frequent and confusing bug.

## 37.2 @ManyToOne and @OneToMany (the most common pair)

```java
// ---------- MANY side — OWNS the relationship (holds the FK) ----------
@Entity
@Table(name = "order_items")
@Getter @Setter @NoArgsConstructor
public class OrderItem {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)        // ★ ALWAYS set LAZY explicitly
    @JoinColumn(name = "order_id", nullable = false)   // this is the FK column
    private Order order;                                // ← OWNING SIDE

    private String productName;
    private Integer quantity;
    private BigDecimal price;
}

// ---------- ONE side — INVERSE (no FK column) ----------
@Entity
@Table(name = "orders")
@Getter @Setter @NoArgsConstructor
public class Order {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "order",             // ← the FIELD NAME in OrderItem
               cascade = CascadeType.ALL,
               orphanRemoval = true,
               fetch = FetchType.LAZY)         // default for @OneToMany, but be explicit
    private List<OrderItem> items = new ArrayList<>();   // initialise to avoid NPE

    // ★ HELPER METHODS — keep both sides of the relationship in sync
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);          // CRITICAL: set the owning side too
    }

    public void removeItem(OrderItem item) {
        items.remove(item);
        item.setOrder(null);          // with orphanRemoval=true this deletes the row
    }
}
```

### Why the helper methods matter

```java
// ❌ WRONG — only the inverse side is set
Order order = new Order();
OrderItem item = new OrderItem();
order.getItems().add(item);          // inverse side only
orderRepository.save(order);
// The order_id column in order_items is NULL, or the row isn't inserted at all.
// Why? Hibernate reads the FK from the OWNING side (item.order), which is null.

// ✅ CORRECT
order.addItem(item);                 // sets BOTH sides
orderRepository.save(order);
```

**This is one of the most common JPA bugs in real projects.** Always provide `addX`/`removeX` helpers on the inverse side, and make it a code-review rule.

`mappedBy = "order"` means: *"I am not the owner. The `order` field in `OrderItem` owns this relationship."* Without it, Hibernate assumes both sides are independent and creates an unnecessary **join table**.

## 37.3 @OneToOne

```java
@Entity
public class User {
    @Id @GeneratedValue private Long id;

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL,
              fetch = FetchType.LAZY, orphanRemoval = true)
    private UserProfile profile;                 // inverse side
}

@Entity
public class UserProfile {
    @Id @GeneratedValue private Long id;

    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", unique = true)
    private User user;                           // OWNING side — holds the FK
}
```

**The `@OneToOne` lazy-loading catch (a strong senior-level point):**
Lazy loading on the **inverse** side of a `@OneToOne` **does not work** by default. Hibernate must know whether to create a proxy or set `null`, and the only way to know is to query — so it queries eagerly regardless of the annotation.

Workarounds: make the association **optional = false** (if it's guaranteed to exist), use bytecode enhancement, or **map it as `@ManyToOne` with a unique constraint** — which is often the pragmatic choice.

## 37.4 @ManyToMany

```java
@Entity
public class Student {
    @Id @GeneratedValue private Long id;

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(name = "student_course",                          // join table
               joinColumns = @JoinColumn(name = "student_id"),
               inverseJoinColumns = @JoinColumn(name = "course_id"))
    private Set<Course> courses = new HashSet<>();               // Set, not List
}

@Entity
public class Course {
    @Id @GeneratedValue private Long id;

    @ManyToMany(mappedBy = "courses", fetch = FetchType.LAZY)
    private Set<Student> students = new HashSet<>();
}
```

**Use `Set`, not `List`, for `@ManyToMany`.** With a `List`, removing one element makes Hibernate **delete every join row and re-insert the remainder** — catastrophically inefficient on large collections. A `Set` deletes only the one row.

### The strong recommendation: replace `@ManyToMany` with two `@OneToMany`

```java
// Break the many-to-many into an explicit join ENTITY
@Entity
@Table(name = "student_course")
@Getter @Setter
public class Enrollment {

    @Id @GeneratedValue private Long id;

    @ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name = "student_id")
    private Student student;

    @ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name = "course_id")
    private Course course;

    // ★ Now we can store data ABOUT the relationship — impossible with @ManyToMany
    private LocalDate enrolledOn;
    private String grade;
    private EnrollmentStatus status;
}
```

**Why this is almost always better:** a pure `@ManyToMany` join table can hold *only* the two foreign keys. The moment the business asks "when did they enrol?" or "what grade did they get?" — and it always does — you must refactor to a join entity anyway. Starting with the entity avoids a painful migration later.

**This is an excellent point to raise with a TL**, because it's a design judgement rather than a syntax fact.

## 37.5 Bidirectional vs Unidirectional

| | Unidirectional | Bidirectional |
|---|---|---|
| Navigation | One direction only | Both directions |
| Complexity | Lower | Higher (sync both sides) |
| Jackson recursion risk | No | **Yes** — needs DTOs or `@JsonIgnore` |
| `equals`/`hashCode`/`toString` risk | Lower | **Higher** |
| **Recommendation** | **Prefer this** | Only when you genuinely navigate both ways |

**Advice:** make the association unidirectional unless you actually need to navigate from both ends. Bidirectional adds synchronisation burden and serialization hazards for a convenience you may never use. If you do need it, use DTOs (Section 21), which removes the recursion problem entirely.

## 37.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Setting only the inverse side | FK is null; relationship not persisted |
| Missing `mappedBy` on the inverse side | Unwanted extra join table |
| `EAGER` on collections | N+1 and over-fetching everywhere |
| `List` with `@ManyToMany` | Delete-all-and-reinsert on every change |
| Returning bidirectional entities as JSON | Infinite recursion |
| Uninitialised collection field | `NullPointerException` on `getItems().add(...)` |
| `CascadeType.ALL` on `@ManyToOne` | Deleting an item deletes the parent order |
| `@ManyToMany` where relationship attributes will be needed | Painful later refactor |

## 37.7 TL Questions

**Q: What is the owning side?**
A: The entity holding the foreign key column — normally the `@ManyToOne` side. Only changes to the owning side are written to the database, which is why setting just the inverse collection doesn't persist anything.

**Q: What does `mappedBy` do?**
A: It marks the inverse side and names the field on the owning side that holds the relationship. Without it Hibernate treats the two sides as separate relationships and creates an unnecessary join table.

**Q: Why do we write `addItem`/`removeItem` helpers?**
A: To keep both sides in sync. Adding to the collection alone leaves the foreign key null, so the child row is never linked. The helper sets both references in one call, which makes it hard to get wrong.

**Q: Why do we avoid `@ManyToMany`?**
A: Because a pure join table can only hold the two foreign keys. As soon as the business wants an enrolment date or a status on the relationship, we have to refactor to a join entity. Starting with an explicit join entity with two `@ManyToOne`s avoids that migration and gives us more control.

**Q: What happens if we return bidirectional entities as JSON?**
A: Jackson recurses infinitely — order serializes items, each item serializes its order, and so on until the stack overflows. DTOs solve it properly; `@JsonIgnore` patches it but leaves the entity exposed.

**Q: Why `Set` rather than `List` for many-to-many?**
A: With a `List`, removing one element makes Hibernate delete every join row and re-insert the rest. With a `Set` it deletes just the one row.

## 37.8 Interview Questions

**Beginner — Which side holds the foreign key?**
The owning side — usually `@ManyToOne`.

**Intermediate — What happens without `mappedBy`?**
Hibernate treats each side as its own unidirectional association and creates an extra join table, so the schema and the updates are both wrong.

**Intermediate — Why doesn't lazy loading work on `@OneToOne` inverse sides?**
Hibernate must decide between a proxy and `null`, and the only way to know which is to query. So it queries eagerly regardless of the fetch type. Mapping it as `@ManyToOne` with a unique constraint, or using bytecode enhancement, avoids it.

**Advanced — What's the difference between `orphanRemoval = true` and `CascadeType.REMOVE`?**
`CascadeType.REMOVE` deletes children when the **parent** is deleted. `orphanRemoval = true` additionally deletes a child when it is **removed from the parent's collection**, even if the parent survives. Orphan removal expresses a genuine parent-child ownership relationship.

**Advanced — Why is `@ManyToMany` with `List` inefficient?**
Hibernate cannot identify individual rows in a bag-semantics `List` without an index column, so on any modification it deletes all join rows for the owner and re-inserts the current contents. A `Set` has identity semantics and allows targeted deletes.

## 37.9 Quick Revision

1. Four types: `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`.
2. **Owning side holds the FK**; only its changes are persisted.
3. `mappedBy` marks the inverse side and names the owning field.
4. **Always write `addX`/`removeX` helpers to sync both sides.**
5. Initialise collection fields (`= new ArrayList<>()`).
6. `@ManyToMany` → use `Set`, and prefer a **join entity** instead.
7. Prefer unidirectional unless both directions are genuinely needed.
8. Bidirectional + entity JSON = infinite recursion → use DTOs.
9. `orphanRemoval` deletes children removed from the collection.
10. Never `CascadeType.ALL` on `@ManyToOne`.

## 37.10 TL Explanation (speak this)

> "The owning side of a relationship is whichever entity holds the foreign key — normally the `@ManyToOne` side — and only changes to that side get written. That's why we always have `addItem`/`removeItem` helpers on the parent that set both references; adding to the collection alone leaves the foreign key null and the link is never saved. We generally avoid `@ManyToMany` and model the join as its own entity with two `@ManyToOne`s, because a plain join table can't carry any attributes, and the business always ends up wanting a date or a status on the relationship. And we keep entities out of our JSON responses, which removes the bidirectional recursion problem entirely."

---

[← 36. Query Methods, JPQL, @Query and Pagination](36-query-methods-jpql-query-and-pagination.md) | [Contents](../README.md) | [38. Cascade Types →](38-cascade-types.md)
