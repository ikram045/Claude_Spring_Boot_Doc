[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 26. AOP (Aspect Oriented Programming)

## 26.1 What is it?

**Simple words:**
Some code needs to run in many places but isn't really part of the business logic — logging, security checks, transactions, performance timing, retries. Copying it into every method is repetitive and easy to forget.

AOP lets you **write that code once and tell Spring where to apply it** — "run this before every method in the service package".

**Technical wording:**
AOP is a programming paradigm that modularises **cross-cutting concerns** — behaviour that spans multiple modules — by separating them into **aspects**, which are woven into the target code at defined **join points** matched by **pointcut** expressions.

## 26.2 The problem AOP solves

**Without AOP:**

```java
@Service
public class OrderService {

    public Order placeOrder(Order order) {
        log.info("Entering placeOrder");                 // logging
        long start = System.currentTimeMillis();          // timing
        if (!securityService.hasPermission("ORDER_CREATE")) {   // security
            throw new AccessDeniedException("Denied");
        }
        try {
            Order result = doPlaceOrder(order);           // ← the ONLY business logic
            log.info("Exiting placeOrder in {}ms", System.currentTimeMillis() - start);
            return result;
        } catch (Exception e) {
            log.error("placeOrder failed", e);
            throw e;
        }
    }
}
```
Roughly **10 lines of plumbing around 1 line of business logic** — repeated in every method. Change the log format and you edit 200 methods.

**With AOP:**

```java
@Service
public class OrderService {
    public Order placeOrder(Order order) {
        return doPlaceOrder(order);        // ONLY business logic
    }
}
```
Logging, timing and security live in aspects, applied declaratively.

## 26.3 Core terminology (know these words exactly)

| Term | Plain meaning | Example |
|---|---|---|
| **Aspect** | The class holding the cross-cutting code | `LoggingAspect` |
| **Join Point** | A point where an aspect *can* be applied. **In Spring AOP this is always a method execution.** | `OrderService.placeOrder()` running |
| **Pointcut** | The expression selecting which join points | `execution(* com.company..service.*.*(..))` |
| **Advice** | The code that runs, and when | `@Before`, `@After`, `@Around` |
| **Target** | The object being advised | The real `OrderService` instance |
| **Proxy** | The wrapper Spring creates around the target | `OrderService$$SpringCGLIB$$0` |
| **Weaving** | Linking aspects to targets | Done at **runtime** in Spring, via proxies |

**Important limitation to state clearly:** Spring AOP supports **method execution join points only**. It cannot advise field access, constructors or object creation. Full AspectJ can do those through compile-time or load-time weaving — Spring AOP deliberately trades that power for simplicity.

## 26.4 The five advice types

```java
@Aspect
@Component
@Slf4j
public class LoggingAspect {

    // 1. BEFORE — runs before the method
    @Before("execution(* com.company.orderservice.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        log.info("Entering {} with args {}",
                 joinPoint.getSignature().getName(), Arrays.toString(joinPoint.getArgs()));
    }

    // 2. AFTER RETURNING — only on successful return; can read the result
    @AfterReturning(pointcut = "execution(* com.company..service.*.*(..))",
                    returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        log.info("{} returned {}", joinPoint.getSignature().getName(), result);
    }

    // 3. AFTER THROWING — only when an exception is thrown
    @AfterThrowing(pointcut = "execution(* com.company..service.*.*(..))",
                   throwing = "ex")
    public void logAfterThrowing(JoinPoint joinPoint, Exception ex) {
        log.error("{} threw {}", joinPoint.getSignature().getName(), ex.getMessage());
    }

    // 4. AFTER (finally) — always runs, success or failure
    @After("execution(* com.company..service.*.*(..))")
    public void logAfter(JoinPoint joinPoint) {
        log.debug("Completed {}", joinPoint.getSignature().getName());
    }

    // 5. AROUND — the most powerful: wraps the call completely
    @Around("execution(* com.company..service.*.*(..))")
    public Object measure(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            Object result = pjp.proceed();          // ← CALL THE ACTUAL METHOD
            return result;                          //   can modify the return value
        } finally {
            long duration = System.currentTimeMillis() - start;
            if (duration > 500) {
                log.warn("SLOW: {} took {}ms", pjp.getSignature().getName(), duration);
            }
        }
    }
}
```

| Advice | Runs | Can prevent execution? | Can modify return? | Parameter |
|---|---|---|---|---|
| `@Before` | Before | No | No | `JoinPoint` |
| `@AfterReturning` | On success | No | No (can read) | `JoinPoint` + result |
| `@AfterThrowing` | On exception | No | No | `JoinPoint` + exception |
| `@After` | Always (finally) | No | No | `JoinPoint` |
| **`@Around`** | **Wraps the call** | **Yes** | **Yes** | **`ProceedingJoinPoint`** |

**`@Around` is the only one that can skip the method entirely** — by simply not calling `proceed()`. That's how caching and circuit breakers are implemented.

**Critical rule:** in `@Around` you **must** call `proceed()` and **must** return its result. Forgetting either means the real method never runs, or the caller gets `null` — a silent and very confusing bug.

**Execution order:** `@Around` (before part) → `@Before` → **method** → `@AfterReturning`/`@AfterThrowing` → `@After` → `@Around` (after part).

## 26.5 Pointcut expressions

```java
// execution() — the one you'll use 95% of the time
execution(* com.company.orderservice.service.*.*(..))
//        │  │                                │ │  │
//        │  │                                │ │  └─ any arguments
//        │  │                                │ └─ any method name
//        │  │                                └─ any class in that package
//        │  └─ package
//        └─ any return type

execution(public * *(..))                          // any public method
execution(* com.company..service.*.*(..))          // .. = package and sub-packages
execution(* *..OrderService.placeOrder(..))        // one specific method
execution(* com.company..*.save*(..))              // any method starting with "save"
execution(* com.company..*.*(String, ..))          // first argument is a String

// within() — all methods within a type/package
within(com.company.orderservice.service..*)

// @annotation() — methods carrying a specific annotation  ★ MOST USEFUL IN PRACTICE
@annotation(com.company.orderservice.aspect.Auditable)

// @within() — all methods in classes carrying an annotation
@within(org.springframework.stereotype.Service)

// bean() — Spring-specific, by bean name
bean(orderService)
bean(*Service)

// Combining with && || !
execution(* com.company..service.*.*(..)) && !execution(* *.get*(..))
```

**Named pointcuts — reuse instead of repeating the expression:**

```java
@Aspect
@Component
public class CommonPointcuts {

    @Pointcut("within(com.company.orderservice.service..*)")
    public void serviceLayer() {}

    @Pointcut("@annotation(com.company.orderservice.aspect.Auditable)")
    public void auditable() {}

    @Pointcut("serviceLayer() && auditable()")      // compose them
    public void auditableServiceMethod() {}
}

@Aspect
@Component
public class AuditAspect {
    @Around("com.company.orderservice.aspect.CommonPointcuts.auditableServiceMethod()")
    public Object audit(ProceedingJoinPoint pjp) throws Throwable { ... }
}
```

**Best practice:** prefer **annotation-driven pointcuts** (`@annotation(...)`) over broad package expressions. `@Auditable` on a method makes the behaviour visible to whoever reads that method. A wildcard pointcut applies invisible behaviour that the next developer will not expect — a maintainability argument worth making to a TL.

## 26.6 Real project example — annotation-driven auditing

```java
// 1. The marker annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Auditable {
    String action();
    String entityType();
}

// 2. The aspect
@Aspect
@Component
@RequiredArgsConstructor
@Slf4j
public class AuditAspect {

    private final AuditLogRepository auditLogRepository;
    private final CurrentUserService currentUserService;

    @Around("@annotation(auditable)")        // binds the annotation instance as a parameter
    public Object audit(ProceedingJoinPoint pjp, Auditable auditable) throws Throwable {

        String user = currentUserService.getCurrentUsername();
        Instant start = Instant.now();

        try {
            Object result = pjp.proceed();                    // run the real method

            auditLogRepository.save(AuditLog.builder()
                    .username(user)
                    .action(auditable.action())
                    .entityType(auditable.entityType())
                    .status("SUCCESS")
                    .durationMs(Duration.between(start, Instant.now()).toMillis())
                    .correlationId(MDC.get("correlationId"))
                    .timestamp(Instant.now())
                    .build());

            return result;                                    // MUST return it

        } catch (Exception ex) {
            auditLogRepository.save(AuditLog.builder()
                    .username(user).action(auditable.action())
                    .entityType(auditable.entityType())
                    .status("FAILURE").errorMessage(ex.getMessage())
                    .timestamp(Instant.now()).build());
            throw ex;                                         // MUST rethrow
        }
    }
}

// 3. Usage — intent is visible right on the method
@Service
public class OrderService {

    @Auditable(action = "DELETE_ORDER", entityType = "ORDER")
    @Transactional
    public void deleteOrder(Long orderId) { ... }
}
```

**Two rules highlighted in that code:**
- **Always return `proceed()`'s result.**
- **Always rethrow** the caught exception. Swallowing it in an aspect makes the failure invisible to the caller *and* prevents the transaction rollback.

> **Transaction caution:** writing the audit row inside the same transaction means a rollback erases the audit of the failure. For audit trails that must survive rollback, use `@Transactional(propagation = Propagation.REQUIRES_NEW)` on the audit write. This is a good senior-level detail.

## 26.7 How AOP works internally — PROXIES

This is the most important mechanism to understand, because it explains AOP's main limitation.

```
   You ask for OrderService
             │
             ▼
   Spring does NOT give you the real object.
   It gives you a PROXY that wraps it.
             │
             ▼
  ┌──────────────────────────────────┐
  │  OrderService$$SpringCGLIB$$0    │   ← what you actually hold
  │                                   │
  │  placeOrder(order) {              │
  │      // 1. run @Before advice     │
  │      // 2. start transaction      │
  │      target.placeOrder(order);  ──┼──► ┌────────────────────┐
  │      // 3. commit transaction     │    │ REAL OrderService  │
  │      // 4. run @After advice      │    │ (the target)       │
  │  }                                │    └────────────────────┘
  └──────────────────────────────────┘
```

### Two proxy types

| | JDK Dynamic Proxy | CGLIB Proxy |
|---|---|---|
| Requires | The class implements an **interface** | No interface needed |
| Creates | A new class implementing the same interface | A **subclass** of the target |
| Limitation | Only interface methods are advised | **Cannot proxy `final` classes or `final` methods** |
| Spring Boot default | — | **CGLIB** (since Boot 2.0, `proxyTargetClass=true`) |

**Why Boot defaults to CGLIB:** it works whether or not an interface exists, avoiding the classic "expected a `OrderService` but found a `$Proxy42`" injection failures that confused developers for years.

### ★ THE SELF-INVOCATION PROBLEM ★

This is **the** AOP interview question, and a real production bug source.

```java
@Service
public class OrderService {

    public void processOrders(List<Order> orders) {
        for (Order order : orders) {
            saveOrder(order);        // ← internal call — DOES NOT GO THROUGH THE PROXY
        }
    }

    @Transactional                   // ← THIS IS IGNORED when called as above
    public void saveOrder(Order order) {
        repository.save(order);
    }
}
```

**Why it fails:** `saveOrder(order)` compiles to `this.saveOrder(order)`. `this` is the **real target object**, not the proxy. The proxy is only involved when the call comes **from outside**. So the advice — transaction, cache, audit — never runs.

```
External caller ──► PROXY ──► target.processOrders()
                                     │
                                     └─► this.saveOrder()   ← proxy bypassed!
```

**Solutions:**

```java
// Solution 1 — move the method to another bean (CLEANEST, preferred)
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderPersistenceService persistenceService;   // separate bean

    public void processOrders(List<Order> orders) {
        orders.forEach(persistenceService::saveOrder);   // external call → proxy applies
    }
}

// Solution 2 — self-injection (works, but a design smell)
@Service
public class OrderService {
    @Lazy @Autowired
    private OrderService self;                  // inject the PROXY of itself

    public void processOrders(List<Order> orders) {
        orders.forEach(self::saveOrder);        // goes through the proxy
    }
}

// Solution 3 — AopContext (requires exposeProxy = true; rarely used)
@EnableAspectJAutoProxy(exposeProxy = true)
// ...
((OrderService) AopContext.currentProxy()).saveOrder(order);
```

**What to say to a TL:** *"Self-invocation bypasses the proxy, so `@Transactional`, `@Cacheable` and our custom aspects silently don't apply. The right fix is to move the annotated method into a separate bean rather than self-injecting, because the need to self-inject usually indicates the class has two responsibilities."*

### Other proxy limitations

| Limitation | Reason |
|---|---|
| `private` methods can't be advised | The proxy can't override them |
| `final` methods/classes can't be advised by CGLIB | CGLIB subclasses; `final` blocks that |
| `static` methods can't be advised | Not polymorphic |
| Only **Spring beans** can be advised | An object created with `new` has no proxy |
| `@PostConstruct` runs before the proxy exists | See Section 6 |

## 26.8 Where AOP is already used (you're using it every day)

| Feature | Annotation | What the aspect does |
|---|---|---|
| **Transactions** | `@Transactional` | Begin, commit or roll back |
| **Caching** | `@Cacheable`, `@CacheEvict` | Check cache; skip the method on a hit |
| **Async** | `@Async` | Run on a different thread |
| **Security** | `@PreAuthorize`, `@Secured` | Check authority before execution |
| **Retry** | `@Retryable` | Re-invoke on failure with backoff |
| **Validation** | `@Validated` | Validate method parameters |

**Strong interview point:** *"`@Transactional` is not magic — it's AOP. Spring wraps the bean in a proxy whose advice opens a transaction before the method and commits or rolls back after. That's exactly why self-invocation breaks it."* Understanding AOP explains a whole family of Spring behaviours at once.

## 26.9 Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
Boot auto-enables `@EnableAspectJAutoProxy` when this is present, so you rarely declare it.

**Ordering multiple aspects:**
```java
@Aspect @Component @Order(1) public class SecurityAspect { }   // outermost
@Aspect @Component @Order(2) public class LoggingAspect { }
@Aspect @Component @Order(3) public class AuditAspect { }      // innermost
```
Lower order = outer wrapper = runs first on the way in, last on the way out.

## 26.10 Common Mistakes

| Mistake | Consequence |
|---|---|
| **Self-invocation** | Advice silently doesn't apply. The #1 AOP bug. |
| Not calling `proceed()` in `@Around` | The real method never executes |
| Not returning `proceed()`'s result | Caller always gets `null` |
| Swallowing the exception in an aspect | Failure invisible; transaction doesn't roll back |
| Advising `private`/`final`/`static` methods | Silently ignored |
| Over-broad pointcuts (`execution(* *(..))`) | Huge performance cost, unpredictable behaviour |
| Heavy logic in an aspect | Every advised call pays the cost |
| Business logic in aspects | Hidden behaviour; very hard to debug |
| Audit write in the same transaction | Rollback erases the failure audit |
| Missing `spring-boot-starter-aop` | Aspects silently never run |

## 26.11 TL Questions

**Q: What is AOP and why do we use it?**
A: It lets us separate cross-cutting concerns — logging, auditing, timing, transactions — from business logic. Instead of repeating that code in every method, we write it once in an aspect and declare where it applies.

**Q: How does it work internally?**
A: Spring wraps the bean in a proxy, CGLIB by default. When we inject `OrderService` we actually get the proxy. Calls go to the proxy, which runs the advice before and after delegating to the real object.

**Q: What's the biggest gotcha?**
A: Self-invocation. If a method calls another method in the same class, the call is `this.method()`, which bypasses the proxy — so `@Transactional`, `@Cacheable` and our aspects silently don't apply. The fix is to move the annotated method into a separate bean.

**Q: Where would you use it in a real project?**
A: An audit aspect driven by an `@Auditable` annotation, recording who did what and whether it succeeded, and a performance aspect that warns on slow service calls. We also use it implicitly through `@Transactional` and `@Cacheable`.

**Q: What happens if an aspect throws an exception?**
A: It propagates to the caller as if the target method threw it. If it's a `RuntimeException` inside a transaction, the transaction rolls back. That's why an aspect must never swallow exceptions — it would hide the failure and prevent rollback.

**Q: What's the performance cost?**
A: Small per call — one extra method dispatch plus the advice. The risk is over-broad pointcuts applying to thousands of methods, so we scope pointcuts with annotations rather than wildcards.

**Q: What alternative can we use?**
A: Interceptors for HTTP-layer concerns, or just calling a shared helper method explicitly. Explicit calls are more visible but reintroduce the duplication AOP removes. We use AOP where the concern is genuinely orthogonal.

**Q: `@Transactional` is AOP?**
A: Yes — it's an aspect Spring provides. The proxy opens a transaction before the method and commits or rolls back after. Understanding that explains why self-invocation breaks it and why it doesn't work in `@PostConstruct`.

## 26.12 Interview Questions

**Beginner — What is a cross-cutting concern?**
Functionality needed across many modules but not part of any one's core responsibility — logging, security, transactions, caching.

**Beginner — Name the advice types.**
`@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`, `@Around`.

**Intermediate — What's special about `@Around`?**
It wraps the invocation and receives a `ProceedingJoinPoint`, so it can inspect and modify arguments, decide whether to call `proceed()` at all, modify the return value, and handle exceptions. It's the only advice that can prevent execution.

**Intermediate — JDK proxy vs CGLIB?**
JDK dynamic proxies require an interface and proxy only interface methods. CGLIB subclasses the target class and needs no interface, but can't proxy `final` classes or methods. Spring Boot defaults to CGLIB.

**Intermediate — Spring AOP vs AspectJ?**
Spring AOP is proxy-based, runtime-woven, method-execution-only, and limited to Spring beans — but requires no special build setup. AspectJ does compile-time or load-time weaving, supports field access, constructors and any object, and performs better, at the cost of build complexity. Spring AOP covers the vast majority of needs.

**Advanced — Why does self-invocation bypass the proxy?**
Because an internal call resolves to `this`, which is the target instance, not the proxy. The proxy only intercepts calls that arrive through the injected reference from outside the object.

**Advanced — At what point in the bean lifecycle is the proxy created?**
In `postProcessAfterInitialization` by `AbstractAutoProxyCreator`, a `BeanPostProcessor` — i.e. **after** `@PostConstruct`. That's precisely why `@Transactional` doesn't work inside `@PostConstruct`.

**Advanced — How do you control the order of multiple aspects?**
`@Order` or `Ordered`. Lower value means higher precedence, so that aspect is the outermost wrapper — first on the way in, last on the way out.

**Advanced — Can you advise a method on an object created with `new`?**
No. AOP only applies to container-managed beans, because the container is what creates the proxy. This is the same reason `new`-ing a service breaks `@Transactional`.

## 26.13 Quick Revision

1. AOP separates **cross-cutting concerns** from business logic.
2. Terms: **Aspect, Join Point, Pointcut, Advice, Target, Proxy, Weaving**.
3. Spring AOP advises **method executions on Spring beans only**.
4. Five advice types; **`@Around` is the most powerful** — can skip the method.
5. In `@Around`: **always call `proceed()`, always return its result, always rethrow**.
6. Implemented by **proxies** — CGLIB by default in Boot.
7. **Self-invocation bypasses the proxy** — the #1 gotcha.
8. `private`, `final`, `static` methods and `new`-ed objects can't be advised.
9. Proxy is created in `postProcessAfterInitialization` — after `@PostConstruct`.
10. `@Transactional`, `@Cacheable`, `@Async`, `@PreAuthorize` are all AOP.
11. Prefer **annotation-driven pointcuts** — behaviour stays visible at the call site.
12. `@Order`: lower = outer.

## 26.14 TL Explanation (speak this)

> "AOP lets us pull cross-cutting concerns out of the business logic — we use it for an annotation-driven audit trail and for slow-call warnings. Spring implements it with proxies: when we inject `OrderService` we actually get a CGLIB proxy that runs the advice around the real object. The critical thing to know is self-invocation — if a method calls another method in the same class, that call goes through `this` and skips the proxy, so `@Transactional` or our aspects silently don't apply. The fix is to move the annotated method into a separate bean. It's also worth knowing that `@Transactional` and `@Cacheable` are themselves aspects, so the same rule explains why those sometimes appear not to work."

---
---

# PART C — DATA LAYER (JDBC / JPA / HIBERNATE)

---

[← 25. Interceptors](25-interceptors.md) | [Contents](../README.md) | [27. JDBC and DataSource →](../part-c-data-layer/27-jdbc-and-datasource.md)
