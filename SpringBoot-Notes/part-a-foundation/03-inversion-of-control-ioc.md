[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

---

# 3. Inversion of Control (IoC)

## 3.1 What is it?

**Simple words:**
Normally, *your* code is in charge. Your class decides when to create the objects it needs:

```java
OrderService service = new OrderService();   // I am in control
```

**Inversion of Control** means flipping that. The **framework** is in charge of creating objects. Your class just says "I need an `OrderService`" and the framework hands one over.

The word "inversion" = the **control of object creation has been inverted** — moved from your code to the container.

**Technical wording:**
IoC is a design principle in which the control of object creation, configuration and lifecycle is transferred from the application code to an external container. In Spring, this container is the `ApplicationContext`, and the objects it manages are called **beans**.

## 3.2 Why do we use it?

Look at the problem concretely.

**Without IoC (tightly coupled):**

```java
public class OrderService {

    // OrderService itself decides which repository implementation to use
    private OrderRepository repository = new MySqlOrderRepository();

    public void placeOrder(Order order) {
        repository.save(order);
    }
}
```

Problems:

1. **Tight coupling** — `OrderService` is permanently bonded to `MySqlOrderRepository`. Switching to Postgres means editing `OrderService`.
2. **Not testable** — a unit test will hit a real MySQL database. You cannot substitute a fake.
3. **Duplicated creation logic** — if `MySqlOrderRepository` needs a connection string, every class creating it must know it.
4. **No lifecycle control** — nobody manages when the object is created or destroyed.

**With IoC (loosely coupled):**

```java
@Service
public class OrderService {

    private final OrderRepository repository;   // just an interface

    // "I need an OrderRepository. I don't care which one. Container, give me one."
    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }

    public void placeOrder(Order order) {
        repository.save(order);
    }
}
```

Now `OrderService` has **no idea** which implementation it gets. That decision moved to the container.

## 3.3 What problem does it solve?

| Problem | IoC solution |
|---|---|
| Tight coupling between classes | Classes depend on interfaces; container supplies implementations |
| Hard to unit test | Inject mocks instead of real dependencies |
| Object creation scattered everywhere | Centralised in the container |
| No lifecycle management | Container manages create → init → use → destroy |
| Hard to swap implementations | Change one annotation/config, not the consuming code |

## 3.4 The Hollywood Principle

The classic way to remember IoC:

> **"Don't call us, we'll call you."**

Your class does not call the framework to fetch dependencies. The framework calls your class to give them.

## 3.5 IoC vs DI — the most common confusion

People use these words interchangeably. They are **not** the same.

| | IoC | DI |
|---|---|---|
| **What it is** | A **principle** / concept | A **design pattern** — one way to implement IoC |
| **Meaning** | Control of object creation moves to a container | Dependencies are *supplied from outside* rather than created inside |
| **Scope** | Broad (also covers template method, event-driven flows, callbacks) | Narrow and specific |
| **Relationship** | DI is **one implementation of** IoC | DI is the technique Spring uses to achieve IoC |

**One line for interviews:** *"IoC is the principle. DI is how Spring implements that principle."*

## 3.6 How does it work? (Internal flow)

```
STEP 1  Application starts
           │
           ▼
STEP 2  Spring creates the IoC container (ApplicationContext)
           │
           ▼
STEP 3  Container scans packages for @Component/@Service/@Repository/
        @Controller and reads @Bean methods
           │
           ▼
STEP 4  Container builds a BEAN DEFINITION for each
        (class name, scope, dependencies, init/destroy methods)
           │  — at this point NO objects exist yet, only "blueprints"
           ▼
STEP 5  Container resolves the dependency graph
        (OrderService needs OrderRepository → create repository FIRST)
           │
           ▼
STEP 6  Container instantiates beans and INJECTS dependencies
           │
           ▼
STEP 7  Beans are stored in the container, ready to use
           │
           ▼
STEP 8  On shutdown, container calls destroy callbacks
```

**Key insight for interviews:** Spring works in **two phases** — first it reads *bean definitions* (metadata), then it *instantiates*. That two-phase design is exactly what allows it to resolve dependency order, detect circular dependencies, and apply proxies (AOP, transactions) before handing the bean to you.

## 3.7 Types of IoC Containers

| Container | Interface | Description |
|---|---|---|
| **BeanFactory** | `org.springframework.beans.factory.BeanFactory` | Basic container. **Lazy** — creates a bean only when requested. Minimal features. |
| **ApplicationContext** | `org.springframework.context.ApplicationContext` | Extends `BeanFactory`. **Eager** — creates singletons at startup. Adds events, i18n, AOP integration, annotation support. |

**In practice you always use `ApplicationContext`.** `BeanFactory` matters only for very memory-constrained environments and for interview questions. Full comparison in **Section 8**.

## 3.8 Real-Time Project Example

**Scenario:** An order service that must send a notification after an order is placed. The business wants email today, and SMS later without changing the service.

```java
// ---------- package: com.company.orderservice.notification ----------
public interface NotificationService {
    void notifyCustomer(String customerId, String message);
}

@Service
public class EmailNotificationService implements NotificationService {
    @Override
    public void notifyCustomer(String customerId, String message) {
        System.out.println("EMAIL to " + customerId + ": " + message);
    }
}

// ---------- package: com.company.orderservice.service ----------
@Service
public class OrderService {

    private final NotificationService notificationService;

    // OrderService depends on the INTERFACE only.
    public OrderService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void placeOrder(Order order) {
        // ... business logic to save the order ...
        notificationService.notifyCustomer(order.getCustomerId(), "Order placed");
    }
}
```

**The payoff:** When the business asks for SMS instead of email, we write `SmsNotificationService implements NotificationService`, mark it as the active bean, and **`OrderService` is not touched at all**. That is the business value of IoC — cheaper change.

## 3.9 Common Mistakes

| Mistake | Result |
|---|---|
| Using `new` inside a Spring bean for another bean | That object is **not** managed — no injection, no transactions, no AOP. A very common cause of "`@Transactional` is not working" and "`@Autowired` field is null". |
| Thinking IoC and DI are identical | Interview mark lost |
| Expecting Spring to manage objects you created yourself | Spring only manages what it creates |

**The `new` trap — memorise this:**

```java
@Service
public class OrderService {
    // WRONG — Spring does not manage this object
    private PaymentService paymentService = new PaymentService();
}
```
Inside that manually created `PaymentService`, every `@Autowired` field will be `null` and every `@Transactional` method will be non-transactional, because Spring never created or proxied it.

## 3.10 TL Questions

**Q: What is IoC?**
A: It is the principle that the framework, not our code, is responsible for creating objects and wiring their dependencies. We declare what we need; the container supplies it.

**Q: Why are we using it?**
A: It gives loose coupling. Our services depend on interfaces, so implementations can be swapped and mocked. Without it, every class hard-codes its dependencies and becomes untestable.

**Q: How does it work internally?**
A: At startup Spring scans for components, builds bean definitions as metadata, resolves the dependency graph in the right order, instantiates the beans and injects dependencies, then keeps them in the context.

**Q: What happens if we create a dependency with `new` inside a bean?**
A: That object is outside the container. It gets no injection, no transaction proxy and no AOP. This is the usual root cause when someone reports that `@Transactional` or `@Autowired` "isn't working".

**Q: What problem does it solve for the team?**
A: Change cost. When a requirement changes — a new payment provider, a new notification channel — we add an implementation instead of editing and re-testing existing service code.

**Q: What alternative can we use?**
A: Manual dependency injection (plain constructors wired in a factory), or another DI container like Google Guice or CDI. Manual DI is fine for a tiny app but does not scale to hundreds of beans with lifecycles and proxies.

## 3.11 Interview Questions

**Beginner — What does "inversion" mean in IoC?**
The control over creating and wiring objects is inverted — moved out of the application class into the container.

**Beginner — What is the Spring IoC container?**
The `ApplicationContext` (or the simpler `BeanFactory`). It creates beans, injects dependencies, and manages their lifecycle.

**Intermediate — Is IoC the same as DI?**
No. IoC is the principle; DI is a pattern implementing it. Other IoC forms exist, for example the template method pattern and event/callback-driven designs.

**Intermediate — How does the container know the creation order?**
It builds a dependency graph from the bean definitions. A bean is created only after all its constructor dependencies exist. If it detects a cycle in constructor injection, it fails fast with `BeanCurrentlyInCreationException`.

**Advanced — Why does Spring separate bean definition from instantiation?**
Because the metadata phase lets Spring order creation correctly, apply `BeanFactoryPostProcessor`s (e.g. property placeholder resolution) before any object exists, detect cycles, and decide where proxies are needed — all before committing to real objects.

**Advanced — Does IoC have any downsides?**
Yes: runtime wiring errors instead of compile-time ones, a startup cost for scanning and instantiation, harder debugging through proxies, and a learning curve. Compile-time DI frameworks (Dagger, Micronaut) trade flexibility for these.

## 3.12 Quick Revision

1. IoC = control of object creation moves from your code to the container.
2. IoC is the **principle**; DI is the **pattern** that implements it.
3. The container is the `ApplicationContext`.
4. Container works in two phases: **bean definitions**, then **instantiation**.
5. Benefit: loose coupling, testability, centralised lifecycle.
6. "Don't call us, we'll call you" — the Hollywood Principle.
7. `new` inside a bean = object outside the container = no DI, no AOP, no transactions.
8. `BeanFactory` is lazy and basic; `ApplicationContext` is eager and full-featured.

## 3.13 TL Explanation (speak this)

> "IoC means the container owns object creation instead of our classes doing it with `new`. Our service just declares the interface it needs in its constructor, and Spring injects a suitable implementation at startup. That keeps our layers loosely coupled — we can swap an implementation or inject a mock in tests without touching the consuming class. The important practical rule is that anything we create with `new` ourselves is outside the container, so it won't get transactions or AOP."

---

[← 2. Spring Boot](02-spring-boot.md) | [Contents](../README.md) | [4. Dependency Injection (DI) →](04-dependency-injection-di.md)
