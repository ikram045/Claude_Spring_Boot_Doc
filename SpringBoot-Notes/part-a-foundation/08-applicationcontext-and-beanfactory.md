[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

---

# 8. ApplicationContext and BeanFactory

## 8.1 What is it?

**Simple words:**
The `ApplicationContext` **is** the Spring container. It is the box that holds all your beans. When your app starts, Spring builds this box, fills it with objects, and wires them together.

**Technical wording:**
`ApplicationContext` is the central interface of the Spring IoC container. It extends `BeanFactory` and adds enterprise features: event publication, internationalization, resource loading, environment/property abstraction, and automatic `BeanPostProcessor`/`BeanFactoryPostProcessor` registration.

## 8.2 BeanFactory vs ApplicationContext

| Feature | `BeanFactory` | `ApplicationContext` |
|---|---|---|
| Basic DI | Yes | Yes |
| Bean instantiation | **Lazy** — on first `getBean()` | **Eager** — singletons created at startup |
| Annotation support (`@Autowired`) | Manual registration needed | Automatic |
| `BeanPostProcessor` registration | Manual | Automatic |
| Event publishing | No | Yes (`ApplicationEventPublisher`) |
| Internationalization (i18n) | No | Yes (`MessageSource`) |
| Resource loading | Basic | Yes (`ResourceLoader`) |
| Environment / properties | No | Yes (`Environment`) |
| AOP integration | Manual | Automatic |
| Memory footprint | Lower | Higher |
| **Used in practice** | Almost never | **Always** |

**Why eager initialization is actually an advantage:**
If a bean is misconfigured, an eager container fails **at startup** — during deployment, when you can roll back safely. A lazy container would fail on the first user request that touches that bean, in production, at an unpredictable time. **Fail fast is a feature, not a cost.**

## 8.3 Implementations you should recognise

| Class | Used for |
|---|---|
| `AnnotationConfigApplicationContext` | Standalone app, Java config |
| `ClassPathXmlApplicationContext` | Legacy XML config |
| `AnnotationConfigServletWebServerApplicationContext` | **What Spring Boot web apps actually use** |
| `GenericWebApplicationContext` | Generic web scenarios |

You almost never instantiate these yourself in Boot — `SpringApplication.run()` picks the right one by detecting the application type.

## 8.4 Useful things you can do with the context

```java
@Component
@RequiredArgsConstructor
public class ContextDemo {

    private final ApplicationContext context;   // the container can inject itself

    public void demo() {
        OrderService service = context.getBean(OrderService.class);       // by type
        OrderService byName  = context.getBean("orderService", OrderService.class);
        String[] names       = context.getBeanDefinitionNames();          // all bean names
        boolean exists       = context.containsBean("orderService");
        Map<String, PaymentService> all = context.getBeansOfType(PaymentService.class);
    }
}
```

> **Important practice note:** calling `getBean()` in business code is an **anti-pattern** — it is the Service Locator pattern, and it re-couples your class to the container, defeating DI. Legitimate uses are limited to framework/infrastructure code, dynamic lookup by runtime key, and tests.

## 8.5 Application Events (a genuinely useful feature)

The context is also an event bus. This is how you decouple side-effects from core business logic.

```java
// 1. The event (a simple immutable carrier)
public record OrderPlacedEvent(Long orderId, String customerId) { }

// 2. Publisher — the core business flow
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ApplicationEventPublisher eventPublisher;
    private final OrderRepository orderRepository;

    @Transactional
    public void placeOrder(Order order) {
        Order saved = orderRepository.save(order);
        // OrderService does NOT know or care who reacts to this
        eventPublisher.publishEvent(new OrderPlacedEvent(saved.getId(), saved.getCustomerId()));
    }
}

// 3. Listeners — side effects, added without touching OrderService
@Component
public class EmailNotificationListener {

    // Runs only AFTER the transaction commits — no email for a rolled-back order
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void onOrderPlaced(OrderPlacedEvent event) {
        // send confirmation email
    }
}

@Component
public class InventoryListener {

    @Async                       // runs on a separate thread
    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        // reserve stock
    }
}
```

**Why a TL will like this:** adding a new reaction to "order placed" (analytics, loyalty points, warehouse notification) means adding a listener class — **zero changes to `OrderService`**. That is the Open/Closed Principle in practice.

**`@EventListener` vs `@TransactionalEventListener`:** by default events are published **synchronously and inside the caller's transaction**. If the transaction later rolls back, a plain `@EventListener` has already sent the email — for a failed order. `@TransactionalEventListener(AFTER_COMMIT)` prevents exactly that. This is a strong senior-level point.

**Built-in events worth knowing:** `ApplicationStartingEvent`, `ApplicationEnvironmentPreparedEvent`, `ContextRefreshedEvent`, `ApplicationReadyEvent`, `ContextClosedEvent`.

## 8.6 TL Questions

**Q: What is the ApplicationContext?**
A: It is the Spring IoC container — the object that holds and manages every bean, resolves dependencies, publishes events and exposes configuration through the `Environment`.

**Q: Why do we use it instead of BeanFactory?**
A: `BeanFactory` only does basic lazy DI. `ApplicationContext` adds annotation support, automatic post-processor registration, AOP integration, events, property handling, and eager singleton creation, which makes configuration errors fail at startup instead of in production traffic.

**Q: What happens internally when the context starts?**
A: It loads bean definitions, runs `BeanFactoryPostProcessor`s on that metadata, then instantiates the singletons, injects dependencies, runs lifecycle callbacks, applies `BeanPostProcessor`s (which create the AOP proxies), starts the embedded web server, and finally publishes `ApplicationReadyEvent`.

**Q: Should we call `context.getBean()` in our services?**
A: No. That is the Service Locator anti-pattern — it couples our class back to the container and defeats dependency injection. We only use it in infrastructure code or where the bean must be chosen by a runtime key, and even then we prefer injecting a `Map<String, T>`.

**Q: Where would you use application events in a real project?**
A: For side effects that shouldn't be in the core flow — sending confirmation emails, updating analytics, reserving inventory after an order. We use `@TransactionalEventListener(AFTER_COMMIT)` so we never notify a customer about an order whose transaction rolled back.

## 8.7 Interview Questions

**Beginner — What is the Spring container?**
The `ApplicationContext` — it creates, configures and manages beans.

**Intermediate — Why are singletons created eagerly?**
To surface configuration and wiring errors at startup, and to avoid first-request latency. It can be made lazy globally with `spring.main.lazy-initialization=true` or per bean with `@Lazy`, which speeds up startup at the cost of moving failures to runtime.

**Advanced — What is `refresh()` in the context lifecycle?**
`AbstractApplicationContext.refresh()` is the template method driving startup: prepare the bean factory, invoke `BeanFactoryPostProcessor`s, register `BeanPostProcessor`s, initialise the `MessageSource` and event multicaster, call `onRefresh()` (which starts the embedded web server in Boot), register listeners, instantiate all remaining singletons, then publish `ContextRefreshedEvent`.

**Advanced — Are Spring events synchronous?**
By default yes — the publisher thread runs the listeners, and an exception in a listener propagates back to the publisher. Add `@Async` (plus `@EnableAsync`) for asynchronous handling, but then be aware that you lose the caller's transaction and exception propagation.

## 8.8 Quick Revision

1. `ApplicationContext` = the Spring IoC container.
2. It extends `BeanFactory` and adds events, i18n, resources, `Environment`, auto post-processors.
3. `BeanFactory` = lazy and minimal; `ApplicationContext` = eager and full-featured.
4. Eager singleton creation gives **fail-fast** behaviour.
5. `refresh()` is the startup template method.
6. `getBean()` in business code = Service Locator anti-pattern.
7. Events decouple side effects from core logic.
8. Use `@TransactionalEventListener(AFTER_COMMIT)` for anything that must not happen on rollback.

## 8.9 TL Explanation (speak this)

> "The `ApplicationContext` is the Spring container itself — it holds every bean, resolves the dependency graph and manages lifecycles. It extends `BeanFactory` but adds annotation support, AOP integration, property handling and events, and it creates singletons eagerly so a wiring mistake breaks the startup rather than a live request. We also use it as an event bus: `OrderService` publishes an `OrderPlacedEvent` and separate listeners handle email and inventory, so adding a new side effect never means editing the order logic."

---

[← 7. Bean Scopes](07-bean-scopes.md) | [Contents](../README.md) | [9. @SpringBootApplication →](09-springbootapplication.md)
