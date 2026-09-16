[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 30. JPA vs Hibernate vs Spring Data JPA

## 30.1 The three things people confuse

This is one of the most commonly asked and most commonly muddled questions. Get it precisely right.

```
┌──────────────────────────────────────────────────────────────┐
│  SPRING DATA JPA                                              │
│  A Spring project. Generates repository implementations       │
│  from interfaces. You write findByCustomerId(); it writes     │
│  the query.                                                   │
│  ──────────────────────────────────────────────────────────  │
│                        uses ▼                                 │
│  JPA  (Jakarta Persistence API)                               │
│  A SPECIFICATION — just interfaces and annotations.           │
│  @Entity, @Id, EntityManager, JPQL. It has NO code that runs. │
│  ──────────────────────────────────────────────────────────  │
│                   implemented by ▼                            │
│  HIBERNATE                                                    │
│  The actual IMPLEMENTATION (ORM provider). Contains the real  │
│  logic: SQL generation, caching, dirty checking, lazy loading.│
│  ──────────────────────────────────────────────────────────  │
│                        uses ▼                                 │
│  JDBC → Driver → DATABASE                                     │
└──────────────────────────────────────────────────────────────┘
```

**The analogy that makes it stick:**
- **JPA** = the *rules of the game* (the specification)
- **Hibernate** = a *team that plays by those rules* (an implementation)
- **Spring Data JPA** = a *coach who plays the game for you* (a convenience layer)

## 30.2 Comparison table

| Aspect | JPA | Hibernate | Spring Data JPA |
|---|---|---|---|
| **What it is** | Specification | ORM implementation | Abstraction layer |
| **Contains runnable code?** | **No — interfaces only** | **Yes** | Yes |
| **Provided by** | Jakarta EE | Red Hat | Spring (Pivotal/VMware) |
| **Key artifacts** | `@Entity`, `EntityManager`, JPQL | `Session`, `SessionFactory`, HQL, Criteria | `JpaRepository`, query methods |
| **Can be used alone?** | No — needs a provider | **Yes** | No — needs JPA + a provider |
| **Extra features** | — | Caching, `@BatchSize`, filters, multi-tenancy, Envers | Derived queries, paging, auditing, specifications |
| **Vendor lock-in** | None | Hibernate-specific APIs lock you in | None (JPA-based) |

**Alternative JPA providers:** EclipseLink (the reference implementation), OpenJPA. Hibernate is by far the most used, and is what Spring Boot's JPA starter includes.

## 30.3 The same operation at each level

```java
// ---------- Level 1: Pure JPA (EntityManager) ----------
@Repository
public class OrderRepositoryJpa {
    @PersistenceContext
    private EntityManager entityManager;

    public Order findById(Long id) {
        return entityManager.find(Order.class, id);
    }

    public List<Order> findByStatus(OrderStatus status) {
        return entityManager.createQuery(
                    "SELECT o FROM Order o WHERE o.status = :status", Order.class)
                .setParameter("status", status)
                .getResultList();
    }

    @Transactional
    public Order save(Order order) {
        if (order.getId() == null) { entityManager.persist(order); return order; }
        return entityManager.merge(order);
    }
}

// ---------- Level 2: Hibernate-specific (Session) ----------
// Works, but couples you to Hibernate. Only do this for Hibernate-only features.
Session session = entityManager.unwrap(Session.class);
Order order = session.get(Order.class, id);

// ---------- Level 3: Spring Data JPA ----------
public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByStatus(OrderStatus status);        // implementation GENERATED
}
// That's it. No implementation class at all.
```

**The progression is the point:** each level removes code. Spring Data JPA removes essentially all of it for standard operations, while still letting you drop down to `EntityManager` or native SQL when you need to.

## 30.4 What is ORM, and its trade-offs

**ORM (Object-Relational Mapping)** bridges two different worlds:

| Java (objects) | Relational database |
|---|---|
| Objects with references | Rows with foreign keys |
| Inheritance | No native inheritance |
| Collections | Join tables |
| Identity by reference | Identity by primary key |
| Navigate with `.` | Navigate with JOINs |

This mismatch is called the **object-relational impedance mismatch**, and ORM exists to bridge it.

**A balanced view — state both sides to a TL:**

| ORM advantages | ORM disadvantages |
|---|---|
| Far less boilerplate | Hides the generated SQL |
| Database-portable | Performance traps (N+1, over-fetching) |
| Caching, dirty checking | Steep learning curve |
| Type-safe domain model | Complex queries can be awkward |
| Automatic relationship handling | Debugging requires understanding internals |

**The honest position:** *"ORM is excellent for CRUD on a well-modelled domain and poor for complex reporting. We use JPA for the domain and `JdbcTemplate` or native queries for reports."* That answer shows judgement rather than dogma.

## 30.5 Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```
The starter brings Spring Data JPA, **Hibernate**, HikariCP, and `spring-tx`.

```properties
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.open-in-view=false
spring.jpa.properties.hibernate.jdbc.batch_size=50
```

### `ddl-auto` — get this right

| Value | Behaviour | Use in |
|---|---|---|
| `none` | Do nothing | Production (with migrations) |
| `validate` | **Verify the schema matches the entities; fail if not** | **Production** |
| `update` | Alter the schema to match entities | Local dev only |
| `create` | Drop and create at startup | Tests |
| `create-drop` | Create at startup, drop at shutdown | Tests |

**Never use `update` in production.** It never drops columns, can't do renames properly, produces unpredictable DDL, and gives no migration history or rollback. **Use Flyway or Liquibase for schema changes and set `ddl-auto=validate`** so the app refuses to start if the schema and entities disagree — catching a missed migration at deploy time rather than at runtime.

## 30.6 TL Questions

**Q: What's the difference between JPA and Hibernate?**
A: JPA is a specification — just interfaces and annotations, with no executable logic. Hibernate is an implementation of that specification and contains the actual code that generates SQL, manages caching and does dirty checking. We code against JPA so we could swap providers, and Hibernate does the work underneath.

**Q: Where does Spring Data JPA fit?**
A: It's a layer above JPA that generates repository implementations from our interfaces. We declare `findByStatus` and it builds the query. It doesn't replace JPA or Hibernate — it uses both.

**Q: Why not use Hibernate's `Session` directly?**
A: It locks us to Hibernate. The JPA `EntityManager` covers nearly everything, and we can always `unwrap` to `Session` for the rare Hibernate-only feature. Keeping to the standard API keeps our options open at no cost.

**Q: What `ddl-auto` do we use and why?**
A: `validate` in production. Hibernate checks the schema matches our entities and refuses to start otherwise, so a missed migration fails at deploy time. Schema changes go through Flyway, never through `update` — which never drops columns and gives no migration history or rollback.

**Q: When is ORM the wrong tool?**
A: Complex reporting with multi-table aggregation, and bulk operations. There we use native SQL or `JdbcTemplate`, because ORM hides the SQL and it's easy to generate something inefficient without noticing.

## 30.7 Interview Questions

**Beginner — Is Hibernate a framework or a specification?**
An implementation (ORM framework). JPA is the specification.

**Beginner — Which JPA provider does Spring Boot use?**
Hibernate, included in `spring-boot-starter-data-jpa`.

**Intermediate — Can we use Hibernate without JPA?**
Yes — through its native `SessionFactory`/`Session` API, which predates JPA. Modern projects use the JPA API for portability.

**Intermediate — JPQL vs HQL vs native SQL?**
JPQL is the JPA standard query language, operating on **entities and fields**. HQL is Hibernate's superset with extra features. Native SQL is raw, database-specific SQL run through the same API. Prefer JPQL; use native when you need vendor features.

**Advanced — What is the object-relational impedance mismatch?**
The structural differences between object models and relational models: inheritance, collections, associations, and identity semantics have no direct relational equivalents. ORM maps between them, and most ORM complexity and performance traps arise from this mapping.

**Advanced — Why avoid `ddl-auto=update` in production?**
It never removes columns or constraints, handles renames as add+orphan, produces vendor-dependent DDL, has no versioning or rollback, and can lock large tables unpredictably during deployment. Versioned migrations with Flyway or Liquibase plus `validate` is the correct approach.

## 30.8 Quick Revision

1. **JPA = specification. Hibernate = implementation. Spring Data JPA = abstraction on top.**
2. JPA has no runnable code — only interfaces and annotations.
3. Hibernate does SQL generation, caching, dirty checking, lazy loading.
4. Spring Data JPA generates repository implementations from interfaces.
5. Prefer the JPA API; `unwrap` to Hibernate only for provider-specific features.
6. ORM bridges the **object-relational impedance mismatch**.
7. ORM is great for CRUD, weak for complex reporting — mix in `JdbcTemplate`.
8. **`ddl-auto=validate` in production**; Flyway/Liquibase for schema changes.
9. `spring.jpa.open-in-view=false` — always.

## 30.9 TL Explanation (speak this)

> "JPA is just a specification — annotations and interfaces with no logic in them. Hibernate is the implementation that actually generates the SQL and manages caching and dirty checking, and Spring Data JPA sits on top generating our repository implementations from interface method names. We code against the JPA API rather than Hibernate's `Session` so we're not locked to a provider. In production we run `ddl-auto=validate` with Flyway for migrations, so the app refuses to start if the schema and entities disagree — that catches a missed migration at deploy time. And for complex reports we drop to native SQL or `JdbcTemplate`, because ORM is strong for CRUD and weak for aggregation-heavy queries."

---

[← 29. JdbcTemplate](29-jdbctemplate.md) | [Contents](../README.md) | [31. Entity and Mapping Annotations →](31-entity-and-mapping-annotations.md)
