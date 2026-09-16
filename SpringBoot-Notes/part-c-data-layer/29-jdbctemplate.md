[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 29. JdbcTemplate

## 29.1 What is it?

**Simple words:**
`JdbcTemplate` is Spring's helper that removes all the JDBC boilerplate. **You write the SQL; Spring handles the connection, statement, result set, closing and exception translation.**

**Technical wording:**
`JdbcTemplate` is Spring's central JDBC abstraction implementing the template method pattern. It manages resource acquisition and release, statement creation and execution, and translates `SQLException` into Spring's `DataAccessException` hierarchy.

## 29.2 The same query, with JdbcTemplate

```java
@Repository
@RequiredArgsConstructor
public class OrderJdbcRepository {

    private final JdbcTemplate jdbcTemplate;         // auto-configured by Boot

    public Order findById(Long id) {
        String sql = "SELECT id, customer_id, amount FROM orders WHERE id = ?";
        return jdbcTemplate.queryForObject(sql, orderRowMapper(), id);
    }

    private RowMapper<Order> orderRowMapper() {
        return (rs, rowNum) -> Order.builder()
                .id(rs.getLong("id"))
                .customerId(rs.getString("customer_id"))
                .amount(rs.getBigDecimal("amount"))
                .build();
    }
}
```
**25 lines became 5.** No connection handling, no `finally`, no checked exceptions.

## 29.3 Common operations

```java
// Query a list
List<Order> orders = jdbcTemplate.query(
        "SELECT * FROM orders WHERE status = ?", orderRowMapper(), status);

// Single value
Integer count = jdbcTemplate.queryForObject(
        "SELECT COUNT(*) FROM orders WHERE customer_id = ?", Integer.class, customerId);

// Insert / update / delete — returns rows affected
int rows = jdbcTemplate.update(
        "INSERT INTO orders (customer_id, amount, status) VALUES (?, ?, ?)",
        customerId, amount, status.name());

// Insert and retrieve the generated key
KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.update(connection -> {
    PreparedStatement ps = connection.prepareStatement(
            "INSERT INTO orders (customer_id, amount) VALUES (?, ?)",
            Statement.RETURN_GENERATED_KEYS);
    ps.setString(1, customerId);
    ps.setBigDecimal(2, amount);
    return ps;
}, keyHolder);
Long generatedId = keyHolder.getKey().longValue();

// Batch update — MUCH faster than a loop of single updates
jdbcTemplate.batchUpdate(
        "INSERT INTO order_items (order_id, product_id, qty) VALUES (?, ?, ?)",
        items, 500,                                  // batch size
        (ps, item) -> {
            ps.setLong(1, item.getOrderId());
            ps.setLong(2, item.getProductId());
            ps.setInt(3, item.getQuantity());
        });
```

### NamedParameterJdbcTemplate — more readable for many parameters

```java
@Repository
@RequiredArgsConstructor
public class OrderSearchRepository {

    private final NamedParameterJdbcTemplate namedJdbcTemplate;

    public List<Order> search(OrderSearchCriteria criteria) {
        String sql = """
                SELECT * FROM orders
                WHERE status = :status
                  AND created_at BETWEEN :fromDate AND :toDate
                  AND amount >= :minAmount
                ORDER BY created_at DESC
                """;

        MapSqlParameterSource params = new MapSqlParameterSource()
                .addValue("status", criteria.getStatus().name())
                .addValue("fromDate", criteria.getFromDate())
                .addValue("toDate", criteria.getToDate())
                .addValue("minAmount", criteria.getMinAmount());

        return namedJdbcTemplate.query(sql, params, orderRowMapper());
    }
}
```
With five positional `?` markers it's easy to swap two arguments and introduce a silent bug. **Named parameters remove that class of error** — prefer them beyond about three parameters.

## 29.4 Exception translation

```java
try {
    jdbcTemplate.update("INSERT INTO customers (email) VALUES (?)", email);
} catch (DuplicateKeyException e) {          // Spring's unchecked exception
    throw new DuplicateResourceException("email", email);
}
```
`@Repository` activates exception translation, converting vendor-specific `SQLException`s (MySQL error 1062, Postgres 23505) into a consistent hierarchy:

```
DataAccessException                        (unchecked root)
├── DataIntegrityViolationException
│     └── DuplicateKeyException
├── DataAccessResourceFailureException
├── EmptyResultDataAccessException
├── IncorrectResultSizeDataAccessException
└── OptimisticLockingFailureException
```
**The benefit:** your service layer catches `DuplicateKeyException` regardless of the database vendor. Switching from MySQL to Postgres doesn't change your error-handling code.

## 29.5 JdbcTemplate vs JPA — when to use which

| | JdbcTemplate | Spring Data JPA |
|---|---|---|
| SQL | **You write it** | Generated (or JPQL/native) |
| Control over SQL | **Total** | Indirect |
| Boilerplate | Moderate (RowMappers) | Minimal |
| Object mapping | Manual | Automatic |
| Caching, dirty checking, lazy loading | No | Yes |
| Learning curve | Low | Higher |
| Performance predictability | **High — what you write is what runs** | Lower (N+1 risk) |
| Best for | Complex reports, bulk operations, legacy schemas, precise tuning | **CRUD and domain-driven persistence** |

**They coexist happily.** A very common and sensible real-world setup: **Spring Data JPA for CRUD, `JdbcTemplate` for complex reporting queries and bulk operations.** Using both is a sign of good judgement, not inconsistency — worth saying explicitly in an interview.

## 29.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| **String-concatenating SQL** | **SQL injection.** Always use `?` or named parameters. |
| `queryForObject` when no row matches | Throws `EmptyResultDataAccessException` — catch it or use `query().stream().findFirst()` |
| Looping single inserts instead of `batchUpdate` | Orders of magnitude slower |
| `SELECT *` | Breaks when columns change; fetches unneeded data |
| Forgetting `@Repository` | No exception translation |
| Building dynamic SQL by concatenation | Injection risk and unreadable — use `NamedParameterJdbcTemplate` with conditional clauses |

**The injection point, explicitly:**
```java
// ❌ CATASTROPHIC — never do this
String sql = "SELECT * FROM users WHERE email = '" + email + "'";
// email = "x' OR '1'='1" returns every user.

// ✅ Correct — the driver binds the value, never parses it as SQL
jdbcTemplate.query("SELECT * FROM users WHERE email = ?", mapper, email);
```

## 29.7 TL Questions

**Q: What is `JdbcTemplate` and why use it?**
A: Spring's JDBC abstraction. It manages connections, statements, result sets and closing, and translates SQL exceptions into Spring's unchecked hierarchy. We write only the SQL and a row mapper.

**Q: When do we use it instead of JPA?**
A: For complex reporting queries, bulk operations, and anywhere we need exact control of the SQL. JPA is better for CRUD on our domain entities. We use both in the same service, which is a normal and deliberate split.

**Q: How does exception translation help us?**
A: It gives vendor-independent exceptions. We catch `DuplicateKeyException` whether the database is MySQL or Postgres, so our error handling doesn't need to know vendor error codes. It's enabled by `@Repository`.

**Q: How do we prevent SQL injection?**
A: Always bind values as parameters, never concatenate them into the SQL string. The driver sends the statement and values separately, so user input is never parsed as SQL.

**Q: Why use `NamedParameterJdbcTemplate`?**
A: Readability and safety once there are more than about three parameters. With positional markers it's easy to transpose two arguments and get a bug that compiles and runs.

## 29.8 Interview Questions

**Beginner — What does `JdbcTemplate` remove?**
Connection and resource management, statement creation, result set iteration, closing, and checked exception handling.

**Intermediate — What is a `RowMapper`?**
A callback mapping one `ResultSet` row to an object. `RowMapper` maps row by row; `ResultSetExtractor` receives the whole `ResultSet` (useful for aggregating a join into a single object graph).

**Intermediate — What happens if `queryForObject` returns no rows?**
It throws `EmptyResultDataAccessException`. To treat "not found" as normal, use `query(...)` and take the first element, or catch the exception.

**Advanced — What is `DataAccessException` and why is it unchecked?**
Spring's consistent, vendor-neutral data-access exception hierarchy. It is unchecked because most data-access failures are not recoverable at the call site, so forcing every caller to catch them would only add noise — and unchecked exceptions also trigger Spring's default transaction rollback.

**Advanced — How much faster is `batchUpdate`?**
Substantially — often ten times or more for large inserts, because it sends many statements in one round trip instead of paying network latency per row. With JDBC batching properly enabled, the difference grows with row count.

## 29.9 Quick Revision

1. `JdbcTemplate` = SQL you write, plumbing Spring handles.
2. Key methods: `query`, `queryForObject`, `update`, `batchUpdate`.
3. `RowMapper` maps a row to an object.
4. `@Repository` enables translation to `DataAccessException`.
5. `DataAccessException` is **unchecked** and vendor-neutral.
6. **Always bind parameters** — never concatenate SQL.
7. `NamedParameterJdbcTemplate` for readability with many parameters.
8. `batchUpdate` for bulk work — dramatically faster.
9. Mixing JPA (CRUD) and `JdbcTemplate` (reports/bulk) is good practice.

## 29.10 TL Explanation (speak this)

> "`JdbcTemplate` is Spring's JDBC wrapper — it handles the connection, statement, result set and closing, so we just write SQL and a row mapper. We use it alongside JPA: JPA for CRUD on our entities, `JdbcTemplate` for reporting queries and bulk inserts where we want exact control of the SQL and predictable performance. It also gives us Spring's `DataAccessException` hierarchy, so we catch `DuplicateKeyException` without caring whether we're on MySQL or Postgres. And we always bind parameters rather than concatenating, so SQL injection isn't possible."

---

[← 28. HikariCP (Connection Pooling)](28-hikaricp-connection-pooling.md) | [Contents](../README.md) | [30. JPA vs Hibernate vs Spring Data JPA →](30-jpa-vs-hibernate-vs-spring-data-jpa.md)
