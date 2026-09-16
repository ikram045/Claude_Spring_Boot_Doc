[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

---

# 7. Bean Scopes

## 7.1 What is it?

**Simple words:**
A scope answers one question: **"How many copies of this bean should exist, and how long should each live?"**

By default Spring creates **one single object** and gives that same object to everyone who asks for it. That is the *singleton* scope.

**Technical wording:**
A bean scope defines the lifecycle and visibility of a bean instance within the container — specifically how many instances the container creates and the boundary within which a given instance is shared.

## 7.2 The Scopes

| Scope | Instances created | Lifetime | Available in |
|---|---|---|---|
| **singleton** (default) | **One per container** | Whole application | All apps |
| **prototype** | **A new one every time it is requested** | Until garbage collected | All apps |
| **request** | One per HTTP request | That request only | Web apps |
| **session** | One per HTTP session | That user's session | Web apps |
| **application** | One per `ServletContext` | Whole web application | Web apps |
| **websocket** | One per WebSocket session | That socket | Web apps |

In practice, **99% of your beans are singleton**, and you will rarely change it.

## 7.3 Singleton scope — the default

```java
@Service                       // singleton by default
public class OrderService { }

// or explicitly:
@Service
@Scope("singleton")
public class OrderService { }
```

**Very important clarification (asked constantly in interviews):**

> Spring's singleton is **one instance per Spring container**, *not* the Java "Singleton design pattern" (one instance per JVM/classloader).

If you start two `ApplicationContext`s in the same JVM, you get **two** instances. The Java singleton pattern enforces one per classloader via a private constructor and a static instance.

| | Spring Singleton | GoF Singleton Pattern |
|---|---|---|
| Uniqueness per | Container | JVM / classloader |
| Enforced by | The container (a map of bean name → instance) | Private constructor + static field |
| Multiple instances possible? | Yes — two containers, or a different bean name | No |
| Testability | High | Low (global state) |

### THE critical rule about singletons

**A singleton bean must be STATELESS.**

Because one instance serves **all concurrent requests on all threads**, any mutable field is shared across threads.

```java
// ❌ DANGEROUS — a real production bug
@Service
public class OrderService {

    private Order currentOrder;      // shared mutable state!

    public void process(Order order) {
        this.currentOrder = order;   // Thread A writes...
        validate();                  // ...Thread B overwrites it here
        save();                      // Thread A now saves Thread B's order
    }
}
```
Under load this leaks one customer's data into another customer's transaction. It usually passes all tests and only fails in production.

```java
// ✅ CORRECT — stateless, all data is in method parameters / local variables
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository repository;   // final dependency = safe to share

    public void process(Order order) {          // state lives on the stack, per thread
        validate(order);
        repository.save(order);
    }
}
```

**Rule of thumb:** in a singleton, the only fields allowed are `final` dependencies and immutable configuration. Never mutable request data.

## 7.4 Prototype scope

```java
@Component
@Scope("prototype")     // or @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class ReportGenerator {
    private final List<String> rows = new ArrayList<>();   // safe: each caller gets a fresh object
}
```

Every `getBean("reportGenerator")` returns a **brand new object**.

**Use when:** the bean holds per-use mutable state — a document builder, a report accumulator, a stateful parser.

**Two critical facts about prototype beans:**

1. **Spring does not manage their full lifecycle.** It creates and injects them, then forgets them. **`@PreDestroy` is never called.** You are responsible for cleanup.
2. **The singleton-injecting-prototype trap** (extremely common interview question):

```java
@Service                                   // singleton — created ONCE
public class ReportService {

    private final ReportGenerator generator;   // prototype

    public ReportService(ReportGenerator generator) {
        this.generator = generator;            // injected ONCE, at startup
    }

    public void generate() {
        generator.addRow("...");   // ALWAYS the same instance — prototype has no effect!
    }
}
```

**Why?** The singleton is created once, so its dependencies are resolved **once**. The prototype's "new instance every time" applies to *injection points*, not to *method calls*.

**Three correct fixes:**

```java
// Fix 1 — ObjectProvider (cleanest, recommended)
@Service
@RequiredArgsConstructor
public class ReportService {
    private final ObjectProvider<ReportGenerator> generatorProvider;

    public void generate() {
        ReportGenerator generator = generatorProvider.getObject();  // NEW instance each call
        generator.addRow("...");
    }
}

// Fix 2 — @Lookup method injection
@Service
public abstract class ReportService {
    @Lookup
    protected abstract ReportGenerator getGenerator();   // Spring implements this method

    public void generate() {
        getGenerator().addRow("...");   // new instance each call
    }
}

// Fix 3 — scoped proxy on the prototype bean itself
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ReportGenerator { }
```

## 7.5 Web scopes — request and session

```java
@Component
@RequestScope                      // shorthand for @Scope("request")
public class RequestContextHolder {
    private String correlationId;
    private String userId;
}

@Component
@SessionScope
public class ShoppingCart {
    private final List<CartItem> items = new ArrayList<>();
}
```

**The proxy requirement:** you cannot inject a request-scoped bean directly into a singleton — at startup there is no HTTP request, so there is nothing to inject.

Spring solves this with a **scoped proxy**. It injects a lightweight stand-in object into the singleton. When a method is called on that proxy, it looks up the *real* bean for the *current* request and delegates to it.

```
Singleton OrderService
        │ holds
        ▼
  [ Scoped Proxy ]  ← injected once, at startup
        │ at call time, resolves:
        ▼
  Real RequestContextHolder for THIS thread's HTTP request
```

`@RequestScope` and `@SessionScope` enable this proxying automatically (`proxyMode = TARGET_CLASS`), which is why you should prefer them to the raw `@Scope("request")`.

## 7.6 Real-Time Project Example

```java
// ---- Correlation ID for distributed tracing: one per HTTP request ----
@Component
@RequestScope
@Getter @Setter
public class RequestContext {
    private String correlationId;
    private String userId;
    private Instant startedAt;
}

// ---- Populated by a filter at the start of every request ----
@Component
@RequiredArgsConstructor
public class CorrelationIdFilter extends OncePerRequestFilter {

    private final RequestContext requestContext;   // proxy injected into this singleton

    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {
        String id = Optional.ofNullable(req.getHeader("X-Correlation-Id"))
                            .orElse(UUID.randomUUID().toString());
        requestContext.setCorrelationId(id);       // proxy resolves THIS request's instance
        requestContext.setStartedAt(Instant.now());
        MDC.put("correlationId", id);              // so every log line carries it
        try {
            chain.doFilter(req, res);
        } finally {
            MDC.clear();                           // MUST clear — thread is reused from the pool
        }
    }
}

// ---- Any singleton service can now read the current request's context ----
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderService {

    private final RequestContext requestContext;   // proxy

    public void placeOrder(Order order) {
        log.info("Placing order for user {} [correlationId={}]",
                 requestContext.getUserId(), requestContext.getCorrelationId());
    }
}
```

**Why this matters in production:** when a customer reports a failure, you search the logs for one correlation ID and see that request's entire journey across every microservice. This is standard practice in corporate systems.

## 7.7 Common Mistakes

| Mistake | Consequence |
|---|---|
| **Mutable state in a singleton** | Race conditions, data leaking between users. The most serious one on this list. |
| Injecting prototype into singleton directly | Only one instance ever used |
| Expecting `@PreDestroy` on a prototype | Never called; resources leak |
| Using `@Scope("request")` without a proxy mode | Startup failure or `ScopeNotActiveException` |
| Using request/session scope in `@Async` or `@Scheduled` code | No request bound to that thread → exception |
| Overusing session scope | Breaks horizontal scaling unless sessions are shared (Spring Session + Redis) |
| Not clearing `MDC`/`ThreadLocal` | Values leak into the next request on a pooled thread |

## 7.8 TL Questions

**Q: What is a bean scope?**
A: It defines how many instances of a bean the container creates and how long each one lives. The default is singleton — one shared instance for the whole application.

**Q: Why is singleton the default?**
A: Because most beans — controllers, services, repositories — are stateless behaviour holders. One shared instance saves memory and creation cost, and there is no state to conflict.

**Q: What is the risk with singleton beans?**
A: Shared mutable state. One instance serves all concurrent threads, so an instance field holding request data will be overwritten by other requests. We keep singletons stateless and hold per-request data in method parameters or local variables.

**Q: What happens if we inject a prototype bean into a singleton?**
A: The singleton is created once, so the prototype is injected once — you keep getting the same instance and the prototype scope has no effect. We use `ObjectProvider.getObject()` or `@Lookup` when we genuinely need a fresh instance per call.

**Q: How do request-scoped beans work internally?**
A: Spring injects a scoped proxy into the singleton at startup. On each method call the proxy looks up the real instance bound to the current thread's HTTP request and delegates to it.

**Q: Where would you use request scope in a real project?**
A: For per-request context like a correlation ID and the authenticated user, so any service can log and trace them without threading those values through every method signature.

**Q: What happens if we make a controller prototype-scoped?**
A: A new controller object per request — extra object churn with no benefit, since controllers should be stateless anyway. It would also be surprising to the next developer, so we keep them singleton.

## 7.9 Interview Questions

**Beginner — What is the default bean scope?**
Singleton — one instance per Spring container.

**Beginner — List the bean scopes.**
singleton, prototype, request, session, application, websocket.

**Intermediate — Is a Spring singleton thread-safe?**
The container guarantees safe *publication* of the instance, but **not** the thread safety of your code. If the bean has mutable state, it is unsafe. Statelessness is the developer's responsibility.

**Intermediate — Spring singleton vs GoF singleton?**
Spring: one per container, enforced by the container, multiple containers give multiple instances. GoF: one per JVM/classloader, enforced by a private constructor and static field.

**Intermediate — Why is `@PreDestroy` not called on a prototype?**
The container hands the instance to the caller and does not keep a reference to it, so it has no way to know when the object is finished. Cleanup is the caller's responsibility.

**Advanced — How does a scoped proxy actually work?**
Spring registers a CGLIB (or JDK) proxy as the injected dependency. Each method call on the proxy consults the active `Scope` implementation, which resolves the target instance for the current request/session from a `ThreadLocal`-backed holder (`RequestContextHolder`), then delegates.

**Advanced — Can we define a custom scope?**
Yes — implement `org.springframework.beans.factory.config.Scope` and register it via `ConfigurableBeanFactory.registerScope()` or a `CustomScopeConfigurer`. A tenant-per-scope in a multi-tenant system is a realistic example.

**Advanced — Why do request-scoped beans fail inside `@Async` methods?**
The request context is stored in a `ThreadLocal` bound to the request-handling thread. An `@Async` method runs on a different pool thread with no bound request, so the proxy cannot resolve a target and throws. The fix is to pass the needed values explicitly, or propagate the context (`RequestContextHolder.setRequestAttributes(..., true)` / a `TaskDecorator`).

## 7.10 Quick Revision

1. Default scope = **singleton** (one per container, not per JVM).
2. **Singletons must be stateless** — mutable fields cause cross-request data bugs.
3. Prototype = new instance per request for the bean; **no destroy callback**.
4. Prototype injected into singleton = injected once → use `ObjectProvider` or `@Lookup`.
5. Web scopes: request, session, application, websocket.
6. Request/session beans are injected as **scoped proxies** that resolve per thread.
7. Prefer `@RequestScope`/`@SessionScope` — they enable proxying automatically.
8. Request scope does not work in `@Async`/`@Scheduled` threads.
9. Always clear `ThreadLocal`/`MDC` in a `finally` block — threads are pooled and reused.

## 7.11 TL Explanation (speak this)

> "Scope controls how many instances of a bean exist. Everything we write — controllers, services, repositories — is singleton by default, which means one shared instance across all threads, so the hard rule is that they stay stateless and all per-request data lives in method parameters. Where we genuinely need per-request state, like a correlation ID for tracing, we use a request-scoped bean; Spring injects a proxy that resolves the right instance for the current request. The trap we watch for is injecting a prototype into a singleton — it gets injected once, so we use an `ObjectProvider` when we need a fresh instance per call."

---

[← 6. Bean Lifecycle](06-bean-lifecycle.md) | [Contents](../README.md) | [8. ApplicationContext and BeanFactory →](08-applicationcontext-and-beanfactory.md)
