[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

---

# 4. Dependency Injection (DI)

## 4.1 What is it?

**Simple words:**
A "dependency" is simply another object your class needs to do its job. `OrderService` needs `OrderRepository` — so `OrderRepository` is a dependency.

**Dependency Injection** means: instead of the class building that object itself, someone from outside **hands it in**.

Real-life analogy: you don't build a car to travel. You call a cab — the cab is *injected* into your journey. You don't care about the model, only that it drives.

**Technical wording:**
DI is a design pattern in which an object's dependencies are provided by an external entity (the IoC container) rather than being constructed by the object itself. Spring performs DI by resolving bean definitions and supplying collaborators via constructor, setter, or field injection.

## 4.2 Why do we use it?

- **Loose coupling** — depend on abstraction, not implementation.
- **Testability** — inject mocks in unit tests. This is the single biggest practical benefit.
- **Single Responsibility** — the class does its job; it is not also an object factory.
- **Reusability** — the same service works with any implementation of its dependency.
- **Centralised configuration** — wiring decisions live in one place.

## 4.3 The Three Types of Dependency Injection

### Type 1 — Constructor Injection (RECOMMENDED)

Dependencies are passed through the constructor.

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final PaymentService paymentService;

    // Since Spring 4.3, @Autowired is OPTIONAL when there is only one constructor
    public OrderService(OrderRepository orderRepository,
                        PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
    }
}
```

**Why this is the industry standard:**

| Advantage | Explanation |
|---|---|
| **Truly immutable** | Fields can be `final` — nobody can change them later |
| **Guaranteed non-null** | Object cannot be created without its dependencies |
| **Fail fast** | Missing dependency = startup failure, not a 3 a.m. `NullPointerException` |
| **Easy testing** | `new OrderService(mockRepo, mockPayment)` — no framework needed |
| **Exposes bad design** | A constructor with 8 parameters *screams* that the class does too much. Field injection hides this. |

### Type 2 — Setter Injection

```java
@Service
public class ReportService {

    private EmailService emailService;

    @Autowired    // required here
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

**Use when:** the dependency is genuinely **optional**, or must be **reconfigurable at runtime**. That is rare in normal business code.

### Type 3 — Field Injection (AVOID)

```java
@Service
public class OrderService {

    @Autowired                        // looks convenient...
    private OrderRepository orderRepository;
}
```

**Why teams ban this in code review:**

| Problem | Explanation |
|---|---|
| **Cannot be `final`** | Field is mutable — no immutability guarantee |
| **Hard to unit test** | `new OrderService()` gives an object with `null` fields. You need reflection or a Spring context just to test. |
| **Hides design smells** | You can add 15 dependencies and the class still "looks" fine |
| **Framework lock-in** | The class cannot be constructed properly outside Spring |
| **Silent circular dependencies** | Cycles get resolved quietly instead of failing loudly |

**Spring's own team recommends constructor injection.** Modern IDEs show a warning on `@Autowired` fields for exactly these reasons.

### Comparison Table

| Feature | Constructor | Setter | Field |
|---|---|---|---|
| Immutability (`final`) | Yes | No | No |
| Mandatory dependency | Yes | No | No |
| Fails fast at startup | Yes | No | No |
| Unit-testable without Spring | Yes | Yes | No |
| Optional dependencies | Awkward | Good | Poor |
| Circular dependency | Fails loudly (good) | Allowed | Allowed (hidden) |
| Readability of dependencies | Explicit | Medium | Hidden |
| **Recommended?** | **YES — default** | Rarely | **No** |

## 4.4 Using Lombok to remove constructor boilerplate

Most corporate projects use Lombok:

```java
@Service
@RequiredArgsConstructor     // generates a constructor for all final fields
public class OrderService {

    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    // Lombok writes the constructor at compile time → still constructor injection
}
```

This gives you constructor injection with field-injection-level brevity. **This is the pattern you will see most often in real codebases.**

## 4.5 Important annotations for DI

| Annotation | What it does |
|---|---|
| `@Autowired` | Tells Spring to inject a matching bean. Optional on a single constructor. |
| `@Qualifier("beanName")` | Chooses **which** bean when several match the same type |
| `@Primary` | Marks one bean as the default choice when multiple candidates exist |
| `@Value("${property}")` | Injects a value from properties, not a bean |
| `@Resource` | JSR-250 alternative; matches **by name** first |
| `@Inject` | JSR-330 standard equivalent of `@Autowired` |
| `@Lazy` | Delay creation until first use; can break some circular dependencies |

### Solving "multiple beans of same type"

The problem:

```java
public interface PaymentService { void pay(BigDecimal amount); }

@Service public class CardPaymentService implements PaymentService { ... }
@Service public class UpiPaymentService  implements PaymentService { ... }
```

Injecting `PaymentService` now fails:
`NoUniqueBeanDefinitionException: expected single matching bean but found 2`

**Solution A — `@Qualifier` (explicit, preferred):**

```java
@Service
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(@Qualifier("upiPaymentService") PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**Solution B — `@Primary` (a default winner):**

```java
@Service
@Primary                 // chosen whenever no @Qualifier is given
public class CardPaymentService implements PaymentService { ... }
```

**Solution C — inject all of them (very useful in real projects):**

```java
@Service
public class PaymentRouter {

    private final Map<String, PaymentService> strategies;   // beanName -> bean

    public PaymentRouter(Map<String, PaymentService> strategies) {
        this.strategies = strategies;
    }

    public void pay(String mode, BigDecimal amount) {
        strategies.get(mode + "PaymentService").pay(amount);
    }
}
```
Spring injects **every** implementation into the map. This is the clean way to implement the **Strategy pattern** in Spring — no `if/else` chain, and adding a new payment mode requires no change here.

## 4.6 How Spring resolves a dependency (internal flow)

```
Spring needs to inject a dependency of type PaymentService
              │
              ▼
1. Find ALL beans of type PaymentService
              │
              ├── 0 found ──► Required? ──► NoSuchBeanDefinitionException
              │                    └── optional/@Autowired(required=false) → inject null
              │
              ├── 1 found ──► inject it ✔
              │
              └── many found
                      │
                      ▼
              2. Is there a @Qualifier?  ── yes ──► use that bean ✔
                      │ no
                      ▼
              3. Is any bean @Primary?   ── yes ──► use that bean ✔
                      │ no
                      ▼
              4. Does a bean name match the field/parameter name?
                                         ── yes ──► use that bean ✔
                      │ no
                      ▼
                 NoUniqueBeanDefinitionException ✘
```

Memorise this order: **type → @Qualifier → @Primary → name → error.**

## 4.7 Circular Dependency

**What it is:** A needs B, and B needs A.

```java
@Service
public class AService {
    public AService(BService b) { }     // A needs B
}

@Service
public class BService {
    public BService(AService a) { }     // B needs A
}
```

With **constructor injection**, Spring cannot create either one first, so it fails at startup:
`BeanCurrentlyInCreationException` / "Requested bean is currently in creation".

**Note:** Since Spring Boot 2.6, circular references are **disabled by default** even for field injection. The property `spring.main.allow-circular-references=true` re-enables them.

**How to fix it properly:**

| Fix | When to use |
|---|---|
| **Redesign** — extract the shared logic into a third class `CService` | **Best fix.** A cycle usually means wrong responsibility split. |
| `@Lazy` on one dependency | Injects a proxy, resolving the dependency on first call. A workaround, not a cure. |
| Use setter injection for one side | Works, but hides the design problem |
| Publish an application event instead of calling directly | Good for decoupled notifications |

**What to say to a TL:** *"A circular dependency is a design smell, not a Spring problem. I'd extract the common behaviour into a third component rather than paper over it with `@Lazy`."*

## 4.8 Real-Time Project Example (full layered wiring)

```java
// ============ CONTROLLER LAYER ============
// package com.company.orderservice.controller
@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;      // injected by Spring

    @PostMapping
    public ResponseEntity<OrderResponseDto> create(@Valid @RequestBody OrderRequestDto request) {
        return ResponseEntity.status(HttpStatus.CREATED)
                             .body(orderService.placeOrder(request));
    }
}

// ============ SERVICE LAYER ============
// package com.company.orderservice.service
@Service
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final OrderRepository orderRepository;      // data access
    private final PaymentService paymentService;        // another service
    private final NotificationService notificationService;

    @Override
    @Transactional
    public OrderResponseDto placeOrder(OrderRequestDto request) {
        Order order = new Order(request.getCustomerId(), request.getAmount());
        paymentService.charge(request.getCustomerId(), request.getAmount());
        Order saved = orderRepository.save(order);
        notificationService.notifyCustomer(request.getCustomerId(), "Order confirmed");
        return OrderResponseDto.from(saved);
    }
}

// ============ REPOSITORY LAYER ============
// package com.company.orderservice.repository
public interface OrderRepository extends JpaRepository<Order, Long> { }
```

**Notice:** not a single `new` for any dependency. Every arrow in `Controller → Service → Repository` is wired by the container.

**And the unit test becomes trivial:**

```java
class OrderServiceImplTest {

    @Test
    void shouldPlaceOrder() {
        OrderRepository repo = mock(OrderRepository.class);
        PaymentService payment = mock(PaymentService.class);
        NotificationService notify = mock(NotificationService.class);

        // Constructor injection = we can build the object directly. No Spring needed.
        OrderServiceImpl service = new OrderServiceImpl(repo, payment, notify);

        service.placeOrder(new OrderRequestDto("CUST1", BigDecimal.TEN));

        verify(payment).charge("CUST1", BigDecimal.TEN);
        verify(repo).save(any(Order.class));
    }
}
```
**This test runs in milliseconds and needs no database and no Spring context.** That is the payoff of DI — and the single strongest argument to give a TL for constructor injection.

## 4.9 Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| Field injection everywhere | Untestable, mutable, hides cycles | Constructor injection + `@RequiredArgsConstructor` |
| `new` instead of injection | No AOP, no transactions, `@Autowired` null | Inject it |
| Injecting a concrete class instead of the interface | Tight coupling returns | Depend on the interface |
| Two beans of same type, no `@Qualifier` | `NoUniqueBeanDefinitionException` | `@Qualifier` or `@Primary` |
| Forgetting `@Component`/`@Service` on the class | `NoSuchBeanDefinitionException` | Add the stereotype annotation |
| Injecting a prototype bean into a singleton | Only **one** instance is ever injected | Use `ObjectProvider`/`@Lookup` (see Section 7) |
| `@Autowired` on a static field | Silently does nothing — Spring cannot inject statics | Make it an instance field |

## 4.10 TL Questions

**Q: What is Dependency Injection?**
A: A pattern where a class receives its dependencies from outside instead of creating them. In Spring, the container injects them at startup, usually through the constructor.

**Q: Which injection type are we using and why?**
A: Constructor injection with Lombok's `@RequiredArgsConstructor`. It lets fields be `final`, guarantees dependencies are never null, fails at startup rather than at runtime, and lets us unit test with plain mocks and no Spring context.

**Q: What happens if we remove `@Autowired` from a single-constructor class?**
A: Nothing — it still works. Since Spring 4.3, a class with exactly one constructor is autowired implicitly. `@Autowired` is only required when there are multiple constructors, or for setter/field injection.

**Q: What happens internally when two beans match the same type?**
A: Spring throws `NoUniqueBeanDefinitionException` unless we disambiguate. Its resolution order is: match by type, then `@Qualifier`, then `@Primary`, then bean name matching the parameter name.

**Q: What happens when an error occurs during injection?**
A: The context fails to refresh and the application does not start. That is deliberate — a wiring error surfaces at deployment time instead of as a `NullPointerException` in production traffic.

**Q: Where would you use it in a real project?**
A: Everywhere — every controller, service and repository dependency. A concrete pattern we use is injecting `Map<String, PaymentService>` to get all strategy implementations, which removes if/else routing.

**Q: What do we do about circular dependencies?**
A: Treat them as a design problem and extract the shared logic into a third component. `@Lazy` works as a stop-gap but leaves the bad design in place. Note that Boot 2.6+ rejects circular references by default.

## 4.11 Interview Questions

**Beginner — What are the types of DI in Spring?**
Constructor, setter and field injection. Constructor is recommended.

**Beginner — What is `@Qualifier` for?**
To choose a specific bean by name when multiple beans of the same type exist.

**Intermediate — `@Autowired` vs `@Resource` vs `@Inject`?**

| | `@Autowired` | `@Resource` | `@Inject` |
|---|---|---|---|
| Origin | Spring | JSR-250 (Jakarta) | JSR-330 |
| Matches by | Type first | **Name** first, then type | Type first |
| `required=false` | Supported | Not supported | Not supported |
| Qualifier | `@Qualifier` | `name` attribute | `@Named` |

**Intermediate — Can we inject a `List` of beans?**
Yes. `List<PaymentService>` injects every implementation; `Map<String, PaymentService>` injects them keyed by bean name. Ordering can be controlled with `@Order` or by implementing `Ordered`.

**Intermediate — Why is field injection considered bad?**
Fields cannot be `final`, the object can exist in an invalid state, unit tests need reflection or a container, the class is unusable outside Spring, and unlimited dependencies can be added without the design smell being visible.

**Advanced — How does Spring inject into a `final` field with constructor injection?**
It does not "inject into" the field at all. It calls the constructor with the resolved arguments, and the constructor assigns the `final` field normally — which is exactly why immutability is preserved. Field injection, by contrast, uses **reflection** to set the field after construction, which is why `final` is impossible there.

**Advanced — What is `ObjectProvider` used for?**
It is a deferred, optional lookup handle (`getIfAvailable()`, `getIfUnique()`, `stream()`). It is the clean way to handle an optional dependency, or to pull a fresh prototype instance from inside a singleton without using `@Lookup` or `ApplicationContextAware`.

**Advanced — What happens if a dependency isn't found?**
`NoSuchBeanDefinitionException`, and the context fails to start. To make it tolerant, use `@Autowired(required = false)`, `Optional<MyBean>`, or `ObjectProvider<MyBean>`.

## 4.12 Quick Revision

1. DI = dependencies handed in from outside, never created inside with `new`.
2. Three types: **constructor (use this)**, setter (optional deps), field (avoid).
3. `@Autowired` is optional on a single constructor since Spring 4.3.
4. `@RequiredArgsConstructor` + `final` fields = the standard corporate pattern.
5. Resolution order: **type → @Qualifier → @Primary → bean name → exception.**
6. `List<T>` / `Map<String,T>` injection gives you every implementation — the Strategy pattern.
7. Circular dependency = design smell; fix by extraction, not by `@Lazy`.
8. Constructor injection makes unit tests pure Java — no Spring, no database.
9. Always depend on the **interface**, not the implementation.

## 4.13 TL Explanation (speak this)

> "Dependency injection means our classes declare what they need and the container supplies it. We use constructor injection with `@RequiredArgsConstructor`, so dependencies are `final` and can never be null, and a wiring mistake fails at startup instead of becoming a null pointer in production. The big practical win is testing — because the constructor takes the dependencies, we can pass in Mockito mocks and unit test the service with no Spring context and no database. Where we have several implementations of one interface we either use `@Qualifier` or inject them as a Map and route by key."

---

[← 3. Inversion of Control (IoC)](03-inversion-of-control-ioc.md) | [Contents](../README.md) | [5. Beans →](05-beans.md)
