[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 27. JDBC and DataSource

## 27.1 What is JDBC?

**Simple words:**
JDBC (Java Database Connectivity) is the **standard Java API for talking to a database**. It defines interfaces like `Connection`, `Statement` and `ResultSet`. Each database vendor writes a **driver** that implements them.

That's why your Java code looks the same for MySQL and PostgreSQL — you only swap the driver.

**Technical wording:**
JDBC is a Java standard API providing vendor-neutral database access. Vendor drivers implement `java.sql` interfaces, and `DriverManager`/`DataSource` supplies `Connection` objects.

## 27.2 Raw JDBC — and why nobody writes it anymore

```java
public Order findById(Long id) {
    Connection conn = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        conn = dataSource.getConnection();
        ps = conn.prepareStatement("SELECT id, customer_id, amount FROM orders WHERE id = ?");
        ps.setLong(1, id);
        rs = ps.executeQuery();
        if (rs.next()) {
            Order order = new Order();
            order.setId(rs.getLong("id"));
            order.setCustomerId(rs.getString("customer_id"));
            order.setAmount(rs.getBigDecimal("amount"));
            return order;
        }
        return null;
    } catch (SQLException e) {
        throw new RuntimeException(e);            // checked exception forced on us
    } finally {
        // ~15 lines of nested null-checked closing — and leaking ANY of these
        // eventually exhausts the connection pool and takes the service down
        if (rs   != null) try { rs.close();   } catch (SQLException ignored) {}
        if (ps   != null) try { ps.close();   } catch (SQLException ignored) {}
        if (conn != null) try { conn.close(); } catch (SQLException ignored) {}
    }
}
```

**Roughly 25 lines for one simple query.** Problems: enormous boilerplate, manual resource management (leaks kill production), checked `SQLException` everywhere, vendor-specific error codes, and manual `ResultSet` → object mapping.

**Spring's answer, at three levels of abstraction:**

```
Raw JDBC          → you manage everything
   ↓
JdbcTemplate      → Spring manages connections/exceptions; you write SQL
   ↓
Spring Data JPA   → Spring generates SQL too; you write method names
```

## 27.3 DataSource

**Simple words:**
A `DataSource` is a **factory for database connections**. Instead of opening a new connection every time (slow — 50–100ms of TCP handshake and authentication), it hands you one from a **pool** of already-open connections.

**Technical wording:**
`javax.sql.DataSource` is the standard abstraction for obtaining `Connection` objects, typically backed by a connection pool that manages creation, reuse, validation and eviction.

### Why connection pooling is essential

```
WITHOUT POOLING — per request:
  open TCP socket    ~20ms
  authenticate       ~30ms
  run query           ~2ms   ← the only useful part
  close connection    ~5ms
  ───────────────────────────
  TOTAL              ~57ms   for 2ms of actual work

WITH POOLING — per request:
  borrow from pool   ~0.1ms
  run query           ~2ms
  return to pool     ~0.1ms
  ───────────────────────────
  TOTAL              ~2.2ms   → roughly 25x faster
```

Pooling also **caps** the number of connections, protecting the database. A database has a hard connection limit (MySQL's default `max_connections` is 151); without a pool, a traffic spike opens thousands of connections and the database refuses everything — including your monitoring.

### Configuration

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/orderdb
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
Boot's `DataSourceAutoConfiguration` sees these plus a driver on the classpath and creates a **HikariCP** `DataSource`. The driver class name is usually auto-detected from the URL.

## 27.4 TL Questions

**Q: What is JDBC?**
A: The standard Java API for database access. Vendors supply drivers implementing it, so our code is portable across databases.

**Q: Why don't we write raw JDBC?**
A: It's about 25 lines for a simple query, all of it resource management that leaks the connection pool if you get it wrong, plus checked exceptions and manual result mapping. Spring's `JdbcTemplate` or Spring Data JPA removes all of that.

**Q: What is a DataSource and why do we need one?**
A: A factory for connections, backed by a pool. Opening a connection costs 50+ milliseconds, so reusing pooled connections is roughly 25 times faster. It also caps concurrent connections so a traffic spike doesn't exhaust the database's connection limit.

**Q: What happens if we don't close connections?**
A: They're never returned to the pool. Eventually the pool is exhausted and every request blocks waiting for a connection until it times out — the service appears completely hung while the database itself is fine.

## 27.5 Interview Questions

**Beginner — What does a JDBC driver do?**
Implements the `java.sql` interfaces for a specific database, translating standard calls into that database's wire protocol.

**Intermediate — `Statement` vs `PreparedStatement`?**
`Statement` sends raw SQL — vulnerable to SQL injection and reparsed every time. `PreparedStatement` uses bind parameters, so it's **injection-safe**, can be cached and reused by the database, and handles type conversion. Always use `PreparedStatement`.

**Advanced — What is `DriverManager` vs `DataSource`?**
`DriverManager` is the old static factory that opens a fresh connection each call, with no pooling. `DataSource` is the modern abstraction, supports pooling, distributed transactions and JNDI lookup, and is what Spring uses.

## 27.6 Quick Revision

1. JDBC = standard Java DB API; drivers implement it per vendor.
2. Raw JDBC = heavy boilerplate and leak risk — don't write it.
3. `DataSource` = connection factory, normally pool-backed.
4. Pooling is ~25x faster and protects the DB's connection limit.
5. Always use `PreparedStatement` — SQL injection safety.
6. Boot auto-configures a HikariCP `DataSource` from `spring.datasource.*`.
7. Leaked connections exhaust the pool and hang the service.

## 27.7 TL Explanation (speak this)

> "JDBC is the standard API underneath everything we do with the database — Hibernate and `JdbcTemplate` both sit on top of it. We never write raw JDBC because it's a lot of manual resource management and a single missed `close()` leaks a connection until the pool is exhausted and the service hangs. The `DataSource` is our connection factory, backed by a HikariCP pool, which is roughly twenty-five times faster than opening a connection per request and also caps how many connections we can open so we don't overwhelm the database."

---

[← 26. AOP (Aspect Oriented Programming)](../part-b-web-layer/26-aop-aspect-oriented-programming.md) | [Contents](../README.md) | [28. HikariCP (Connection Pooling) →](28-hikaricp-connection-pooling.md)
