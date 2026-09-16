[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

---

# 5. Beans

## 5.1 What is it?

**Simple words:**
A **bean is just an object that Spring created and is looking after.**

That's the whole definition. If Spring made it, it's a bean. If you made it with `new`, it's an ordinary object — not a bean.

**Technical wording:**
A Spring bean is an object that is instantiated, assembled, configured and lifecycle-managed by the Spring IoC container, based on a `BeanDefinition` registered in that container.

**The distinction that matters:**

```java
Order order = new Order();        // plain Java object — NOT a bean
                                  // no DI, no AOP, no transactions

@Service
public class OrderService { }     // Spring creates it — IS a bean
                                  // gets DI, AOP, transactions, lifecycle callbacks
```

**Important rule:** entities and DTOs are **not** beans. `new Order()` in your service is correct — those are data objects, created per request. Beans are your *components*: controllers, services, repositories, configuration objects.

## 5.2 Why do we use beans?

Because being managed by the container is what unlocks every Spring feature:

- Dependencies get injected automatically
- Lifecycle callbacks (`@PostConstruct`, `@PreDestroy`) fire
- AOP proxies can be applied (transactions, security, logging, caching, retry)
- Scope is controlled (one shared instance vs one per request)
- The bean can be replaced/mocked in tests via the context

## 5.3 How to create a bean — the three ways

### Way 1 — Stereotype annotations (most common)

```java
@Component     // generic
@Service       // business logic layer
@Repository     // data access layer
@Controller     // web layer (returns views)
@RestController // web layer (returns JSON)
```

These are found by **component scanning** (Section 11).

**Critical point:** `@Service`, `@Repository` and `@Controller` are all **`@Component` under the hood** — technically they are *meta-annotated* with it.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Component                       // ← @Service IS a @Component
public @interface Service { ... }
```

**So why do they exist separately?**

| Annotation | Functional difference | Why use it |
|---|---|---|
| `@Component` | Base — no extra behaviour | Generic components (utilities, mappers, validators) |
| `@Service` | **None functionally** | Documents intent: "this holds business logic". Also a target for AOP pointcuts. |
| `@Repository` | **Yes — real extra behaviour.** Enables exception translation: vendor-specific `SQLException`/Hibernate exceptions are converted into Spring's `DataAccessException` hierarchy | Data access classes |
| `@Controller` | **Yes** — makes the class a Spring MVC handler; scanned for `@RequestMapping` | Web layer |
| `@RestController` | **Yes** — `@Controller` + `@ResponseBody` | REST APIs |

**Interview gold:** The only stereotype with real *hidden* behaviour in the service/data layers is **`@Repository` (exception translation)**. `@Service` is purely semantic — but you should still use it, because readability and pointcut targeting matter.

### Way 2 — `@Bean` methods in a `@Configuration` class

Use this when you **cannot annotate the class** — typically a third-party library class.

```java
@Configuration
public class AppConfig {

    @Bean                                 // method return value becomes a bean
    public RestTemplate restTemplate() {  // bean name = method name = "restTemplate"
        return new RestTemplate();
    }

    @Bean
    public ModelMapper modelMapper() {
        ModelMapper mapper = new ModelMapper();
        mapper.getConfiguration().setSkipNullEnabled(true);
        return mapper;
    }
}
```
You cannot put `@Component` on `RestTemplate` — you don't own that source code. `@Bean` is the answer.

### Way 3 — XML configuration (legacy)

```xml
<bean id="orderService" class="com.company.service.OrderService"/>
```
You will only meet this in older projects. Know it exists; don't use it in new code.

## 5.4 `@Component` vs `@Bean` — very common interview question

| Aspect | `@Component` | `@Bean` |
|---|---|---|
| Applied to | **Class** | **Method** |
| Detected by | Component scanning | Declared inside `@Configuration` |
| Control over creation | Spring calls the constructor | **You write the creation code** |
| Use for | Classes **you own** | Classes you **don't own** (third-party) |
| Bean name | Class name, camelCase (`orderService`) | Method name |
| Conditional creation | Harder | Easy — plain `if` inside the method |
| Number of beans per source | One per class | Many methods = many beans |

**One line:** *"`@Component` is auto-detection for my own classes; `@Bean` is manual registration for objects I must construct myself."*

## 5.5 Bean naming

```java
@Service
public class OrderService { }          // bean name → "orderService"

@Service("customOrderService")
public class OrderService { }          // bean name → "customOrderService"

@Bean
public RestTemplate restTemplate() { } // bean name → "restTemplate"

@Bean(name = "externalApiClient")
public RestTemplate restTemplate() { } // bean name → "externalApiClient"
```

Default rule: the class's simple name with the **first letter lowercased**.
Edge case: if the first two letters are both uppercase, the name is left as-is (`URLParser` → `URLParser`, not `uRLParser`).

## 5.6 Real-Time Project Example

```java
// ---------- config/AppConfig.java ----------
@Configuration
public class AppConfig {

    // Third-party class → must use @Bean
    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder
                .setConnectTimeout(Duration.ofSeconds(3))
                .setReadTimeout(Duration.ofSeconds(5))
                .build();
    }

    // Reading config values into a bean
    @Bean
    public PaymentGatewayClient paymentGatewayClient(
            @Value("${payment.gateway.url}") String url,
            @Value("${payment.gateway.api-key}") String apiKey,
            RestTemplate restTemplate) {        // ← another bean injected as a parameter
        return new PaymentGatewayClient(url, apiKey, restTemplate);
    }
}
```

**Key line explanation:** `@Bean` methods can take **parameters**, and Spring injects matching beans into them. That is how `restTemplate` gets passed into `paymentGatewayClient` — bean-to-bean wiring inside configuration.

## 5.7 Common Mistakes

| Mistake | Consequence |
|---|---|
| Forgetting the stereotype annotation | `NoSuchBeanDefinitionException` at startup |
| Class outside the scanned package tree | Bean never registered |
| Putting `@Bean` in a plain class (not `@Configuration`) | Works partially, but inter-bean calls create **duplicate objects** (see Section 12 on proxyBeanMethods) |
| Making entities/DTOs beans | Wrong — they are per-request data, not shared components |
| Two beans with the same name | `BeanDefinitionOverrideException` (Boot 2.1+ disallows overriding by default) |

## 5.8 TL Questions

**Q: What is a bean?**
A: Any object whose creation, wiring and lifecycle is managed by the Spring container. If Spring created it, it's a bean; if we created it with `new`, it isn't.

**Q: Why does it matter whether something is a bean?**
A: Only beans get dependency injection, AOP proxies and lifecycle callbacks. That is exactly why `@Transactional` doesn't work on an object we instantiated ourselves.

**Q: What's the difference between `@Component` and `@Service`?**
A: Functionally nothing — `@Service` is meta-annotated with `@Component`. It's a semantic marker telling the reader this class holds business logic, and it gives us a clean target for AOP pointcuts. `@Repository` is the one stereotype that adds real behaviour: JDBC/Hibernate exception translation.

**Q: When do we use `@Bean` instead of `@Component`?**
A: When we don't own the class. We can't annotate `RestTemplate` or `ModelMapper`, so we register them from a `@Configuration` method where we control the construction.

**Q: What happens if we remove `@Service` from a class?**
A: Component scanning no longer registers it, so anything injecting it fails at startup with `NoSuchBeanDefinitionException`. The app won't boot.

**Q: How are beans named, and why should we care?**
A: Default is the class name with a lowercase first letter. It matters because `@Qualifier` and name-based resolution use it, and duplicate names cause a startup failure.

## 5.9 Interview Questions

**Beginner — What is a Spring bean?**
An object instantiated, configured and managed by the Spring IoC container.

**Beginner — Name the stereotype annotations.**
`@Component`, `@Service`, `@Repository`, `@Controller`, `@RestController`.

**Intermediate — Are `@Service` and `@Component` interchangeable?**
Technically yes — the container treats them identically. Semantically no: layering information is lost, and some AOP pointcuts / tooling target `@Service` specifically.

**Intermediate — What does `@Repository` actually do beyond marking a bean?**
It activates `PersistenceExceptionTranslationPostProcessor`, which proxies the bean and converts vendor exceptions (`SQLException`, Hibernate's `ConstraintViolationException`) into Spring's consistent `DataAccessException` hierarchy. That keeps the service layer independent of the persistence technology.

**Advanced — Can one class produce multiple beans?**
Yes — declare several `@Bean` methods returning the same type (differently configured), then disambiguate with `@Qualifier` or `@Primary`. A common example is two `DataSource` beans: primary and read-replica.

**Advanced — What is a `BeanDefinition`?**
The metadata object describing how to create a bean: class name, scope, lazy flag, constructor arguments, property values, init/destroy method names, primary flag. The container registers all definitions first, then instantiates from them — the two-phase design that makes ordering and proxying possible.

## 5.10 Quick Revision

1. Bean = object created and managed by the Spring container.
2. `new` → not a bean → no DI, no AOP, no transactions.
3. Three ways to create: stereotype annotations, `@Bean` methods, XML (legacy).
4. `@Service`/`@Repository`/`@Controller` are all `@Component` underneath.
5. `@Repository` uniquely adds **exception translation**.
6. `@Component` = my classes; `@Bean` = third-party classes.
7. Default bean name = class name with lowercase first letter.
8. Entities and DTOs are **not** beans.
9. Duplicate bean names fail the startup in Boot 2.1+.

## 5.11 TL Explanation (speak this)

> "A bean is simply an object the Spring container creates and manages. That management is what gives us injection, transaction proxies and AOP — so anything we build with `new` ourselves is outside all of that. We register our own classes with `@Service` or `@Repository` and let component scanning pick them up, and for third-party classes like `RestTemplate` we register them with an `@Bean` method in a config class. The one stereotype with real hidden behaviour is `@Repository`, which translates database exceptions into Spring's `DataAccessException` hierarchy."

---

[← 4. Dependency Injection (DI)](04-dependency-injection-di.md) | [Contents](../README.md) | [6. Bean Lifecycle →](06-bean-lifecycle.md)
