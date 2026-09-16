[← Back to Contents](../README.md) · Part F — Master Flows and Final Revision

---

# 52. Complete Transaction Flow

```
 externalCaller.placeOrder(dto)      ← must come from ANOTHER bean
        │                              (self-invocation bypasses the proxy)
        ▼
 ╔══════════════════════════════════════════════════════════════╗
 ║  TRANSACTIONAL PROXY  (CGLIB, created by a BeanPostProcessor) ║
 ║                                                               ║
 ║  TransactionInterceptor.invoke()                              ║
 ║        │                                                      ║
 ║        ▼                                                      ║
 ║  PlatformTransactionManager.getTransaction(definition)        ║
 ║        │                                                      ║
 ║        ├─ PROPAGATION check:                                  ║
 ║        │    REQUIRED      → existing? JOIN : CREATE           ║
 ║        │    REQUIRES_NEW  → SUSPEND existing, CREATE new      ║
 ║        │                    (⚠ uses a SECOND connection)      ║
 ║        │    MANDATORY     → none? throw                       ║
 ║        │    NESTED        → savepoint on the same connection  ║
 ║        │                                                      ║
 ║        ├─ borrow a connection from HikariCP                   ║
 ║        ├─ connection.setAutoCommit(false)                     ║
 ║        ├─ apply ISOLATION level                               ║
 ║        ├─ apply readOnly → Hibernate flush mode MANUAL        ║
 ║        │                   (no dirty checking, no snapshots)  ║
 ║        └─ bind to the thread via                              ║
 ║             TransactionSynchronizationManager (ThreadLocal)   ║
 ║        │                                                      ║
 ║        ▼                                                      ║
 ║   ═══ INVOKE THE REAL METHOD ═══════════════════════════      ║
 ║        repository calls find the SAME connection and          ║
 ║        persistence context via the thread binding —           ║
 ║        nothing is passed as a parameter                       ║
 ║   ══════════════════════════════════════════════════════      ║
 ║        │                                                      ║
 ║   ┌────┴──────────────────────┐                               ║
 ║   │                            │                              ║
 ║ normal return          exception thrown                       ║
 ║   │                            │                              ║
 ║   │                   ┌────────┴────────┐                     ║
 ║   │            RuntimeException    checked Exception          ║
 ║   │            or Error                  │                    ║
 ║   │                   │            ⚠ COMMITS by default!      ║
 ║   │                   │            (unless rollbackFor)       ║
 ║   ▼                   ▼                  ▼                    ║
 ║ FLUSH              ROLLBACK           COMMIT                  ║
 ║  dirty check                                                  ║
 ║  → SQL                                                        ║
 ║ COMMIT                                                        ║
 ║   │                   │                  │                    ║
 ║   └───────────────────┴──────────────────┘                    ║
 ║        │                                                      ║
 ║        ▼                                                      ║
 ║  release connection → HikariCP                                ║
 ║  persistence context closes → entities DETACHED               ║
 ║  resume any suspended transaction                             ║
 ╚══════════════════════════════════════════════════════════════╝
```

## The four ways `@Transactional` silently fails

| Cause | Why |
|---|---|
| **Self-invocation** | `this.method()` bypasses the proxy |
| **Non-public method** | CGLIB cannot proxy it |
| **Checked exception thrown** | Default rollback covers unchecked only |
| **Exception caught and not rethrown** | Method returns normally → commit |

Plus: not a Spring bean (`new`-ed), `final` method/class, or called from `@PostConstruct`.

## The rules

1. `@Transactional` on the **service**, never the controller.
2. **Keep transactions short — no external HTTP calls inside.**
3. Default `readOnly = true` at class level; override on writes.
4. Business exceptions extend `RuntimeException`.
5. `REQUIRES_NEW` for audit that must survive a rollback — sparingly (second connection).
6. Optimistic locking (`@Version`) in preference to raising isolation.

---

[← 51. Complete Exception Handling Flow](51-complete-exception-handling-flow.md) | [Contents](../README.md) | [53. Filter vs Interceptor vs AOP — Complete Comparison →](53-filter-vs-interceptor-vs-aop-complete-comparison.md)
