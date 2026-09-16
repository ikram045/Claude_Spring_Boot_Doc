[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 31. Entity and Mapping Annotations

## 31.1 What is an Entity?

**Simple words:**
An entity is a **Java class that maps to a database table**. One object = one row. Each field = one column.

**Technical wording:**
An entity is a lightweight persistent domain object annotated `@Entity`, representing a table, whose instances are managed by the persistence provider and correspond to rows identified by a primary key.

## 31.2 A complete entity

```java
package com.company.orderservice.entity;

@Entity
@Table(name = "orders",
       indexes = {
           @Index(name = "idx_order_customer", columnList = "customer_id"),
           @Index(name = "idx_order_status",   columnList = "status")
       },
       uniqueConstraints = @UniqueConstraint(name = "uk_order_number",
                                             columnNames = "order_number"))
@Getter @Setter
@NoArgsConstructor                 // REQUIRED by JPA
@AllArgsConstructor
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "order_number", nullable = false, length = 20, updatable = false)
    private String orderNumber;

    @Column(name = "customer_id", nullable = false)
    private String customerId;

    @Column(name = "total_amount", nullable = false, precision = 19, scale = 2)
    private BigDecimal totalAmount;        // NEVER use double for money

    @Enumerated(EnumType.STRING)           // store "PENDING", not 0
    @Column(nullable = false, length = 20)
    private OrderStatus status;

    @Column(name = "created_at", nullable = false, updatable = false)
    @CreationTimestamp                     // Hibernate sets this on insert
    private LocalDateTime createdAt;

    @UpdateTimestamp                       // Hibernate sets this on every update
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @Version                               // OPTIMISTIC LOCKING
    private Long version;

    @Lob
    @Column(name = "notes")
    private String notes;

    @Transient                             // NOT persisted — calculated in memory
    private String displayLabel;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}
```

## 31.3 The essential annotations

| Annotation | Purpose |
|---|---|
| `@Entity` | Marks the class as persistent |
| `@Table` | Table name, indexes, unique constraints |
| `@Id` | Primary key |
| `@GeneratedValue` | How the PK is generated |
| `@Column` | Column name, nullability, length, precision |
| `@Enumerated` | How an enum is stored |
| `@Temporal` | Date precision (legacy `Date`/`Calendar` only) |
| `@Lob` | Large object (CLOB/BLOB) |
| `@Transient` | **Do not persist this field** |
| `@Version` | Optimistic locking version column |
| `@Embedded` / `@Embeddable` | Value object mapped into the same table |
| `@CreationTimestamp` / `@UpdateTimestamp` | Hibernate auto-timestamps |

### JPA requirements for an entity class

1. Annotated `@Entity`
2. Has an `@Id`
3. Has a **public or protected no-argument constructor**
4. Class is **not `final`**; persistent fields/methods not `final`
5. Should implement `Serializable` if detached instances travel over the wire

**Why the no-arg constructor?** Hibernate instantiates entities **reflectively** when loading rows. It cannot guess constructor arguments. **Why not `final`?** Hibernate creates **proxy subclasses** for lazy loading — it cannot subclass a `final` class. These two rules cause real, confusing failures when Lombok's `@Builder` is used alone (it removes the no-arg constructor).

## 31.4 `@GeneratedValue` strategies — important

| Strategy | How it works | Batch inserts? | Notes |
|---|---|---|---|
| **`IDENTITY`** | Database auto-increment | **NO** | Simple; **disables JDBC batching** |
| **`SEQUENCE`** | Database sequence object | **YES** | **Best for Postgres/Oracle** |
| `TABLE` | A separate table simulates a sequence | Yes | Slow, contention-prone — avoid |
| `AUTO` | Provider picks | Depends | Unpredictable — be explicit |

**The `IDENTITY` batching trap (genuinely valuable to know):**
With `IDENTITY`, Hibernate must execute each INSERT immediately to learn the generated ID, so it **cannot batch inserts**. Setting `hibernate.jdbc.batch_size=50` has no effect. With `SEQUENCE` and an allocation size, Hibernate pre-fetches IDs and batches inserts properly.

```java
// Optimised sequence generation — 50 IDs per database round trip
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "order_seq")
@SequenceGenerator(name = "order_seq", sequenceName = "order_sequence",
                   allocationSize = 50)     // must match the DB sequence INCREMENT BY
private Long id;
```
> **Caution:** `allocationSize` must match the actual sequence's increment in the database, or you get duplicate-key errors. MySQL has no sequences, so `IDENTITY` is the practical choice there.

## 31.5 `@Enumerated` — a genuine production trap

```java
@Enumerated(EnumType.ORDINAL)     // ❌ stores 0, 1, 2 — THE INDEX
private OrderStatus status;

@Enumerated(EnumType.STRING)      // ✅ stores "PENDING", "SHIPPED"
private OrderStatus status;
```

**Why `ORDINAL` is dangerous:**
```java
enum OrderStatus { PENDING, CONFIRMED, SHIPPED }        // PENDING=0, CONFIRMED=1, SHIPPED=2
```
Later someone inserts a value:
```java
enum OrderStatus { PENDING, CANCELLED, CONFIRMED, SHIPPED }   // CANCELLED=1 now!
```
**Every existing row storing `1` silently changes meaning from CONFIRMED to CANCELLED.** No error, no warning — just corrupted data across the whole table. `ORDINAL` is also unreadable in the database.

**Always use `EnumType.STRING`.** The only cost is a few bytes per row.

## 31.6 `equals()` and `hashCode()` for entities (advanced but important)

A subtle area that causes real bugs.

```java
// ❌ WRONG — Lombok's default uses ALL fields
@Data                       // generates equals/hashCode over every field
@Entity
public class Order { }
```
Problems: the hash code changes when any field changes (breaking `HashSet` membership), and it triggers lazy loading of associations just to compute equality.

```java
// ✅ CORRECT — business key, or ID with a stable hashCode
@Entity
@Getter @Setter
public class Order {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "order_number", nullable = false, unique = true, updatable = false)
    private String orderNumber;         // natural business key

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Order other)) return false;
        return orderNumber != null && orderNumber.equals(other.orderNumber);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();   // constant — stable before and after persist
    }
}
```

**Why a constant `hashCode()`?** Before `persist()`, `id` is `null`. After `persist()`, it has a value. If `hashCode()` depends on `id`, an entity added to a `HashSet` *before* saving becomes unfindable *after* saving, because its bucket changed. A constant hash keeps it locatable; `equals` still distinguishes instances correctly.

**Never use `@Data` on an entity.** It also generates `toString()` over all fields, which triggers lazy loading and can cause infinite recursion on bidirectional relationships. Use `@Getter`/`@Setter` and write `equals`/`hashCode` deliberately.

## 31.7 `@Embeddable` — value objects

```java
@Embeddable
@Getter @Setter @NoArgsConstructor @AllArgsConstructor
public class Address {
    @Column(name = "street")   private String street;
    @Column(name = "city")     private String city;
    @Column(name = "pincode")  private String pincode;
}

@Entity
public class Customer {
    @Id @GeneratedValue
    private Long id;

    @Embedded
    private Address billingAddress;      // → columns street, city, pincode

    @Embedded
    @AttributeOverrides({                // reuse the same type with different columns
        @AttributeOverride(name = "street",  column = @Column(name = "ship_street")),
        @AttributeOverride(name = "city",    column = @Column(name = "ship_city")),
        @AttributeOverride(name = "pincode", column = @Column(name = "ship_pincode"))
    })
    private Address shippingAddress;
}
```
All columns live in the `customer` table — no join. It groups related fields into a reusable, meaningful type instead of six loose fields.

## 31.8 Auditing — who changed what, when

```java
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaAuditingConfig {
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
                .map(SecurityContext::getAuthentication)
                .filter(Authentication::isAuthenticated)
                .map(Authentication::getName)
                .or(() -> Optional.of("SYSTEM"));
    }
}

@MappedSuperclass                       // shared columns, NOT its own table
@EntityListeners(AuditingEntityListener.class)
@Getter @Setter
public abstract class BaseAuditEntity {

    @CreatedDate     @Column(updatable = false) private LocalDateTime createdAt;
    @LastModifiedDate                           private LocalDateTime updatedAt;
    @CreatedBy       @Column(updatable = false) private String createdBy;
    @LastModifiedBy                             private String updatedBy;
}

@Entity
public class Order extends BaseAuditEntity {    // inherits all four audit columns
    @Id @GeneratedValue private Long id;
}
```
**`@MappedSuperclass` vs `@Entity` inheritance:** `@MappedSuperclass` contributes **columns** to subclasses but is not itself queryable and has no table. That's exactly what you want for audit fields.

## 31.9 Common Mistakes

| Mistake | Consequence |
|---|---|
| `@EnumType.ORDINAL` | **Silent data corruption** when the enum changes |
| `double`/`float` for money | Rounding errors — use `BigDecimal` with `precision`/`scale` |
| `@Data` on an entity | Broken `equals`/`hashCode`, lazy loading in `toString`, recursion |
| Missing no-arg constructor (Lombok `@Builder` alone) | Hibernate can't instantiate the entity |
| `final` entity class | Lazy proxies impossible |
| Exposing entities in the API | See Section 21 |
| `@Column(nullable=false)` without a real DB constraint | Only enforced when Hibernate generates the DDL |
| No index on foreign keys / filter columns | Full table scans as data grows |
| Mutable `@Id` | Breaks identity and the persistence context |

## 31.10 TL Questions

**Q: What is an entity?**
A: A Java class mapped to a database table — one instance corresponds to one row, and Hibernate manages its persistent state.

**Q: Why must entities have a no-arg constructor and not be final?**
A: Hibernate instantiates entities reflectively when loading rows, so it needs a no-arg constructor, and it creates proxy subclasses for lazy loading, which is impossible on a final class. This is why Lombok's `@Builder` on its own breaks entities — it removes the default constructor.

**Q: Why `EnumType.STRING`?**
A: `ORDINAL` stores the enum's index. If anyone inserts a new constant in the middle, every existing row silently changes meaning — corrupt data with no error. `STRING` stores the name, so it's stable and readable in the database.

**Q: Why don't we use `@Data` on entities?**
A: It generates `equals`/`hashCode` over all fields, which breaks `HashSet` behaviour when a field changes and triggers lazy loading just to compare. Its `toString` also loads lazy associations and can recurse infinitely on bidirectional relationships.

**Q: How do we track who changed a record?**
A: JPA auditing — a `@MappedSuperclass` with `@CreatedDate`, `@LastModifiedDate`, `@CreatedBy` and `@LastModifiedBy`, populated automatically from the security context via an `AuditorAware` bean.

**Q: Why `BigDecimal` for money?**
A: `double` can't represent decimal fractions exactly, so amounts drift over repeated arithmetic. `BigDecimal` with explicit precision and scale is exact, which is mandatory for financial data.

**Q: What does `@Version` do?**
A: Optimistic locking. Hibernate increments it on each update and includes the old value in the WHERE clause. If another transaction changed the row meanwhile, zero rows match and we get an `OptimisticLockException` instead of silently overwriting their change.

## 31.11 Interview Questions

**Beginner — What does `@Transient` do?**
Excludes a field from persistence — it exists in the object but has no column.

**Beginner — What is `@MappedSuperclass`?**
A class whose mapped fields are inherited as columns by subclasses, but which has no table and cannot be queried itself.

**Intermediate — `IDENTITY` vs `SEQUENCE`?**
`IDENTITY` uses the database's auto-increment and requires an immediate INSERT to obtain the key, which **disables JDBC batch inserts**. `SEQUENCE` pre-fetches IDs from a sequence, allowing batching and better bulk performance. MySQL supports only `IDENTITY` practically; Postgres and Oracle favour `SEQUENCE`.

**Intermediate — Why is a constant `hashCode()` recommended for entities?**
Because the ID is null before persist and populated after. An ID-based hash changes on save, so an entity placed in a `HashSet` before saving cannot be found afterwards. A constant hash keeps bucket placement stable while `equals` still distinguishes instances.

**Advanced — `@Embeddable` vs `@OneToOne`?**
`@Embeddable` maps a value object into the **same table** — no join, no separate identity, lifecycle tied to the owner. `@OneToOne` is a separate table and entity with its own identity and lifecycle. Use embeddable for value semantics like an address, and a relationship for something with independent identity.

**Advanced — What are the entity inheritance strategies?**
`SINGLE_TABLE` (one table plus a discriminator — fast, but subclass columns must be nullable), `JOINED` (one table per class joined by PK — normalised, requires joins), and `TABLE_PER_CLASS` (one table per concrete class — polymorphic queries become UNIONs and perform poorly).

## 31.12 Quick Revision

1. `@Entity` + `@Id` + no-arg constructor + non-final class.
2. **`@Enumerated(EnumType.STRING)` always.**
3. `BigDecimal` with `precision`/`scale` for money.
4. Never `@Data` on entities — write `equals`/`hashCode` deliberately.
5. Constant `hashCode()`; `equals` on a business key.
6. `IDENTITY` blocks batch inserts; `SEQUENCE` with `allocationSize` enables them.
7. `@Version` = optimistic locking.
8. `@MappedSuperclass` + `@EnableJpaAuditing` for audit columns.
9. `@Embeddable` for value objects in the same table.
10. Index foreign keys and frequent filter columns.
11. `@Transient` excludes a field from persistence.

## 31.13 TL Explanation (speak this)

> "Entities are our table mappings — one class per table, managed by Hibernate. A few conventions we hold to: enums are always stored as `STRING` rather than ordinal, because an ordinal silently corrupts every existing row if someone inserts a constant in the middle; money is always `BigDecimal` with explicit precision; and we never put Lombok's `@Data` on an entity because its generated `equals` and `toString` trigger lazy loading and break `HashSet` behaviour. Audit columns come from a shared `@MappedSuperclass` with JPA auditing, so we always know who changed a record and when, and `@Version` gives us optimistic locking so concurrent updates fail loudly instead of overwriting each other."

---

[← 30. JPA vs Hibernate vs Spring Data JPA](30-jpa-vs-hibernate-vs-spring-data-jpa.md) | [Contents](../README.md) | [32. EntityManager and Persistence Context →](32-entitymanager-and-persistence-context.md)
