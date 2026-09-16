[← Back to Contents](../README.md) · Part C — Data Layer (JDBC / JPA / Hibernate)

---

# 41. Transactions and @Transactional

## 41.1 What is it?

**Simple words:**
A transaction groups several database operations into **one unit that either fully succeeds or fully fails**. If any step fails, everything is undone.

The classic example: transfer money. Debit account A, credit account B. If the credit fails after the debit succeeded, the money has **vanished**. A transaction guarantees both happen or neither does.

**Technical wording:**
A transaction is a unit of work with ACID guarantees. Spring provides declarative transaction management through `@Transactional`, implemented via AOP proxies delegating to a `PlatformTransactionManager`.

## 41.2 ACID

| Property | Meaning | Example |
|---|---|---|
| **Atomicity** | All operations succeed or all are rolled back | Debit and credit both happen, or neither |
| **Consistency** | The DB moves from one valid state to another | Constraints are never violated |
| **Isolation** | Concurrent transactions don't corrupt each other | Two simultaneous transfers don't interleave incorrectly |
| **Durability** | Committed data survives a crash | Written to disk/WAL before commit returns |

## 41.3 Basic usage

```java
@Service
@RequiredArgsConstructor
public class PaymentService {

    private final AccountRepository accountRepository;
    private final TransactionLogRepository logRepository;

    @Transactional
    public void transfer(Long fromId, Long toId, BigDecimal amount) {

        Account from = accountRepository.findById(fromId)
                .orElseThrow(() -> new ResourceNotFoundException("Account", fromId));
        Account to = accountRepository.findById(toId)
                .orElseThrow(() -> new ResourceNotFoundException("Account", toId));

        if (from.getBalance().compareTo(amount) < 0) {
            // RuntimeException → automatic rollback
            throw new InsufficientBalanceException(amount, from.getBalance());
        }

        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));
        // No save() needed — dirty checking handles it

        logRepository.save(new TransactionLog(fromId, toId, amount));

        // Commit here. If ANY step threw, everything above is rolled back.
    }
}
```

## 41.4 How it works internally — AOP again

```
Caller calls paymentService.transfer(...)
              │
              ▼
   ╔══════════════════════════════════════════════════╗
   ║  TRANSACTIONAL PROXY (created by AOP)            ║
   ║                                                  ║
   ║  1. TransactionInterceptor intercepts the call   ║
   ║  2. PlatformTransactionManager.getTransaction()  ║
   ║       → obtains a Connection from the pool       ║
   ║       → connection.setAutoCommit(false)          ║
   ║       → binds it to the current thread           ║
   ║         (TransactionSynchronizationManager)      ║
   ║  3. ─────────────► invoke the REAL method ───────╫──► your code runs
   ║                                                  ║      (same thread → same
   ║  4a. Normal return:                              ║       connection/session)
   ║        flush persistence context → commit()      ║
   ║  4b. RuntimeException:                           ║
   ║        rollback()                                ║
   ║  5. Release the connection back to the pool      ║
   ╚══════════════════════════════════════════════════╝
```

**Key insight:** the transaction is bound to the **thread**, via `TransactionSynchronizationManager`. That is how the repository, several layers down, finds the same connection and persistence context without anything being passed as a parameter. It's also why `@Async` methods do **not** join the caller's transaction — different thread, different binding.

## 41.5 Rollback rules — critical

```java
@Transactional
public void method() {
    throw new RuntimeException();      // ✅ ROLLS BACK
}

@Transactional
public void method() throws Exception {
    throw new Exception();             // ❌ COMMITS! Checked exception — no rollback
}

@Transactional(rollbackFor = Exception.class)      // ✅ now it rolls back
public void method() throws Exception {
    throw new Exception();
}

@Transactional(noRollbackFor = ValidationException.class)   // commit despite this exception
public void method() { }
```

| Exception type | Default behaviour |
|---|---|
| `RuntimeException` (unchecked) | **ROLLBACK** |
| `Error` | **ROLLBACK** |
| Checked `Exception` | **COMMIT** ← surprises everyone |

**Why this default exists:** it follows the EJB convention that checked exceptions represent *recoverable business conditions* the caller is expected to handle, while unchecked exceptions represent *system failures*. Whether you agree or not, it is the behaviour.

**The practical consequence:** this is precisely why **business exceptions must extend `RuntimeException`** (Section 23). A checked business exception would let a half-completed transaction commit — corrupt data with no error anywhere.

### The silent killer: catching exceptions inside a transaction

```java
// ❌ BROKEN — the transaction commits despite the failure
@Transactional
public void processOrder(Order order) {
    orderRepository.save(order);
    try {
        paymentService.charge(order);          // throws
    } catch (Exception e) {
        log.error("Payment failed", e);        // swallowed!
    }
    // Method returns normally → COMMIT.
    // The order is saved with NO payment. Data is now inconsistent.
}

// ✅ CORRECT — rethrow so the rollback happens
@Transactional
public void processOrder(Order order) {
    orderRepository.save(order);
    try {
        paymentService.charge(order);
    } catch (PaymentException e) {
        log.error("Payment failed for order {}", order.getId(), e);
        throw new OrderProcessingException("Payment failed", e);   // rollback
    }
}
```

**Also know `UnexpectedRollbackException`:** if an inner `REQUIRED` transaction throws and you catch it in the outer method, the inner call has already marked the shared transaction **rollback-only**. The outer method then tries to commit and Spring throws `UnexpectedRollbackException: Transaction silently rolled back`. The fix is either to rethrow, or to make the inner method `REQUIRES_NEW` so it has its own transaction.

## 41.6 Propagation — 7 types

Propagation answers: **"What should happen if a transaction already exists when this method is called?"**

| Propagation | Existing transaction | No transaction |
|---|---|---|
| **`REQUIRED`** (default) | **Join it** | **Create one** |
| **`REQUIRES_NEW`** | **Suspend it, create a new one** | Create one |
| `SUPPORTS` | Join it | Run without a transaction |
| `NOT_SUPPORTED` | Suspend it, run without | Run without |
| `MANDATORY` | Join it | **Throw exception** |
| `NEVER` | **Throw exception** | Run without |
| `NESTED` | Create a **savepoint** | Create one |

### The two you actually use

```java
// REQUIRED (default) — one transaction for the whole operation
@Transactional
public void placeOrder(Order order) {
    orderRepository.save(order);      // same transaction
    inventoryService.reserve(order);  // joins it — rollback affects both
}

// REQUIRES_NEW — must survive the outer rollback
@Service
public class AuditService {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logAttempt(String action, String outcome) {
        auditRepository.save(new AuditLog(action, outcome));
        // Commits INDEPENDENTLY. The outer transaction rolling back
        // does NOT erase this audit record.
    }
}
```

**The classic use case for `REQUIRES_NEW`:** audit logging and failure records. If the business transaction rolls back, you still want the record that it was attempted and failed. With `REQUIRED`, the rollback deletes the evidence.

**Cost warning:** `REQUIRES_NEW` uses a **second database connection** while the first is suspended. If N concurrent requests each do this, you need 2N connections. With a pool of 10, five concurrent requests can deadlock waiting for connections. **Use it deliberately, not casually.**

**`NESTED`** uses JDBC savepoints — the inner part can roll back without killing the outer transaction, and it uses the *same* connection. It's only supported by some transaction managers (`DataSourceTransactionManager` yes, JTA generally no).

## 41.7 Isolation levels

Isolation controls what one transaction can see of another's uncommitted or concurrent work.

### The three concurrency problems

| Problem | What happens |
|---|---|
| **Dirty read** | You read data another transaction wrote but hasn't committed — it may be rolled back |
| **Non-repeatable read** | You read the same row twice and get different values, because another transaction committed an UPDATE in between |
| **Phantom read** | You run the same query twice and get **different rows**, because another transaction committed an INSERT/DELETE |

### The levels

| Isolation | Dirty read | Non-repeatable | Phantom | Performance |
|---|---|---|---|---|
| `READ_UNCOMMITTED` | Possible | Possible | Possible | Fastest |
| **`READ_COMMITTED`** | Prevented | Possible | Possible | Good — **Postgres/Oracle/SQL Server default** |
| **`REPEATABLE_READ`** | Prevented | **Prevented** | Possible* | Moderate — **MySQL InnoDB default** |
| `SERIALIZABLE` | Prevented | Prevented | **Prevented** | Slowest |

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void criticalOperation() { }
```

> \* MySQL InnoDB actually prevents most phantom reads at `REPEATABLE_READ` using next-key locking, which is stricter than the SQL standard requires. **Know your database's default** — it differs between MySQL and Postgres, and assuming the wrong one causes subtle bugs.

**Practical guidance:** leave isolation at the database default (`Isolation.DEFAULT`) unless you have a demonstrated problem. Raising isolation increases locking and deadlock risk. For most contention problems, **optimistic locking with `@Version` is a better answer than a higher isolation level**, because it doesn't hold locks.

## 41.8 readOnly — a real optimisation

```java
@Transactional(readOnly = true)
public List<OrderDto> getAllOrders() { ... }
```

**What it actually does:**
1. Hibernate sets the flush mode to `MANUAL` — **no dirty checking, no snapshots** → less memory and CPU
2. The JDBC connection is marked read-only — some databases optimise, and replicas can be routed
3. Accidental writes are prevented — a safety net

**Standard pattern:**
```java
@Service
@Transactional(readOnly = true)              // class-level default: all reads
public class OrderService {

    public List<OrderDto> findAll() { ... }        // inherits readOnly = true

    @Transactional                                  // override for writes
    public OrderDto create(OrderRequestDto dto) { ... }
}
```
**This is a genuinely good pattern to propose** — it makes read-only the default, saves the dirty-checking overhead on every query, and makes write methods explicit and visible in review.

## 41.9 The self-invocation problem (again)

`@Transactional` is AOP, so it has the same limitation as Section 26.

```java
@Service
public class OrderService {

    public void processAll(List<Order> orders) {      // no @Transactional
        for (Order o : orders) {
            saveOrder(o);          // ❌ internal call — proxy bypassed
        }
    }

    @Transactional                 // ❌ NEVER APPLIES when called as above
    public void saveOrder(Order o) { ... }
}
```

**Additional rules that silently break `@Transactional`:**

| Rule | Why |
|---|---|
| Must be **public** | CGLIB can't proxy private/protected methods |
| Must be called **from outside** the bean | Self-invocation bypasses the proxy |
| Class must be a **Spring bean** | `new`-ed objects have no proxy |
| Doesn't work in `@PostConstruct` | Proxy created after initialization |
| Doesn't apply to `final` methods/classes | CGLIB can't override them |

**Checklist to state to a TL when `@Transactional` "isn't working":** *is the method public, is it called from another bean, is the class a Spring bean, and is the exception unchecked?* That covers virtually every case.

## 41.10 Optimistic vs Pessimistic locking

```java
// OPTIMISTIC — assume conflicts are rare; detect them at write time
@Entity
public class Product {
    @Version
    private Long version;        // Hibernate increments this on every update
}
// UPDATE products SET stock = ?, version = 4 WHERE id = ? AND version = 3
// Zero rows updated → someone else changed it → OptimisticLockException

// PESSIMISTIC — assume conflicts are likely; lock the row up front
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT p FROM Product p WHERE p.id = :id")
Optional<Product> findByIdForUpdate(@Param("id") Long id);
// → SELECT ... FOR UPDATE — other transactions BLOCK until this one commits
```

| | Optimistic | Pessimistic |
|---|---|---|
| Mechanism | Version column | Database row lock |
| Locks held | **None** | Row locked until commit |
| Conflict detected | **At commit** | Prevented upfront |
| Throughput | **High** | Lower |
| Deadlock risk | None | **Yes** |
| Best for | Low contention (most web apps) | High contention (inventory, seat booking) |

**Default to optimistic.** It scales better and has no deadlock risk. Handle `OptimisticLockException` by returning 409 Conflict and letting the client retry. Use pessimistic only where conflicts are genuinely frequent and retrying is unacceptable — the last seat on a flight, the last unit of stock.

## 41.11 Common Mistakes

| Mistake | Consequence |
|---|---|
| **Catching an exception without rethrowing** | Transaction commits despite failure — inconsistent data |
| Checked business exception | **No rollback** |
| **Self-invocation** | `@Transactional` silently ignored |
| `@Transactional` on a private method | Silently ignored |
| `@Transactional` on the controller | Transaction spans HTTP serialization — connection held too long |
| **External API calls inside a transaction** | Connection held for the call's duration; pool exhausted |
| Long-running transactions | Locks held, pool starved, deadlocks |
| `REQUIRES_NEW` used casually | Connection pool exhaustion |
| No `readOnly` on read methods | Wasted dirty-checking overhead |
| Assuming `@Async` joins the caller's transaction | Different thread — it does not |

**The external-call rule, stated explicitly:**
```java
// ❌ BAD — holds a DB connection during a 5-second HTTP call
@Transactional
public void placeOrder(Order order) {
    orderRepository.save(order);
    paymentGateway.charge(order);      // external HTTP — could take seconds or hang
    emailService.send(order);          // another external call
}

// ✅ BETTER — keep the transaction tight around the DB work
public void placeOrder(Order order) {
    Order saved = saveOrder(order);                  // @Transactional, short
    PaymentResult result = paymentGateway.charge(saved);   // outside the transaction
    updateOrderStatus(saved.getId(), result);        // @Transactional, short
}
```
**Why this matters:** a transaction holds a pooled connection for its entire duration. If an external service becomes slow, every in-flight transaction holds its connection, the pool drains, and your service stops — brought down by *someone else's* outage. **Keep transactions short and free of network I/O.** This is one of the most valuable practical points in this document.

## 41.12 TL Questions

**Q: What does `@Transactional` do?**
A: It wraps the method in a database transaction via an AOP proxy — begin before, commit on normal return, roll back on an unchecked exception. So the whole method is atomic.

**Q: How does it work internally?**
A: The proxy's `TransactionInterceptor` asks the `PlatformTransactionManager` for a transaction, which takes a connection, disables auto-commit and binds it to the current thread. Everything downstream on that thread uses the same connection and persistence context, which is why nothing has to be passed around.

**Q: Why don't checked exceptions roll back?**
A: Spring's default rule follows the EJB convention that checked exceptions are recoverable business conditions. It's why all our business exceptions extend `RuntimeException` — a checked one would let a half-completed transaction commit.

**Q: What happens if we catch an exception inside a transactional method?**
A: The method returns normally so the transaction commits, even though the operation failed. That's how we end up with an order saved and no payment. We always rethrow, wrapping if needed.

**Q: When do we use `REQUIRES_NEW`?**
A: For work that must survive the outer rollback — audit records of a failed attempt, mainly. We use it sparingly because it holds a second connection while the first is suspended, so heavy use can exhaust the pool.

**Q: Why is `readOnly = true` worth setting?**
A: Hibernate skips dirty checking and the change snapshots, so read queries use less memory and CPU. It also marks the connection read-only and prevents accidental writes. We default the service class to read-only and override on write methods.

**Q: Why must we not call external APIs inside a transaction?**
A: Because the transaction holds a pooled database connection the whole time. If the external service is slow or hangs, every in-flight request holds its connection until the pool drains and our service stops responding — taken down by someone else's outage. We commit the database work first and make the call outside.

**Q: What if `@Transactional` appears not to work?**
A: The checklist is: is the method public, is it called from another bean rather than internally, is the class a Spring bean, and is the exception unchecked. Self-invocation is the most common cause.

**Q: Optimistic or pessimistic locking?**
A: Optimistic by default, using a `@Version` column. It holds no locks and can't deadlock, and we return 409 on conflict so the client retries. Pessimistic only where contention is genuinely high and a retry isn't acceptable, like allocating the last unit of stock.

## 41.13 Interview Questions

**Beginner — What is ACID?**
Atomicity, Consistency, Isolation, Durability.

**Beginner — What is the default propagation?**
`REQUIRED` — join an existing transaction, or create one.

**Intermediate — Which exceptions trigger rollback by default?**
`RuntimeException` and `Error`. Checked exceptions do not; use `rollbackFor` to change that.

**Intermediate — `REQUIRED` vs `REQUIRES_NEW`?**
`REQUIRED` joins the existing transaction, so a rollback affects everything. `REQUIRES_NEW` suspends it and runs in an independent transaction that commits or rolls back separately — at the cost of a second connection.

**Intermediate — What does `readOnly = true` do?**
Sets Hibernate's flush mode to manual, disabling dirty checking and snapshots, and marks the JDBC connection read-only. A performance optimisation and a safety net, not a security control.

**Advanced — How is the transaction bound to the thread?**
`TransactionSynchronizationManager` holds `ThreadLocal` maps of resources — the connection holder and the `EntityManager` — keyed by `DataSource`/`EntityManagerFactory`. Everything on that thread resolves the same resources, which is why `@Async` methods don't join the caller's transaction.

**Advanced — What is `UnexpectedRollbackException`?**
When an inner `REQUIRED` transaction throws, it marks the shared transaction rollback-only. If the caller catches the exception and tries to commit, Spring refuses and throws this. The fix is to rethrow or to use `REQUIRES_NEW` for the inner call.

**Advanced — What is `NESTED` propagation?**
It creates a JDBC savepoint within the current transaction. The inner work can roll back to the savepoint without aborting the outer transaction, and it reuses the same connection — unlike `REQUIRES_NEW`. Support depends on the transaction manager.

**Advanced — Optimistic locking mechanics?**
A `@Version` column is included in the UPDATE's WHERE clause and incremented. If another transaction committed in the meantime the version no longer matches, zero rows are affected, and Hibernate raises `OptimisticLockException` — surfacing a lost update instead of silently overwriting.

## 41.14 Quick Revision

1. `@Transactional` = declarative transactions via an **AOP proxy**.
2. Commit on normal return; **rollback on unchecked exceptions only**.
3. **Checked exceptions commit** — use `rollbackFor`, or extend `RuntimeException`.
4. **Catching without rethrowing commits the failure** — always rethrow.
5. Transaction is bound to the **thread** — `@Async` does not inherit it.
6. Default propagation `REQUIRED`; `REQUIRES_NEW` for audit that must survive rollback.
7. `REQUIRES_NEW` uses a second connection — use sparingly.
8. `readOnly = true` disables dirty checking — default it at class level.
9. **Self-invocation bypasses the proxy**; method must be public and called externally.
10. Put `@Transactional` on the **service**, never the controller.
11. **Never make external API calls inside a transaction.**
12. Keep transactions short — they hold a pooled connection.
13. Prefer **optimistic locking** (`@Version`) over higher isolation levels.
14. Know your DB's default isolation — MySQL `REPEATABLE_READ`, Postgres `READ_COMMITTED`.

## 41.15 TL Explanation (speak this)

> "`@Transactional` on our service methods wraps them in a database transaction through an AOP proxy — begin before, commit on success, roll back on an unchecked exception. Two things we're strict about. First, our business exceptions extend `RuntimeException`, because Spring doesn't roll back on checked exceptions, and we never swallow an exception inside a transactional method — that commits the failure and leaves inconsistent data. Second, no external HTTP calls inside a transaction, because the transaction holds a pooled connection for its whole duration, so a slow payment gateway would drain the pool and take our service down. We default service classes to `readOnly = true` and override on writes, which skips dirty checking on all our read queries, and we use optimistic locking with `@Version` rather than raising isolation levels."

---
---

# PART D — SECURITY

---

[← 40. The N+1 Select Problem and EntityGraph](40-the-n-1-select-problem-and-entitygraph.md) | [Contents](../README.md) | [42. Spring Security →](../part-d-security/42-spring-security.md)
