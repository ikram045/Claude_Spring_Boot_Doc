[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 28. HikariCP (Connection Pooling)

## 28.1 What is it?

**Simple words:**
HikariCP is the **connection pool** Spring Boot uses by default. It keeps a set of open database connections ready and hands them out.

**Technical wording:**
HikariCP is a high-performance JDBC connection pool, the default `DataSource` implementation in Spring Boot since 2.0, chosen for its low latency, small footprint and correctness under concurrency.

## 28.2 Why HikariCP?

| Pool | Notes |
|---|---|
| **HikariCP** | **Boot default.** Fastest, smallest, zero-overhead design |
| Tomcat JDBC | Previous Boot default |
| Apache DBCP2 | Older, heavier |
| C3P0 | Legacy |

Boot selects it automatically if it's on the classpath — which it is, because `spring-boot-starter-data-jpa` and `spring-boot-starter-jdbc` both bring it in.

## 28.3 Configuration — and what each setting actually means

```properties
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
spring.datasource.hikari.pool-name=OrderServiceHikariPool
spring.datasource.hikari.leak-detection-threshold=60000
```

| Property | Default | Meaning | Practical guidance |
|---|---|---|---|
| `maximum-pool-size` | 10 | **Max connections in the pool** | The most important setting — see below |
| `minimum-idle` | = max | Idle connections kept ready | Set equal to max for steady load, to avoid churn |
| `connection-timeout` | 30s | How long a thread waits for a connection before failing | Lower it (e.g. 5s) to fail fast rather than pile up |
| `idle-timeout` | 10m | Idle connection eviction time | Only applies if `minimum-idle < maximum-pool-size` |
| `max-lifetime` | 30m | Max age of a connection | **Must be shorter than the DB/firewall idle timeout** |
| `leak-detection-threshold` | 0 (off) | Log a warning if a connection is held this long | Set to 60s in non-prod to find leaks |

### Pool sizing — the counter-intuitive part

**Bigger is not better.** A common mistake is setting `maximum-pool-size=100` "for performance". That usually makes things *worse*: the database context-switches between 100 connections, contends on locks and I/O, and throughput drops.

HikariCP's own guidance is a formula based on the database server:

```
pool size = (CPU cores × 2) + effective spindle count
```

For a typical 4-core database server with SSD storage, that's roughly **10** — which is exactly the default.

**The sizing constraint people forget:**
```
total connections = pool size × number of application instances
```
With 10 pods each using `maximum-pool-size=20`, you need **200** database connections. If MySQL's `max_connections` is 151, your deployment fails at scale — and the failure appears only under load. **This is worth raising in any capacity-planning discussion.**

### `max-lifetime` and the firewall problem

If your database or a network firewall silently drops idle connections after, say, 10 minutes, but Hikari believes them valid for 30 minutes, you get intermittent, unexplainable `Connection is closed` errors under low traffic. **Set `max-lifetime` a few seconds below the shortest idle timeout in the network path.** This is a classic hard-to-diagnose production issue.

## 28.4 Monitoring

```properties
management.endpoints.web.exposure.include=health,metrics
management.metrics.enable.hikaricp=true
```
Key metrics: `hikaricp.connections.active`, `.idle`, `.pending`, `.usage`, `.timeout`.

**`hikaricp.connections.pending` is the alert to set.** A non-zero pending count means threads are queuing for a connection — the pool is too small, queries are too slow, or connections are leaking. It's the earliest warning before user-visible timeouts.

## 28.5 Common Mistakes

| Mistake | Consequence |
|---|---|
| Pool size far too large | Database thrashes; throughput drops |
| `pool size × instances` > DB `max_connections` | Failures at scale only |
| `max-lifetime` longer than the DB/firewall idle timeout | Intermittent "connection closed" errors |
| Long-running transactions | Connections held far longer than needed; pool starved |
| `open-in-view` left enabled | **Connection held for the whole HTTP request, including view rendering** |
| No leak detection in non-prod | Leaks only found in production |

## 28.6 TL Questions

**Q: What connection pool do we use?**
A: HikariCP — it's Boot's default since 2.0 and the fastest available. It comes in automatically with the JDBC and JPA starters.

**Q: How do we size the pool?**
A: Small and deliberate, roughly `(cores × 2)` on the database server, which lands near the default of 10. The constraint people miss is that total connections equal pool size times instance count, so ten pods at twenty each needs two hundred database connections — more than MySQL's default limit.

**Q: Why isn't a bigger pool better?**
A: Because the database can only do so much work concurrently. More connections mean more context switching and lock contention, so throughput actually drops while latency rises.

**Q: What happens when the pool is exhausted?**
A: Threads block waiting for a connection and fail after `connection-timeout` with `SQLTransientConnectionException`. The application looks hung even though the database is healthy — which is why we alert on the pending-connections metric.

**Q: What's `max-lifetime` for?**
A: It retires connections before something else in the network closes them. If the firewall drops idle connections after ten minutes but Hikari keeps them for thirty, we get intermittent "connection closed" errors. It must be set below the shortest idle timeout in the path.

**Q: How do we find a connection leak?**
A: Enable `leak-detection-threshold` in non-production — Hikari logs a stack trace for any connection held longer than the threshold, pointing straight at the offending code.

## 28.7 Interview Questions

**Beginner — What is the default connection pool in Spring Boot?**
HikariCP, since Boot 2.0.

**Intermediate — What does `connection-timeout` control?**
How long a thread waits to borrow a connection before an exception is thrown. It is not a query timeout.

**Advanced — Why does a larger pool often reduce throughput?**
The database's real concurrency is bounded by cores and disk. Beyond that, extra connections add context switching, cache thrashing and lock contention. Queuing at the pool is cheaper than queuing inside the database.

**Advanced — How does `open-in-view` affect the pool?**
With it enabled, the Hibernate session and its connection stay open for the entire request, including response serialization. Connections are held far longer than the actual database work, so the pool is exhausted at much lower load. It should be disabled.

## 28.8 Quick Revision

1. HikariCP = Boot's default pool; fastest available.
2. `maximum-pool-size` default 10 — usually right; bigger is often worse.
3. Formula: `(cores × 2) + spindles`.
4. **Total connections = pool size × instances** — check against DB `max_connections`.
5. `max-lifetime` must be below the DB/firewall idle timeout.
6. Alert on `hikaricp.connections.pending`.
7. Use `leak-detection-threshold` in non-prod.
8. Long transactions and `open-in-view` starve the pool.

## 28.9 TL Explanation (speak this)

> "HikariCP is our connection pool — Boot's default and the fastest one available. We keep the pool small, around ten, because a bigger pool usually reduces throughput: the database can only do so much concurrently, and extra connections just add contention. The number we watch in capacity planning is pool size times instance count, since ten pods at twenty connections each already exceeds MySQL's default limit. We set `max-lifetime` below the firewall's idle timeout to avoid intermittent closed-connection errors, and we alert on the pending-connections metric because that's the earliest sign the pool is too small or something is leaking."

---

[← 27. JDBC and DataSource](27-jdbc-and-datasource.md) | [Contents](../README.md) | [29. JdbcTemplate →](29-jdbctemplate.md)
