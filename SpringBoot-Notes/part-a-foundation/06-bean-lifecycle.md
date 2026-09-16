[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

---

# 6. Bean Lifecycle

## 6.1 What is it?

**Simple words:**
A bean is not just created and used. It goes through **stages**, like a person: born → educated → given a job → works → retires.

Spring lets you plug your own code into two of those moments:
- **just after the bean is fully ready** (initialization)
- **just before the application shuts down** (destruction)

**Technical wording:**
The bean lifecycle is the sequence of container-managed phases a bean passes through from instantiation to destruction, including dependency population, aware-interface callbacks, `BeanPostProcessor` hooks, initialization callbacks and destruction callbacks.

## 6.2 Why do we care?

Because real applications need setup and cleanup:

- Load a cache or lookup table once at startup
- Validate that required configuration is actually present — **fail fast at boot rather than on first request**
- Open a connection pool, a Kafka consumer, a file handle
- Close those resources cleanly on shutdown so nothing leaks

## 6.3 The Complete Lifecycle — step by step

```
   ┌─────────────────────────────────────────────────────────┐
   │              CONTAINER STARTUP                          │
   └─────────────────────────────────────────────────────────┘
                          │
   1. Bean definitions loaded (metadata only — no objects yet)
                          │
                          ▼
   2. INSTANTIATION — constructor is called
      (constructor injection happens HERE)
                          │
                          ▼
   3. POPULATE PROPERTIES — setter and field injection (@Autowired)
                          │
                          ▼
   4. AWARE INTERFACES called in order:
        BeanNameAware.setBeanName()
        BeanClassLoaderAware.setBeanClassLoader()
        BeanFactoryAware.setBeanFactory()
        ApplicationContextAware.setApplicationContext()
                          │
                          ▼
   5. BeanPostProcessor.postProcessBeforeInitialization()
                          │
                          ▼
   6. INITIALIZATION callbacks, in this exact order:
        a) @PostConstruct annotated method
        b) InitializingBean.afterPropertiesSet()
        c) custom init-method (@Bean(initMethod="..."))
                          │
                          ▼
   7. BeanPostProcessor.postProcessAfterInitialization()
      ★ AOP PROXIES ARE CREATED HERE ★
        (@Transactional, @Async, @Cacheable wrapping happens now)
                          │
                          ▼
   ┌─────────────────────────────────────────────────────────┐
   │        BEAN IS READY AND IN USE                          │
   └─────────────────────────────────────────────────────────┘
                          │
                  ... application runs ...
                          │
                          ▼
   ┌─────────────────────────────────────────────────────────┐
   │             CONTAINER SHUTDOWN                           │
   └─────────────────────────────────────────────────────────┘
                          │
   8. DESTRUCTION callbacks, in this exact order:
        a) @PreDestroy annotated method
        b) DisposableBean.destroy()
        c) custom destroy-method (@Bean(destroyMethod="..."))
                          │
                          ▼
   9. Bean removed, JVM exits
```

**Two things to memorise from this diagram:**

1. **Order of init callbacks:** `@PostConstruct` → `afterPropertiesSet()` → custom `initMethod`.
2. **AOP proxies are created in step 7** — *after* initialization. This single fact explains the most famous Spring bug, covered below.

## 6.4 Code example showing every hook

```java
package com.company.orderservice.service;

@Service
public class LifecycleDemoService implements InitializingBean, DisposableBean {

    private final OrderRepository repository;

    // STEP 2 — instantiation + constructor injection
    public LifecycleDemoService(OrderRepository repository) {
        this.repository = repository;
        System.out.println("1. Constructor — dependencies injected");
        // WARNING: @Value fields and field-injected beans are NOT set yet here
    }

    // STEP 6a — runs once, after ALL dependencies are injected
    @PostConstruct
    public void init() {
        System.out.println("2. @PostConstruct — safe to use dependencies");
        // Typical real use: warm a cache, validate config, preload reference data
    }

    // STEP 6b — same purpose, but couples the class to Spring's interface
    @Override
    public void afterPropertiesSet() {
        System.out.println("3. afterPropertiesSet()");
    }

    // STEP 8a — runs on graceful shutdown
    @PreDestroy
    public void cleanup() {
        System.out.println("4. @PreDestroy — release resources");
    }

    @Override
    public void destroy() {
        System.out.println("5. destroy()");
    }
}
```

### The same hooks via `@Bean` (for third-party classes)

```java
@Configuration
public class AppConfig {

    @Bean(initMethod = "start", destroyMethod = "shutdown")
    public MessageConsumer messageConsumer() {
        return new MessageConsumer();   // third-party class we can't annotate
    }
}
```

## 6.5 Which initialization approach should you use?

| Approach | Coupled to Spring? | Recommendation |
|---|---|---|
| `@PostConstruct` / `@PreDestroy` | No — Jakarta annotations | **Preferred.** Standard, clean, portable. |
| `InitializingBean` / `DisposableBean` | **Yes** — implements Spring interfaces | Avoid in business code |
| `@Bean(initMethod/destroyMethod)` | No | Best for **third-party** classes |

> **Note for Spring Boot 3+ / Java 17+:** `@PostConstruct` and `@PreDestroy` moved from `javax.annotation` to **`jakarta.annotation`**. If you see `@PostConstruct` silently not firing after an upgrade, it is almost always the wrong import.

## 6.6 The classic bug: `@Transactional` does not work in `@PostConstruct`

```java
@Service
public class DataLoaderService {

    @PostConstruct
    @Transactional              // ← THIS DOES NOT WORK
    public void loadInitialData() {
        repository.saveAll(referenceData);   // runs with NO transaction
    }
}
```

**Why?** Look at the lifecycle diagram again:

- `@PostConstruct` runs at **step 6**.
- The transactional proxy is created at **step 7**.

So when `@PostConstruct` executes, the proxy that would start the transaction **does not exist yet**. You are calling the raw object.

**Correct solutions:**

```java
// Solution 1 — use an application event (fires after the context is fully ready)
@Component
@RequiredArgsConstructor
public class DataLoader {

    private final ReferenceDataService service;

    @EventListener(ApplicationReadyEvent.class)
    public void loadData() {
        service.loadInitialData();   // proxy exists now → @Transactional works
    }
}

// Solution 2 — CommandLineRunner / ApplicationRunner
@Component
@RequiredArgsConstructor
public class StartupRunner implements ApplicationRunner {

    private final ReferenceDataService service;

    @Override
    public void run(ApplicationArguments args) {
        service.loadInitialData();
    }
}
```

**This is a genuine senior-level interview question.** If you can explain *why* using the lifecycle order, you stand out immediately.

## 6.7 Real-Time Project Example

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class CountryCacheService {

    private final CountryRepository countryRepository;
    private Map<String, String> countryCodeToName;

    @PostConstruct
    public void loadCache() {
        // Reference data changes rarely — load once instead of hitting DB per request
        countryCodeToName = countryRepository.findAll().stream()
                .collect(Collectors.toMap(Country::getCode, Country::getName));
        log.info("Country cache loaded with {} entries", countryCodeToName.size());

        // Fail fast: better to crash at deploy time than serve wrong data all day
        if (countryCodeToName.isEmpty()) {
            throw new IllegalStateException("Country reference data is empty — check DB seed");
        }
    }

    public String resolveName(String code) {
        return countryCodeToName.get(code);
    }

    @PreDestroy
    public void clear() {
        log.info("Clearing country cache before shutdown");
        countryCodeToName.clear();
    }
}
```

**Why this is good practice:** the failure surfaces during deployment, when a rollback is easy — not at 2 a.m. when a customer hits the endpoint.

## 6.8 Important interfaces and annotations

| Name | Type | When it runs |
|---|---|---|
| `@PostConstruct` | Annotation | After dependency injection |
| `@PreDestroy` | Annotation | Before bean destruction |
| `InitializingBean` | Interface | After `@PostConstruct` |
| `DisposableBean` | Interface | After `@PreDestroy` |
| `BeanPostProcessor` | Interface | Around **every** bean's initialization |
| `BeanFactoryPostProcessor` | Interface | On **bean definitions**, before any instantiation |
| `ApplicationContextAware` | Interface | Gives the bean the context |
| `CommandLineRunner` / `ApplicationRunner` | Interface | After the context is fully started |
| `ApplicationReadyEvent` | Event | After the app is fully up, including the web server |

**`BeanPostProcessor` vs `BeanFactoryPostProcessor` (advanced but commonly asked):**

| | `BeanFactoryPostProcessor` | `BeanPostProcessor` |
|---|---|---|
| Operates on | Bean **definitions** (metadata) | Bean **instances** (objects) |
| Runs | Before any bean is instantiated | Around each bean's initialization |
| Can modify | Configuration/metadata | The object itself; can return a **proxy** |
| Real example | `PropertySourcesPlaceholderConfigurer` (resolves `${...}`) | `AutoProxyCreator` (creates `@Transactional` proxies) |

## 6.9 Common Mistakes

| Mistake | Consequence |
|---|---|
| `@Transactional` on `@PostConstruct` | Silently non-transactional |
| Using field-injected dependencies inside the constructor | `NullPointerException` — fields aren't set yet |
| Wrong import (`javax` vs `jakarta`) in Boot 3 | Callback silently never runs |
| Expecting `@PreDestroy` on a **prototype** bean | Never called — Spring does not track prototype destruction |
| Expecting `@PreDestroy` after `kill -9` | Never called — only graceful shutdown (SIGTERM) triggers it |
| Long-running work in `@PostConstruct` | Slows startup; blocks health checks and container readiness probes |

## 6.10 TL Questions

**Q: What is the bean lifecycle?**
A: The stages a bean passes through — instantiation, dependency injection, aware callbacks, post-processing, initialization, use, and destruction — all managed by the container.

**Q: Why do we use `@PostConstruct` in our project?**
A: For one-time setup that needs the dependencies to be ready — loading reference data caches and validating that required configuration exists, so a misconfigured deployment fails immediately instead of failing on the first request.

**Q: What happens internally between the constructor and `@PostConstruct`?**
A: Field and setter injection are completed, then the aware interfaces are invoked, then `BeanPostProcessor.postProcessBeforeInitialization` runs. Only after all of that does `@PostConstruct` fire — which is why dependencies are guaranteed to be available there but not in the constructor for field injection.

**Q: Why doesn't `@Transactional` work inside `@PostConstruct`?**
A: Because AOP proxies are created in `postProcessAfterInitialization`, which runs *after* `@PostConstruct`. At that point the proxy doesn't exist, so the raw method executes with no transaction. We move such work to an `ApplicationReadyEvent` listener or an `ApplicationRunner`.

**Q: What happens if we remove `@PreDestroy`?**
A: Cleanup code stops running at shutdown. For caches that's harmless; for open connections, consumers or file handles it means leaked resources and possibly unflushed data.

**Q: When is `@PreDestroy` NOT called?**
A: On a `kill -9` or JVM crash, and for prototype-scoped beans, which the container does not track after handing them out.

**Q: Where would you use this in a real project?**
A: Warming reference-data caches, validating configuration at startup, registering/deregistering from a service discovery registry, and closing Kafka consumers cleanly during shutdown.

## 6.11 Interview Questions

**Beginner — Name the two main lifecycle callback annotations.**
`@PostConstruct` and `@PreDestroy`.

**Beginner — When does `@PostConstruct` run?**
Once, after the bean is constructed and all dependencies are injected, before the bean is made available for use.

**Intermediate — What is the exact order of initialization callbacks?**
`@PostConstruct` → `InitializingBean.afterPropertiesSet()` → custom `initMethod`. Destruction mirrors it: `@PreDestroy` → `DisposableBean.destroy()` → custom `destroyMethod`.

**Intermediate — `@PostConstruct` vs `CommandLineRunner`?**
`@PostConstruct` runs per bean, during context startup, before the context is fully refreshed. `CommandLineRunner` runs once, after the entire context is started and the web server is listening. Use `@PostConstruct` for bean-local setup; use `CommandLineRunner`/`ApplicationReadyEvent` for anything that needs the whole application (especially proxies) to be ready.

**Advanced — What is `BeanPostProcessor` and name a real one.**
An extension point invoked around every bean's initialization, able to return a *replacement* object. `AbstractAutoProxyCreator` is the crucial real example — it is what wraps your beans in AOP proxies for `@Transactional`, `@Async` and `@Cacheable`.

**Advanced — Where exactly in the lifecycle is the AOP proxy created?**
In `postProcessAfterInitialization`, i.e. after all initialization callbacks. This is why self-invocation and `@PostConstruct` bypass proxies.

**Advanced — Are lifecycle callbacks honoured for prototype beans?**
Initialization callbacks yes; destruction callbacks **no**. The container does not retain a reference to prototype instances, so the client code is responsible for cleanup.

## 6.12 Quick Revision

1. Lifecycle: instantiate → inject → aware → BPP-before → **init** → BPP-after (**proxy created**) → use → destroy.
2. Init order: `@PostConstruct` → `afterPropertiesSet()` → `initMethod`.
3. Destroy order: `@PreDestroy` → `destroy()` → `destroyMethod`.
4. Prefer `@PostConstruct`/`@PreDestroy` — no Spring coupling.
5. **AOP proxies are created after initialization** → `@Transactional` fails in `@PostConstruct`.
6. Use `ApplicationReadyEvent` or `ApplicationRunner` for startup work needing proxies.
7. Prototype beans get **no** destroy callback.
8. Boot 3 uses `jakarta.annotation.PostConstruct`, not `javax`.
9. `@PostConstruct` is the right place to **fail fast** on bad configuration.

## 6.13 TL Explanation (speak this)

> "The container creates the bean, injects its dependencies, runs the initialization callbacks, and only then wraps it in any AOP proxies — that ordering is the part worth remembering. We use `@PostConstruct` for one-time setup like warming a reference cache and validating config, so a bad deployment fails at startup instead of on the first customer request, and `@PreDestroy` to release resources on graceful shutdown. One gotcha we watch for: `@Transactional` inside `@PostConstruct` silently does nothing because the proxy isn't created yet, so that work belongs in an `ApplicationReadyEvent` listener instead."

---

[← 5. Beans](05-beans.md) | [Contents](../README.md) | [7. Bean Scopes →](07-bean-scopes.md)
