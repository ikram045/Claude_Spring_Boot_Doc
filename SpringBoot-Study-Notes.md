# Spring Boot — Complete Study Notes

**Prepared for:** Corporate Java / Spring Boot development work
**Level:** Beginner → Intermediate → TL-discussion ready
**Format:** Professional technical documentation

---

## About This Document (Read This First)

**Source note — important for honesty:**
No playlist transcript or notes file was supplied before this document was generated. The workspace was empty. This document was therefore written from the **topic curriculum you listed** in your request, using standard, widely-accepted Spring Boot behaviour.

That means:

- Every topic here comes from **your requested topic list**, not from a video transcript.
- Where I explain something that is deeper than a normal tutorial would cover, it is labelled **Additional clarification**.
- When you send your actual playlist parts, I will **merge them into this same document** — deepening sections where your videos go further, and flagging anything where your instructor's explanation differs from what is written here.

**How to use this document:**

| If you want to... | Go to... |
|---|---|
| Learn a topic properly | The numbered topic sections |
| Revise the night before a discussion | "Quick Revision" boxes at the end of each topic |
| Speak to your TL/senior confidently | "TL Explanation" boxes (3–5 lines, speak them as-is) |
| Prepare for interviews | "Interview Questions" in each topic + Section 55 |
| Understand the big picture | Part F — Master Flows (Sections 47–56) |

**Reading tip:** Every topic first explains the idea in **plain, simple words**, then gives the **technical/corporate wording** you should actually use in a meeting. Learn both. Speak the second one.

---

# TABLE OF CONTENTS

## PART A — FOUNDATION (Core Spring)
1. Spring Framework
2. Spring Boot
3. Inversion of Control (IoC)
4. Dependency Injection (DI)
5. Beans
6. Bean Lifecycle
7. Bean Scopes
8. ApplicationContext and BeanFactory
9. @SpringBootApplication
10. Auto-Configuration
11. Component Scanning
12. Java Configuration — @Configuration and @Bean
13. application.properties and application.yml
14. Profiles

## PART B — WEB LAYER (Spring MVC / REST)
15. Spring MVC and the DispatcherServlet
16. HandlerMapping and HandlerAdapter
17. REST API Fundamentals
18. Controllers
19. Reading Request Data — @PathVariable, @RequestParam, @RequestBody, Headers
20. Sending Responses — ResponseEntity and HTTP Status Codes
21. DTO Pattern
22. Validation
23. Exception Handling
24. Filters
25. Interceptors
26. AOP (Aspect Oriented Programming)

## PART C — DATA LAYER (JDBC / JPA / Hibernate)
27. JDBC and DataSource
28. HikariCP (Connection Pooling)
29. JdbcTemplate
30. JPA vs Hibernate vs Spring Data JPA
31. Entity and Mapping Annotations
32. EntityManager and Persistence Context
33. First Level Cache
34. Entity Lifecycle States
35. Spring Data Repositories
36. Query Methods, JPQL, @Query and Pagination
37. Entity Relationships
38. Cascade Types
39. Lazy vs Eager Loading
40. The N+1 Select Problem and EntityGraph
41. Transactions and @Transactional

## PART D — SECURITY
42. Spring Security
43. JWT (JSON Web Token) Authentication

## PART E — BUILD, CONFIG AND DEPLOYMENT
44. Maven
45. Spring Boot Actuator and Production Readiness
46. Packaging and Deployment

## PART F — MASTER FLOWS AND FINAL REVISION
47. Complete Spring Boot Architecture Flow
48. Complete REST API Request/Response Flow
49. Complete JPA/Hibernate Flow
50. Complete Spring Security / JWT Flow
51. Complete Exception Handling Flow
52. Complete Transaction Flow
53. Filter vs Interceptor vs AOP — Complete Comparison
54. Most Important Spring Boot Annotations
55. Most Important TL / Senior-Level Interview Questions
56. One-Page Spring Boot Quick Revision Sheet

---
---

# PART A — FOUNDATION (CORE SPRING)

---

# 1. Spring Framework

## 1.1 What is it?

**Simple words:**
Spring is a big toolbox for building Java applications. Before Spring, if your class needed another class, you had to create it yourself with `new`. You also had to write a lot of repeated plumbing code — opening database connections, closing them, handling transactions, wiring objects together.

Spring takes over that plumbing. **You write the business logic; Spring creates and connects the objects for you.**

**Technical wording:**
Spring Framework is an open-source, lightweight application framework for the Java platform. Its core is an **IoC (Inversion of Control) container** that manages the lifecycle and dependencies of application objects (called *beans*). Around that core, Spring provides modules for web (Spring MVC), data access (Spring JDBC, Spring ORM), transaction management, AOP, security, testing and messaging.

## 1.2 Why do we use it?

- **Loose coupling** — classes depend on interfaces, not on concrete `new` objects. Easier to change and test.
- **Less boilerplate** — no manual JDBC connection open/close, no manual transaction commit/rollback.
- **Modular** — take only what you need (just JDBC, or just web, etc.).
- **Testable** — dependencies are injected, so in a test you inject a fake/mock instead of a real database.
- **Industry standard** — nearly every Java backend job uses it, so the ecosystem and documentation are huge.

## 1.3 What problem does it solve?

Before Spring, Java enterprise development had these pain points:

| Problem (before Spring) | How Spring solves it |
|---|---|
| Objects created with `new` everywhere → tight coupling | IoC container creates objects and injects them |
| Hard to unit test (can't replace a real DB) | Inject a mock implementation instead |
| Repeated JDBC try/catch/finally code | `JdbcTemplate` / Spring Data handles it |
| Manual transaction begin / commit / rollback | `@Transactional` handles it declaratively |
| Cross-cutting code (logging, security) copied into every method | AOP applies it in one place |
| Heavy EJB servers needed | Runs in a plain JVM, POJO-based |

## 1.4 How does it work? (Very short version)

1. Spring reads your **configuration** (annotations, Java config, or XML in old projects).
2. It builds an **IoC container** (`ApplicationContext`).
3. It creates the required objects (**beans**) inside that container.
4. It **injects** each bean's dependencies into it.
5. Your code simply asks for a bean, or receives it injected, and uses it.

```
Configuration (annotations / @Bean methods)
              │
              ▼
      Spring IoC Container (ApplicationContext)
              │  creates + wires
              ▼
   Your Beans (Controller, Service, Repository)
              │
              ▼
        Ready-to-use application
```

## 1.5 Main Spring modules (know these names)

| Module | What it gives you |
|---|---|
| **spring-core / spring-beans** | The IoC container and DI itself |
| **spring-context** | `ApplicationContext`, events, annotation support |
| **spring-aop** | Aspect Oriented Programming |
| **spring-web / spring-webmvc** | Servlet-based web and REST support |
| **spring-jdbc** | `JdbcTemplate`, `DataSource` support |
| **spring-orm** | Hibernate/JPA integration |
| **spring-tx** | Transaction management (`@Transactional`) |
| **spring-test** | Testing support |

**Note:** Spring Framework is the *core*. Spring Boot, Spring Data, Spring Security are **separate projects built on top of it**. That is why people say "the Spring ecosystem".

## 1.6 TL Questions

**Q: What is Spring Framework in one line?**
A: It is a Java framework whose core job is to create your objects, inject their dependencies, and handle repetitive infrastructure work like transactions and data access, so we only write business logic.

**Q: Why are we using Spring instead of plain Java?**
A: Plain Java forces us to create dependencies manually with `new`, which makes classes tightly coupled and very hard to unit test. Spring injects dependencies, so we code against interfaces, swap implementations easily, and mock them in tests.

**Q: What problem does it actually solve in our project?**
A: It removes infrastructure boilerplate. Without it we would write connection handling, transaction commit/rollback and object wiring by hand in every class — that is where most production bugs and leaks come from.

**Q: What is the difference between Spring and Spring Boot?**
A: Spring is the framework itself. Spring Boot is a layer on top that auto-configures Spring, bundles an embedded server, and removes manual setup. Spring Boot does not replace Spring — it configures it for us.

**Q: What alternative could we use?**
A: Jakarta EE (formerly Java EE) with CDI, Micronaut, or Quarkus. Micronaut and Quarkus resolve dependencies at compile time so they start faster, but Spring has by far the largest ecosystem and hiring pool, which usually decides it for corporate projects.

## 1.7 Interview Questions

**Beginner — What are the main features of Spring?**
IoC/DI, AOP, transaction management, data access abstraction (JDBC/ORM), MVC web framework, testing support, and a huge integration ecosystem.

**Beginner — Is Spring heavyweight?**
No. Spring is *lightweight* in the sense that it is POJO-based — your classes do not extend or implement framework classes, and it does not require a heavy application server. It runs in a plain JVM.

**Intermediate — What does "loose coupling" actually mean here?**
Class `A` declares that it needs a `PaymentGateway` **interface**. It never decides *which* implementation. Spring decides that and injects it. So replacing `RazorpayGateway` with `StripeGateway` needs zero change inside `A`.

**Intermediate — Spring Framework vs Spring Boot vs Spring Data — how are they related?**
Spring Framework is the base container. Spring Boot auto-configures the framework and provides opinionated defaults + embedded server. Spring Data is a separate project that generates repository implementations for you. Boot ties them together with starters.

**Advanced — Why is POJO-based design considered a major Spring advantage?**
Because your business classes stay free of framework types. They can be instantiated and tested with plain JUnit, without starting a container. It also means the framework can be upgraded or replaced with much less code churn.

## 1.8 Quick Revision

1. Spring = Java framework whose core is an **IoC container**.
2. Its two headline features are **IoC/DI** and **AOP**.
3. It makes code **loosely coupled** and **testable**.
4. It removes **boilerplate**: JDBC plumbing, transaction handling, object wiring.
5. It is **modular** — use only the modules you need.
6. It is **POJO-based** — your classes don't extend framework classes.
7. Spring Framework ≠ Spring Boot. Boot sits **on top of** Spring.
8. Everything Spring manages is called a **bean**.

## 1.9 TL Explanation (speak this)

> "Spring is the core framework we build on. Its main job is dependency injection — instead of our classes creating their own dependencies with `new`, the Spring container creates them and injects them. That keeps the layers loosely coupled and lets us mock dependencies in unit tests. On top of that it gives us declarative transactions and data-access abstractions, so we don't write JDBC boilerplate. Spring Boot then sits on top and auto-configures all of that for us."

---

# 2. Spring Boot

## 2.1 What is it?

**Simple words:**
Plain Spring is powerful but needs a lot of setup — you must declare which database pool to use, configure the web layer, set up the view resolver, package a WAR and deploy it to a Tomcat server. That setup is repetitive and every project does almost the same thing.

Spring Boot says: *"I already know what a normal project looks like. I will configure it for you, and I will even bring my own web server. You just start writing code."*

**Technical wording:**
Spring Boot is an **opinionated, convention-over-configuration** layer on top of Spring Framework. It provides **auto-configuration**, **starter dependencies**, an **embedded servlet container**, and **production-ready features** (Actuator), enabling stand-alone, executable Spring applications with minimal explicit configuration.

## 2.2 Why do we use it?

| Benefit | Meaning in practice |
|---|---|
| **Auto-configuration** | Sees libraries on the classpath and configures them automatically |
| **Starter dependencies** | One dependency pulls a whole, version-compatible stack |
| **Embedded server** | Tomcat/Jetty/Undertow runs inside the app — no external server install |
| **Executable JAR** | `java -jar app.jar` and it runs. Perfect for Docker/cloud |
| **No XML** | Configuration through `application.properties` / `application.yml` |
| **Actuator** | Health checks, metrics, info endpoints out of the box |
| **Opinionated defaults** | Sensible defaults you can always override |

## 2.3 What problem does it solve?

Before Spring Boot, a new Spring web project required:

- Manually choosing compatible versions of Spring Core, Spring MVC, Jackson, Hibernate, validation API, logging… (version conflicts were a daily problem — "JAR hell")
- Writing a `web.xml` or a `WebApplicationInitializer` to register the `DispatcherServlet`
- Configuring a `DataSource`, `EntityManagerFactory`, `TransactionManager` by hand
- Building a WAR and deploying it to an externally installed Tomcat

Spring Boot removes **all four**.

## 2.4 The Four Pillars of Spring Boot

```
┌──────────────────────────────────────────────┐
│               SPRING BOOT                     │
├───────────────┬───────────────┬──────────────┤
│  1. Starters  │ 2. Auto-Config│ 3. Embedded  │
│               │               │    Server    │
│  version-     │ configure     │ no external  │
│  managed dep  │ beans based   │ Tomcat needed│
│  bundles      │ on classpath  │              │
├───────────────┴───────────────┴──────────────┤
│  4. Actuator — health, metrics, production    │
└──────────────────────────────────────────────┘
```

### Pillar 1 — Starters

A **starter** is a dependency that pulls in a curated, version-compatible set of libraries.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

That single line brings in: Spring MVC, embedded Tomcat, Jackson (JSON), Hibernate Validator, spring-core, logging — all at versions guaranteed to work together.

**Common starters to remember:**

| Starter | Gives you |
|---|---|
| `spring-boot-starter-web` | REST APIs, Spring MVC, embedded Tomcat, Jackson |
| `spring-boot-starter-data-jpa` | Spring Data JPA, Hibernate, HikariCP, spring-tx |
| `spring-boot-starter-security` | Spring Security |
| `spring-boot-starter-validation` | Hibernate Validator (`@NotNull`, `@Email`…) |
| `spring-boot-starter-test` | JUnit 5, Mockito, AssertJ, Spring Test |
| `spring-boot-starter-actuator` | Health/metrics endpoints |
| `spring-boot-starter-thymeleaf` | Server-side HTML templates |

### Pillar 2 — Auto-Configuration
Covered in depth in **Section 10**.

### Pillar 3 — Embedded Server
The web server is a **library inside your JAR**, not a separate product you install.

```
OLD WAY:                          SPRING BOOT WAY:
build app.war                     build app.jar (server inside)
install Tomcat on server          java -jar app.jar
copy WAR into webapps/            done
restart Tomcat
```

This is what makes Spring Boot ideal for **Docker and cloud deployment** — the container image only needs a JRE.

### Pillar 4 — Actuator
Covered in **Section 45**.

## 2.5 Spring vs Spring Boot — Comparison

| Aspect | Spring Framework | Spring Boot |
|---|---|---|
| **Purpose** | Core container + modules | Auto-configure Spring, reduce setup |
| **Configuration** | Manual (XML or Java config) | Auto-configuration + properties |
| **Dependency management** | You pick every version | Starters manage versions |
| **Server** | External Tomcat/JBoss required | Embedded server included |
| **Packaging** | Usually WAR | Usually executable JAR |
| **Boilerplate** | High | Very low |
| **Flexibility** | Total control | Opinionated, but fully overridable |
| **Time to first API** | Hours | Minutes |
| **Production tooling** | Add yourself | Actuator built in |

**Key sentence for interviews:** *"Spring Boot is not a replacement for Spring. It is Spring, pre-configured."*

## 2.6 A Minimal Spring Boot Application

**Package:** `com.company.orderservice`

```java
package com.company.orderservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication          // 1
public class OrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);   // 2
    }
}
```

**Line explanations:**

1. `@SpringBootApplication` — the master annotation. It turns on component scanning, auto-configuration and Java configuration. Fully explained in **Section 9**.
2. `SpringApplication.run(...)` — this one call:
   - creates the `ApplicationContext` (the IoC container),
   - runs auto-configuration,
   - scans and registers all your beans,
   - starts the embedded Tomcat server,
   - keeps the application running.

That is the entire startup of a Spring Boot app.

## 2.7 Standard Project Structure (follow this in real projects)

```
src/main/java/com/company/orderservice/
│
├── OrderServiceApplication.java        ← main class (root package)
│
├── controller/          ← REST endpoints only. No business logic.
│     └── OrderController.java
│
├── service/             ← Business logic + @Transactional lives here
│     ├── OrderService.java             (interface)
│     └── impl/OrderServiceImpl.java    (implementation)
│
├── repository/          ← Database access only
│     └── OrderRepository.java
│
├── entity/              ← JPA entities (@Entity) = DB table mapping
│     └── Order.java
│
├── dto/                 ← Request/response objects exposed to clients
│     ├── OrderRequestDto.java
│     └── OrderResponseDto.java
│
├── mapper/              ← Entity ↔ DTO conversion
│     └── OrderMapper.java
│
├── exception/           ← Custom exceptions + @RestControllerAdvice
│     ├── ResourceNotFoundException.java
│     └── GlobalExceptionHandler.java
│
├── config/              ← @Configuration classes (Security, Swagger, beans)
│     └── SecurityConfig.java
│
└── util/                ← Helpers, constants
```

```
src/main/resources/
├── application.properties      ← or application.yml
├── application-dev.properties  ← profile-specific
├── application-prod.properties
└── static/ , templates/
```

**Golden rule of the main class location:**
`OrderServiceApplication.java` must sit in the **root package**, above all other packages. Component scanning starts from that package downward. If you put the main class inside `controller/`, Spring will never find `service/` or `repository/` — a classic beginner bug.

## 2.8 Common Mistakes

| Mistake | What happens | Fix |
|---|---|---|
| Main class not in root package | `NoSuchBeanDefinitionException` for your services | Move main class up to the root package |
| Writing business logic in controllers | Untestable, unreusable, no transactions | Keep controllers thin, logic in `@Service` |
| Exposing `@Entity` directly in REST responses | Leaks DB structure + lazy-loading errors | Use DTOs |
| Specifying versions for starter dependencies | Version conflicts return | Let the Boot parent POM manage versions |
| Thinking Boot "replaces" Spring | Wrong mental model in interviews | Boot *configures* Spring |

## 2.9 TL Questions

**Q: What is Spring Boot?**
A: It is a layer on top of Spring Framework that auto-configures the application based on what is on the classpath, manages dependency versions through starters, and embeds the web server so we can ship one runnable JAR.

**Q: Why are we using Spring Boot in this project?**
A: It removes the entire manual setup — no version juggling, no `web.xml`, no external Tomcat installation. We get a production-ready service faster and with fewer configuration bugs.

**Q: How does it work internally at startup?**
A: `SpringApplication.run()` creates the `ApplicationContext`, component-scans our packages to register beans, runs auto-configuration classes conditionally based on the classpath, then starts the embedded server.

**Q: What happens if we remove `@SpringBootApplication`?**
A: No component scanning and no auto-configuration. The context will start empty — no controllers, no services, no DataSource, no embedded server configuration. The application effectively does nothing.

**Q: What if we need to override what Boot auto-configured?**
A: Just define our own bean of that type. Auto-configuration is conditional — most auto-config classes are annotated `@ConditionalOnMissingBean`, so our bean wins. We can also exclude a specific auto-config class via `@SpringBootApplication(exclude = ...)`.

**Q: Where would you use it in a real project?**
A: Every microservice we build. Each service is a Boot app producing an executable JAR that goes into a Docker image and runs on Kubernetes.

**Q: What alternative can we use?**
A: Quarkus or Micronaut for faster startup and lower memory (they do dependency resolution at build time), or plain Jakarta EE. For our team, Boot wins on ecosystem maturity and available skills.

## 2.10 Interview Questions

**Beginner — What are Spring Boot starters?**
Curated dependency bundles. Adding `spring-boot-starter-web` brings Spring MVC, Tomcat, Jackson and validation at mutually compatible versions, so we don't manage versions manually.

**Beginner — Can we still use an external Tomcat?**
Yes. Change packaging to `war`, extend `SpringBootServletInitializer`, and mark `spring-boot-starter-tomcat` as `provided`. Some corporate environments still require WAR deployment.

**Intermediate — How does Spring Boot know which database to configure?**
Auto-configuration inspects the classpath. If it finds a JDBC driver and a `spring.datasource.url` property, `DataSourceAutoConfiguration` creates a HikariCP `DataSource`. If it also finds Hibernate, `HibernateJpaAutoConfiguration` creates the `EntityManagerFactory` and `JpaTransactionManager`.

**Intermediate — Is Spring Boot's "opinionated" nature a limitation?**
No, because every opinion is a **default**, not a rule. Defining your own bean or setting a property overrides it. The opinion only applies when you have expressed no preference.

**Advanced — What actually happens inside `SpringApplication.run()`?**
It deduces the application type (servlet/reactive/none), loads `ApplicationContextInitializer`s and `ApplicationListener`s from `spring.factories`, prepares the `Environment` (properties, profiles), prints the banner, creates the appropriate `ApplicationContext`, loads bean definitions, refreshes the context (which instantiates singletons, runs `BeanPostProcessor`s and starts the embedded server), then calls any `ApplicationRunner`/`CommandLineRunner` beans.

**Advanced — Executable JAR vs normal JAR?**
A Boot "fat JAR" nests dependency JARs under `BOOT-INF/lib` and uses a custom `JarLauncher` plus a nested-JAR-aware classloader, because standard Java cannot load a JAR inside a JAR. Your code lives in `BOOT-INF/classes`.

## 2.11 Quick Revision

1. Spring Boot = Spring + auto-configuration + starters + embedded server + Actuator.
2. It **configures** Spring; it does not replace it.
3. Starters solve **version compatibility**; auto-config solves **bean configuration**.
4. Embedded server → executable JAR → ideal for Docker/cloud.
5. `SpringApplication.run()` creates the context and starts everything.
6. Main class must be in the **root package**.
7. Every default is **overridable** — define your own bean or set a property.
8. Standard layering: Controller → Service → Repository → Entity, with DTOs at the edge.

## 2.12 TL Explanation (speak this)

> "Spring Boot is Spring with the setup already done. Starters give us version-compatible dependency bundles, auto-configuration creates the standard beans based on what's on the classpath, and the embedded Tomcat means we ship a single executable JAR straight into a Docker image. Everything it configures is conditional, so the moment we define our own bean or property, ours wins. It saves us the whole XML and server-installation layer without taking away control."

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

# 9. @SpringBootApplication

## 9.1 What is it?

**Simple words:**
It is the one annotation you put on your main class. It is a shortcut that switches on three things at once.

**Technical wording:**
`@SpringBootApplication` is a convenience meta-annotation combining `@SpringBootConfiguration`, `@EnableAutoConfiguration` and `@ComponentScan` with sensible defaults.

## 9.2 What it is made of

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@SpringBootConfiguration        // = @Configuration → this class can declare @Bean methods
@EnableAutoConfiguration        // turn on Spring Boot's auto-configuration
@ComponentScan(                 // scan this package and everything below it
    excludeFilters = { ... }
)
public @interface SpringBootApplication { }
```

| Composed annotation | What it turns on |
|---|---|
| **`@SpringBootConfiguration`** | Marks the class as a configuration source. It is `@Configuration` specialised — Boot's test framework searches for exactly this annotation to locate your main config, which is why there must be only **one** per application. |
| **`@EnableAutoConfiguration`** | Activates auto-configuration: Boot inspects the classpath and configures beans automatically. |
| **`@ComponentScan`** | Scans the **package of this class and all sub-packages** for `@Component`, `@Service`, `@Repository`, `@Controller`, `@Configuration`. |

**So it is equivalent to writing:**

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class OrderServiceApplication { }
```

## 9.3 Why the main class location matters

```
com.company.orderservice
│
├── OrderServiceApplication.java   ← @SpringBootApplication HERE
│                                     scanning starts at com.company.orderservice
├── controller/     ✔ scanned
├── service/        ✔ scanned
├── repository/     ✔ scanned
└── config/         ✔ scanned

com.company.shared
└── AuditService.java              ✘ NOT scanned — different package tree!
```

If you need beans from outside the tree:

```java
@SpringBootApplication(scanBasePackages = {"com.company.orderservice", "com.company.shared"})
public class OrderServiceApplication { }
```

## 9.4 Useful attributes

```java
// Exclude an auto-configuration class you don't want
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class MyApp { }
// Real use: a service with no database — without this, Boot fails at startup
// complaining it cannot determine a suitable JDBC URL.

// Exclude by class name (when the class isn't on the compile classpath)
@SpringBootApplication(excludeName = "org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration")

// Scan extra packages
@SpringBootApplication(scanBasePackages = "com.company")
```

## 9.5 Common Mistakes

| Mistake | Consequence |
|---|---|
| Main class inside a sub-package (e.g. `controller`) | Other packages never scanned → `NoSuchBeanDefinitionException` |
| Main class in the **default package** (no `package` statement) | Spring would scan the entire classpath — Boot refuses to start |
| Multiple `@SpringBootApplication` classes | Test context loading becomes ambiguous |
| Adding `@ComponentScan` again on the same class | **Overrides** the built-in one — if you specify packages, the default (own package) is no longer included unless you list it |

## 9.6 TL Questions

**Q: What is `@SpringBootApplication`?**
A: A meta-annotation on the main class that combines `@SpringBootConfiguration`, `@EnableAutoConfiguration` and `@ComponentScan` — so one annotation enables configuration, auto-configuration and component scanning.

**Q: What happens if we remove it?**
A: No component scanning and no auto-configuration. The context starts essentially empty — no controllers, services, `DataSource` or web server configuration — so the application does nothing.

**Q: Why must the main class sit in the root package?**
A: Because `@ComponentScan` defaults to the package of the annotated class and scans downward. If the main class is inside a sub-package, sibling packages like `service` and `repository` are never scanned.

**Q: How do we exclude something Boot auto-configures?**
A: `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)`, or the `spring.autoconfigure.exclude` property. We use that for services with no database, which otherwise fail at startup asking for a JDBC URL.

**Q: Can we have two `@SpringBootApplication` classes in one app?**
A: Technically it compiles, but it breaks Boot's test support, which searches for a single `@SpringBootConfiguration` to build the test context. Keep exactly one.

## 9.7 Interview Questions

**Beginner — What three annotations does it combine?**
`@SpringBootConfiguration`, `@EnableAutoConfiguration`, `@ComponentScan`.

**Intermediate — `@SpringBootConfiguration` vs `@Configuration`?**
`@SpringBootConfiguration` is `@Configuration` with a marker meaning "this is the application's primary configuration". `@SpringBootTest` uses it to auto-detect the context to load, which is why only one should exist.

**Advanced — Does `@ComponentScan` scan the whole classpath?**
No — only the annotated class's package and its sub-packages. Scanning is also restricted to classes with stereotype annotations, and Boot speeds it up with `spring-context-indexer` or, in large apps, explicit `scanBasePackages`.

## 9.8 Quick Revision

1. `@SpringBootApplication` = `@SpringBootConfiguration` + `@EnableAutoConfiguration` + `@ComponentScan`.
2. Put it on the **main class in the root package**.
3. Scanning covers that package and **all sub-packages only**.
4. `scanBasePackages` extends the scan; `exclude` removes auto-config classes.
5. Exactly **one** per application.
6. Never put the main class in the default (unnamed) package.

## 9.9 TL Explanation (speak this)

> "`@SpringBootApplication` on the main class is three annotations in one — it marks the class as configuration, enables auto-configuration, and turns on component scanning from that package downwards. That last part is why the main class lives in the root package: if it sat inside `controller`, Spring would never scan `service` or `repository` and the app wouldn't start. If we need to switch off something Boot configures for us — say `DataSourceAutoConfiguration` in a service with no database — we exclude it right there in the annotation."

---

# 10. Auto-Configuration

## 10.1 What is it?

**Simple words:**
Auto-configuration is Spring Boot **looking at what libraries you added and configuring them for you**.

You add the JPA starter → Boot says *"I see Hibernate and a database driver, and a datasource URL in the properties. Let me create the connection pool, the `EntityManagerFactory` and the transaction manager for you."*

You never wrote those beans. Boot did.

**Technical wording:**
Auto-configuration is a mechanism by which Spring Boot conditionally registers bean definitions based on the classpath contents, existing beans and configuration properties, using `@Conditional` evaluation over auto-configuration classes discovered from `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`.

## 10.2 Why it matters

Without auto-configuration, a simple JPA + web application needs you to hand-write:

```java
@Bean public DataSource dataSource() { ... }
@Bean public LocalContainerEntityManagerFactoryBean entityManagerFactory() { ... }
@Bean public PlatformTransactionManager transactionManager() { ... }
@Bean public ObjectMapper objectMapper() { ... }
@Bean public DispatcherServlet dispatcherServlet() { ... }
// ... and about 15 more
```

Auto-configuration writes all of that. You write **zero**.

## 10.3 How it works internally — the complete flow

```
STEP 1  @EnableAutoConfiguration is present (via @SpringBootApplication)
              │
              ▼
STEP 2  It imports AutoConfigurationImportSelector
              │
              ▼
STEP 3  That selector reads EVERY jar on the classpath for the file:
        META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
        (Boot 2.7+;  older versions used META-INF/spring.factories)
              │
              ▼
STEP 4  It collects a candidate list — roughly 150+ auto-configuration classes
        e.g. DataSourceAutoConfiguration, JpaRepositoriesAutoConfiguration,
             WebMvcAutoConfiguration, JacksonAutoConfiguration, SecurityAutoConfiguration
              │
              ▼
STEP 5  It filters out anything in `exclude` / spring.autoconfigure.exclude
              │
              ▼
STEP 6  ★ EACH remaining class is evaluated against its @Conditional annotations ★
              │
              ├── conditions PASS  →  its @Bean methods run → beans registered
              └── conditions FAIL  →  class skipped entirely, nothing registered
              │
              ▼
STEP 7  Auto-configured beans are registered LAST, so YOUR beans always win
```

**The most important idea in this whole section:**
Auto-configuration is **conditional**, and it **backs off** when you have defined your own bean. That is why Boot is opinionated *and* flexible at the same time.

## 10.4 The `@Conditional` family

| Annotation | Registers the bean only if... |
|---|---|
| `@ConditionalOnClass` | This class **is** on the classpath |
| `@ConditionalOnMissingClass` | This class is **not** on the classpath |
| `@ConditionalOnBean` | Such a bean already exists |
| **`@ConditionalOnMissingBean`** | **No such bean exists — this is the "back off" mechanism** |
| `@ConditionalOnProperty` | A property has a given value |
| `@ConditionalOnWebApplication` | It is a web application |
| `@ConditionalOnResource` | A resource file exists |
| `@ConditionalOnExpression` | A SpEL expression is true |

### A real auto-configuration class (simplified from Boot's source)

```java
@AutoConfiguration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })   // JDBC on classpath?
@EnableConfigurationProperties(DataSourceProperties.class)              // bind spring.datasource.*
public class DataSourceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean(DataSource.class)     // ← ONLY if YOU didn't define one
    public DataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
    }
}
```

**Read those three lines carefully — they explain the whole philosophy:**
1. `@ConditionalOnClass` — "only bother if JDBC is even present"
2. `@EnableConfigurationProperties` — "read the user's `spring.datasource.*` settings"
3. `@ConditionalOnMissingBean` — **"if the developer defined their own `DataSource`, get out of the way"**

## 10.5 How to override auto-configuration (three ways)

```java
// Way 1 — define your own bean. Auto-config backs off automatically. (Most common)
@Configuration
public class DataSourceConfig {
    @Bean
    public DataSource dataSource() {
        return new HikariDataSource(customConfig());
    }
}
```

```properties
# Way 2 — set properties (auto-config reads them)
spring.datasource.url=jdbc:mysql://localhost:3306/orders
spring.jpa.hibernate.ddl-auto=validate
server.port=8081
```

```java
// Way 3 — exclude the auto-configuration entirely
@SpringBootApplication(exclude = { SecurityAutoConfiguration.class })
```

## 10.6 Debugging: "why is this bean here / missing?"

This is a genuinely valuable practical skill.

```properties
debug=true
```
or run with `--debug`. Boot then prints a **Conditions Evaluation Report**:

```
============================
CONDITIONS EVALUATION REPORT
============================

Positive matches:          ← auto-config that WAS applied, and why
-----------------
   DataSourceAutoConfiguration matched:
      - @ConditionalOnClass found required classes 'javax.sql.DataSource' (OnClassCondition)

Negative matches:          ← auto-config that was SKIPPED, and why
-----------------
   MongoAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'com.mongodb.client.MongoClient'
```

**What to tell a TL:** *"When a bean isn't behaving as expected, I run with `--debug` and read the conditions report — it tells me exactly which auto-configuration matched and which backed off, and why."* Alternatively, the Actuator `/actuator/conditions` endpoint gives the same data in JSON.

## 10.7 Writing your own auto-configuration (shared library scenario)

Relevant when your team builds an internal starter used by many microservices.

```java
// 1. The configuration class
@AutoConfiguration
@ConditionalOnClass(AuditService.class)
@ConditionalOnProperty(prefix = "company.audit", name = "enabled", havingValue = "true",
                       matchIfMissing = true)
@EnableConfigurationProperties(AuditProperties.class)
public class AuditAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean          // let consuming apps override
    public AuditService auditService(AuditProperties properties) {
        return new AuditService(properties.getEndpoint());
    }
}
```

```
2. Register it in:
   src/main/resources/META-INF/spring/
       org.springframework.boot.autoconfigure.AutoConfiguration.imports

   File content (one fully-qualified class name per line):
   com.company.audit.AuditAutoConfiguration
```

Now any microservice that adds your `company-audit-starter` dependency automatically gets a configured `AuditService` — with no code. That is exactly how `spring-boot-starter-web` works.

## 10.8 Common Mistakes

| Mistake | Consequence |
|---|---|
| Defining a bean **and** expecting auto-config too | Yours wins; auto-config backs off. Sometimes surprising. |
| Adding `spring-boot-starter-data-jpa` with no datasource config | Startup fails: *"Failed to configure a DataSource: 'url' attribute is not specified"* |
| Adding `spring-boot-starter-security` unknowingly | Every endpoint suddenly requires login with a generated password |
| Excluding an auto-config that others depend on | Cascading `NoSuchBeanDefinitionException` |
| Putting `@Configuration` auto-config classes in a scanned package | They get applied unconditionally, defeating the ordering/conditional design |

## 10.9 TL Questions

**Q: What is auto-configuration?**
A: Spring Boot inspecting the classpath and existing beans, then registering the standard beans for us — datasource, entity manager factory, transaction manager, dispatcher servlet, JSON mapper — so we don't hand-write that configuration.

**Q: How does it work internally?**
A: `@EnableAutoConfiguration` imports `AutoConfigurationImportSelector`, which reads the `AutoConfiguration.imports` files from every jar, builds a candidate list of around 150 configuration classes, and evaluates each one's `@Conditional` annotations. Only the matching ones contribute beans.

**Q: How does Boot avoid overriding our configuration?**
A: Nearly every auto-configured bean is annotated `@ConditionalOnMissingBean`, and auto-configurations are processed after user configuration. So if we define our own bean of that type, the auto-configuration backs off silently.

**Q: What happens if we remove `@EnableAutoConfiguration`?**
A: Nothing is configured automatically. No `DataSource`, no `EntityManagerFactory`, no MVC setup, no embedded server config — we would have to declare every one of those beans manually, as in classic Spring.

**Q: How do you debug an auto-configuration problem?**
A: Start the app with `--debug` and read the conditions evaluation report, or hit `/actuator/conditions`. It lists positive and negative matches with the exact reason each condition passed or failed.

**Q: Where does this show up in our project?**
A: Everywhere — our HikariCP pool, Hibernate setup, Jackson serialization and the embedded Tomcat all come from auto-configuration. We only override where we need something non-default, like custom Jackson date handling or a second datasource.

**Q: Is auto-configuration a performance problem?**
A: There is a small startup cost from evaluating conditions, but Boot mitigates it with class-level `@ConditionalOnClass` short-circuiting and auto-configuration metadata filtering. It has no effect on runtime request performance.

## 10.10 Interview Questions

**Beginner — What enables auto-configuration?**
`@EnableAutoConfiguration`, included inside `@SpringBootApplication`.

**Beginner — Where is the list of auto-configuration classes?**
In `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` inside the `spring-boot-autoconfigure` jar (Boot 2.7+). Before that it was the `EnableAutoConfiguration` key in `META-INF/spring.factories`.

**Intermediate — How do you disable a specific auto-configuration?**
`@SpringBootApplication(exclude = X.class)`, `@EnableAutoConfiguration(exclude = X.class)`, or the property `spring.autoconfigure.exclude=fully.qualified.X`.

**Intermediate — What is `@ConditionalOnMissingBean` and why is it central?**
It registers the bean only when no bean of that type already exists. It is the mechanism that makes Boot's opinions into defaults rather than constraints — user beans always take precedence.

**Advanced — How does Boot ensure user beans are evaluated before auto-configuration?**
Auto-configuration classes are registered as *deferred* imports via `DeferredImportSelector`, so they are processed after all regular `@Configuration` classes. By the time `@ConditionalOnMissingBean` is evaluated, user bean definitions already exist.

**Advanced — Auto-configuration ordering?**
`@AutoConfigureBefore`, `@AutoConfigureAfter` and `@AutoConfigureOrder` control relative order — for example `JpaRepositoriesAutoConfiguration` must run after `DataSourceAutoConfiguration`.

**Advanced — Why did Boot 2.7 move away from `spring.factories`?**
`spring.factories` was a general-purpose multi-key file that had to be fully parsed; the dedicated `.imports` file is a simple line-per-class list that is faster to read and clearer in intent, and it separates auto-configuration registration from other factory mechanisms.

## 10.11 Quick Revision

1. Auto-configuration = Boot configuring beans based on the **classpath**, **existing beans** and **properties**.
2. Enabled by `@EnableAutoConfiguration` inside `@SpringBootApplication`.
3. Driven by `AutoConfigurationImportSelector` reading `AutoConfiguration.imports`.
4. Every auto-config class is **conditional** — `@ConditionalOnClass`, `@ConditionalOnMissingBean`, etc.
5. **`@ConditionalOnMissingBean` = the back-off mechanism.** Your bean always wins.
6. Auto-configurations are processed **last** (deferred import).
7. Override by: defining your own bean, setting properties, or excluding the class.
8. Debug with `--debug` or `/actuator/conditions`.
9. Custom starters register via `META-INF/spring/...AutoConfiguration.imports`.

## 10.12 TL Explanation (speak this)

> "Auto-configuration is Boot looking at what's on the classpath and registering the standard beans for us — the datasource, entity manager factory, transaction manager, Jackson, the dispatcher servlet. Internally `@EnableAutoConfiguration` imports a selector that loads about 150 candidate config classes and evaluates each one's `@Conditional` annotations. The key part is `@ConditionalOnMissingBean`: the moment we define our own bean of that type, the auto-configuration backs off, so Boot's opinions are only defaults. When something isn't configured the way we expect, I run with `--debug` and read the conditions report, which says exactly which config matched and why."

---

# 11. Component Scanning

## 11.1 What is it?

**Simple words:**
Component scanning is Spring **walking through your packages, looking at every class, and asking "does this have `@Component`, `@Service`, `@Repository`, `@Controller` or `@Configuration` on it?"** Every class that does becomes a bean.

**Technical wording:**
Component scanning is the process by which the container discovers candidate components by scanning the classpath under specified base packages, reading class metadata (via ASM, without loading the class), and registering matching classes as bean definitions.

## 11.2 How it works internally

```
1. @ComponentScan declares base packages
        (default = the package of the annotated class)
                    │
                    ▼
2. ClassPathBeanDefinitionScanner walks those packages
                    │
                    ▼
3. For each .class file, read annotation metadata with ASM
   (bytecode is inspected — the class is NOT loaded yet; this keeps startup fast)
                    │
                    ▼
4. Apply include filters (default: any @Component meta-annotation)
   then apply exclude filters
                    │
                    ▼
5. Matching classes → ScannedGenericBeanDefinition registered
                    │
                    ▼
6. Bean name generated: class simple name, first letter lowercased
```

## 11.3 Controlling the scan

```java
// Default — package of the annotated class and below
@SpringBootApplication
public class App { }

// Explicit packages
@ComponentScan(basePackages = {"com.company.orderservice", "com.company.shared"})

// Type-safe: uses the package of the given class (survives refactoring/renaming)
@ComponentScan(basePackageClasses = {OrderService.class, SharedMarker.class})

// Exclude by annotation
@ComponentScan(excludeFilters =
    @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Deprecated.class))

// Exclude by regex
@ComponentScan(excludeFilters =
    @ComponentScan.Filter(type = FilterType.REGEX, pattern = "com\\.company\\.legacy\\..*"))
```

**Best practice:** prefer `basePackageClasses` over `basePackages` — a `String` package name silently breaks when someone renames a package; a class reference is checked by the compiler.

## 11.4 Common Mistakes

| Mistake | Consequence |
|---|---|
| Bean class outside the scanned tree | `NoSuchBeanDefinitionException` |
| Adding `@ComponentScan` with `basePackages` on the main class | **Replaces** the default scan — your own package is no longer scanned unless you list it |
| Scanning a very broad package (e.g. `com`) | Slow startup; may pick up third-party components unintentionally |
| Expecting interfaces or abstract classes to be scanned | They are not instantiable, so they are not registered (Spring Data repository interfaces are a special case — registered by a different mechanism) |

## 11.5 TL Questions

**Q: What is component scanning?**
A: Spring scanning the configured base packages for classes annotated with stereotype annotations and registering each one as a bean definition.

**Q: What packages does it scan by default?**
A: The package of the `@SpringBootApplication` class and all its sub-packages. That is the reason the main class must live in the root package.

**Q: What happens internally — does it load every class?**
A: No. It reads bytecode metadata with ASM, so it only checks annotations without initialising classes. That keeps startup fast even in large codebases.

**Q: What happens if a class has no stereotype annotation?**
A: It is ignored entirely, so injecting it fails at startup with `NoSuchBeanDefinitionException`. If we can't annotate it — a third-party class — we register it with an `@Bean` method instead.

**Q: How do we include beans from a shared library module?**
A: Either add its package via `scanBasePackages`, or better, have the library ship its own auto-configuration so consuming services get the beans automatically without knowing its package structure.

## 11.6 Interview Questions

**Beginner — Which annotations are picked up by scanning?**
`@Component` and everything meta-annotated with it: `@Service`, `@Repository`, `@Controller`, `@RestController`, `@Configuration`.

**Intermediate — `basePackages` vs `basePackageClasses`?**
`basePackages` takes `String` names — not refactor-safe. `basePackageClasses` takes class references and uses their packages — compile-time checked and refactor-safe. Prefer the latter.

**Advanced — Does scanning hurt startup time in large applications?**
It can. Mitigations: narrow the base packages, use `spring-context-indexer` (which generates a compile-time `META-INF/spring.components` index so the scanner reads a file instead of walking the classpath), or use explicit `@Bean` configuration in extreme cases.

## 11.7 Quick Revision

1. Scanning finds stereotype-annotated classes and registers them as beans.
2. Default base package = package of the `@SpringBootApplication` class, downward only.
3. Metadata is read via ASM — no class loading, fast.
4. Bean name = class simple name, lowercase first letter.
5. `basePackageClasses` is safer than `basePackages`.
6. Declaring `@ComponentScan` explicitly **replaces** the default behaviour.
7. Third-party classes cannot be scanned — use `@Bean`.

## 11.8 TL Explanation (speak this)

> "Component scanning is how Spring finds our beans — it walks the package tree under the main class and registers every class carrying `@Service`, `@Repository`, `@Controller` or `@Component`. It reads annotations straight from bytecode rather than loading classes, so it's fast. The practical consequences are that the main class has to sit in the root package, and that anything outside that tree — a shared library, say — either needs `scanBasePackages` or should ship its own auto-configuration."

---

# 12. Java Configuration — @Configuration and @Bean

## 12.1 What is it?

**Simple words:**
A `@Configuration` class is a place where **you write the code that creates beans yourself**, instead of letting Spring create them from a `@Component` annotation.

**Technical wording:**
`@Configuration` designates a class as a source of bean definitions. Its `@Bean`-annotated methods are invoked by the container, and their return values are registered as beans under the container's management.

## 12.2 Basic usage

```java
package com.company.orderservice.config;

@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();          // bean name = "restTemplate"
    }

    @Bean
    public PaymentClient paymentClient(RestTemplate restTemplate,        // injected bean
                                       @Value("${payment.url}") String url) {  // injected property
        return new PaymentClient(restTemplate, url);
    }
}
```

## 12.3 The critical concept: `proxyBeanMethods` (Full vs Lite mode)

This is a favourite senior-level interview question.

```java
@Configuration
public class AppConfig {

    @Bean
    public DataSource dataSource() {
        return new HikariDataSource();
    }

    @Bean
    public OrderRepository orderRepository() {
        return new OrderRepository(dataSource());   // ← calling another @Bean method directly
    }

    @Bean
    public AuditRepository auditRepository() {
        return new AuditRepository(dataSource());   // ← calling it AGAIN
    }
}
```

**Question: how many `DataSource` objects are created — one or two?**

**Answer: ONE.** And here is why.

`@Configuration` classes are enhanced by **CGLIB at runtime** — Spring creates a subclass proxy of `AppConfig`. That proxy intercepts every call to a `@Bean` method and checks: *"Does this bean already exist in the container? If yes, return the existing one instead of executing the method."*

This is called **Full mode** (`proxyBeanMethods = true`, the default). It guarantees singleton semantics even when `@Bean` methods call each other.

**Now compare with `@Component`:**

```java
@Component                       // NOT @Configuration — no CGLIB proxy
public class AppConfig {

    @Bean public DataSource dataSource() { return new HikariDataSource(); }

    @Bean public OrderRepository orderRepository() {
        return new OrderRepository(dataSource());   // plain Java call → NEW object!
    }
}
```
Here `dataSource()` is an ordinary method call, so you get **two different `DataSource` objects** — meaning **two connection pools**. In production this shows up as double the expected database connections and transactions that don't see each other's data. A nasty, hard-to-find bug.

### Lite mode

```java
@Configuration(proxyBeanMethods = false)     // no CGLIB proxy — faster startup
public class AppConfig {

    @Bean
    public OrderRepository orderRepository(DataSource dataSource) {   // inject as PARAMETER
        return new OrderRepository(dataSource);    // never call dataSource() directly
    }
}
```

**When to use lite mode:** when `@Bean` methods never call each other (always take dependencies as method parameters). It skips CGLIB subclassing, giving faster startup and better native-image compatibility. **Spring Boot's own auto-configuration classes use `proxyBeanMethods = false` everywhere** for exactly this reason.

| | Full mode (`true`, default) | Lite mode (`false`) |
|---|---|---|
| CGLIB proxy created | Yes | No |
| Inter-`@Bean` method calls return the singleton | Yes | **No — creates a new object** |
| Startup cost | Slightly higher | Lower |
| Class must be non-final | Yes (CGLIB subclasses it) | No restriction |
| Recommended style | Convenient | **Take dependencies as parameters** |

**Golden rule:** always pass dependencies as **method parameters** rather than calling other `@Bean` methods. Then both modes behave identically and you can safely use lite mode.

## 12.4 `@Configuration` vs `@Component` for bean declaration

| | `@Configuration` | `@Component` with `@Bean` methods |
|---|---|---|
| CGLIB proxied | Yes (default) | No |
| Inter-bean calls | Return singleton | Create new objects — **bug risk** |
| Intended for | Declaring beans | Being a bean itself |
| Recommendation | **Use for bean declaration** | Don't put `@Bean` methods here |

## 12.5 Other useful configuration annotations

| Annotation | Purpose |
|---|---|
| `@Import(OtherConfig.class)` | Pull another configuration class in |
| `@PropertySource("classpath:custom.properties")` | Load an extra properties file |
| `@EnableConfigurationProperties(X.class)` | Activate a `@ConfigurationProperties` class |
| `@Profile("dev")` | Register beans only for a given profile |
| `@ConditionalOnProperty` | Register conditionally on a property value |
| `@EnableTransactionManagement`, `@EnableAsync`, `@EnableScheduling`, `@EnableCaching` | Turn on framework features (mostly automatic in Boot) |

## 12.6 Real-Time Project Example

```java
@Configuration
@EnableConfigurationProperties(ExternalApiProperties.class)
public class ExternalApiConfig {

    // A tuned RestTemplate — never use a bare `new RestTemplate()` in production,
    // because it has NO timeouts and can hang a thread forever.
    @Bean
    public RestTemplate paymentRestTemplate(RestTemplateBuilder builder,
                                            ExternalApiProperties props) {
        return builder
                .setConnectTimeout(Duration.ofMillis(props.getConnectTimeoutMs()))
                .setReadTimeout(Duration.ofMillis(props.getReadTimeoutMs()))
                .defaultHeader("X-Api-Key", props.getApiKey())
                .build();
    }

    @Bean
    @ConditionalOnProperty(name = "feature.retry.enabled", havingValue = "true")
    public RetryTemplate retryTemplate() {
        RetryTemplate template = new RetryTemplate();
        template.setRetryPolicy(new SimpleRetryPolicy(3));
        return template;
    }
}
```

**Key teaching point:** the timeout configuration is a real production concern. A `RestTemplate` with no read timeout will block a Tomcat worker thread indefinitely if the remote service hangs — exhaust the thread pool and your entire service goes down because of *someone else's* outage.

## 12.7 TL Questions

**Q: When do we use `@Configuration` and `@Bean` instead of `@Component`?**
A: When we don't own the class or need custom construction logic — `RestTemplate`, `ObjectMapper`, a `DataSource`, third-party clients. `@Component` only works on classes whose source we control.

**Q: What happens internally with a `@Configuration` class?**
A: Spring subclasses it with CGLIB and intercepts the `@Bean` methods, so calling one from another returns the existing singleton instead of creating a new object.

**Q: What if we use `@Component` instead of `@Configuration` for bean methods?**
A: No proxy is created, so an inter-bean method call is just a normal Java call and produces a second object. With a `DataSource` that means two connection pools — a genuine production bug.

**Q: What is `proxyBeanMethods = false`?**
A: Lite mode — it skips the CGLIB proxy for faster startup. It's safe as long as `@Bean` methods take their dependencies as parameters rather than calling each other. Boot's own auto-configuration classes all use it.

**Q: Where would you use this in a real project?**
A: Our `config` package — the `RestTemplate` with explicit connect and read timeouts, the Jackson `ObjectMapper` with our date format, the Spring Security filter chain, and Swagger/OpenAPI setup.

## 12.8 Interview Questions

**Beginner — What does `@Bean` do?**
It tells Spring that the method's return value should be registered and managed as a bean.

**Intermediate — `@Bean` vs `@Component`?**
`@Bean` is method-level and used for explicit construction, typically of third-party classes. `@Component` is class-level and discovered by scanning. `@Bean` gives full control of construction; `@Component` is automatic.

**Intermediate — Can a `@Bean` method be `private` or `final`?**
In full mode, no — CGLIB must be able to override it, so it must be non-private and non-final (and the class non-final). In lite mode those restrictions do not apply.

**Advanced — How does Spring guarantee singleton semantics for inter-bean calls?**
`ConfigurationClassEnhancer` creates a CGLIB subclass with a `BeanMethodInterceptor`. The interceptor checks whether the bean is already in the factory and, if so, returns the existing instance rather than invoking the method body.

**Advanced — Why do auto-configuration classes use `proxyBeanMethods = false`?**
To avoid the CGLIB subclassing cost for ~150 configuration classes at every startup, and to remain compatible with GraalVM native image, which dislikes runtime bytecode generation.

## 12.9 Quick Revision

1. `@Configuration` + `@Bean` = manual bean registration.
2. Use it for **third-party classes** and custom construction.
3. `@Configuration` is **CGLIB-proxied** by default → inter-bean calls return the singleton.
4. `@Component` + `@Bean` is **not** proxied → duplicate objects. Avoid.
5. `proxyBeanMethods = false` = lite mode, faster, no proxy.
6. **Best practice: pass dependencies as method parameters**, never call `@Bean` methods directly.
7. Bean name = method name, unless `@Bean(name = "...")`.
8. Always set timeouts on any HTTP client bean.

## 12.10 TL Explanation (speak this)

> "We use `@Configuration` classes to register beans we can't annotate — `RestTemplate`, `ObjectMapper`, the security filter chain. The detail worth knowing is that Spring CGLIB-proxies `@Configuration` classes, so if one `@Bean` method calls another it gets the existing singleton back rather than creating a second object. If you put `@Bean` methods in a plain `@Component` you lose that and can end up with two datasources and two connection pools. Our convention is to take dependencies as method parameters, which is correct in both modes."

---

# 13. application.properties and application.yml

## 13.1 What is it?

**Simple words:**
It is the settings file for your application — the port it runs on, the database URL, log levels, timeouts. Anything you might want to change **without recompiling the code**.

**Technical wording:**
Externalized configuration files read by Spring Boot's `Environment` abstraction and bound to beans via `@Value` or `@ConfigurationProperties`. Located in `src/main/resources`.

## 13.2 properties vs yml

```properties
# application.properties — flat key=value
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/orders
spring.datasource.username=root
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true
app.payment.timeout-ms=5000
app.payment.retry-count=3
```

```yaml
# application.yml — hierarchical
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/orders
    username: root
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true

app:
  payment:
    timeout-ms: 5000
    retry-count: 3
```

| | `.properties` | `.yml` |
|---|---|---|
| Format | Flat `key=value` | Hierarchical, indented |
| Repetition of prefixes | High | Low |
| Lists | `a[0]=x` clumsy | Clean `- x` syntax |
| Multiple profiles in one file | No | **Yes** (`---` separators) |
| Indentation sensitivity | None | **Strict — tabs break it** |
| Readability for large config | Poor | Good |

**Both are equally valid.** Most modern corporate projects use `.yml`. If both files exist, `.properties` wins for duplicate keys.

## 13.3 Reading values — `@Value` vs `@ConfigurationProperties`

### `@Value` — single values

```java
@Service
public class PaymentService {

    @Value("${app.payment.timeout-ms}")            // required — fails if missing
    private int timeoutMs;

    @Value("${app.payment.retry-count:3}")         // default value after the colon
    private int retryCount;

    @Value("${app.payment.modes}")                 // comma-separated → List
    private List<String> modes;

    @Value("#{systemProperties['user.timezone']}") // SpEL expression
    private String timezone;
}
```

### `@ConfigurationProperties` — grouped, type-safe (PREFERRED)

```java
@Component
@ConfigurationProperties(prefix = "app.payment")
@Getter @Setter
@Validated                                  // enables validation on startup
public class PaymentProperties {

    @NotBlank
    private String gatewayUrl;

    @Min(1000) @Max(30000)
    private int timeoutMs = 5000;           // field name maps to timeout-ms (relaxed binding)

    @Min(0) @Max(5)
    private int retryCount = 3;

    private List<String> supportedModes = new ArrayList<>();
    private Map<String, String> headers = new HashMap<>();
}
```

```java
// Injected like any bean — fully type-safe
@Service
@RequiredArgsConstructor
public class PaymentService {
    private final PaymentProperties properties;

    public void pay() {
        int timeout = properties.getTimeoutMs();   // compile-time checked, IDE-completed
    }
}
```

### Comparison — know this table

| Feature | `@Value` | `@ConfigurationProperties` |
|---|---|---|
| Binds | One property at a time | A whole group by prefix |
| Type safety | Weak (string → parsed) | **Strong** |
| Validation (JSR-380) | No | **Yes, with `@Validated`** |
| Relaxed binding (`timeout-ms` → `timeoutMs`) | **No — exact match required** | **Yes** |
| Complex types (List, Map, nested) | Awkward | Natural |
| SpEL support | Yes | No |
| IDE autocomplete | No | Yes (with the config-processor) |
| Best for | One-off values | **Grouped configuration — prefer this** |

**Relaxed binding** means `@ConfigurationProperties` matches `app.payment.timeout-ms`, `APP_PAYMENT_TIMEOUTMS`, `app.payment.timeoutMs` and `app.payment.timeout_ms` all to the field `timeoutMs`. This is what makes environment-variable overrides work cleanly in Docker/Kubernetes — and `@Value` does **not** do it.

## 13.4 Property precedence (higher overrides lower) — IMPORTANT

```
1. Command-line arguments        --server.port=9090          ← HIGHEST
2. SPRING_APPLICATION_JSON environment variable
3. OS environment variables      SERVER_PORT=9090
4. Java system properties        -Dserver.port=9090
5. Profile-specific outside the jar   ./config/application-prod.yml
6. Profile-specific inside the jar    application-prod.yml
7. application.yml outside the jar
8. application.yml inside the jar
9. @PropertySource
10. Default properties (SpringApplication.setDefaultProperties)  ← LOWEST
```

**Why this matters in production:** it is what lets one immutable Docker image run in dev, QA and prod. The image contains sensible defaults; the environment supplies overrides through environment variables. **Never bake production secrets into the jar** — inject them at runtime.

```bash
# The same jar, different environment:
java -jar app.jar --spring.profiles.active=prod --server.port=9090
SPRING_DATASOURCE_PASSWORD=secret java -jar app.jar
```

## 13.5 Common Mistakes

| Mistake | Consequence |
|---|---|
| **Committing passwords to `application.yml`** | Credentials in git history forever — a security incident |
| Using tabs in YAML | Parse failure at startup |
| `@Value` with a kebab-case name (`timeout-ms`) into a camelCase expectation | Fails — `@Value` has no relaxed binding |
| No default in `@Value` for an optional property | `IllegalArgumentException: Could not resolve placeholder` |
| `@ConfigurationProperties` without `@Component`/`@EnableConfigurationProperties` | Bean not created |
| `@ConfigurationProperties` with only `final` fields and no constructor binding | Values never bound (setters are needed unless you use `@ConstructorBinding`) |
| `ddl-auto=update` in production | **Hibernate silently alters your production schema.** Use `validate`. |

## 13.6 TL Questions

**Q: Why do we externalize configuration?**
A: So the same build artifact runs in every environment. Code stays identical; only the environment's properties change. It also keeps secrets out of the source tree.

**Q: `@Value` or `@ConfigurationProperties` — which should we use?**
A: `@ConfigurationProperties` for anything grouped. It is type-safe, supports validation at startup, handles lists and maps, and supports relaxed binding so environment variables map cleanly. We use `@Value` only for a stray single value.

**Q: How do we manage passwords?**
A: Never in the repo. They come from environment variables, or a secrets manager such as Vault, AWS Secrets Manager or Kubernetes secrets, injected at runtime. Property precedence means an environment variable overrides anything packaged in the jar.

**Q: What happens internally when the app reads a property?**
A: Boot builds an `Environment` containing an ordered list of `PropertySource`s. Lookup walks that list in precedence order and returns the first match, which is why command-line args beat environment variables, which beat the packaged file.

**Q: What if a required property is missing?**
A: With `@Value` and no default, startup fails with "Could not resolve placeholder". With `@ConfigurationProperties` plus `@Validated` and `@NotBlank`, it fails at startup with a clear validation message. Both are good — a missing config should stop the deployment, not surface later as a runtime error.

## 13.7 Interview Questions

**Beginner — Where do these files live?**
`src/main/resources/application.properties` or `application.yml`.

**Intermediate — What is relaxed binding?**
`@ConfigurationProperties` matching several naming styles (kebab-case, camelCase, snake_case, UPPER_CASE) to the same field. It is what makes `SPRING_DATASOURCE_URL` as an environment variable bind to `spring.datasource.url`.

**Intermediate — How do you override a property at deploy time?**
Command-line argument, environment variable or JVM system property — all of which outrank the packaged file in the precedence order.

**Advanced — How do you bind immutable configuration?**
Use constructor binding: a class with `final` fields and a single constructor, registered with `@ConfigurationProperties` plus `@EnableConfigurationProperties` (in Boot 3, `@ConstructorBinding` is inferred for a single-constructor class). Java `record`s work well for this.

**Advanced — How do you validate configuration at startup?**
Add `@Validated` to the `@ConfigurationProperties` class and JSR-380 constraints on the fields. Binding failures then abort startup with a readable report — fail fast rather than a `NullPointerException` on first use.

## 13.8 Quick Revision

1. `application.properties` / `application.yml` in `src/main/resources`.
2. YAML is hierarchical and supports multiple profiles in one file; indentation is strict.
3. `@Value` = single values; `@ConfigurationProperties` = grouped and type-safe (**preferred**).
4. Only `@ConfigurationProperties` supports **relaxed binding** and **validation**.
5. Precedence: **command line > env vars > system properties > external file > packaged file**.
6. One artifact, many environments — never bake secrets into the jar.
7. `@Value("${key:default}")` provides a fallback.
8. Use `ddl-auto=validate` in production, never `update` or `create`.

## 13.9 TL Explanation (speak this)

> "All environment-specific settings live in `application.yml` rather than in code, so the same jar runs in dev, QA and prod with only the properties changing. For anything grouped we bind with `@ConfigurationProperties` instead of `@Value`, because it's type-safe, validates at startup, and supports relaxed binding so Kubernetes environment variables map straight onto the fields. Secrets never go in the file — they come from environment variables or the secrets manager, and Boot's precedence order means those override whatever is packaged in the jar."

---

# 14. Profiles

## 14.1 What is it?

**Simple words:**
A profile is a **named set of configuration** — "dev", "qa", "prod". You tell Spring which one is active, and it loads the matching settings and beans.

It lets one codebase behave differently per environment without any `if (environment.equals("prod"))` code.

**Technical wording:**
Profiles provide conditional registration of beans and conditional loading of configuration based on the active profile set in the `Environment`.

## 14.2 Profile-specific configuration files

```
src/main/resources/
├── application.yml            ← always loaded (common settings)
├── application-dev.yml        ← loaded when 'dev' is active
├── application-qa.yml
└── application-prod.yml
```

Naming rule: `application-{profile}.yml`.

**Loading order:** `application.yml` loads first, then the profile file **overrides** matching keys. So put common settings in the base file and only the differences in the profile files.

```yaml
# application.yml — common to all environments
spring:
  application:
    name: order-service
  jpa:
    open-in-view: false           # always disable this (see JPA section)
server:
  port: 8080
```

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
  jpa:
    hibernate:
      ddl-auto: create-drop        # fine for an in-memory dev DB
    show-sql: true
logging:
  level:
    com.company: DEBUG
    org.hibernate.SQL: DEBUG
```

```yaml
# application-prod.yml
spring:
  datasource:
    url: ${DB_URL}                 # from environment — no secrets in the file
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 20
  jpa:
    hibernate:
      ddl-auto: validate           # NEVER modify a production schema automatically
    show-sql: false                # SQL logging kills performance and leaks data
logging:
  level:
    com.company: INFO
```

### Multiple profiles in one YAML file

```yaml
spring:
  application:
    name: order-service
---
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:h2:mem:testdb
---
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: ${DB_URL}
```
(`spring.config.activate.on-profile` is the Boot 2.4+ syntax; older projects used `spring.profiles`.)

## 14.3 Activating a profile

```properties
# 1. In application.yml (fine for a default, bad for prod)
spring.profiles.active=dev
```
```bash
# 2. Command line — typical for deployments
java -jar app.jar --spring.profiles.active=prod

# 3. Environment variable — typical for Docker/Kubernetes
export SPRING_PROFILES_ACTIVE=prod

# 4. JVM system property
java -Dspring.profiles.active=prod -jar app.jar

# 5. Multiple profiles at once
java -jar app.jar --spring.profiles.active=prod,monitoring
```
```java
// 6. In tests
@SpringBootTest
@ActiveProfiles("test")
class OrderServiceIntegrationTest { }
```

**If no profile is set, the `default` profile is active** and only `application.yml` (plus any `application-default.yml`) is loaded.

## 14.4 Profile-specific beans with `@Profile`

```java
// Dev: log the email instead of sending it
@Service
@Profile("dev")
public class MockEmailService implements EmailService {
    public void send(String to, String body) {
        log.info("[MOCK EMAIL] to={} body={}", to, body);
    }
}

// Prod: real SMTP
@Service
@Profile("prod")
public class SmtpEmailService implements EmailService {
    public void send(String to, String body) { /* real send */ }
}

// Active for anything EXCEPT prod
@Service
@Profile("!prod")
public class DebugToolbarService { }

// Multiple profiles
@Service
@Profile({"dev", "qa"})
public class TestDataSeeder { }
```

**Why this is powerful:** `OrderService` injects `EmailService` and has **no idea** which implementation it receives. No environment checks leak into business logic. This is IoC plus profiles working together.

**Important warning:** if exactly one bean of a required type is `@Profile`-annotated and that profile isn't active, startup fails with `NoSuchBeanDefinitionException`. Always ensure every environment has exactly one implementation available — for example by making the dev one `@Profile("!prod")`.

## 14.5 Real-time usage patterns

| Profile | Typical configuration |
|---|---|
| `dev` | H2 in-memory DB, `ddl-auto=create-drop`, SQL logging on, mock external services, security relaxed |
| `test` | H2 or Testcontainers, seeded test data, fast timeouts |
| `qa` | Real DB (QA instance), real integrations pointing at sandbox endpoints, INFO logging |
| `prod` | Real DB, `ddl-auto=validate`, SQL logging **off**, full security, larger connection pool, Actuator endpoints restricted |

## 14.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Hardcoding `spring.profiles.active=prod` in the packaged `application.yml` | Dev machines accidentally point at production. **Serious.** |
| Forgetting to activate a profile in deployment | App silently uses default config — often the dev database |
| Only one implementation of an interface and it's profile-gated | `NoSuchBeanDefinitionException` when that profile is off |
| Putting secrets in `application-prod.yml` in the repo | Credentials leaked to version control |
| `ddl-auto=update` in the prod profile | Hibernate alters the production schema on deploy |
| Too many profiles | Combinatorial confusion; keep it to a few well-understood ones |

## 14.7 TL Questions

**Q: What are profiles and why do we use them?**
A: Named configuration sets per environment. They let one artifact run in dev, QA and prod with different datasources, logging levels and even different bean implementations, with no environment checks in the code.

**Q: How do we activate one in production?**
A: Through `SPRING_PROFILES_ACTIVE` as an environment variable in the container, or `--spring.profiles.active=prod` on the command line. We deliberately do not hardcode it in the packaged file, so nothing can accidentally start in prod mode locally.

**Q: How does `@Profile` work internally?**
A: It is a `@Conditional` — during bean definition registration Spring checks the annotated profile expression against the active profiles in the `Environment`, and skips registering the bean if it doesn't match.

**Q: What happens if we forget to set a profile?**
A: The `default` profile applies, so only the base `application.yml` is used. That is usually the dev-oriented configuration, which is exactly why production deployments must set the profile explicitly and why we don't put production defaults in the base file.

**Q: Where would you use profile-specific beans?**
A: Mocking external systems in dev — a `MockEmailService` and a stubbed payment gateway under `@Profile("dev")`, with the real implementations under `@Profile("prod")`. The service layer is unchanged; only the wiring differs.

**Q: What's the difference between a profile and a feature flag?**
A: A profile is fixed at startup and selects an environment's whole configuration. A feature flag is toggled at runtime for a specific behaviour, often per user. We don't use profiles for business feature toggles.

## 14.8 Interview Questions

**Beginner — How do you activate a profile?**
`spring.profiles.active` via command line, environment variable, system property, or `@ActiveProfiles` in tests.

**Beginner — What's the file naming convention?**
`application-{profile}.properties` or `application-{profile}.yml`.

**Intermediate — Can multiple profiles be active simultaneously?**
Yes — comma-separated. Later profiles override earlier ones for conflicting keys, and all matching `@Profile` beans are registered.

**Intermediate — What is `@Profile("!prod")`?**
A negated expression — the bean registers whenever `prod` is *not* active. Expressions support `!`, `&` and `|`.

**Advanced — How does `@Profile` differ from `@ConditionalOnProperty`?**
`@Profile` is environment-oriented and driven by the active profile list. `@ConditionalOnProperty` keys off any arbitrary property value and is better for feature toggles. Both are `@Conditional` implementations evaluated at bean-definition time.

**Advanced — What are `spring.profiles.include` and `spring.profiles.group`?**
`include` unconditionally adds extra profiles. `group` (Boot 2.4+) defines a named set — e.g. `spring.profiles.group.prod=prod,monitoring,ssl` — so activating `prod` activates all three. Useful for keeping deployment commands simple.

## 14.9 Quick Revision

1. Profiles = environment-specific configuration and beans.
2. Files: `application-{profile}.yml`; base file loads first, profile file overrides.
3. Activate with `spring.profiles.active` — prefer environment variable or command line.
4. `@Profile` conditionally registers beans; supports `!`, `&`, `|`.
5. Multiple profiles can be active at once.
6. No profile set = `default` profile.
7. **Never hardcode the prod profile into the packaged jar.**
8. Prod rules: `ddl-auto=validate`, SQL logging off, secrets from environment.
9. `spring.profiles.group` bundles profiles under one name.

## 14.10 TL Explanation (speak this)

> "Profiles let one build artifact behave correctly in every environment. We keep common settings in `application.yml` and the differences in `application-dev.yml` and `application-prod.yml`, and the profile is activated through the `SPRING_PROFILES_ACTIVE` environment variable in the container rather than being hardcoded in the jar. We also use `@Profile` on beans, so dev gets a mock email service and prod gets the real SMTP one — the service layer injects the interface and never knows the difference. In prod we always run `ddl-auto=validate` with SQL logging off."

---
---

# PART B — WEB LAYER (SPRING MVC / REST)

---

# 15. Spring MVC and the DispatcherServlet

## 15.1 What is it?

**Simple words:**
When a request arrives at your application, **something** has to decide which of your methods should handle it. That something is the **DispatcherServlet**.

It is the single entry door for every HTTP request. It receives the request, works out which controller method matches the URL, calls it, takes the returned value, converts it to JSON, and writes the response.

It is often called the **Front Controller**, because *every* request goes through this one object first.

**Technical wording:**
Spring MVC is Spring's servlet-based web framework implementing the Front Controller pattern. The `DispatcherServlet` is the central dispatcher that receives all matching requests and delegates to specialised strategy components — `HandlerMapping`, `HandlerAdapter`, `HandlerExceptionResolver`, `ViewResolver` and `HttpMessageConverter` — to produce a response.

## 15.2 Why do we use a Front Controller?

Without it, every servlet would duplicate: authentication checks, logging, character encoding, exception handling, JSON conversion.

With one front controller, all of that lives in **one place**, and your controllers contain only business-facing code.

| Benefit | Explanation |
|---|---|
| Centralised request handling | One place for cross-cutting concerns |
| Consistent behaviour | Every endpoint behaves the same way |
| Pluggable strategies | Swap the JSON converter, add argument resolvers, add interceptors |
| Clean controllers | Your methods just take parameters and return objects |

## 15.3 THE COMPLETE REQUEST FLOW (memorise this)

This is the diagram to draw on a whiteboard when asked "explain the Spring Boot request flow".

```
  CLIENT (browser / Postman / another microservice)
     │
     │  HTTP POST /api/orders   { "customerId": "C1", "amount": 500 }
     ▼
┌─────────────────────────────────────────────────────────────────┐
│  EMBEDDED TOMCAT (servlet container)                            │
│  - accepts the TCP connection                                   │
│  - assigns a worker thread from its thread pool                 │
│  - parses the raw HTTP into an HttpServletRequest               │
└─────────────────────────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────────────┐
│  FILTER CHAIN  (servlet-level, before Spring MVC)               │
│  CharacterEncodingFilter → Spring Security filters →            │
│  your custom filters (correlation id, request logging)          │
└─────────────────────────────────────────────────────────────────┘
     │
     ▼
╔═════════════════════════════════════════════════════════════════╗
║  DISPATCHER SERVLET   (the Front Controller)                    ║
╚═════════════════════════════════════════════════════════════════╝
     │
     │ STEP 1 ─────────────────────────────────────────────────────
     │ Ask each HandlerMapping: "who handles POST /api/orders?"
     │ RequestMappingHandlerMapping searches its registry of
     │ @RequestMapping methods and returns a HandlerExecutionChain
     │   = the handler method + the matching interceptors
     ▼
     │ STEP 2 ─────────────────────────────────────────────────────
     │ INTERCEPTOR preHandle()  ← runs BEFORE the controller
     │   returns false → request stops here
     ▼
     │ STEP 3 ─────────────────────────────────────────────────────
     │ Pick a HandlerAdapter that knows how to invoke this handler
     │ (RequestMappingHandlerAdapter for annotated controllers)
     ▼
     │ STEP 4 ─────────────────────────────────────────────────────
     │ ARGUMENT RESOLVERS build each method parameter:
     │   @PathVariable      → PathVariableMethodArgumentResolver
     │   @RequestParam      → RequestParamMethodArgumentResolver
     │   @RequestBody       → RequestResponseBodyMethodProcessor
     │                          └─► HttpMessageConverter (Jackson)
     │                              reads JSON → Java DTO object
     │   @Valid present?    → run Bean Validation NOW
     │                          └─► fails → MethodArgumentNotValidException
     ▼
     │ STEP 5 ─────────────────────────────────────────────────────
     │ ★ YOUR CONTROLLER METHOD IS FINALLY CALLED ★
     │
     │      Controller  →  Service  (@Transactional starts here)
     │                        │
     │                        ▼
     │                    Repository
     │                        │
     │                        ▼
     │                 JPA / Hibernate
     │                        │
     │                        ▼
     │                     DATABASE
     │                        │
     │                   result returns back up
     ▼
     │ STEP 6 ─────────────────────────────────────────────────────
     │ RETURN VALUE HANDLER processes what you returned
     │   @ResponseBody / ResponseEntity
     │        └─► HttpMessageConverter (Jackson)
     │            writes Java object → JSON into the response body
     │   (a plain String view name → ViewResolver → HTML template)
     ▼
     │ STEP 7 ─────────────────────────────────────────────────────
     │ INTERCEPTOR postHandle()   ← after controller, before response committed
     ▼
     │ STEP 8 ─────────────────────────────────────────────────────
     │ INTERCEPTOR afterCompletion()  ← always runs, even on exception
     ▼
┌─────────────────────────────────────────────────────────────────┐
│  FILTER CHAIN unwinds (code after chain.doFilter)               │
└─────────────────────────────────────────────────────────────────┘
     │
     ▼
  HTTP RESPONSE  201 Created   { "orderId": 1001, "status": "CONFIRMED" }
     │
     ▼
  CLIENT


  ══ IF AN EXCEPTION IS THROWN ANYWHERE IN STEPS 4–6 ══
     │
     ▼
  HandlerExceptionResolver chain
     └─► ExceptionHandlerExceptionResolver
            └─► finds a matching @ExceptionHandler
                (in the controller, or in @RestControllerAdvice)
                   └─► builds the error response
     └─► if none matches → DefaultHandlerExceptionResolver
            → falls back to /error → 500 Internal Server Error
```

**How to summarise this in an interview in 30 seconds:**
> "Tomcat hands the request to the filter chain, then to the DispatcherServlet. It uses HandlerMapping to find the controller method, a HandlerAdapter to invoke it, argument resolvers and message converters to build the parameters from JSON, runs my controller, then converts the return value back to JSON with a message converter. Interceptors wrap the controller call, and any exception goes to a HandlerExceptionResolver, which is how `@RestControllerAdvice` works."

## 15.4 The strategy components inside DispatcherServlet

| Component | Responsibility | Default implementation |
|---|---|---|
| **`HandlerMapping`** | URL + method → handler | `RequestMappingHandlerMapping` |
| **`HandlerAdapter`** | Knows how to invoke that handler | `RequestMappingHandlerAdapter` |
| **`HandlerMethodArgumentResolver`** | Builds each method parameter | ~30 built-in resolvers |
| **`HttpMessageConverter`** | JSON/XML ↔ Java object | `MappingJackson2HttpMessageConverter` |
| **`HandlerExceptionResolver`** | Turns exceptions into responses | `ExceptionHandlerExceptionResolver` |
| **`ViewResolver`** | View name → template (not used in pure REST) | `ThymeleafViewResolver` |
| **`LocaleResolver`** | Determines the request locale | `AcceptHeaderLocaleResolver` |
| **`MultipartResolver`** | File uploads | `StandardServletMultipartResolver` |

**Key insight:** DispatcherServlet itself contains **almost no logic**. It orchestrates these pluggable strategies. That is why Spring MVC is so extensible — you add a custom argument resolver or message converter without touching the dispatcher.

## 15.5 How does DispatcherServlet get registered?

In old Spring, you wrote it in `web.xml`:

```xml
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

In Spring Boot, `DispatcherServletAutoConfiguration` does it automatically and maps it to `/`. You can change the path:

```properties
spring.mvc.servlet.path=/api      # DispatcherServlet now handles /api/**
```

## 15.6 Spring MVC vs Spring WebFlux (Additional clarification)

You will hear WebFlux mentioned; know the distinction.

| | Spring MVC | Spring WebFlux |
|---|---|---|
| Model | **Blocking**, one thread per request | **Non-blocking**, event loop |
| Stack | Servlet API | Reactive Streams |
| Server | Tomcat/Jetty/Undertow | Netty (default) |
| Return types | Objects, `ResponseEntity` | `Mono<T>`, `Flux<T>` |
| Best for | Standard CRUD, JDBC/JPA | High concurrency, streaming, non-blocking clients |
| Learning curve | Low | High |
| **Use when** | **Almost always in corporate CRUD systems** | Very high concurrency with reactive data access end-to-end |

**Important honesty point:** WebFlux only helps if the *whole* chain is non-blocking. Using WebFlux with blocking JDBC/JPA gives you the complexity with none of the benefit. Most corporate Spring Boot projects correctly use MVC.

## 15.7 Common Mistakes

| Mistake | Consequence |
|---|---|
| Thinking you must configure DispatcherServlet in Boot | It's automatic |
| Two methods mapped to the same URL + method | `IllegalStateException: Ambiguous mapping` at startup |
| Blocking I/O in a controller with no timeout | Tomcat worker threads exhausted → whole service unresponsive |
| Confusing filter order with interceptor order | Filters are servlet-level and always run **before** any interceptor |
| Expecting `@ExceptionHandler` to catch filter exceptions | It cannot — filters run outside DispatcherServlet (see Section 24) |

## 15.8 TL Questions

**Q: What is the DispatcherServlet?**
A: The front controller — the single servlet that receives every HTTP request and orchestrates handler lookup, parameter binding, controller invocation, response conversion and exception resolution.

**Q: Walk me through what happens when a request hits our API.**
A: Tomcat accepts it and assigns a worker thread, the filter chain runs, then DispatcherServlet asks HandlerMapping which controller method matches. A HandlerAdapter invokes it, argument resolvers and Jackson build the parameters from the JSON body, validation runs if `@Valid` is present, our controller calls the service and repository, and the return value goes back through Jackson to become the JSON response. Interceptors wrap the controller call and any exception is routed to our `@RestControllerAdvice`.

**Q: Why is it called a front controller?**
A: Because all requests enter through it. Cross-cutting concerns live there instead of being duplicated in every endpoint.

**Q: What happens if two methods map to the same path?**
A: The application fails to start with an ambiguous mapping error. That's deliberate — it's a configuration error caught at deploy time rather than at runtime.

**Q: How does JSON become our DTO object?**
A: `@RequestBody` triggers `RequestResponseBodyMethodProcessor`, which picks an `HttpMessageConverter` based on the `Content-Type` header. For `application/json` that's Jackson's converter, which deserializes the body into our DTO class.

**Q: What alternative can we use?**
A: Spring WebFlux for non-blocking handling, but it only pays off when the entire chain including data access is reactive. With JPA and JDBC, MVC is the correct choice for us.

## 15.9 Interview Questions

**Beginner — What design pattern does DispatcherServlet implement?**
Front Controller.

**Beginner — Who registers it in Spring Boot?**
`DispatcherServletAutoConfiguration`, mapped to `/` by default.

**Intermediate — What is a `HandlerExecutionChain`?**
The object returned by `HandlerMapping`. It contains the handler method plus the ordered list of `HandlerInterceptor`s applicable to that request.

**Intermediate — How are method parameters populated?**
By `HandlerMethodArgumentResolver` implementations. Each resolver declares which parameter types it supports (`supportsParameter`) and builds the value (`resolveArgument`). You can register custom ones for things like injecting the authenticated user.

**Advanced — What is `doDispatch()`?**
The core method of `DispatcherServlet`: checks for multipart, resolves the handler, resolves the adapter, handles conditional-GET/`Last-Modified`, runs `preHandle`, invokes the handler, applies the default view name, runs `postHandle`, then `processDispatchResult` which handles the exception or renders the view, and finally `afterCompletion`.

**Advanced — Can there be multiple DispatcherServlets?**
Yes — an older pattern mapped different servlets to different URL spaces, each with its own child `WebApplicationContext` sharing a common root context. Rare today; microservices solve the same problem by separation of services.

**Advanced — What is the thread model?**
One thread per request, taken from Tomcat's pool (default max 200). The thread is held for the entire request, including blocking database and HTTP calls. This is exactly why timeouts and connection pool sizing matter — a slow downstream service can consume all 200 threads and take the whole application down.

## 15.10 Quick Revision

1. `DispatcherServlet` = front controller, single entry point for all requests.
2. Flow: **Tomcat → Filters → DispatcherServlet → HandlerMapping → Interceptor preHandle → HandlerAdapter → Argument Resolvers → Controller → Service → Repository → DB → Return Value Handler → Message Converter → postHandle → afterCompletion → Response.**
3. `HandlerMapping` finds the handler; `HandlerAdapter` invokes it.
4. `HttpMessageConverter` (Jackson) does JSON ↔ Java in **both** directions.
5. Exceptions go to `HandlerExceptionResolver` → `@ExceptionHandler` / `@RestControllerAdvice`.
6. Boot auto-registers the servlet at `/`; change with `spring.mvc.servlet.path`.
7. DispatcherServlet delegates everything — it is pure orchestration.
8. Thread-per-request model: default 200 Tomcat threads; blocking calls need timeouts.
9. Filters run **outside** DispatcherServlet; interceptors **inside** it.

## 15.11 TL Explanation (speak this)

> "The DispatcherServlet is the front controller — every request goes through it. It asks HandlerMapping which controller method matches the URL and HTTP method, uses a HandlerAdapter to invoke it, and argument resolvers plus Jackson's message converter to turn the JSON body into our DTO. Our controller then calls the service and repository, and the return value goes back through Jackson to become the JSON response. Interceptors wrap the controller call and exceptions are routed to a HandlerExceptionResolver, which is the mechanism behind our `@RestControllerAdvice`. Boot registers all of this automatically."

---

# 16. HandlerMapping and HandlerAdapter

## 16.1 What are they?

**Simple words:**
- **HandlerMapping** answers: *"Which method should handle this URL?"*
- **HandlerAdapter** answers: *"How do I actually call that method?"*

They are separated because a handler is not necessarily an annotated controller method — it could be an older `Controller` interface implementation or a functional route. The adapter hides that difference from the dispatcher.

## 16.2 How HandlerMapping builds its registry (internal)

At startup:

```
1. RequestMappingHandlerMapping implements InitializingBean
              │
              ▼
2. afterPropertiesSet() → scans EVERY bean in the context
              │
              ▼
3. Is the bean annotated @Controller or @RequestMapping?
              │ yes
              ▼
4. For each method, read its @RequestMapping/@GetMapping/etc.
   and build a RequestMappingInfo:
       - path patterns      /api/orders/{id}
       - HTTP methods       GET
       - params / headers   conditions
       - consumes / produces  media types
              │
              ▼
5. Register into a MultiValueMap:  RequestMappingInfo → HandlerMethod
              │
              ▼
6. At request time, match the incoming request against all
   RequestMappingInfos, sort by specificity, pick the best match
```

**Why this matters:** the registry is built **once at startup**, not per request. Lookup at runtime is a fast map match, not reflection scanning. It is also why an ambiguous mapping fails at startup, not on the first call.

**Matching specificity:** exact path beats a path variable, which beats a single wildcard `*`, which beats `**`. So `/api/orders/latest` wins over `/api/orders/{id}` — a useful fact when designing URLs.

## 16.3 The main implementations

| HandlerMapping | Handles |
|---|---|
| `RequestMappingHandlerMapping` | `@RequestMapping` methods — **the one you use** |
| `RouterFunctionMapping` | Functional endpoints (WebFlux style) |
| `SimpleUrlHandlerMapping` | Explicit URL → bean maps |
| `BeanNameUrlHandlerMapping` | Bean names starting with `/` (legacy) |

| HandlerAdapter | Invokes |
|---|---|
| `RequestMappingHandlerAdapter` | `@RequestMapping` methods — **the one you use** |
| `HttpRequestHandlerAdapter` | `HttpRequestHandler` implementations (e.g. static resources) |
| `SimpleControllerHandlerAdapter` | Legacy `Controller` interface |

## 16.4 TL Questions

**Q: What does HandlerMapping do?**
A: It maps an incoming request to the controller method that should handle it, based on path, HTTP method, headers, and content types. It builds that registry once at startup by scanning all `@Controller` beans.

**Q: Why do we need a separate HandlerAdapter?**
A: Because a handler can be different things — an annotated method, a legacy `Controller` implementation, a resource handler. The adapter knows how to invoke each type, so DispatcherServlet stays generic.

**Q: What happens internally when no mapping matches?**
A: DispatcherServlet either throws `NoHandlerFoundException` or, by default in Boot, forwards to `/error`, producing a 404. You can turn the exception on with `spring.mvc.throw-exception-if-no-handler-found=true` to handle it in `@RestControllerAdvice`.

**Q: How does Spring choose between `/api/orders/latest` and `/api/orders/{id}`?**
A: By specificity — a literal path segment always beats a path variable, so `/latest` wins. That's why we can safely have both.

## 16.5 Interview Questions

**Beginner — Which HandlerMapping is used for annotated controllers?**
`RequestMappingHandlerMapping`.

**Intermediate — When is the mapping registry built?**
At startup, in `afterPropertiesSet()`, by scanning all beans. Request-time lookup is then a fast map match.

**Advanced — What is `RequestMappingInfo`?**
The composite matching condition for a handler method: path patterns, HTTP methods, params, headers, consumable and producible media types. Requests are matched against it and results sorted by specificity.

**Advanced — What is `PathPatternParser`?**
The modern, pre-parsed path matching implementation (default since Spring 5.3/Boot 2.6), replacing `AntPathMatcher`. It is significantly faster and stricter — notably it does not support `**` in the middle of a pattern.

## 16.6 Quick Revision

1. HandlerMapping: request → handler method. HandlerAdapter: how to invoke it.
2. `RequestMappingHandlerMapping` + `RequestMappingHandlerAdapter` are the pair you use.
3. Registry built **once at startup**, so ambiguous mappings fail fast.
4. Matching considers path, method, headers, params, consumes, produces.
5. Literal paths beat path variables beat wildcards.
6. No match → 404 (or `NoHandlerFoundException` if enabled).

## 16.7 TL Explanation (speak this)

> "HandlerMapping decides which controller method handles a request and HandlerAdapter knows how to invoke it. At startup Spring scans every `@Controller` bean and builds a registry keyed on path, HTTP method, headers and content type, so runtime lookup is just a fast match rather than reflection. That's also why two methods mapped to the same path fail the startup instead of failing on a live request."

---

# 17. REST API Fundamentals

## 17.1 What is REST?

**Simple words:**
REST is a **set of rules for designing web APIs** so that different systems can talk to each other in a predictable way. It says: treat everything as a *resource* (an order, a customer), give each resource a URL, and use standard HTTP methods to act on it.

**Technical wording:**
REST (REpresentational State Transfer) is an architectural style defining constraints for distributed hypermedia systems: client-server separation, statelessness, cacheability, a uniform interface, layered system, and optionally code-on-demand.

## 17.2 The REST constraints that matter daily

| Constraint | What it means in practice |
|---|---|
| **Client–Server** | Frontend and backend evolve independently |
| **Stateless** | **Every request carries everything needed.** The server keeps no session between requests. This is what allows horizontal scaling — any instance can serve any request. |
| **Cacheable** | Responses declare whether they can be cached |
| **Uniform Interface** | Resources have URIs; standard methods act on them |
| **Layered System** | Client can't tell if it's talking to the server or a load balancer/gateway |

**Statelessness is the one to emphasise to a TL:** it is why JWT (self-contained token) suits microservices better than server-side sessions — no sticky sessions, no shared session store required, and any pod can handle any request.

## 17.3 HTTP methods

| Method | Purpose | Idempotent? | Safe? | Request body | Typical success |
|---|---|---|---|---|---|
| **GET** | Read a resource | Yes | Yes | No | 200 OK |
| **POST** | Create a resource | **No** | No | Yes | 201 Created |
| **PUT** | Full replace/update | **Yes** | No | Yes | 200 OK / 204 |
| **PATCH** | Partial update | No (usually) | No | Yes | 200 OK |
| **DELETE** | Remove a resource | **Yes** | No | No | 204 No Content |

**Definitions you must get right in interviews:**
- **Safe** = does not modify server state. Only GET (and HEAD/OPTIONS).
- **Idempotent** = calling it N times has the **same effect** as calling it once. GET, PUT, DELETE are idempotent; POST is not.

**Why idempotency matters in production:** if a client times out and retries, an idempotent call is harmless. A POST retry can create a **duplicate order**. This is why payment APIs require an **idempotency key** header — a real, practical concern worth raising with a TL.

**PUT vs PATCH — the classic question:**

```http
PUT /api/orders/1          →  replaces the ENTIRE resource
{ "customerId": "C1", "amount": 500, "status": "CONFIRMED" }
   Any field you omit is set to null/default.

PATCH /api/orders/1        →  updates ONLY the given fields
{ "status": "SHIPPED" }
   Everything else is left untouched.
```

## 17.4 URL design rules

| Rule | Bad | Good |
|---|---|---|
| Use **nouns**, not verbs | `/getOrder`, `/createOrder` | `/orders` |
| Use **plural** resource names | `/order/1` | `/orders/1` |
| The HTTP method is the verb | `POST /createOrder` | `POST /orders` |
| Hierarchy for relations | `/getOrderItems?orderId=1` | `/orders/1/items` |
| lowercase with hyphens | `/orderItems`, `/order_items` | `/order-items` |
| Version the API | `/orders` | `/api/v1/orders` |
| Filters as query params | `/orders/status/pending` | `/orders?status=PENDING` |

**A complete, well-designed resource:**

```
GET    /api/v1/orders                 list orders (paginated, filterable)
GET    /api/v1/orders/{id}            get one order
POST   /api/v1/orders                 create an order
PUT    /api/v1/orders/{id}            replace an order
PATCH  /api/v1/orders/{id}            update part of an order
DELETE /api/v1/orders/{id}            delete an order
GET    /api/v1/orders/{id}/items      items of that order (sub-resource)
GET    /api/v1/orders?status=PENDING&page=0&size=20&sort=createdAt,desc
```

## 17.5 HTTP status codes (know these cold)

### 2xx — Success
| Code | Name | Use when |
|---|---|---|
| **200** | OK | Successful GET / PUT / PATCH |
| **201** | Created | **POST created a resource** — include a `Location` header |
| **202** | Accepted | Async processing started, not yet complete |
| **204** | No Content | **DELETE succeeded**, nothing to return |

### 4xx — Client's fault
| Code | Name | Use when |
|---|---|---|
| **400** | Bad Request | Validation failed, malformed JSON |
| **401** | Unauthorized | **Not authenticated** — no/invalid token |
| **403** | Forbidden | **Authenticated but not allowed** |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | Wrong HTTP method for that URL |
| **409** | Conflict | Duplicate resource, optimistic-lock failure |
| **415** | Unsupported Media Type | Wrong `Content-Type` |
| **422** | Unprocessable Entity | Syntactically fine, semantically invalid |
| **429** | Too Many Requests | Rate limit exceeded |

### 5xx — Server's fault
| Code | Name | Use when |
|---|---|---|
| **500** | Internal Server Error | Unhandled exception — a bug |
| **502** | Bad Gateway | Upstream service returned garbage |
| **503** | Service Unavailable | Down / overloaded / circuit breaker open |
| **504** | Gateway Timeout | Upstream timed out |

**The single most important distinction — 401 vs 403:**
- **401 Unauthorized** = *"I don't know who you are."* Missing or invalid credentials. **Log in.**
- **403 Forbidden** = *"I know who you are, and you're not allowed."* Valid token, insufficient role. **Logging in again won't help.**

(The name "Unauthorized" for 401 is a historical misnomer — it actually means unauthenticated.)

**Second most important — 4xx vs 5xx:** 4xx means the client must change something. 5xx means **we** have a problem. Returning 500 for a validation failure is a common bug: it makes your error dashboards alert on the customer's typo.

## 17.6 HTTP headers worth knowing

| Header | Direction | Purpose |
|---|---|---|
| `Content-Type` | Both | Format of **this** message's body (`application/json`) |
| `Accept` | Request | Format the client **wants** back |
| `Authorization` | Request | `Bearer <jwt>` or `Basic <base64>` |
| `Location` | Response | URI of the newly created resource (with 201) |
| `Cache-Control` | Response | Caching policy |
| `ETag` / `If-None-Match` | Both | Conditional requests, optimistic concurrency |
| `X-Correlation-Id` | Both | Request tracing across microservices |

## 17.7 TL Questions

**Q: What makes an API RESTful?**
A: Resources identified by URIs, standard HTTP methods as the verbs, statelessness, appropriate status codes, and a uniform, predictable interface. In practice: nouns in URLs, correct methods, correct status codes, no server-side session state.

**Q: Why does statelessness matter for us?**
A: It allows horizontal scaling. Any instance can serve any request because no session is held in memory, so we can add pods behind a load balancer without sticky sessions or a shared session store. It's also the reason we use JWT rather than server sessions.

**Q: PUT or PATCH for our update endpoint?**
A: PUT if the client sends the complete resource and we replace it; PATCH if they send only changed fields. PUT is idempotent, which makes retries safe. We use PUT for full updates and PATCH for partial ones like a status change.

**Q: What status code should a POST return?**
A: 201 Created, with a `Location` header pointing at the new resource. 200 is acceptable but 201 is more precise and tells the client a resource was created.

**Q: What's the difference between 401 and 403?**
A: 401 means we don't know who you are — the token is missing or invalid. 403 means we know exactly who you are but your role doesn't permit this action. Logging in again fixes 401, not 403.

**Q: Why do we version the API?**
A: So we can make breaking changes without breaking existing clients. Mobile apps in particular can't be force-upgraded, so `/api/v1` and `/api/v2` have to coexist for a period.

**Q: What happens if the client retries a failed POST?**
A: It could create a duplicate, because POST isn't idempotent. For critical operations like payments we accept an idempotency key header and return the original result if the key repeats.

## 17.8 Interview Questions

**Beginner — What does REST stand for?**
Representational State Transfer.

**Beginner — Which methods are idempotent?**
GET, PUT, DELETE, HEAD, OPTIONS. POST is not; PATCH usually is not.

**Intermediate — Difference between safe and idempotent?**
Safe = no state change at all (GET). Idempotent = repeating it produces the same final state (PUT, DELETE). All safe methods are idempotent, but not vice versa — DELETE changes state yet is idempotent.

**Intermediate — Is DELETE really idempotent if the second call returns 404?**
Yes. Idempotency concerns the **server state**, not the status code. After the first DELETE the resource is gone; further DELETEs leave it gone. The differing response code doesn't break idempotency.

**Intermediate — REST vs SOAP?**
REST: HTTP-native, usually JSON, lightweight, stateless, no formal contract required. SOAP: XML envelope, strict WSDL contract, transport-independent, built-in WS-Security and standardised transactions. SOAP still appears in banking and legacy enterprise integration.

**Advanced — What is HATEOAS and do we use it?**
Hypermedia As The Engine Of Application State — responses include links to the next available actions, so a client discovers the API rather than hardcoding URLs. It is REST's highest maturity level (Richardson Level 3), but most corporate APIs stop at Level 2 (resources + verbs + status codes) because of the added complexity and limited client support.

**Advanced — How do you version an API?**
URI versioning (`/api/v1/orders`) — simplest and most visible, most common. Header versioning (`Accept: application/vnd.company.v1+json`) — cleaner URLs, purer REST, harder to test. Query param (`?version=1`) — easy but untidy. Most teams choose URI versioning for its clarity.

## 17.9 Quick Revision

1. REST = resources + URIs + standard HTTP methods + status codes + statelessness.
2. Nouns, plural, lowercase-hyphenated, versioned: `/api/v1/orders/{id}/items`.
3. GET read, POST create, PUT replace, PATCH partial, DELETE remove.
4. Idempotent: GET, PUT, DELETE. **Not** POST.
5. 200 OK, 201 Created (+`Location`), 204 No Content.
6. 400 validation, **401 not authenticated**, **403 not allowed**, 404 missing, 409 conflict.
7. 4xx = client's problem, 5xx = our problem. Never return 500 for validation.
8. Stateless → horizontal scaling → JWT over sessions.
9. Filters and paging go in query params, not the path.

## 17.10 TL Explanation (speak this)

> "We design the API around resources rather than actions — so `POST /api/v1/orders` instead of `/createOrder`, with the HTTP method carrying the verb. Everything is stateless, meaning each request carries its own JWT and no session lives on the server, which is what lets us scale horizontally without sticky sessions. We're careful with status codes: 201 with a `Location` header on create, 204 on delete, 400 for validation, and 401 versus 403 distinguished properly — 401 means we don't know who you are, 403 means you're known but not permitted. And we version the URL from day one so mobile clients don't break when we change contracts."

---

# 18. Controllers

## 18.1 What is it?

**Simple words:**
A controller is the class that **receives web requests**. Each method maps to a URL. Its only job is: take the request, pass it to the service, return the result. **No business logic.**

**Technical wording:**
A controller is a `@Controller`-annotated bean whose `@RequestMapping` methods are registered as handlers by `RequestMappingHandlerMapping`. `@RestController` additionally applies `@ResponseBody` to every method, so return values are serialized directly into the response body.

## 18.2 `@Controller` vs `@RestController` — key comparison

| | `@Controller` | `@RestController` |
|---|---|---|
| Composed of | `@Component` | **`@Controller` + `@ResponseBody`** |
| Return value meaning | **View name** (resolved to an HTML template) | **Response body** (serialized to JSON) |
| Needs `@ResponseBody` per method | Yes, for data responses | No — applied to all methods |
| Use for | Server-rendered pages (Thymeleaf, JSP) | **REST APIs** |
| Typical return | `"order-list"` → `order-list.html` | `OrderDto` → `{"id":1,...}` |

```java
// @Controller — returns a VIEW NAME
@Controller
public class OrderViewController {

    @GetMapping("/orders")
    public String list(Model model) {
        model.addAttribute("orders", orderService.findAll());
        return "order-list";        // → resolves to templates/order-list.html
    }

    @GetMapping("/orders/data")
    @ResponseBody                   // must add this for JSON
    public List<OrderDto> data() {
        return orderService.findAll();
    }
}

// @RestController — returns DATA. This is what you use for APIs.
@RestController
public class OrderApiController {

    @GetMapping("/api/orders")
    public List<OrderDto> list() {
        return orderService.findAll();    // automatically serialized to JSON
    }
}
```

**If you use `@Controller` and return `"order-list"` from a REST endpoint by mistake**, Spring tries to resolve a *view* called "order-list", fails, and returns a 404 or 500 — a confusing bug with an obvious cause once you know this distinction.

## 18.3 Mapping annotations

```java
@RequestMapping(value = "/orders", method = RequestMethod.GET)   // generic form
@GetMapping("/orders")        // shortcut — preferred
@PostMapping("/orders")
@PutMapping("/orders/{id}")
@PatchMapping("/orders/{id}")
@DeleteMapping("/orders/{id}")
```

The shortcuts are **composed annotations** — `@GetMapping` is `@RequestMapping(method = GET)`. Prefer them: they're shorter and make the HTTP method impossible to miss when reading the code.

**Class-level mapping** sets a common prefix:

```java
@RestController
@RequestMapping("/api/v1/orders")     // all methods inherit this prefix
public class OrderController {

    @GetMapping                       // → GET /api/v1/orders
    @GetMapping("/{id}")              // → GET /api/v1/orders/{id}
}
```

**Narrowing conditions:**

```java
@PostMapping(
    value    = "/orders",
    consumes = MediaType.APPLICATION_JSON_VALUE,   // only accept JSON requests
    produces = MediaType.APPLICATION_JSON_VALUE    // declare JSON responses
)
// Wrong Content-Type → 415 Unsupported Media Type
// Unacceptable Accept header → 406 Not Acceptable
```

## 18.4 A complete, production-shaped controller

```java
package com.company.orderservice.controller;

@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
@Validated                                   // enables validation on @PathVariable/@RequestParam
@Slf4j
public class OrderController {

    private final OrderService orderService;   // ONLY dependency — no repository here

    // ---------- CREATE ----------
    @PostMapping
    public ResponseEntity<OrderResponseDto> createOrder(
            @Valid @RequestBody OrderRequestDto request) {

        OrderResponseDto created = orderService.createOrder(request);

        // 201 + Location header pointing at the new resource
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(created.getId())
                .toUri();

        return ResponseEntity.created(location).body(created);
    }

    // ---------- READ ONE ----------
    @GetMapping("/{id}")
    public ResponseEntity<OrderResponseDto> getOrder(@PathVariable Long id) {
        // Service throws ResourceNotFoundException → handled by @RestControllerAdvice → 404
        return ResponseEntity.ok(orderService.getOrderById(id));
    }

    // ---------- READ MANY (paginated + filtered) ----------
    @GetMapping
    public ResponseEntity<Page<OrderResponseDto>> getOrders(
            @RequestParam(required = false) OrderStatus status,
            @RequestParam(defaultValue = "0")  @Min(0)  int page,
            @RequestParam(defaultValue = "20") @Min(1) @Max(100) int size) {

        // size is CAPPED at 100 — never let a client request 1,000,000 rows
        Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
        return ResponseEntity.ok(orderService.getOrders(status, pageable));
    }

    // ---------- FULL UPDATE ----------
    @PutMapping("/{id}")
    public ResponseEntity<OrderResponseDto> updateOrder(
            @PathVariable Long id,
            @Valid @RequestBody OrderRequestDto request) {
        return ResponseEntity.ok(orderService.updateOrder(id, request));
    }

    // ---------- PARTIAL UPDATE ----------
    @PatchMapping("/{id}/status")
    public ResponseEntity<OrderResponseDto> updateStatus(
            @PathVariable Long id,
            @Valid @RequestBody StatusUpdateDto request) {
        return ResponseEntity.ok(orderService.updateStatus(id, request.getStatus()));
    }

    // ---------- DELETE ----------
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteOrder(@PathVariable Long id) {
        orderService.deleteOrder(id);
        return ResponseEntity.noContent().build();      // 204
    }
}
```

**What makes this controller "correct" — point these out to a TL:**

1. **It is thin.** Every method is 1–5 lines. All logic is in the service.
2. **It depends only on the service**, never on the repository.
3. **It uses DTOs**, never entities.
4. **`@Valid` on every request body.**
5. **Page size is capped** — protects the database from a hostile or careless client.
6. **No try/catch** — exceptions are handled centrally.
7. **Correct status codes**: 201 + Location, 204 for delete.
8. **Versioned URL.**

## 18.5 Common Mistakes

| Mistake | Why it's wrong |
|---|---|
| **Business logic in the controller** | Untestable, unreusable, outside transaction boundaries |
| **Injecting the repository into the controller** | Skips the service layer and the transaction boundary |
| **Returning entities instead of DTOs** | Leaks the DB schema; causes lazy-loading serialization errors |
| **try/catch in every method** | Duplicated; use `@RestControllerAdvice` |
| **Always returning 200** | Clients can't distinguish create from update from failure |
| **Missing `@Valid`** | Invalid data reaches the service and the database |
| **Uncapped page size** | One request can pull the whole table into memory |
| **`@Autowired` field injection** | See Section 4 |
| **Storing state in controller fields** | Controllers are singletons — race conditions |

## 18.6 TL Questions

**Q: What should a controller contain?**
A: Only request/response handling — bind the input, call one service method, map the result to a status code. No business rules, no database access, no transaction management.

**Q: Why not inject the repository directly into the controller?**
A: It bypasses the service layer, which is where our transaction boundary and business rules live. It also makes the logic unreusable, since another caller — a scheduled job or a message listener — would have to duplicate it.

**Q: `@Controller` or `@RestController`?**
A: `@RestController` for our APIs. It's `@Controller` plus `@ResponseBody`, so return values are serialized to JSON instead of being treated as view names.

**Q: What happens if we remove `@RestController` and use `@Controller`?**
A: Return values are interpreted as view names. Returning an `OrderDto` would make Spring look for a view, and we'd get a 404 or 500 instead of JSON — unless we add `@ResponseBody` to every method.

**Q: Why are controllers singletons, and what's the risk?**
A: Spring creates one instance shared across all requests for efficiency. The risk is that any mutable instance field would be shared across concurrent requests, so we keep controllers stateless and hold all request data in method parameters.

**Q: What happens when an exception is thrown in a controller?**
A: It propagates to DispatcherServlet, which passes it to the `HandlerExceptionResolver` chain. `ExceptionHandlerExceptionResolver` finds the matching `@ExceptionHandler` in our `@RestControllerAdvice` and builds a consistent error response. That's why we have no try/catch in controllers.

## 18.7 Interview Questions

**Beginner — Difference between `@Controller` and `@RestController`?**
`@RestController` = `@Controller` + `@ResponseBody`. It returns data, not view names.

**Beginner — What is `@RequestMapping`?**
It maps HTTP requests to handler methods, optionally narrowing by method, params, headers, consumes and produces.

**Intermediate — Can a controller method return `void`?**
Yes. Combined with `@ResponseStatus`, or by writing to the `HttpServletResponse` directly. Spring treats the response as already handled. `ResponseEntity<Void>` is usually clearer.

**Intermediate — What does `produces` actually do?**
It restricts the mapping to requests whose `Accept` header is compatible, and sets the response `Content-Type`. A mismatch produces 406 Not Acceptable.

**Advanced — How would you inject the authenticated user into a controller method?**
Spring Security supports `@AuthenticationPrincipal UserDetails user` or a `Principal` parameter. For anything custom, implement a `HandlerMethodArgumentResolver` and register it via `WebMvcConfigurer.addArgumentResolvers` — that's the extension point argument binding is built on.

**Advanced — Are controller methods thread-safe?**
The method invocation is, because local variables and parameters are per-thread. The *bean* is not, because it's a shared singleton — so instance fields must never hold request state.

## 18.8 Quick Revision

1. `@RestController` = `@Controller` + `@ResponseBody`; use it for APIs.
2. Controllers are **thin**: bind → delegate → map to status code.
3. Inject the **service**, never the repository.
4. Return **DTOs**, never entities.
5. Use `@GetMapping`/`@PostMapping` shortcuts; class-level `@RequestMapping` for the prefix.
6. Always `@Valid` the request body; always cap page size.
7. No try/catch — central `@RestControllerAdvice`.
8. Correct codes: 201+Location on create, 204 on delete.
9. Controllers are singletons — keep them stateless.

## 18.9 TL Explanation (speak this)

> "Our controllers are deliberately thin — they bind the request, call a single service method and map the result to the right status code, and that's all. Business logic and the transaction boundary sit in the service, so a scheduled job or a Kafka listener can reuse the same logic. We use `@RestController`, so returns are serialized to JSON, we validate every request body with `@Valid`, and we always return DTOs rather than entities so we're not leaking the database schema. There's no try/catch in the controllers because exceptions go to a central `@RestControllerAdvice`."

---

# 19. Reading Request Data — @PathVariable, @RequestParam, @RequestBody

## 19.1 The three ways data arrives

```
POST /api/v1/orders/42/items?includeTax=true&currency=INR
Content-Type: application/json
Authorization: Bearer eyJhbGc...

{ "productId": 100, "quantity": 2 }

      │              │                    │              │
      │              │                    │              └─► HEADER  → @RequestHeader
      │              │                    └─► BODY          → @RequestBody
      │              └─► QUERY PARAMS  ?includeTax=true    → @RequestParam
      └─► PATH VARIABLE  /orders/{orderId}/items            → @PathVariable
```

## 19.2 The comparison table (very common interview question)

| Feature | `@PathVariable` | `@RequestParam` | `@RequestBody` |
|---|---|---|---|
| Reads from | URL **path** | URL **query string** or form data | Request **body** |
| Example | `/orders/42` | `/orders?status=NEW` | `{"amount":500}` |
| Typical use | **Identify** a specific resource | **Filter / sort / paginate** | **Send complex data** |
| Data type | Simple (String, Long, enum) | Simple, or List | Complex object (DTO) |
| How many per method | Many | Many | **Only one** |
| Optional? | `required=false` (needs a second mapping) | `required=false` or `defaultValue` | `required=false` |
| Conversion by | `WebDataBinder` / `Converter` | `WebDataBinder` / `Converter` | **`HttpMessageConverter` (Jackson)** |
| Used with | GET, PUT, DELETE, PATCH | Mostly GET | POST, PUT, PATCH |
| Visible in logs/history | Yes | Yes — **never put secrets here** | No (body isn't usually logged) |

**Rule of thumb:**
- Identifies **which** resource → `@PathVariable`
- Modifies **how** you retrieve it → `@RequestParam`
- **Is** the data → `@RequestBody`

## 19.3 `@PathVariable`

```java
// Simple case — parameter name matches {id}
@GetMapping("/orders/{id}")
public OrderDto getOrder(@PathVariable Long id) { ... }

// Explicit name — needed when names differ
@GetMapping("/orders/{orderId}")
public OrderDto getOrder(@PathVariable("orderId") Long id) { ... }

// Multiple path variables — nested resources
@GetMapping("/customers/{customerId}/orders/{orderId}")
public OrderDto get(@PathVariable Long customerId,
                    @PathVariable Long orderId) { ... }

// All at once
@GetMapping("/customers/{customerId}/orders/{orderId}")
public OrderDto get(@PathVariable Map<String, String> vars) {
    String customerId = vars.get("customerId");
}
```

> **Compilation note:** relying on the parameter name requires the class to be compiled with debug symbols (`-parameters`). Spring Boot's Maven plugin enables this by default; if you ever see *"Name for argument of type [Long] not specified"*, add the explicit name.

## 19.4 `@RequestParam`

```java
// Required (default)
@GetMapping("/orders")
public List<OrderDto> get(@RequestParam String status) { }
// Missing → 400 Bad Request: "Required request parameter 'status' is not present"

// Optional with default — the common production pattern
@GetMapping("/orders")
public Page<OrderDto> get(
        @RequestParam(defaultValue = "0")  int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(required = false) OrderStatus status) { }

// Optional wrapper
@RequestParam Optional<String> status

// List:  ?ids=1&ids=2&ids=3   OR   ?ids=1,2,3
@GetMapping("/orders")
public List<OrderDto> byIds(@RequestParam List<Long> ids) { }

// All params as a map
@RequestParam Map<String, String> allParams
```

**Important:** `@RequestParam` also reads `application/x-www-form-urlencoded` form bodies, not just query strings. That's why HTML form posts bind with `@RequestParam`, not `@RequestBody`.

### Better: bind many params into an object

When a search endpoint has 6+ filters, stop listing parameters:

```java
@Getter @Setter
public class OrderSearchCriteria {
    private OrderStatus status;
    private LocalDate fromDate;
    private LocalDate toDate;
    private BigDecimal minAmount;
    @Min(0) private int page = 0;
    @Min(1) @Max(100) private int size = 20;
}

@GetMapping("/orders")
public Page<OrderDto> search(@Valid OrderSearchCriteria criteria) { ... }
// NOTE: no annotation needed — Spring binds query params onto the object fields.
// This also gets you validation for free.
```

## 19.5 `@RequestBody`

```java
@PostMapping("/orders")
public ResponseEntity<OrderResponseDto> create(
        @Valid @RequestBody OrderRequestDto request) { ... }
```

**What happens internally, step by step:**

```
1. RequestResponseBodyMethodProcessor handles the parameter
                │
2. Reads the Content-Type header  →  application/json
                │
3. Selects an HttpMessageConverter that can read JSON
   → MappingJackson2HttpMessageConverter
                │
4. Jackson's ObjectMapper deserializes the body into OrderRequestDto
   - matches JSON field names to setters/fields
   - converts types (string → BigDecimal, string → LocalDate)
   - unknown field → FAIL by default in Boot? No: Boot sets
     FAIL_ON_UNKNOWN_PROPERTIES=false, so extra fields are ignored
                │
5. @Valid present → run Bean Validation
   - violation → MethodArgumentNotValidException → 400
                │
6. The populated, validated DTO is passed to your method
```

**Requirements for Jackson deserialization:**
- A **no-argument constructor** (or `@JsonCreator` / a `record` / a Lombok `@NoArgsConstructor`)
- **Setters**, or field access
- **Field names matching the JSON** (or `@JsonProperty("json_name")`)

**Common failure:** using Lombok's `@Builder` alone removes the no-arg constructor, causing *"Cannot construct instance ... no Creators"*. Fix: add `@NoArgsConstructor @AllArgsConstructor` alongside `@Builder`.

**Only one `@RequestBody` per method** — there's a single body to read, and the stream can be consumed once.

## 19.6 Other useful binding annotations

```java
@GetMapping("/orders")
public List<OrderDto> get(
        @RequestHeader("Authorization") String authHeader,
        @RequestHeader(value = "X-Correlation-Id", required = false) String correlationId,
        @CookieValue(value = "sessionId", required = false) String sessionId,
        HttpServletRequest request,          // raw servlet request
        Principal principal) { }             // authenticated user (Spring Security)

// File upload
@PostMapping(value = "/orders/{id}/invoice", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public ResponseEntity<Void> upload(@PathVariable Long id,
                                   @RequestPart("file") MultipartFile file) { }
```

## 19.7 Type conversion and a real gotcha

Spring converts `String` → target type automatically for primitives, wrappers, enums and common types. For dates you usually need a format hint:

```java
// Without this, "2024-01-15" often fails to bind to LocalDate
@GetMapping("/orders")
public List<OrderDto> get(
    @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate fromDate) { }
```

**Enum binding:** `?status=PENDING` binds to `OrderStatus.PENDING`. An invalid value throws `MethodArgumentTypeMismatchException` → handle it in your advice to return a clean 400 listing the allowed values, instead of a raw 500.

## 19.8 Common Mistakes

| Mistake | Consequence |
|---|---|
| Two `@RequestBody` parameters | `HttpMessageNotReadableException` — body already consumed |
| Sensitive data in query params | Logged in access logs, browser history, proxies. **Use the body.** |
| No `defaultValue` on an optional param | 400 when the client omits it |
| DTO without a no-arg constructor | Jackson can't instantiate it |
| Mismatch between `{id}` and the parameter name without an explicit name | Binding failure |
| `@RequestBody` on a GET | GET bodies are not reliably supported by clients/proxies |
| Using `@RequestParam` for a JSON body | Body won't bind — 400 |

## 19.9 TL Questions

**Q: When do we use `@PathVariable` versus `@RequestParam`?**
A: `@PathVariable` identifies which resource — the ID is part of the resource's address. `@RequestParam` modifies how we retrieve it — filters, sorting, pagination. So `/orders/42?includeItems=true`.

**Q: Why can there be only one `@RequestBody`?**
A: There's a single HTTP body and the input stream can only be read once. If we need several pieces of data, they belong in one DTO.

**Q: How does JSON become our DTO internally?**
A: `RequestResponseBodyMethodProcessor` picks an `HttpMessageConverter` based on `Content-Type`; for JSON that's Jackson, which deserializes the body into the DTO. Then `@Valid` runs Bean Validation before our method is called.

**Q: What happens if a client sends a field we don't have in the DTO?**
A: Spring Boot configures Jackson with `FAIL_ON_UNKNOWN_PROPERTIES` disabled, so unknown fields are ignored rather than causing an error. That's usually what we want for forward compatibility, but it does mean a typo in a field name fails silently — which is why validation on required fields matters.

**Q: Why don't we put the token in a query parameter?**
A: Query strings are recorded in access logs, browser history and proxy logs. Credentials belong in the `Authorization` header or the body, never the URL.

**Q: What happens if the client sends an invalid enum value?**
A: Spring throws `MethodArgumentTypeMismatchException`. We handle it in `@RestControllerAdvice` and return a 400 listing the valid values, rather than letting it become a 500.

## 19.10 Interview Questions

**Beginner — Which annotation reads the JSON body?**
`@RequestBody`.

**Beginner — How do you give a query param a default?**
`@RequestParam(defaultValue = "20") int size`.

**Intermediate — `@RequestParam` vs `@RequestPart`?**
`@RequestParam` handles simple values and can handle a `MultipartFile`, converting via `WebDataBinder`. `@RequestPart` handles a named part of a multipart request and applies `HttpMessageConverter`s — so it's what you use when a part is itself JSON.

**Intermediate — How do you bind many query params cleanly?**
Declare a POJO parameter with no annotation. Spring binds matching query params onto its fields, and `@Valid` works on it. Much cleaner than eight `@RequestParam`s.

**Advanced — What is `WebDataBinder` and how do you customise conversion?**
It performs binding and type conversion for non-body parameters. Customise it per controller with `@InitBinder`, or globally by registering a `Converter`/`Formatter` in `WebMvcConfigurer.addFormatters` — the right place for a project-wide custom date format.

**Advanced — Why is `@RequestBody` on a GET request a bad idea?**
The HTTP spec assigns no semantics to a GET body; many proxies, caches and client libraries strip or reject it, and it breaks caching. Use query params, or make it a POST search endpoint if the criteria are genuinely complex.

## 19.11 Quick Revision

1. `@PathVariable` = which resource. `@RequestParam` = how to retrieve. `@RequestBody` = the data.
2. Only **one** `@RequestBody` per method.
3. `@RequestParam` supports `defaultValue`, `required`, `List`, `Map`, and form bodies.
4. `@RequestBody` uses `HttpMessageConverter`/Jackson; the others use `WebDataBinder`.
5. Bind many query params into a POJO — cleaner and validatable.
6. Never put secrets in the URL.
7. Use `@DateTimeFormat` for date parameters.
8. DTOs need a no-arg constructor for Jackson.
9. `@RequestHeader`, `@CookieValue`, `@RequestPart` cover the rest.

## 19.12 TL Explanation (speak this)

> "We use `@PathVariable` for the resource identifier, `@RequestParam` for filters and paging, and `@RequestBody` for the payload — so a create is a POST with a DTO body, and a search is a GET with query params. There's only one `@RequestBody` per method because the request stream is read once. Internally `@RequestBody` goes through Jackson via an `HttpMessageConverter`, and `@Valid` runs right after binding, before our method is called. We never put tokens or sensitive values in query strings because they end up in access logs."

---

# 20. Sending Responses — ResponseEntity and Status Codes

## 20.1 Three ways to return a response

```java
// 1. Return the object directly — always 200 OK
@GetMapping("/{id}")
public OrderDto getOrder(@PathVariable Long id) {
    return orderService.getOrderById(id);
}

// 2. Object + @ResponseStatus — fixed, non-200 status
@PostMapping
@ResponseStatus(HttpStatus.CREATED)          // always 201
public OrderDto create(@Valid @RequestBody OrderRequestDto req) {
    return orderService.createOrder(req);
}

// 3. ResponseEntity — FULL control over status, headers and body (PREFERRED)
@PostMapping
public ResponseEntity<OrderDto> create(@Valid @RequestBody OrderRequestDto req) {
    OrderDto created = orderService.createOrder(req);
    return ResponseEntity
            .status(HttpStatus.CREATED)
            .header("X-Order-Id", created.getId().toString())
            .body(created);
}
```

| | Direct return | `@ResponseStatus` | `ResponseEntity` |
|---|---|---|---|
| Status control | Always 200 | Fixed at compile time | **Dynamic at runtime** |
| Custom headers | No | No | **Yes** |
| Conditional status | No | No | **Yes** |
| Verbosity | Lowest | Low | Higher |
| **Recommended for APIs** | Simple GETs | Simple creates | **Yes — default choice** |

## 20.2 ResponseEntity builder methods

```java
ResponseEntity.ok(dto)                                  // 200 + body
ResponseEntity.ok().build()                             // 200, no body
ResponseEntity.created(location).body(dto)              // 201 + Location header
ResponseEntity.accepted().body(dto)                     // 202
ResponseEntity.noContent().build()                      // 204
ResponseEntity.badRequest().body(errorDto)              // 400
ResponseEntity.notFound().build()                       // 404
ResponseEntity.status(HttpStatus.CONFLICT).body(err)    // any status
ResponseEntity.status(409).header("Retry-After","30").body(err)
```

**Creating with a Location header — the correct REST pattern:**

```java
@PostMapping
public ResponseEntity<OrderDto> create(@Valid @RequestBody OrderRequestDto req) {
    OrderDto created = orderService.createOrder(req);

    URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();                       // → http://host/api/v1/orders/42

    return ResponseEntity.created(location).body(created);
}
```
The client now knows where the new resource lives without guessing the URL pattern.

## 20.3 How the object becomes JSON

```
Controller returns OrderDto
        │
        ▼
HandlerMethodReturnValueHandler  (RequestResponseBodyMethodProcessor)
        │
        ▼
Content negotiation: what does the client's Accept header allow?
        │
        ▼
Select HttpMessageConverter → MappingJackson2HttpMessageConverter
        │
        ▼
Jackson ObjectMapper serializes:
   - getters become JSON fields  (getCustomerId() → "customerId")
   - @JsonProperty renames
   - @JsonIgnore omits
   - null handling per configuration
        │
        ▼
JSON bytes written to the response output stream
```

**Useful Jackson annotations on DTOs:**

```java
public class OrderResponseDto {

    private Long id;

    @JsonProperty("customer_id")                       // rename in JSON
    private String customerId;

    @JsonIgnore                                        // never serialize
    private String internalNotes;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")       // date format
    private LocalDateTime createdAt;

    @JsonInclude(JsonInclude.Include.NON_NULL)         // omit if null
    private String cancelReason;
}
```

```properties
# Global: omit all null fields from every response
spring.jackson.default-property-inclusion=non_null
# Global: ISO dates instead of timestamps
spring.jackson.serialization.write-dates-as-timestamps=false
```

## 20.4 Standard response envelope (a common corporate pattern)

Many teams wrap every response in a consistent shape so clients can parse uniformly:

```java
@Getter @AllArgsConstructor
public class ApiResponse<T> {
    private boolean success;
    private String message;
    private T data;
    private Instant timestamp;

    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, "Success", data, Instant.now());
    }
}

@GetMapping("/{id}")
public ResponseEntity<ApiResponse<OrderDto>> get(@PathVariable Long id) {
    return ResponseEntity.ok(ApiResponse.success(orderService.getOrderById(id)));
}
```

**Balanced view to give a TL:** an envelope gives clients one parsing shape and a place for metadata. The counter-argument is that HTTP *already* has a status code and headers for this, so the envelope duplicates them and can encourage the anti-pattern of returning 200 with `success: false`. **If you use an envelope, still set the correct HTTP status code.** Never return 200 for an error.

## 20.5 Common Mistakes

| Mistake | Consequence |
|---|---|
| **200 with an error body** | Clients, retry logic, monitoring and load balancers all treat it as success |
| Returning entities directly | Schema leak + `LazyInitializationException` |
| Forgetting the `Location` header on 201 | Client must construct the URL itself |
| Returning `null` from a controller | 200 with an empty body — use 404 |
| Exposing stack traces in responses | **Security risk** — leaks internals |
| Returning an unbounded list | Memory blowout — paginate |

## 20.6 TL Questions

**Q: Why do we use `ResponseEntity`?**
A: It gives us runtime control over the status code, headers and body. We can return 201 with a `Location` header on create and 204 on delete, instead of always returning 200.

**Q: How does our object become JSON?**
A: The return value handler performs content negotiation against the `Accept` header, selects the Jackson message converter, and Jackson serializes the object's getters into JSON on the response stream.

**Q: Why not return the entity directly?**
A: It couples the API contract to the database schema, so a column rename becomes a breaking API change. It also exposes fields we don't want public, and serializing a lazy association outside the persistence context throws `LazyInitializationException`.

**Q: Should errors return 200 with a flag?**
A: No. The HTTP status code is the contract that infrastructure understands — load balancers, retry logic, monitoring and the client library all key off it. An error must be 4xx or 5xx.

**Q: What happens if a controller returns null?**
A: The client gets a 200 with an empty body, which is misleading. If a resource doesn't exist we throw a `ResourceNotFoundException` so the advice returns a proper 404.

## 20.7 Interview Questions

**Beginner — What is `ResponseEntity`?**
A wrapper representing the full HTTP response: status code, headers and body.

**Intermediate — `@ResponseStatus` vs `ResponseEntity`?**
`@ResponseStatus` fixes the status at compile time; `ResponseEntity` decides it at runtime and also allows custom headers. `@ResponseStatus` is convenient on exception classes.

**Intermediate — What is content negotiation?**
Choosing the response representation based on the client's `Accept` header, the method's `produces` attribute, and the registered converters. With only Jackson on the classpath, it effectively always resolves to JSON.

**Advanced — How do you add a custom `HttpMessageConverter`?**
Implement `WebMvcConfigurer` and override `extendMessageConverters` (to adjust the defaults) or `configureMessageConverters` (to replace them entirely). Order matters — the first converter that can handle the media type wins.

**Advanced — How do you stream a large response?**
Return `StreamingResponseBody`, or a `ResponseEntity<Resource>` for files, so data is written incrementally rather than buffered fully in memory. For long-lived pushes, `SseEmitter` provides server-sent events.

## 20.8 Quick Revision

1. Prefer `ResponseEntity` — full control of status, headers, body.
2. 201 + `Location` on create; 204 on delete; 200 for reads and updates.
3. Never return 200 for an error.
4. Never return entities — return DTOs.
5. Jackson serializes via `HttpMessageConverter` after content negotiation.
6. `@JsonIgnore`, `@JsonProperty`, `@JsonFormat`, `@JsonInclude` shape the JSON.
7. Never expose stack traces to clients.
8. Always paginate collection endpoints.

## 20.9 TL Explanation (speak this)

> "We return `ResponseEntity` from our controllers so we control the status code and headers at runtime — 201 with a `Location` header when we create something, 204 on delete, 200 on reads. The body is always a DTO, never an entity, so the API contract doesn't move every time the schema changes and we avoid lazy-loading serialization errors. Jackson does the serialization through a message converter after content negotiation. The rule we hold firmly is that errors get a real 4xx or 5xx status — never a 200 with a success flag set to false, because all our infrastructure keys off the status code."

---

# 21. DTO Pattern

## 21.1 What is it?

**Simple words:**
A DTO (Data Transfer Object) is a **simple class used only to carry data between your API and the outside world**. It is not your database entity — it's the shape you agreed to expose to clients.

Think of it as the difference between what's in your warehouse (entity) and what's on the price tag the customer sees (DTO).

**Technical wording:**
A DTO is a flat, serializable object used to transfer data across architectural boundaries, decoupling the external API contract from the internal domain/persistence model.

## 21.2 Why do we use DTOs? (Seven concrete reasons)

### Reason 1 — Security: don't leak sensitive fields

```java
@Entity
public class User {
    private Long id;
    private String username;
    private String password;      // BCrypt hash
    private String ssn;
    private boolean internalFlag;
}
```
Return this entity from a controller and Jackson serializes **every** field — including the password hash and SSN. A DTO exposes only what's safe.

### Reason 2 — Decoupling: schema changes don't break clients
Rename a column, split a table, or switch to a different persistence model — the DTO keeps the API contract stable. Without DTOs, **every database refactor becomes a breaking API change**.

### Reason 3 — Avoid `LazyInitializationException`
Serializing an entity with a `LAZY` association after the transaction has closed throws, because Jackson triggers the proxy outside the persistence context. DTOs are plain objects built while the session is open — no proxies, no surprise.

### Reason 4 — Avoid infinite recursion
A bidirectional relationship (`Order` → `List<OrderItem>` → `Order` → …) makes Jackson loop forever and blow the stack. DTOs are unidirectional by construction.

### Reason 5 — Shape data for the client
One screen may need order + customer name + item count — data from three tables. A DTO models the *response*, not a table.

### Reason 6 — Different shapes for input and output
The create request has no `id` and no `createdAt`; the response has both and no password. One entity can't represent both correctly.

### Reason 7 — Validation belongs on the DTO
Validation rules are API concerns (`@NotBlank`, `@Email`). Putting them on the entity mixes API validation with persistence constraints.

## 21.3 Entity vs DTO

| Aspect | Entity | DTO |
|---|---|---|
| Purpose | Map to a DB table | Transfer data over the API |
| Annotations | `@Entity`, `@Table`, `@Column`, `@OneToMany` | Validation + Jackson only |
| Contains | Persistence state, relationships, lazy proxies | Plain, flat fields |
| Layer | Repository / persistence | Controller / API boundary |
| Managed by | Hibernate persistence context | Nobody — plain object |
| Changes when | The schema changes | The API contract changes |
| Exposed to clients | **Never** | Yes |

## 21.4 Real example — request and response DTOs

```java
// ---------- dto/OrderRequestDto.java (what the client SENDS) ----------
@Getter @Setter
@NoArgsConstructor @AllArgsConstructor     // no-arg needed by Jackson
public class OrderRequestDto {

    @NotBlank(message = "Customer ID is required")
    private String customerId;

    @NotEmpty(message = "Order must contain at least one item")
    @Valid                                  // cascade validation into each item
    private List<OrderItemDto> items;

    @NotNull(message = "Payment mode is required")
    private PaymentMode paymentMode;

    // NOTE: no id, no createdAt, no status — the SERVER decides those.
    // Accepting them from the client is a security hole (mass-assignment).
}

// ---------- dto/OrderResponseDto.java (what the client RECEIVES) ----------
@Getter @Builder
public class OrderResponseDto {

    private Long id;
    private String customerId;
    private String customerName;             // joined from another table
    private BigDecimal totalAmount;
    private OrderStatus status;

    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    private LocalDateTime createdAt;

    private List<OrderItemResponseDto> items;

    // NOTE: no internal audit fields, no version column, no soft-delete flag
}
```

**Mass assignment — a real security issue:** if you bind the request straight onto the entity, a client can send `{"status":"PAID","totalAmount":0}` and set fields they should never control. A request DTO that simply doesn't contain those fields makes the attack impossible.

## 21.5 Mapping between entity and DTO

### Option 1 — Manual mapper (no dependency, fully explicit)

```java
@Component
public class OrderMapper {

    public Order toEntity(OrderRequestDto dto) {
        Order order = new Order();
        order.setCustomerId(dto.getCustomerId());
        order.setPaymentMode(dto.getPaymentMode());
        order.setStatus(OrderStatus.PENDING);      // server-controlled
        return order;
    }

    public OrderResponseDto toDto(Order order) {
        return OrderResponseDto.builder()
                .id(order.getId())
                .customerId(order.getCustomerId())
                .totalAmount(order.getTotalAmount())
                .status(order.getStatus())
                .createdAt(order.getCreatedAt())
                .items(order.getItems().stream()
                            .map(this::toItemDto)
                            .toList())
                .build();
    }
}
```

### Option 2 — MapStruct (compile-time, recommended for large projects)

```java
@Mapper(componentModel = "spring")          // generates a Spring bean
public interface OrderMapper {

    OrderResponseDto toDto(Order order);

    @Mapping(target = "id",     ignore = true)
    @Mapping(target = "status", constant = "PENDING")
    Order toEntity(OrderRequestDto dto);
}
```
MapStruct **generates the implementation at compile time** — so it's as fast as hand-written code, and a field mismatch is a compile error rather than a runtime surprise.

### Mapping approaches compared

| | Manual | MapStruct | ModelMapper |
|---|---|---|---|
| Speed | Fastest | Fastest (generated) | Slower (reflection) |
| Errors caught | Compile time | **Compile time** | **Runtime** |
| Boilerplate | High | Very low | Very low |
| Debuggable | Yes | Yes (readable generated code) | Harder |
| **Recommendation** | Small projects | **Large projects** | Prototypes |

**Warning about ModelMapper:** it maps by name at runtime using reflection. A field rename silently stops mapping and you ship a `null`. For that reason many teams standardise on MapStruct.

## 21.6 Where should mapping happen?

**Recommended: in the service layer.**

```java
@Service
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final OrderRepository orderRepository;
    private final OrderMapper orderMapper;

    @Override
    @Transactional
    public OrderResponseDto createOrder(OrderRequestDto request) {
        Order order = orderMapper.toEntity(request);          // DTO → entity
        order.setTotalAmount(calculateTotal(request.getItems()));
        Order saved = orderRepository.save(order);
        return orderMapper.toDto(saved);                       // entity → DTO
    }
}
```

**Why the service and not the controller?** Because mapping the entity to a DTO must happen **while the transaction is still open**, so lazy associations can still be loaded. If the controller did the mapping, the session would already be closed and you'd get `LazyInitializationException` (unless `open-in-view` is on — which you should disable; see Section 39).

**Key rule: the entity must never escape the service layer.**

```
Controller  ←── DTO ──►  Service  ←── Entity ──►  Repository  ←──►  DB
                            ▲
                    mapping happens HERE,
                    inside the transaction
```

## 21.7 Projections — a lighter alternative

For read-only endpoints you can skip the mapper entirely by having the query return the DTO directly:

```java
// Interface projection — Spring Data implements it
public interface OrderSummary {
    Long getId();
    String getCustomerId();
    BigDecimal getTotalAmount();
}

public interface OrderRepository extends JpaRepository<Order, Long> {
    List<OrderSummary> findByStatus(OrderStatus status);
}

// Class-based (DTO) projection via JPQL constructor expression
@Query("SELECT new com.company.dto.OrderSummaryDto(o.id, o.customerId, o.totalAmount) " +
       "FROM Order o WHERE o.status = :status")
List<OrderSummaryDto> findSummaries(@Param("status") OrderStatus status);
```

**Why this is a performance win:** the generated SQL selects **only those three columns** instead of `SELECT *`, and no entities enter the persistence context, so there's no dirty-checking overhead. For list screens on wide tables this is a measurable improvement — a strong point to raise with a senior.

## 21.8 Common Mistakes

| Mistake | Consequence |
|---|---|
| Returning entities from controllers | Schema leak, lazy exceptions, recursion, security exposure |
| One DTO for both request and response | Exposes server-controlled fields; weakens validation |
| Accepting `id`/`status` in a create request | Mass-assignment vulnerability |
| Mapping in the controller | `LazyInitializationException` |
| Putting business logic in a DTO | DTOs are dumb data carriers |
| JPA annotations on a DTO | Confuses the layers |
| Too many near-identical DTOs | Maintenance burden — consolidate sensibly |

## 21.9 TL Questions

**Q: Why do we use DTOs instead of returning entities?**
A: Three main reasons — security, because entities contain fields like password hashes we must never serialize; decoupling, so a schema change isn't a breaking API change; and stability, because serializing lazy associations outside the transaction throws `LazyInitializationException` and bidirectional relationships cause infinite recursion in Jackson.

**Q: Where do we do the mapping?**
A: In the service layer, inside the transaction, so lazy associations can still be initialised. The entity never leaves the service — the controller only ever sees DTOs.

**Q: Why separate request and response DTOs?**
A: They have genuinely different shapes. The request has no `id`, `status` or `createdAt` because the server owns those; accepting them from the client would be a mass-assignment vulnerability. The response has them but excludes internal audit fields.

**Q: What mapping library do we use and why?**
A: MapStruct. It generates the mapping code at compile time, so it's as fast as hand-written code and a field mismatch is a compile error. Reflection-based mappers like ModelMapper fail silently at runtime when a field is renamed.

**Q: What alternative is there for read-heavy endpoints?**
A: Projections — either an interface projection or a JPQL constructor expression returning the DTO directly. The SQL then selects only the needed columns and nothing enters the persistence context, which is noticeably faster for list screens.

**Q: Isn't this extra boilerplate?**
A: Some, yes, and MapStruct removes most of it. The trade is worth it: without DTOs every database change risks breaking clients, and we'd be one careless `return entity` away from serializing a password hash.

## 21.10 Interview Questions

**Beginner — What is a DTO?**
A plain object used to carry data between layers or across the API boundary, decoupled from the persistence model.

**Intermediate — Name three concrete problems DTOs prevent.**
Leaking sensitive entity fields, `LazyInitializationException` when serializing lazy associations, and infinite recursion from bidirectional relationships.

**Intermediate — Can a DTO be a Java `record`?**
Yes, and it's an excellent fit for **response** DTOs — immutable and concise. Jackson supports records natively. For **request** DTOs it works too, but validation annotations must go on the record components, and some frameworks still expect a no-arg constructor.

**Advanced — DTO vs Value Object vs Entity?**
A DTO is a boundary data carrier with no behaviour. A Value Object (DDD) is immutable, has no identity, is compared by value, and *can* hold domain behaviour (e.g. `Money`). An Entity has identity and lifecycle.

**Advanced — When is it acceptable to skip DTOs?**
Internal tools, throwaway prototypes, or a service with a single trusted consumer and no sensitive fields. Even then, expect the API to outlive that assumption. For anything public or long-lived, DTOs are not optional.

## 21.11 Quick Revision

1. DTO = plain data carrier at the API boundary. Not an entity.
2. Prevents: sensitive-field leaks, schema coupling, lazy exceptions, recursion, mass assignment.
3. **Separate request and response DTOs.**
4. Never accept server-controlled fields (`id`, `status`) in a create request.
5. Map in the **service layer, inside the transaction**.
6. **The entity must never leave the service layer.**
7. MapStruct = compile-time, safe. ModelMapper = runtime reflection, risky.
8. Projections give DTOs straight from the query — fewer columns, no persistence context.
9. DTOs hold validation and Jackson annotations, never business logic or JPA annotations.

## 21.12 TL Explanation (speak this)

> "DTOs are how we keep the API contract separate from the database schema. Entities never leave the service layer — the controller only sees DTOs. That protects us three ways: we can't accidentally serialize a password hash, a column rename doesn't break our clients, and we avoid `LazyInitializationException` and Jackson recursion from bidirectional relationships. We keep request and response DTOs separate so the client can't set server-owned fields like status, and we map with MapStruct inside the transaction so field mismatches are caught at compile time. For read-heavy list screens we skip entities entirely and use a projection, so the SQL selects only the columns we need."

---

# 22. Validation

## 22.1 What is it?

**Simple words:**
Validation means **checking that the data the client sent is acceptable before you use it** — is the email really an email, is the amount positive, is the required field actually present.

Spring lets you declare these rules as annotations on the DTO instead of writing `if` checks everywhere.

**Technical wording:**
Spring integrates Jakarta Bean Validation (JSR-380), implemented by Hibernate Validator. Constraint annotations on DTO fields are evaluated by a `Validator` when `@Valid`/`@Validated` is present, producing `ConstraintViolation`s that Spring converts into `MethodArgumentNotValidException`.

## 22.2 Why validate at the API boundary?

- **Fail fast** — reject bad data before it reaches business logic or the database
- **Security** — the first line of defence against malformed and malicious input
- **Clear errors** — the client learns exactly which field is wrong
- **No duplication** — declared once on the DTO instead of repeated `if` blocks
- **Data integrity** — the database never receives invalid rows

## 22.3 Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```
**Important:** since Spring Boot 2.3 this starter is **no longer included** in `spring-boot-starter-web`. If your `@Valid` annotations appear to do nothing, this missing dependency is the most common cause.

## 22.4 The constraint annotations

| Annotation | Applies to | Checks |
|---|---|---|
| `@NotNull` | Any | Not null (**empty string passes!**) |
| `@NotEmpty` | String, Collection, Map, Array | Not null **and** size > 0 (`" "` passes) |
| `@NotBlank` | String | Not null, and contains non-whitespace |
| `@Size(min, max)` | String, Collection | Length/size in range |
| `@Min` / `@Max` | Numbers | Numeric bounds |
| `@Positive` / `@PositiveOrZero` | Numbers | > 0 / >= 0 |
| `@Negative` / `@NegativeOrZero` | Numbers | < 0 / <= 0 |
| `@DecimalMin` / `@DecimalMax` | BigDecimal | Bounds with precision |
| `@Digits(integer, fraction)` | Numbers | Digit counts — ideal for money |
| `@Email` | String | Email format |
| `@Pattern(regexp)` | String | Regex match |
| `@Past` / `@PastOrPresent` | Dates | In the past |
| `@Future` / `@FutureOrPresent` | Dates | In the future |
| `@AssertTrue` / `@AssertFalse` | boolean | Must be true/false |
| `@Valid` | Object / Collection | **Cascade** validation into nested objects |

### The `@NotNull` vs `@NotEmpty` vs `@NotBlank` trap

This is asked in almost every interview:

| Input | `@NotNull` | `@NotEmpty` | `@NotBlank` |
|---|---|---|---|
| `null` | FAIL | FAIL | FAIL |
| `""` | **PASS** | FAIL | FAIL |
| `"   "` | **PASS** | **PASS** | FAIL |
| `"abc"` | PASS | PASS | PASS |

**For a String field the client must actually fill in, use `@NotBlank`.** Using `@NotNull` on a String is a common bug — an empty string sails straight through into the database.

## 22.5 Example DTO with validation

```java
@Getter @Setter
@NoArgsConstructor
public class CustomerRequestDto {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between {min} and {max} characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Email format is invalid")
    private String email;

    @NotBlank(message = "Mobile number is required")
    @Pattern(regexp = "^[6-9]\\d{9}$", message = "Mobile must be a valid 10-digit number")
    private String mobile;

    @NotNull(message = "Date of birth is required")
    @Past(message = "Date of birth must be in the past")
    private LocalDate dateOfBirth;

    @NotNull(message = "Amount is required")
    @DecimalMin(value = "0.01", message = "Amount must be greater than zero")
    @Digits(integer = 10, fraction = 2, message = "Amount can have at most 2 decimal places")
    private BigDecimal creditLimit;

    @NotEmpty(message = "At least one address is required")
    @Valid                               // ← CASCADES validation into each AddressDto
    private List<AddressDto> addresses;
}
```

**Note the `{min}`/`{max}` placeholders** — they interpolate the constraint's own values, so the message stays correct if you change the bounds.

**The `@Valid` on the list is essential.** Without it, the constraints *inside* `AddressDto` are never checked. Nested validation does not happen automatically — a very common oversight.

## 22.6 Triggering validation

```java
// Request body — the most common case
@PostMapping
public ResponseEntity<CustomerDto> create(@Valid @RequestBody CustomerRequestDto request) { }
// Violation → MethodArgumentNotValidException → 400

// Query params / path variables bound to a POJO
@GetMapping
public Page<OrderDto> search(@Valid OrderSearchCriteria criteria) { }

// Individual params — needs @Validated on the CLASS
@RestController
@Validated                                          // ← REQUIRED for the line below
public class OrderController {
    @GetMapping("/{id}")
    public OrderDto get(@PathVariable @Min(1) Long id) { }
}
// Violation → ConstraintViolationException (NOT MethodArgumentNotValidException!)

// Service-layer method validation
@Service
@Validated
public class OrderService {
    public void process(@Valid OrderRequestDto dto, @NotBlank String userId) { }
}
```

## 22.7 `@Valid` vs `@Validated` — the key comparison

| Feature | `@Valid` | `@Validated` |
|---|---|---|
| Defined by | Jakarta Bean Validation (JSR-380) | **Spring** |
| Placed on | Method parameters, fields | **Classes**, methods, parameters |
| Validation groups | **No** | **Yes** |
| Nested/cascade validation | **Yes** | **No** (not on its own) |
| Method-level validation on params | No | **Yes** (via AOP proxy) |
| Exception thrown | `MethodArgumentNotValidException` | `ConstraintViolationException` |
| Typical use | `@RequestBody` DTOs, nested objects | Class-level to enable param validation; groups |

**The two must be understood together:** use `@Valid` on the request body and on nested fields; use `@Validated` on the class when you want constraints on individual parameters or need validation groups. **They throw different exceptions, so your exception handler must cover both.**

## 22.8 Validation Groups — different rules for create vs update

```java
public interface OnCreate { }
public interface OnUpdate { }

@Getter @Setter
public class CustomerDto {

    @Null(groups = OnCreate.class,    message = "ID must not be provided when creating")
    @NotNull(groups = OnUpdate.class, message = "ID is required when updating")
    private Long id;

    @NotBlank(groups = {OnCreate.class, OnUpdate.class})
    private String name;

    @NotBlank(groups = OnCreate.class)     // required on create, optional on update
    private String password;
}

@RestController
public class CustomerController {

    @PostMapping
    public CustomerDto create(@Validated(OnCreate.class) @RequestBody CustomerDto dto) { }

    @PutMapping("/{id}")
    public CustomerDto update(@Validated(OnUpdate.class) @RequestBody CustomerDto dto) { }
}
```
This lets **one DTO** serve both operations with correctly different rules — the main practical reason `@Validated` exists.

## 22.9 Custom validator (real project need)

Scenario: order dates must be valid business dates and not on a company holiday.

```java
// 1. The annotation
@Documented
@Constraint(validatedBy = ValidBusinessDateValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidBusinessDate {
    String message() default "Date must be a working day";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. The validator — note it CAN be dependency-injected
@Component
@RequiredArgsConstructor
public class ValidBusinessDateValidator
        implements ConstraintValidator<ValidBusinessDate, LocalDate> {

    private final HolidayService holidayService;      // DI works in validators

    @Override
    public boolean isValid(LocalDate value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;       // null-handling is @NotNull's job, not ours.
        }                      // Returning true here keeps constraints composable.
        boolean weekend = value.getDayOfWeek() == DayOfWeek.SATURDAY
                       || value.getDayOfWeek() == DayOfWeek.SUNDAY;
        return !weekend && !holidayService.isHoliday(value);
    }
}

// 3. Use it
public class OrderRequestDto {
    @NotNull
    @ValidBusinessDate
    private LocalDate deliveryDate;
}
```

**Two teaching points:**
1. A `ConstraintValidator` is a Spring bean, so it can inject services and hit the database.
2. **Always return `true` for `null`** and let `@NotNull` handle nullability separately. Otherwise the two constraints fight and you get confusing double errors.

### Class-level validator (cross-field validation)

When a rule spans two fields — "endDate must be after startDate" — the constraint goes on the **class**:

```java
@Constraint(validatedBy = DateRangeValidator.class)
@Target(ElementType.TYPE)                    // ← on the TYPE, not a field
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidDateRange {
    String message() default "End date must be after start date";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class DateRangeValidator implements ConstraintValidator<ValidDateRange, ReportRequestDto> {
    @Override
    public boolean isValid(ReportRequestDto dto, ConstraintValidatorContext ctx) {
        if (dto.getStartDate() == null || dto.getEndDate() == null) return true;
        boolean valid = dto.getEndDate().isAfter(dto.getStartDate());
        if (!valid) {
            ctx.disableDefaultConstraintViolation();
            ctx.buildConstraintViolationWithTemplate("End date must be after start date")
               .addPropertyNode("endDate")        // attach the error to a specific field
               .addConstraintViolation();
        }
        return valid;
    }
}

@ValidDateRange                              // applied to the class
public class ReportRequestDto {
    private LocalDate startDate;
    private LocalDate endDate;
}
```

## 22.10 Handling validation errors properly

By default Spring returns a verbose, unhelpful error blob. Convert it into a clean, field-keyed response:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Triggered by @Valid on @RequestBody
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex,
                                          HttpServletRequest request) {

        Map<String, String> fieldErrors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(error -> fieldErrors.put(error.getField(), error.getDefaultMessage()));

        // Class-level (cross-field) errors have no field name
        ex.getBindingResult().getGlobalErrors()
          .forEach(error -> fieldErrors.put(error.getObjectName(), error.getDefaultMessage()));

        return ErrorResponse.builder()
                .timestamp(Instant.now())
                .status(400)
                .error("Validation Failed")
                .message("Request contains invalid fields")
                .path(request.getRequestURI())
                .fieldErrors(fieldErrors)
                .build();
    }

    // Triggered by @Validated on params — a DIFFERENT exception. Don't forget it.
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleConstraintViolation(ConstraintViolationException ex,
                                                   HttpServletRequest request) {
        Map<String, String> errors = ex.getConstraintViolations().stream()
                .collect(Collectors.toMap(
                        v -> v.getPropertyPath().toString(),
                        ConstraintViolation::getMessage,
                        (a, b) -> a));
        return ErrorResponse.builder()
                .timestamp(Instant.now()).status(400).error("Validation Failed")
                .message("Request parameters are invalid")
                .path(request.getRequestURI()).fieldErrors(errors).build();
    }
}
```

**The response the client actually wants:**

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "status": 400,
  "error": "Validation Failed",
  "message": "Request contains invalid fields",
  "path": "/api/v1/customers",
  "fieldErrors": {
    "name": "Name is required",
    "email": "Email format is invalid",
    "creditLimit": "Amount must be greater than zero"
  }
}
```
A frontend can now highlight exactly the wrong inputs. **This is the difference between a usable API and a frustrating one** — a good point to make to a TL.

## 22.11 The layers of validation (Additional clarification)

Validation is not only at the API boundary. Know all four layers:

| Layer | Mechanism | Catches |
|---|---|---|
| **1. API / DTO** | `@Valid` + constraints | Malformed input, missing fields |
| **2. Business rules** | Explicit code in the service | "Order total exceeds credit limit", "Cannot cancel a shipped order" |
| **3. Entity constraints** | `@Column(nullable=false, length=50)` | Last-line schema integrity |
| **4. Database constraints** | `NOT NULL`, `UNIQUE`, `CHECK`, FK | Absolute guarantee, including other writers |

**Point to make to a senior:** DTO validation handles *format*; the service handles *business rules*; the database is the final authority. A unique-email check in the service is a race condition under concurrency — two simultaneous requests can both pass the check. The **database unique constraint** is what actually guarantees it, and you catch `DataIntegrityViolationException` to return a clean 409.

## 22.12 Common Mistakes

| Mistake | Consequence |
|---|---|
| Missing `spring-boot-starter-validation` | `@Valid` silently does nothing |
| `@NotNull` on a String instead of `@NotBlank` | Empty strings accepted |
| Forgetting `@Valid` on a nested object/list | Nested constraints never run |
| Forgetting `@Validated` on the class for param validation | `@Min` on `@PathVariable` ignored |
| Only handling `MethodArgumentNotValidException` | `ConstraintViolationException` returns an ugly 500 |
| `isValid` returning `false` for null | Fights with `@NotNull`; confusing duplicate messages |
| Business rules as custom constraints | Better in the service, where you have full context and transactions |
| Relying on service-level uniqueness checks | Race condition — use a DB unique constraint |
| Returning 500 for validation failures | Pollutes error dashboards with client mistakes |

## 22.13 TL Questions

**Q: How do we validate incoming requests?**
A: Constraint annotations on the request DTOs, triggered by `@Valid` on the `@RequestBody`. Failures throw `MethodArgumentNotValidException`, which our `@RestControllerAdvice` converts into a 400 with a field-to-message map.

**Q: What's the difference between `@Valid` and `@Validated`?**
A: `@Valid` is the Jakarta standard — we use it on request bodies and on nested objects to cascade validation. `@Validated` is Spring's, added at class level to enable validation of individual method parameters, and it's the only one supporting validation groups. They throw different exceptions, so we handle both.

**Q: Why `@NotBlank` instead of `@NotNull` for strings?**
A: `@NotNull` accepts an empty string and a string of spaces. `@NotBlank` requires actual content. For any field the user must fill in, `@NotBlank` is correct.

**Q: What happens internally when validation fails?**
A: The argument resolver binds the JSON to the DTO, then invokes Hibernate Validator. Violations are collected into a `BindingResult`; if it has errors, Spring throws `MethodArgumentNotValidException` **before** our controller method is called, so invalid data never reaches the service.

**Q: Where do business rules go?**
A: In the service, not in constraint annotations. Constraints are for format and structure — is this a valid email, is the amount positive. Rules like "cannot cancel a shipped order" need entity state and the transaction, so they belong in the service and throw domain exceptions.

**Q: How do we guarantee a unique email?**
A: A unique constraint on the database column. A service-level check is a race condition — two concurrent requests can both see "not taken" and both insert. We catch `DataIntegrityViolationException` and return 409 Conflict.

**Q: What if we remove `@Valid` from the controller?**
A: The constraints are never evaluated. Invalid data flows into the service and probably into the database, and we'd only find out via a database constraint error surfacing as a 500.

## 22.14 Interview Questions

**Beginner — Which dependency enables validation?**
`spring-boot-starter-validation` (Hibernate Validator). Not included in the web starter since Boot 2.3.

**Beginner — `@NotNull` vs `@NotEmpty` vs `@NotBlank`?**
`@NotNull`: not null only. `@NotEmpty`: not null and size > 0. `@NotBlank`: not null and has non-whitespace characters. For strings, use `@NotBlank`.

**Intermediate — How do you validate a nested object?**
Put `@Valid` on the nested field or collection. Cascade is not automatic.

**Intermediate — Which exception does each mechanism throw?**
`@Valid` on `@RequestBody` → `MethodArgumentNotValidException`. `@Validated` on class + constrained params → `ConstraintViolationException`. Binding failures on non-body params → `BindException` / `MethodArgumentTypeMismatchException`.

**Intermediate — How do you write a custom constraint?**
Create an annotation meta-annotated with `@Constraint(validatedBy = X.class)` declaring `message()`, `groups()` and `payload()`, then implement `ConstraintValidator<A, T>`. The validator is a Spring bean, so it can inject dependencies.

**Advanced — How does method-level validation work internally?**
`@Validated` on a class causes `MethodValidationPostProcessor` to create an AOP proxy. The interceptor calls `ExecutableValidator.validateParameters()` before the method and `validateReturnValue()` after. Because it's proxy-based, **self-invocation bypasses it** — the same limitation as `@Transactional`.

**Advanced — What are validation groups and why do they exist?**
Interfaces used as markers to activate subsets of constraints, so one DTO can enforce different rules for create versus update — for example `id` must be null on create and non-null on update. Activated with `@Validated(OnCreate.class)`.

**Advanced — Where should validation live in a layered architecture?**
Format validation at the API boundary on DTOs; business-rule validation in the service where entity state and the transaction are available; structural constraints on the entity; and absolute guarantees as database constraints. Each layer catches what the one before it cannot.

## 22.15 Quick Revision

1. Add `spring-boot-starter-validation` — not bundled with the web starter.
2. `@Valid` on `@RequestBody` triggers DTO validation.
3. `@NotBlank` for strings; `@NotNull` lets `""` through.
4. `@Valid` on nested fields/collections to cascade — not automatic.
5. `@Validated` on the class enables parameter validation and groups.
6. Two exceptions: `MethodArgumentNotValidException` and `ConstraintViolationException` — **handle both**.
7. Custom constraint = annotation + `ConstraintValidator`; validators support DI.
8. Return `true` for `null` in custom validators; let `@NotNull` own nullability.
9. Cross-field rules = class-level constraint.
10. Business rules go in the service; uniqueness is guaranteed by the **database**.
11. Always return 400 with a field-to-message map.

## 22.16 TL Explanation (speak this)

> "We validate at the API boundary with Jakarta Bean Validation annotations on the request DTOs and `@Valid` on the request body, so bad input is rejected before it ever reaches the service. Validation failures are caught in our `@RestControllerAdvice` and returned as a 400 with a field-to-message map, so the frontend can highlight exactly which inputs are wrong. Two details we're careful about: `@NotBlank` rather than `@NotNull` for strings, and handling both `MethodArgumentNotValidException` and `ConstraintViolationException`, because `@Valid` and `@Validated` throw different ones. Business rules stay in the service, and genuine uniqueness is enforced by a database constraint rather than a service check, which would be a race condition."

---

# 23. Exception Handling

## 23.1 What is it?

**Simple words:**
Exception handling means deciding **what the client sees when something goes wrong**. Instead of a stack trace or a bare 500, the client should get a clear, consistent error message with the right status code.

Spring lets you write this **once, in one class**, instead of try/catch in every controller.

**Technical wording:**
Spring MVC resolves exceptions through the `HandlerExceptionResolver` chain. `@ExceptionHandler` methods — declared locally in a controller or globally in a `@RestControllerAdvice` — are discovered by `ExceptionHandlerExceptionResolver` and invoked to produce the error response.

## 23.2 Why centralise it?

**Without central handling:**

```java
@GetMapping("/{id}")
public ResponseEntity<?> getOrder(@PathVariable Long id) {
    try {
        return ResponseEntity.ok(orderService.getOrderById(id));
    } catch (OrderNotFoundException e) {
        return ResponseEntity.status(404).body(Map.of("error", e.getMessage()));
    } catch (Exception e) {
        return ResponseEntity.status(500).body(Map.of("error", "Something went wrong"));
    }
}
```
Repeated in **every** method. Inconsistent formats. Business logic buried in error handling.

**With central handling:**

```java
@GetMapping("/{id}")
public ResponseEntity<OrderDto> getOrder(@PathVariable Long id) {
    return ResponseEntity.ok(orderService.getOrderById(id));   // clean
}
```
The exception propagates, and the advice turns it into a proper response. **Consistent format across the whole API, in one place.**

## 23.3 The complete exception-handling flow

```
    Controller / Service / Repository throws an exception
                       │
                       ▼
        Propagates up to DispatcherServlet
                       │
                       ▼
        processHandlerException()
                       │
                       ▼
        HandlerExceptionResolver chain (in order):
                       │
        ┌──────────────┴──────────────────────────────────┐
        │                                                  │
        ▼                                                  │
  1. ExceptionHandlerExceptionResolver                     │
       - looks for @ExceptionHandler in the SAME controller│
       - then in @ControllerAdvice / @RestControllerAdvice │
       - picks the MOST SPECIFIC matching type             │
       - found? → invoke it → build the response → DONE ───┤
       - not found? ↓                                      │
        │                                                  │
        ▼                                                  │
  2. ResponseStatusExceptionResolver                       │
       - handles @ResponseStatus-annotated exceptions      │
         and ResponseStatusException             ──────────┤
        │                                                  │
        ▼                                                  │
  3. DefaultHandlerExceptionResolver                       │
       - maps standard Spring MVC exceptions:              │
         HttpRequestMethodNotSupportedException → 405      │
         HttpMediaTypeNotSupportedException     → 415      │
         MethodArgumentNotValidException        → 400      │
         NoHandlerFoundException                → 404 ─────┤
        │                                                  │
        ▼                                                  │
  4. Nothing handled it                                    │
       → forwarded to /error (BasicErrorController)        │
       → default 500 JSON response                         │
        └─────────────────────────────────────────────────┘
                       │
                       ▼
              HTTP error response to client
```

## 23.4 The building blocks

### Custom exceptions

```java
// package com.company.orderservice.exception

// Base class — one root for all our business exceptions
public abstract class BusinessException extends RuntimeException {
    private final String errorCode;
    protected BusinessException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
    public String getErrorCode() { return errorCode; }
}

public class ResourceNotFoundException extends BusinessException {
    public ResourceNotFoundException(String resource, Object id) {
        super("RESOURCE_NOT_FOUND", String.format("%s not found with id: %s", resource, id));
    }
}

public class InsufficientBalanceException extends BusinessException {
    public InsufficientBalanceException(BigDecimal required, BigDecimal available) {
        super("INSUFFICIENT_BALANCE",
              String.format("Required %s but available %s", required, available));
    }
}

public class DuplicateResourceException extends BusinessException {
    public DuplicateResourceException(String field, String value) {
        super("DUPLICATE_RESOURCE", String.format("%s '%s' already exists", field, value));
    }
}
```

**Why extend `RuntimeException` and not `Exception`?**
This is important and frequently asked. A **checked** exception forces every caller to `throws` or catch it, polluting every signature up the stack. More critically, **Spring's declarative transaction management rolls back on `RuntimeException` by default but NOT on checked exceptions.** A checked business exception would let a partially completed transaction **commit**. Always use unchecked exceptions for business errors.

### The error response DTO

```java
@Getter @Builder
@JsonInclude(JsonInclude.Include.NON_NULL)     // omit null fields
public class ErrorResponse {
    private Instant timestamp;
    private int status;
    private String error;              // "Not Found"
    private String errorCode;          // "RESOURCE_NOT_FOUND" — machine-readable
    private String message;            // human-readable
    private String path;               // which endpoint
    private String correlationId;      // ties to our logs
    private Map<String, String> fieldErrors;   // only for validation failures
}
```

**Why include both `errorCode` and `message`:** the client's code branches on the stable `errorCode`; the `message` is for humans and can be reworded or localised without breaking clients.

**Why include `correlationId`:** the client reports "I got this error with correlation ID abc-123", and you find that exact request across every microservice log. This is the single most useful field in a production error response.

### The global handler

```java
package com.company.orderservice.exception;

@RestControllerAdvice           // = @ControllerAdvice + @ResponseBody
@Slf4j
@RequiredArgsConstructor
public class GlobalExceptionHandler {

    // ---------- 404 ----------
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex,
                                                        HttpServletRequest request) {
        log.warn("Resource not found: {}", ex.getMessage());       // WARN, not ERROR
        return build(HttpStatus.NOT_FOUND, ex.getErrorCode(), ex.getMessage(), request, null);
    }

    // ---------- 422 business rule violation ----------
    @ExceptionHandler(InsufficientBalanceException.class)
    public ResponseEntity<ErrorResponse> handleInsufficientBalance(
            InsufficientBalanceException ex, HttpServletRequest request) {
        log.warn("Business rule violation: {}", ex.getMessage());
        return build(HttpStatus.UNPROCESSABLE_ENTITY, ex.getErrorCode(),
                     ex.getMessage(), request, null);
    }

    // ---------- 409 duplicate ----------
    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ErrorResponse> handleDuplicate(DuplicateResourceException ex,
                                                         HttpServletRequest request) {
        return build(HttpStatus.CONFLICT, ex.getErrorCode(), ex.getMessage(), request, null);
    }

    // ---------- 400 validation (@Valid on @RequestBody) ----------
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex,
                                                          HttpServletRequest request) {
        Map<String, String> fieldErrors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(e -> fieldErrors.put(e.getField(), e.getDefaultMessage()));
        ex.getBindingResult().getGlobalErrors()
          .forEach(e -> fieldErrors.put(e.getObjectName(), e.getDefaultMessage()));
        return build(HttpStatus.BAD_REQUEST, "VALIDATION_FAILED",
                     "Request contains invalid fields", request, fieldErrors);
    }

    // ---------- 400 validation (@Validated on params) ----------
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ErrorResponse> handleConstraintViolation(
            ConstraintViolationException ex, HttpServletRequest request) {
        Map<String, String> errors = ex.getConstraintViolations().stream()
                .collect(Collectors.toMap(v -> v.getPropertyPath().toString(),
                                          ConstraintViolation::getMessage, (a, b) -> a));
        return build(HttpStatus.BAD_REQUEST, "VALIDATION_FAILED",
                     "Request parameters are invalid", request, errors);
    }

    // ---------- 400 malformed JSON ----------
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<ErrorResponse> handleUnreadable(HttpMessageNotReadableException ex,
                                                          HttpServletRequest request) {
        // Deliberately do NOT echo ex.getMessage() — it can leak class names and structure
        return build(HttpStatus.BAD_REQUEST, "MALFORMED_REQUEST",
                     "Request body is malformed or unreadable", request, null);
    }

    // ---------- 400 wrong parameter type ----------
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ErrorResponse> handleTypeMismatch(
            MethodArgumentTypeMismatchException ex, HttpServletRequest request) {
        String message = String.format("Parameter '%s' has invalid value '%s'",
                                       ex.getName(), ex.getValue());
        return build(HttpStatus.BAD_REQUEST, "INVALID_PARAMETER", message, request, null);
    }

    // ---------- 409 database constraint ----------
    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<ErrorResponse> handleDataIntegrity(DataIntegrityViolationException ex,
                                                             HttpServletRequest request) {
        log.error("Data integrity violation", ex);
        // Never expose the raw SQL/constraint name to the client
        return build(HttpStatus.CONFLICT, "DATA_CONFLICT",
                     "Operation conflicts with existing data", request, null);
    }

    // ---------- 403 ----------
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex,
                                                            HttpServletRequest request) {
        return build(HttpStatus.FORBIDDEN, "ACCESS_DENIED",
                     "You do not have permission to perform this action", request, null);
    }

    // ---------- 500 catch-all — MUST BE LAST ----------
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAll(Exception ex, HttpServletRequest request) {
        // ERROR level + full stack trace → this is a bug we must investigate
        log.error("Unhandled exception on {} {}", request.getMethod(),
                  request.getRequestURI(), ex);
        // Generic message to the client — NEVER leak internals
        return build(HttpStatus.INTERNAL_SERVER_ERROR, "INTERNAL_ERROR",
                     "An unexpected error occurred. Please contact support.", request, null);
    }

    private ResponseEntity<ErrorResponse> build(HttpStatus status, String code, String message,
                                                HttpServletRequest request,
                                                Map<String, String> fieldErrors) {
        ErrorResponse body = ErrorResponse.builder()
                .timestamp(Instant.now())
                .status(status.value())
                .error(status.getReasonPhrase())
                .errorCode(code)
                .message(message)
                .path(request.getRequestURI())
                .correlationId(MDC.get("correlationId"))   // ties response to logs
                .fieldErrors(fieldErrors)
                .build();
        return ResponseEntity.status(status).body(body);
    }
}
```

**Three practices in that code worth explaining to a senior:**

1. **Log levels differ by cause.** Client mistakes (404, validation) are `WARN` — they're expected. Unhandled exceptions are `ERROR` with a stack trace — they're bugs. If you log 404s at ERROR, your alerting becomes noise and real incidents get missed.
2. **Never leak internals.** No stack traces, SQL, constraint names or class names in responses. That's reconnaissance material for an attacker.
3. **The catch-all is a safety net, not a design.** Every *expected* failure should have its own handler with a meaningful status code.

## 23.5 Exception handler resolution order

```java
@ExceptionHandler(ResourceNotFoundException.class)   // most specific
@ExceptionHandler(BusinessException.class)           // parent
@ExceptionHandler(RuntimeException.class)            // grandparent
@ExceptionHandler(Exception.class)                   // catch-all
```
Spring picks the **most specific match by class hierarchy distance**, not declaration order. So a `ResourceNotFoundException` hits the first handler even if `Exception.class` is declared earlier in the file.

**Local vs global precedence:** an `@ExceptionHandler` inside the controller **wins over** one in `@ControllerAdvice`. That lets one controller override the global behaviour for a specific case.

**Ordering multiple advices:** if you have several `@RestControllerAdvice` classes, control precedence with `@Order`. A common pattern is a general advice plus a higher-priority one for a specific package:

```java
@RestControllerAdvice(basePackages = "com.company.orderservice.admin")
@Order(Ordered.HIGHEST_PRECEDENCE)
public class AdminExceptionHandler { }
```

## 23.6 `@ControllerAdvice` vs `@RestControllerAdvice`

| | `@ControllerAdvice` | `@RestControllerAdvice` |
|---|---|---|
| Composition | `@Component` | **`@ControllerAdvice` + `@ResponseBody`** |
| Return value | View name unless `@ResponseBody` | Serialized to JSON |
| Use for | MVC apps with HTML views | **REST APIs** |

`@ControllerAdvice` can also scope itself:
```java
@RestControllerAdvice(basePackages = "com.company.orderservice.controller")
@RestControllerAdvice(assignableTypes = {OrderController.class})
@RestControllerAdvice(annotations = RestController.class)
```

## 23.7 Other mechanisms

```java
// @ResponseStatus directly on an exception class — quick, but less flexible
@ResponseStatus(HttpStatus.NOT_FOUND)
public class OrderNotFoundException extends RuntimeException { }

// ResponseStatusException — throw a status inline without a custom class
throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Order not found");

// ProblemDetail (RFC 7807) — the standardised error format, Spring 6 / Boot 3
@ExceptionHandler(ResourceNotFoundException.class)
public ProblemDetail handle(ResourceNotFoundException ex) {
    ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    pd.setTitle("Resource Not Found");
    pd.setType(URI.create("https://api.company.com/errors/not-found"));
    pd.setProperty("errorCode", ex.getErrorCode());
    return pd;
}
```
```properties
# Enable RFC 7807 responses for Spring's built-in exceptions
spring.mvc.problemdetails.enabled=true
```

**Additional clarification:** `ProblemDetail` is the modern, standards-based approach in Boot 3. If you're starting a new service, it's worth proposing — clients get a predictable `type`/`title`/`status`/`detail`/`instance` structure defined by RFC 7807, rather than a bespoke shape per team.

## 23.8 What `@RestControllerAdvice` CANNOT catch

This is a genuinely important limitation, and a strong interview answer.

```
Request
   │
   ▼
FILTERS  ◄── exceptions HERE are NOT caught by @RestControllerAdvice
   │            (filters run BEFORE DispatcherServlet exists in the call stack)
   ▼
DispatcherServlet
   │
   ▼
Interceptors, Controller, Service, Repository  ◄── these ARE caught
```

**Consequence:** a JWT authentication filter that throws produces the container's default HTML error page, not your clean JSON. **This surprises a lot of developers.**

**Solutions:**
1. Catch inside the filter and write the JSON response yourself with an `ObjectMapper`.
2. Use Spring Security's `AuthenticationEntryPoint` and `AccessDeniedHandler` (the correct approach for security filters — see Section 42).
3. Add a `@Component` `ErrorController` to customise the `/error` fallback.

```java
// Writing a clean JSON error from inside a filter
private void writeError(HttpServletResponse response, HttpStatus status, String message)
        throws IOException {
    response.setStatus(status.value());
    response.setContentType(MediaType.APPLICATION_JSON_VALUE);
    ErrorResponse body = ErrorResponse.builder()
            .timestamp(Instant.now()).status(status.value())
            .error(status.getReasonPhrase()).message(message).build();
    objectMapper.writeValue(response.getOutputStream(), body);
}
```

## 23.9 Common Mistakes

| Mistake | Consequence |
|---|---|
| try/catch in every controller | Duplication, inconsistency |
| **Swallowing exceptions** (`catch (Exception e) { }`) | Silent failure — the worst bug class |
| Exposing stack traces | Security exposure |
| Everything returns 500 | Clients can't react appropriately |
| Checked exceptions for business errors | **Transaction does not roll back** |
| Logging 404s at ERROR | Alert fatigue; real incidents get missed |
| Catching an exception and rethrowing without the cause | Stack trace lost — undebuggable |
| Expecting advice to catch filter exceptions | Ugly HTML error instead of JSON |
| No catch-all handler | Unexpected exceptions leak Spring's default error body |
| Returning the raw DB message | Leaks schema and constraint names |

## 23.10 TL Questions

**Q: How do we handle exceptions?**
A: Centrally, with a `@RestControllerAdvice`. Controllers and services throw domain exceptions; the advice maps each type to a status code and a consistent error body, so every endpoint returns errors in the same shape.

**Q: Why not try/catch in controllers?**
A: It duplicates the same code everywhere, makes formats drift between endpoints, and mixes error handling into business flow. One advice class gives us consistency and a single place to change.

**Q: Why do our business exceptions extend `RuntimeException`?**
A: Two reasons. Checked exceptions force `throws` declarations through every layer, and more importantly Spring rolls back on `RuntimeException` by default but **not** on checked exceptions — a checked business exception would let a partially completed transaction commit.

**Q: What happens internally when an exception is thrown?**
A: It propagates to DispatcherServlet, which runs the `HandlerExceptionResolver` chain. `ExceptionHandlerExceptionResolver` looks for a matching `@ExceptionHandler` first in the controller, then in the advice classes, choosing the most specific type match. If nothing matches, the request forwards to `/error`.

**Q: What if an exception is thrown in a filter?**
A: The advice can't catch it, because filters run before DispatcherServlet. We handle it inside the filter by writing the JSON response directly, and for security filters we use Spring Security's `AuthenticationEntryPoint` and `AccessDeniedHandler`.

**Q: What do we return for an unexpected exception?**
A: A generic 500 with a correlation ID and no internal detail. The full stack trace goes to the logs at ERROR level. Exposing stack traces would leak our internal structure to an attacker.

**Q: How do we make production errors debuggable?**
A: Every request gets a correlation ID from a filter, stored in the MDC so it appears on every log line and is returned in the error response. When a user reports an error we search that one ID and see the whole request across all services.

**Q: What alternative can we use?**
A: `ProblemDetail` (RFC 7807), standard in Boot 3, which gives a standardised error format instead of a custom shape. For new services that's worth adopting.

## 23.11 Interview Questions

**Beginner — What is `@ControllerAdvice`?**
A specialised component that applies `@ExceptionHandler`, `@InitBinder` and `@ModelAttribute` methods globally across controllers.

**Beginner — `@ControllerAdvice` vs `@RestControllerAdvice`?**
`@RestControllerAdvice` adds `@ResponseBody`, so handler return values are serialized to JSON rather than treated as view names.

**Intermediate — How does Spring choose among multiple handlers?**
By the most specific match in the exception class hierarchy — measured by inheritance distance, not declaration order. Handlers in the controller itself take precedence over those in advice classes.

**Intermediate — `@ResponseStatus` vs `@ExceptionHandler`?**
`@ResponseStatus` on an exception class gives a fixed status with no custom body. `@ExceptionHandler` builds a full custom response, can inspect the exception, add headers and log. Use `@ExceptionHandler` for a real API.

**Intermediate — Why don't checked exceptions roll back transactions?**
Spring's default rollback rule is `RuntimeException` and `Error` only, following the EJB convention that checked exceptions are recoverable business conditions. Override with `@Transactional(rollbackFor = Exception.class)`.

**Advanced — What is the `HandlerExceptionResolver` chain?**
Ordered resolvers: `ExceptionHandlerExceptionResolver` (your `@ExceptionHandler`s), `ResponseStatusExceptionResolver` (`@ResponseStatus` and `ResponseStatusException`), and `DefaultHandlerExceptionResolver` (standard MVC exceptions → 405, 415, 400). The first to return a non-null `ModelAndView` wins.

**Advanced — Why can't advice catch filter exceptions?**
Filters execute in the servlet container's chain before `DispatcherServlet.doDispatch()` is entered, so there is no dispatcher frame to catch and resolve the exception. Handle it in the filter or via Spring Security's entry point.

**Advanced — What is `ProblemDetail`?**
Spring 6's implementation of RFC 7807 "Problem Details for HTTP APIs" — a standard error body with `type`, `title`, `status`, `detail` and `instance`, plus custom properties. Enabled for built-in exceptions via `spring.mvc.problemdetails.enabled=true`.

## 23.12 Quick Revision

1. Centralise in `@RestControllerAdvice` — never try/catch per controller.
2. Business exceptions extend **`RuntimeException`** so transactions roll back.
3. Consistent `ErrorResponse`: timestamp, status, errorCode, message, path, correlationId.
4. Map each exception to a **meaningful** status: 404, 409, 422, 400, 403.
5. Always include a **catch-all** `Exception` handler as a safety net.
6. Resolution = most specific type; controller-local beats global advice.
7. **Never expose stack traces, SQL or constraint names.**
8. Log client errors at WARN, unexpected errors at ERROR with the stack trace.
9. Advice **cannot** catch filter exceptions — handle those in the filter.
10. `ProblemDetail` (RFC 7807) is the modern standard format in Boot 3.
11. Include a correlation ID in every error — it makes production debugging possible.

## 23.13 TL Explanation (speak this)

> "All our error handling is centralised in a `@RestControllerAdvice`. Controllers and services just throw domain exceptions — `ResourceNotFoundException`, `InsufficientBalanceException` — and the advice maps each to the right status code and a consistent error body with a timestamp, error code, message and correlation ID. There's no try/catch in the controllers at all. Our business exceptions extend `RuntimeException` deliberately, because Spring only rolls back automatically on unchecked exceptions. We never return stack traces to clients, we log client mistakes at WARN and genuine bugs at ERROR, and the correlation ID in the response lets us find that exact request in the logs across every service. The one gap is that filters run before the DispatcherServlet, so exceptions there are handled in the filter itself or through Spring Security's entry point."

---

# 24. Filters

## 24.1 What is it?

**Simple words:**
A filter is code that runs **before and after every HTTP request**, at the servlet container level — *before* Spring MVC even gets involved. It's the outermost layer of your application.

Think of it as security at the building entrance: everyone passes through it, before they reach any particular office.

**Technical wording:**
A `Filter` is a Jakarta Servlet API component (`jakarta.servlet.Filter`) that intercepts requests and responses at the container level. Filters form a chain executed before the request reaches `DispatcherServlet` and after the response leaves it.

## 24.2 Where filters sit

```
 CLIENT
   │
   ▼
 TOMCAT
   │
   ▼
┌──────────────────────────────────────────────┐
│ FILTER 1  (order 1)                          │
│   code before chain.doFilter()               │
│  ┌────────────────────────────────────────┐  │
│  │ FILTER 2  (order 2)                    │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │ FILTER 3 (Spring Security)       │  │  │
│  │  │  ┌────────────────────────────┐  │  │  │
│  │  │  │  DISPATCHER SERVLET        │  │  │  │
│  │  │  │    Interceptors            │  │  │  │
│  │  │  │      Controller            │  │  │  │
│  │  │  │        Service             │  │  │  │
│  │  │  │          Repository        │  │  │  │
│  │  │  └────────────────────────────┘  │  │  │
│  │  │   code after chain.doFilter()    │  │  │
│  │  └──────────────────────────────────┘  │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
   │
   ▼
 RESPONSE
```

Filters nest like layers of an onion: outermost in, then unwind outermost out.

## 24.3 Why use filters?

- Runs for **every** request, including static resources and error dispatches
- Works even when **no controller matches** (404s still pass through)
- Has access to the raw `HttpServletRequest`/`HttpServletResponse`
- Can **modify or wrap** the request and response
- Can **stop** the request entirely (authentication rejection)

**Typical uses:** authentication/JWT validation, CORS, correlation IDs, request/response logging, character encoding, compression, rate limiting, XSS sanitisation.

## 24.4 Creating a filter

### Approach 1 — `OncePerRequestFilter` (RECOMMENDED)

```java
package com.company.orderservice.filter;

@Component
@Order(1)                                          // lower value = runs earlier
@Slf4j
public class CorrelationIdFilter extends OncePerRequestFilter {

    private static final String CORRELATION_ID_HEADER = "X-Correlation-Id";
    private static final String MDC_KEY = "correlationId";

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        // Reuse the caller's ID if present, so a trace spans microservices
        String correlationId = Optional.ofNullable(request.getHeader(CORRELATION_ID_HEADER))
                                       .filter(s -> !s.isBlank())
                                       .orElse(UUID.randomUUID().toString());
        try {
            MDC.put(MDC_KEY, correlationId);              // every log line now carries it
            response.setHeader(CORRELATION_ID_HEADER, correlationId);  // return it to the client

            filterChain.doFilter(request, response);      // ← CONTINUE THE CHAIN

        } finally {
            MDC.clear();     // MUST clear: Tomcat reuses threads, values would leak
        }
    }

    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) {
        return request.getRequestURI().startsWith("/actuator");   // skip health checks
    }
}
```

**Why `OncePerRequestFilter` rather than implementing `Filter` directly?**
A plain `Filter` can execute **multiple times per request** — on `FORWARD`, `INCLUDE` and `ERROR` dispatches. That means duplicate log entries, double-counted metrics, and authentication running twice. `OncePerRequestFilter` guarantees exactly one execution per request. **Always use it** unless you have a specific reason not to.

**The `finally` block is not optional.** Tomcat pools threads, so a `ThreadLocal`/MDC value left behind will appear on a completely unrelated later request — leaking one user's correlation ID (or worse, identity) into another's logs.

### Approach 2 — `FilterRegistrationBean` (precise control)

```java
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<RequestLoggingFilter> requestLoggingFilter() {
        FilterRegistrationBean<RequestLoggingFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new RequestLoggingFilter());
        registration.addUrlPatterns("/api/*");      // only these URLs — @Component can't do this
        registration.setOrder(2);
        registration.setName("requestLoggingFilter");
        return registration;
    }
}
```

**Use this when** you need URL-pattern scoping, or you must register a third-party filter you can't annotate.

> **Gotcha:** if a filter is both a `@Component` **and** registered via `FilterRegistrationBean`, it gets registered **twice**. Either drop `@Component`, or call `registration.setEnabled(false)` on the auto-registration.

## 24.5 Real example — request/response logging with a wrapper

A filter can only read the request body once — unless you wrap it.

```java
@Component
@Order(2)
@Slf4j
public class RequestLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {

        // Wrappers CACHE the body so it can be read here AND by the controller
        ContentCachingRequestWrapper  wrappedReq  = new ContentCachingRequestWrapper(request);
        ContentCachingResponseWrapper wrappedResp = new ContentCachingResponseWrapper(response);

        long start = System.currentTimeMillis();
        try {
            chain.doFilter(wrappedReq, wrappedResp);
        } finally {
            long duration = System.currentTimeMillis() - start;

            log.info("{} {} -> {} ({}ms)",
                     request.getMethod(), request.getRequestURI(),
                     wrappedResp.getStatus(), duration);

            if (log.isDebugEnabled()) {
                // Mask secrets before logging — never log raw passwords or tokens
                log.debug("Request body: {}", mask(new String(wrappedReq.getContentAsByteArray())));
            }

            // CRITICAL: the wrapper buffered the response — copy it out or the
            // client receives an EMPTY body.
            wrappedResp.copyBodyToResponse();
        }
    }

    private String mask(String body) {
        return body.replaceAll("(\"password\"\\s*:\\s*\")[^\"]*(\")", "$1****$2")
                   .replaceAll("(\"token\"\\s*:\\s*\")[^\"]*(\")", "$1****$2");
    }
}
```

**Two things that bite people here:**
1. Forgetting `copyBodyToResponse()` → the client gets an empty response body. Very confusing to debug.
2. Logging request bodies **without masking** → passwords and tokens land in your log aggregator, which is a compliance incident. Always mask.

## 24.6 Filter ordering

```java
@Order(Ordered.HIGHEST_PRECEDENCE)      // Integer.MIN_VALUE — runs first
@Order(1)
@Order(Ordered.LOWEST_PRECEDENCE)       // runs last
```

**Sensible ordering in a real service:**

```
1  CharacterEncodingFilter        (Boot registers this automatically)
2  CorrelationIdFilter            (so everything downstream can log the ID)
3  CorsFilter                     (must precede security — preflight has no token)
4  RequestLoggingFilter
5  Spring Security filter chain   (default order -100, configurable)
6  RateLimitingFilter
```

**Practical note:** Spring Security's chain runs at order `-100` by default (`SecurityProperties.DEFAULT_FILTER_ORDER`), so a filter with `@Order(1)` actually runs **after** security. If you need something before authentication — like a correlation ID — give it a negative order lower than `-100`, or register it with `addFilterBefore` in the security config.

## 24.7 Stopping the chain

```java
@Component
public class ApiKeyFilter extends OncePerRequestFilter {

    private final ObjectMapper objectMapper;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {

        String apiKey = request.getHeader("X-Api-Key");

        if (!isValid(apiKey)) {
            // Do NOT call chain.doFilter() → the request stops here.
            // Remember: @RestControllerAdvice cannot catch this, so we write JSON ourselves.
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            objectMapper.writeValue(response.getOutputStream(),
                    Map.of("status", 401, "error", "Unauthorized",
                           "message", "Invalid or missing API key"));
            return;                          // ← request does not proceed
        }
        chain.doFilter(request, response);
    }
}
```

## 24.8 Common Mistakes

| Mistake | Consequence |
|---|---|
| Forgetting `chain.doFilter()` | **The request silently hangs / returns empty.** The #1 filter bug. |
| Implementing `Filter` instead of `OncePerRequestFilter` | Runs multiple times per request |
| Not clearing `MDC`/`ThreadLocal` in `finally` | Values leak into other users' requests |
| Reading the body without a caching wrapper | Controller gets an empty body — stream already consumed |
| Forgetting `copyBodyToResponse()` | Client receives an empty response |
| Logging unmasked bodies | Passwords and tokens in logs — compliance breach |
| Registering both as `@Component` and via `FilterRegistrationBean` | Filter runs twice |
| Expecting `@RestControllerAdvice` to catch filter exceptions | Ugly HTML error page instead of JSON |
| Heavy work (DB calls) in a filter | Adds latency to **every** request including static resources |

## 24.9 TL Questions

**Q: What is a filter and where does it run?**
A: A servlet-level component that wraps every request, running before the DispatcherServlet and after the response is produced. It's the outermost layer, so it sees requests even when no controller matches.

**Q: Why do we use filters in our project?**
A: For correlation IDs so we can trace a request across services, request/response logging with timing, JWT authentication through the Spring Security chain, and CORS.

**Q: Why `OncePerRequestFilter`?**
A: A plain `Filter` can run multiple times per request on forward, include and error dispatches, which would double-log and re-authenticate. `OncePerRequestFilter` guarantees a single execution.

**Q: What happens if we forget `chain.doFilter()`?**
A: The request stops there — the controller is never reached and the client gets an empty response. It's the most common filter bug and it's silent, which is what makes it painful.

**Q: What happens when a filter throws an exception?**
A: Our `@RestControllerAdvice` can't catch it, because filters run outside the DispatcherServlet. We handle it inside the filter and write the JSON error ourselves, and for security filters we use `AuthenticationEntryPoint`.

**Q: How do we control filter order?**
A: `@Order` or `FilterRegistrationBean.setOrder()`. One detail to know is that Spring Security's chain sits at order -100, so anything that must run before authentication needs a lower order than that.

**Q: Why clear the MDC in a finally block?**
A: Tomcat reuses worker threads from a pool. If we leave a value in a ThreadLocal, the next unrelated request on that thread inherits it — so one user's correlation ID or identity shows up in another user's logs.

## 24.10 Interview Questions

**Beginner — What is a servlet filter?**
A Jakarta Servlet API component that intercepts requests and responses before and after the servlet handles them.

**Beginner — How do you register a filter in Spring Boot?**
Annotate it `@Component` (auto-registered for all URLs), or register a `FilterRegistrationBean` for URL patterns and explicit ordering.

**Intermediate — Filter vs Interceptor?**
Filters are servlet-level, run outside Spring MVC, work on raw request/response, apply to all requests including static resources, and have no knowledge of the handler. Interceptors are Spring MVC-level, run inside DispatcherServlet, know which handler method will run, and only apply to mapped requests. Full table in Section 53.

**Intermediate — How do you read the request body in a filter without breaking the controller?**
Wrap the request in `ContentCachingRequestWrapper` (or a custom wrapper), which caches the bytes so both the filter and the message converter can read them. The raw `ServletInputStream` can only be consumed once.

**Advanced — How does Spring Security fit into the filter chain?**
Boot registers a single `DelegatingFilterProxy` named `springSecurityFilterChain`, which delegates to a `FilterChainProxy`. That proxy holds one or more `SecurityFilterChain`s, each a list of security filters (`SecurityContextPersistenceFilter`, `UsernamePasswordAuthenticationFilter`, `ExceptionTranslationFilter`, `FilterSecurityInterceptor`, plus any custom JWT filter). It runs at order -100 within the container's chain.

**Advanced — What is `DelegatingFilterProxy` for?**
It bridges the servlet container and the Spring context. The container instantiates the proxy, which looks up the real filter **as a Spring bean** by name and delegates to it — so the actual filter can use dependency injection and the full Spring lifecycle, which a container-instantiated filter could not.

## 24.11 Quick Revision

1. Filters are **servlet-level**, running before and after DispatcherServlet.
2. Extend **`OncePerRequestFilter`** — guarantees one execution per request.
3. **Always call `chain.doFilter()`** unless deliberately blocking the request.
4. Register with `@Component` (all URLs) or `FilterRegistrationBean` (patterns + order).
5. `@Order` controls sequence; Spring Security is at **-100**.
6. Always clear `MDC`/`ThreadLocal` in `finally` — threads are pooled.
7. Use `ContentCachingRequestWrapper`/`ResponseWrapper` to read bodies; call `copyBodyToResponse()`.
8. **`@RestControllerAdvice` cannot catch filter exceptions** — handle them in the filter.
9. Mask secrets before logging bodies.
10. Keep filters lightweight — they run on every single request.

## 24.12 TL Explanation (speak this)

> "Filters are the outermost layer — servlet-level, so they run before the DispatcherServlet and see every request, even ones that don't match a controller. We use them for correlation IDs, request logging with timing, and JWT authentication through Spring Security's chain. We always extend `OncePerRequestFilter` so they don't re-run on forwards and error dispatches, and we always clear the MDC in a `finally` block because Tomcat reuses threads and values would leak into other users' requests. The one thing to be aware of is that our `@RestControllerAdvice` can't catch exceptions thrown in a filter, so filters write their own JSON error responses."

---

# 25. Interceptors

## 25.1 What is it?

**Simple words:**
An interceptor is like a filter, but it lives **inside Spring MVC**. It runs after the DispatcherServlet has already worked out *which controller method* will handle the request — so unlike a filter, it **knows the target method**.

**Technical wording:**
A `HandlerInterceptor` is a Spring MVC component invoked by `DispatcherServlet` around handler execution, with access to the resolved `HandlerMethod`, the `ModelAndView`, and any exception raised.

## 25.2 The three methods

```java
@Component
@Slf4j
public class PerformanceInterceptor implements HandlerInterceptor {

    private static final String START_TIME = "startTime";

    // 1. BEFORE the controller method
    //    return false → request STOPS here, controller never runs
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) {
        request.setAttribute(START_TIME, System.currentTimeMillis());

        // We KNOW the target method here — a filter cannot do this
        if (handler instanceof HandlerMethod handlerMethod) {
            log.debug("Invoking {}.{}",
                      handlerMethod.getBeanType().getSimpleName(),
                      handlerMethod.getMethod().getName());
        }
        return true;                          // continue
    }

    // 2. AFTER the controller, BEFORE the view is rendered
    //    NOT called if the controller threw an exception
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) {
        log.debug("Controller completed for {}", request.getRequestURI());
    }

    // 3. ALWAYS runs — after completion, even on exception. Cleanup goes here.
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                                Object handler, Exception ex) {
        long duration = System.currentTimeMillis() - (long) request.getAttribute(START_TIME);

        if (duration > 1000) {
            log.warn("SLOW REQUEST: {} {} took {}ms",
                     request.getMethod(), request.getRequestURI(), duration);
        }
        if (ex != null) {
            log.error("Request failed: {}", request.getRequestURI(), ex);
        }
    }
}
```

| Method | When | Skipped if... | Use for |
|---|---|---|---|
| `preHandle` | Before the controller | — | Auth checks, logging, timing start, rate limiting |
| `postHandle` | After controller, before view render | **Controller threw an exception** | Adding model attributes, response headers |
| `afterCompletion` | After everything | Never (if `preHandle` returned true) | Cleanup, timing end, resource release |

**The critical distinction:** `postHandle` is **skipped on exception**; `afterCompletion` **always runs**. So cleanup and metrics belong in `afterCompletion`, not `postHandle`. Getting this wrong means your timing metrics silently exclude every failed request — exactly the requests you most want to measure.

## 25.3 Registration

```java
@Configuration
@RequiredArgsConstructor
public class WebConfig implements WebMvcConfigurer {

    private final PerformanceInterceptor performanceInterceptor;
    private final AuthorizationInterceptor authorizationInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        registry.addInterceptor(performanceInterceptor)
                .addPathPatterns("/**")
                .order(1);

        registry.addInterceptor(authorizationInterceptor)
                .addPathPatterns("/api/**")                  // apply to these
                .excludePathPatterns("/api/v1/auth/**",      // except these
                                     "/api/v1/public/**")
                .order(2);
    }
}
```

**Note:** unlike filters, `@Component` alone does **not** register an interceptor. You must add it in `addInterceptors`. This catches people out.

**Path-pattern support is a real advantage** over filters, where you'd need a `FilterRegistrationBean` and coarser URL patterns.

## 25.4 Real example — method-aware authorization

This shows what interceptors can do that filters cannot:

```java
// Custom annotation on controller methods
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequiresRole {
    String[] value();
}

@RestController
public class AdminController {
    @DeleteMapping("/api/v1/orders/{id}")
    @RequiresRole({"ADMIN", "SUPER_ADMIN"})        // ← read by the interceptor
    public ResponseEntity<Void> delete(@PathVariable Long id) { ... }
}

@Component
@RequiredArgsConstructor
public class AuthorizationInterceptor implements HandlerInterceptor {

    private final CurrentUserService currentUserService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws IOException {

        if (!(handler instanceof HandlerMethod handlerMethod)) {
            return true;                      // static resource, not a controller method
        }

        // THIS is the interceptor advantage: we can read the method's annotations
        RequiresRole requiresRole = handlerMethod.getMethodAnnotation(RequiresRole.class);
        if (requiresRole == null) {
            return true;                      // endpoint is not role-restricted
        }

        Set<String> userRoles = currentUserService.getCurrentUserRoles();
        boolean allowed = Arrays.stream(requiresRole.value()).anyMatch(userRoles::contains);

        if (!allowed) {
            response.setStatus(HttpStatus.FORBIDDEN.value());
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            response.getWriter().write("{\"status\":403,\"message\":\"Insufficient role\"}");
            return false;                     // ← STOP: controller is never invoked
        }
        return true;
    }
}
```

A filter **could not** do this — at filter time, Spring hasn't yet resolved which method will handle the request, so the `@RequiresRole` annotation is invisible.

> **Additional clarification:** in a project already using Spring Security, prefer `@PreAuthorize("hasRole('ADMIN')")` over a custom interceptor like this. The example above is the clearest way to demonstrate the handler-awareness capability, but re-implementing authorization when Spring Security already provides it is duplicated effort and a security risk.

## 25.5 Filter vs Interceptor

| Aspect | Filter | Interceptor |
|---|---|---|
| Specification | Jakarta Servlet API | Spring MVC |
| Runs | **Outside** DispatcherServlet | **Inside** DispatcherServlet |
| Order relative to each other | **Always first** | After all filters |
| Knows the handler method | **No** | **Yes** (`HandlerMethod`) |
| Can read method annotations | No | **Yes** |
| Applies to static resources | Yes | Only if mapped |
| Applies when no handler matches | Yes | No |
| Can modify request/response objects | **Yes** (wrapping) | No |
| Access to `ModelAndView` | No | **Yes** (`postHandle`) |
| Path patterns | Coarse (URL patterns) | **Fine-grained include/exclude** |
| Spring DI | Yes (as a bean) | Yes |
| Exception caught by `@ControllerAdvice` | **No** | **Yes** |
| Registration | `@Component` / `FilterRegistrationBean` | `WebMvcConfigurer.addInterceptors` |
| Best for | Auth, CORS, encoding, correlation ID, compression | Handler-aware logic, per-endpoint auditing, timing |

**How to choose, in one line:**
> *"If it needs raw request/response manipulation or must apply to everything, use a filter. If it needs to know which controller method is running, use an interceptor."*

## 25.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Forgetting to register in `addInterceptors` | `@Component` alone does nothing — silently never runs |
| Returning `false` without writing a response | Client gets an empty 200 |
| Cleanup in `postHandle` | Skipped whenever an exception occurs — resource leak |
| Using a mutable field on the interceptor | Singleton → thread-safety bug. Use request attributes. |
| Assuming `handler` is always a `HandlerMethod` | `ClassCastException` on static resources — always type-check |
| Heavy DB work in `preHandle` | Latency on every request |
| Re-implementing Spring Security in an interceptor | Duplicated, weaker security |

## 25.7 TL Questions

**Q: What is an interceptor?**
A: A Spring MVC component that runs around the controller method. Unlike a filter, it runs inside the DispatcherServlet and knows which handler method is about to execute, so it can read that method's annotations.

**Q: Filter or interceptor — how do we decide?**
A: Filter when we need the raw request/response or need to cover everything including unmapped URLs — authentication, CORS, correlation IDs. Interceptor when the logic depends on which controller method is running — per-endpoint auditing or annotation-driven checks.

**Q: What's the difference between `postHandle` and `afterCompletion`?**
A: `postHandle` is skipped if the controller throws, `afterCompletion` always runs. So all cleanup and timing goes in `afterCompletion` — otherwise our metrics would silently exclude every failed request.

**Q: What happens if `preHandle` returns false?**
A: The chain stops — the controller is never invoked and `postHandle`/`afterCompletion` are skipped for interceptors that haven't run yet. We must write the response ourselves, or the client gets an empty 200.

**Q: How do we register one?**
A: Through `WebMvcConfigurer.addInterceptors`, where we can also specify include and exclude path patterns. Just annotating it `@Component` isn't enough — a mistake that's easy to miss because it fails silently.

**Q: Can `@RestControllerAdvice` catch exceptions from an interceptor?**
A: Yes for `preHandle` and `postHandle`, because they run inside the DispatcherServlet. That's a real advantage over filters.

## 25.8 Interview Questions

**Beginner — Name the three interceptor methods.**
`preHandle`, `postHandle`, `afterCompletion`.

**Beginner — How do you register an interceptor?**
Implement `WebMvcConfigurer` and add it in `addInterceptors`.

**Intermediate — Which runs first, a filter or an interceptor?**
A filter, always. Filters wrap the entire DispatcherServlet, and interceptors run inside it.

**Intermediate — Why type-check the `handler` parameter?**
It's declared `Object` and is only a `HandlerMethod` for annotated controller methods. For static resources it's a `ResourceHttpRequestHandler`, so an unchecked cast throws.

**Advanced — Interceptor vs AOP for cross-cutting logic?**
Interceptors are web-layer only and see HTTP request/response. AOP works on any Spring bean at any layer and sees method arguments and return values. Use an interceptor for HTTP concerns, AOP for service-layer concerns like transactions, caching and business auditing.

**Advanced — Are interceptors thread-safe?**
The bean is a shared singleton, so instance fields are not safe. Per-request state belongs in `HttpServletRequest` attributes (as in the timing example) or a properly cleared `ThreadLocal`.

## 25.9 Quick Revision

1. Interceptors are **Spring MVC**-level, running inside DispatcherServlet.
2. Three methods: `preHandle` (before), `postHandle` (after controller, skipped on exception), `afterCompletion` (always).
3. **Cleanup and metrics go in `afterCompletion`.**
4. `preHandle` returning `false` stops the request — write the response yourself.
5. Register via `WebMvcConfigurer.addInterceptors`, not `@Component` alone.
6. Supports fine-grained include/exclude path patterns.
7. **Key advantage: knows the `HandlerMethod`** and can read its annotations.
8. Filters always run before interceptors.
9. Interceptors are singletons — use request attributes for per-request state.
10. Exceptions from interceptors **can** be caught by `@ControllerAdvice`.

## 25.10 TL Explanation (speak this)

> "Interceptors sit inside Spring MVC, so unlike filters they run after the DispatcherServlet has resolved which controller method will handle the request — which means they can read that method's annotations. We use one for request timing and slow-request warnings. The important detail is that `postHandle` is skipped when the controller throws, so anything that must always run — cleanup, metrics — goes in `afterCompletion`. And they need registering in `WebMvcConfigurer.addInterceptors`; just marking them `@Component` silently does nothing."

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

# 27. JDBC and DataSource

## 27.1 What is JDBC?

**Simple words:**
JDBC (Java Database Connectivity) is the **standard Java API for talking to a database**. It defines interfaces like `Connection`, `Statement` and `ResultSet`. Each database vendor writes a **driver** that implements them.

That's why your Java code looks the same for MySQL and PostgreSQL — you only swap the driver.

**Technical wording:**
JDBC is a Java standard API providing vendor-neutral database access. Vendor drivers implement `java.sql` interfaces, and `DriverManager`/`DataSource` supplies `Connection` objects.

## 27.2 Raw JDBC — and why nobody writes it anymore

```java
public Order findById(Long id) {
    Connection conn = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        conn = dataSource.getConnection();
        ps = conn.prepareStatement("SELECT id, customer_id, amount FROM orders WHERE id = ?");
        ps.setLong(1, id);
        rs = ps.executeQuery();
        if (rs.next()) {
            Order order = new Order();
            order.setId(rs.getLong("id"));
            order.setCustomerId(rs.getString("customer_id"));
            order.setAmount(rs.getBigDecimal("amount"));
            return order;
        }
        return null;
    } catch (SQLException e) {
        throw new RuntimeException(e);            // checked exception forced on us
    } finally {
        // ~15 lines of nested null-checked closing — and leaking ANY of these
        // eventually exhausts the connection pool and takes the service down
        if (rs   != null) try { rs.close();   } catch (SQLException ignored) {}
        if (ps   != null) try { ps.close();   } catch (SQLException ignored) {}
        if (conn != null) try { conn.close(); } catch (SQLException ignored) {}
    }
}
```

**Roughly 25 lines for one simple query.** Problems: enormous boilerplate, manual resource management (leaks kill production), checked `SQLException` everywhere, vendor-specific error codes, and manual `ResultSet` → object mapping.

**Spring's answer, at three levels of abstraction:**

```
Raw JDBC          → you manage everything
   ↓
JdbcTemplate      → Spring manages connections/exceptions; you write SQL
   ↓
Spring Data JPA   → Spring generates SQL too; you write method names
```

## 27.3 DataSource

**Simple words:**
A `DataSource` is a **factory for database connections**. Instead of opening a new connection every time (slow — 50–100ms of TCP handshake and authentication), it hands you one from a **pool** of already-open connections.

**Technical wording:**
`javax.sql.DataSource` is the standard abstraction for obtaining `Connection` objects, typically backed by a connection pool that manages creation, reuse, validation and eviction.

### Why connection pooling is essential

```
WITHOUT POOLING — per request:
  open TCP socket    ~20ms
  authenticate       ~30ms
  run query           ~2ms   ← the only useful part
  close connection    ~5ms
  ───────────────────────────
  TOTAL              ~57ms   for 2ms of actual work

WITH POOLING — per request:
  borrow from pool   ~0.1ms
  run query           ~2ms
  return to pool     ~0.1ms
  ───────────────────────────
  TOTAL              ~2.2ms   → roughly 25x faster
```

Pooling also **caps** the number of connections, protecting the database. A database has a hard connection limit (MySQL's default `max_connections` is 151); without a pool, a traffic spike opens thousands of connections and the database refuses everything — including your monitoring.

### Configuration

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/orderdb
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
Boot's `DataSourceAutoConfiguration` sees these plus a driver on the classpath and creates a **HikariCP** `DataSource`. The driver class name is usually auto-detected from the URL.

## 27.4 TL Questions

**Q: What is JDBC?**
A: The standard Java API for database access. Vendors supply drivers implementing it, so our code is portable across databases.

**Q: Why don't we write raw JDBC?**
A: It's about 25 lines for a simple query, all of it resource management that leaks the connection pool if you get it wrong, plus checked exceptions and manual result mapping. Spring's `JdbcTemplate` or Spring Data JPA removes all of that.

**Q: What is a DataSource and why do we need one?**
A: A factory for connections, backed by a pool. Opening a connection costs 50+ milliseconds, so reusing pooled connections is roughly 25 times faster. It also caps concurrent connections so a traffic spike doesn't exhaust the database's connection limit.

**Q: What happens if we don't close connections?**
A: They're never returned to the pool. Eventually the pool is exhausted and every request blocks waiting for a connection until it times out — the service appears completely hung while the database itself is fine.

## 27.5 Interview Questions

**Beginner — What does a JDBC driver do?**
Implements the `java.sql` interfaces for a specific database, translating standard calls into that database's wire protocol.

**Intermediate — `Statement` vs `PreparedStatement`?**
`Statement` sends raw SQL — vulnerable to SQL injection and reparsed every time. `PreparedStatement` uses bind parameters, so it's **injection-safe**, can be cached and reused by the database, and handles type conversion. Always use `PreparedStatement`.

**Advanced — What is `DriverManager` vs `DataSource`?**
`DriverManager` is the old static factory that opens a fresh connection each call, with no pooling. `DataSource` is the modern abstraction, supports pooling, distributed transactions and JNDI lookup, and is what Spring uses.

## 27.6 Quick Revision

1. JDBC = standard Java DB API; drivers implement it per vendor.
2. Raw JDBC = heavy boilerplate and leak risk — don't write it.
3. `DataSource` = connection factory, normally pool-backed.
4. Pooling is ~25x faster and protects the DB's connection limit.
5. Always use `PreparedStatement` — SQL injection safety.
6. Boot auto-configures a HikariCP `DataSource` from `spring.datasource.*`.
7. Leaked connections exhaust the pool and hang the service.

## 27.7 TL Explanation (speak this)

> "JDBC is the standard API underneath everything we do with the database — Hibernate and `JdbcTemplate` both sit on top of it. We never write raw JDBC because it's a lot of manual resource management and a single missed `close()` leaks a connection until the pool is exhausted and the service hangs. The `DataSource` is our connection factory, backed by a HikariCP pool, which is roughly twenty-five times faster than opening a connection per request and also caps how many connections we can open so we don't overwhelm the database."

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

# 29. JdbcTemplate

## 29.1 What is it?

**Simple words:**
`JdbcTemplate` is Spring's helper that removes all the JDBC boilerplate. **You write the SQL; Spring handles the connection, statement, result set, closing and exception translation.**

**Technical wording:**
`JdbcTemplate` is Spring's central JDBC abstraction implementing the template method pattern. It manages resource acquisition and release, statement creation and execution, and translates `SQLException` into Spring's `DataAccessException` hierarchy.

## 29.2 The same query, with JdbcTemplate

```java
@Repository
@RequiredArgsConstructor
public class OrderJdbcRepository {

    private final JdbcTemplate jdbcTemplate;         // auto-configured by Boot

    public Order findById(Long id) {
        String sql = "SELECT id, customer_id, amount FROM orders WHERE id = ?";
        return jdbcTemplate.queryForObject(sql, orderRowMapper(), id);
    }

    private RowMapper<Order> orderRowMapper() {
        return (rs, rowNum) -> Order.builder()
                .id(rs.getLong("id"))
                .customerId(rs.getString("customer_id"))
                .amount(rs.getBigDecimal("amount"))
                .build();
    }
}
```
**25 lines became 5.** No connection handling, no `finally`, no checked exceptions.

## 29.3 Common operations

```java
// Query a list
List<Order> orders = jdbcTemplate.query(
        "SELECT * FROM orders WHERE status = ?", orderRowMapper(), status);

// Single value
Integer count = jdbcTemplate.queryForObject(
        "SELECT COUNT(*) FROM orders WHERE customer_id = ?", Integer.class, customerId);

// Insert / update / delete — returns rows affected
int rows = jdbcTemplate.update(
        "INSERT INTO orders (customer_id, amount, status) VALUES (?, ?, ?)",
        customerId, amount, status.name());

// Insert and retrieve the generated key
KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.update(connection -> {
    PreparedStatement ps = connection.prepareStatement(
            "INSERT INTO orders (customer_id, amount) VALUES (?, ?)",
            Statement.RETURN_GENERATED_KEYS);
    ps.setString(1, customerId);
    ps.setBigDecimal(2, amount);
    return ps;
}, keyHolder);
Long generatedId = keyHolder.getKey().longValue();

// Batch update — MUCH faster than a loop of single updates
jdbcTemplate.batchUpdate(
        "INSERT INTO order_items (order_id, product_id, qty) VALUES (?, ?, ?)",
        items, 500,                                  // batch size
        (ps, item) -> {
            ps.setLong(1, item.getOrderId());
            ps.setLong(2, item.getProductId());
            ps.setInt(3, item.getQuantity());
        });
```

### NamedParameterJdbcTemplate — more readable for many parameters

```java
@Repository
@RequiredArgsConstructor
public class OrderSearchRepository {

    private final NamedParameterJdbcTemplate namedJdbcTemplate;

    public List<Order> search(OrderSearchCriteria criteria) {
        String sql = """
                SELECT * FROM orders
                WHERE status = :status
                  AND created_at BETWEEN :fromDate AND :toDate
                  AND amount >= :minAmount
                ORDER BY created_at DESC
                """;

        MapSqlParameterSource params = new MapSqlParameterSource()
                .addValue("status", criteria.getStatus().name())
                .addValue("fromDate", criteria.getFromDate())
                .addValue("toDate", criteria.getToDate())
                .addValue("minAmount", criteria.getMinAmount());

        return namedJdbcTemplate.query(sql, params, orderRowMapper());
    }
}
```
With five positional `?` markers it's easy to swap two arguments and introduce a silent bug. **Named parameters remove that class of error** — prefer them beyond about three parameters.

## 29.4 Exception translation

```java
try {
    jdbcTemplate.update("INSERT INTO customers (email) VALUES (?)", email);
} catch (DuplicateKeyException e) {          // Spring's unchecked exception
    throw new DuplicateResourceException("email", email);
}
```
`@Repository` activates exception translation, converting vendor-specific `SQLException`s (MySQL error 1062, Postgres 23505) into a consistent hierarchy:

```
DataAccessException                        (unchecked root)
├── DataIntegrityViolationException
│     └── DuplicateKeyException
├── DataAccessResourceFailureException
├── EmptyResultDataAccessException
├── IncorrectResultSizeDataAccessException
└── OptimisticLockingFailureException
```
**The benefit:** your service layer catches `DuplicateKeyException` regardless of the database vendor. Switching from MySQL to Postgres doesn't change your error-handling code.

## 29.5 JdbcTemplate vs JPA — when to use which

| | JdbcTemplate | Spring Data JPA |
|---|---|---|
| SQL | **You write it** | Generated (or JPQL/native) |
| Control over SQL | **Total** | Indirect |
| Boilerplate | Moderate (RowMappers) | Minimal |
| Object mapping | Manual | Automatic |
| Caching, dirty checking, lazy loading | No | Yes |
| Learning curve | Low | Higher |
| Performance predictability | **High — what you write is what runs** | Lower (N+1 risk) |
| Best for | Complex reports, bulk operations, legacy schemas, precise tuning | **CRUD and domain-driven persistence** |

**They coexist happily.** A very common and sensible real-world setup: **Spring Data JPA for CRUD, `JdbcTemplate` for complex reporting queries and bulk operations.** Using both is a sign of good judgement, not inconsistency — worth saying explicitly in an interview.

## 29.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| **String-concatenating SQL** | **SQL injection.** Always use `?` or named parameters. |
| `queryForObject` when no row matches | Throws `EmptyResultDataAccessException` — catch it or use `query().stream().findFirst()` |
| Looping single inserts instead of `batchUpdate` | Orders of magnitude slower |
| `SELECT *` | Breaks when columns change; fetches unneeded data |
| Forgetting `@Repository` | No exception translation |
| Building dynamic SQL by concatenation | Injection risk and unreadable — use `NamedParameterJdbcTemplate` with conditional clauses |

**The injection point, explicitly:**
```java
// ❌ CATASTROPHIC — never do this
String sql = "SELECT * FROM users WHERE email = '" + email + "'";
// email = "x' OR '1'='1" returns every user.

// ✅ Correct — the driver binds the value, never parses it as SQL
jdbcTemplate.query("SELECT * FROM users WHERE email = ?", mapper, email);
```

## 29.7 TL Questions

**Q: What is `JdbcTemplate` and why use it?**
A: Spring's JDBC abstraction. It manages connections, statements, result sets and closing, and translates SQL exceptions into Spring's unchecked hierarchy. We write only the SQL and a row mapper.

**Q: When do we use it instead of JPA?**
A: For complex reporting queries, bulk operations, and anywhere we need exact control of the SQL. JPA is better for CRUD on our domain entities. We use both in the same service, which is a normal and deliberate split.

**Q: How does exception translation help us?**
A: It gives vendor-independent exceptions. We catch `DuplicateKeyException` whether the database is MySQL or Postgres, so our error handling doesn't need to know vendor error codes. It's enabled by `@Repository`.

**Q: How do we prevent SQL injection?**
A: Always bind values as parameters, never concatenate them into the SQL string. The driver sends the statement and values separately, so user input is never parsed as SQL.

**Q: Why use `NamedParameterJdbcTemplate`?**
A: Readability and safety once there are more than about three parameters. With positional markers it's easy to transpose two arguments and get a bug that compiles and runs.

## 29.8 Interview Questions

**Beginner — What does `JdbcTemplate` remove?**
Connection and resource management, statement creation, result set iteration, closing, and checked exception handling.

**Intermediate — What is a `RowMapper`?**
A callback mapping one `ResultSet` row to an object. `RowMapper` maps row by row; `ResultSetExtractor` receives the whole `ResultSet` (useful for aggregating a join into a single object graph).

**Intermediate — What happens if `queryForObject` returns no rows?**
It throws `EmptyResultDataAccessException`. To treat "not found" as normal, use `query(...)` and take the first element, or catch the exception.

**Advanced — What is `DataAccessException` and why is it unchecked?**
Spring's consistent, vendor-neutral data-access exception hierarchy. It is unchecked because most data-access failures are not recoverable at the call site, so forcing every caller to catch them would only add noise — and unchecked exceptions also trigger Spring's default transaction rollback.

**Advanced — How much faster is `batchUpdate`?**
Substantially — often ten times or more for large inserts, because it sends many statements in one round trip instead of paying network latency per row. With JDBC batching properly enabled, the difference grows with row count.

## 29.9 Quick Revision

1. `JdbcTemplate` = SQL you write, plumbing Spring handles.
2. Key methods: `query`, `queryForObject`, `update`, `batchUpdate`.
3. `RowMapper` maps a row to an object.
4. `@Repository` enables translation to `DataAccessException`.
5. `DataAccessException` is **unchecked** and vendor-neutral.
6. **Always bind parameters** — never concatenate SQL.
7. `NamedParameterJdbcTemplate` for readability with many parameters.
8. `batchUpdate` for bulk work — dramatically faster.
9. Mixing JPA (CRUD) and `JdbcTemplate` (reports/bulk) is good practice.

## 29.10 TL Explanation (speak this)

> "`JdbcTemplate` is Spring's JDBC wrapper — it handles the connection, statement, result set and closing, so we just write SQL and a row mapper. We use it alongside JPA: JPA for CRUD on our entities, `JdbcTemplate` for reporting queries and bulk inserts where we want exact control of the SQL and predictable performance. It also gives us Spring's `DataAccessException` hierarchy, so we catch `DuplicateKeyException` without caring whether we're on MySQL or Postgres. And we always bind parameters rather than concatenating, so SQL injection isn't possible."

---

# 30. JPA vs Hibernate vs Spring Data JPA

## 30.1 The three things people confuse

This is one of the most commonly asked and most commonly muddled questions. Get it precisely right.

```
┌──────────────────────────────────────────────────────────────┐
│  SPRING DATA JPA                                              │
│  A Spring project. Generates repository implementations       │
│  from interfaces. You write findByCustomerId(); it writes     │
│  the query.                                                   │
│  ──────────────────────────────────────────────────────────  │
│                        uses ▼                                 │
│  JPA  (Jakarta Persistence API)                               │
│  A SPECIFICATION — just interfaces and annotations.           │
│  @Entity, @Id, EntityManager, JPQL. It has NO code that runs. │
│  ──────────────────────────────────────────────────────────  │
│                   implemented by ▼                            │
│  HIBERNATE                                                    │
│  The actual IMPLEMENTATION (ORM provider). Contains the real  │
│  logic: SQL generation, caching, dirty checking, lazy loading.│
│  ──────────────────────────────────────────────────────────  │
│                        uses ▼                                 │
│  JDBC → Driver → DATABASE                                     │
└──────────────────────────────────────────────────────────────┘
```

**The analogy that makes it stick:**
- **JPA** = the *rules of the game* (the specification)
- **Hibernate** = a *team that plays by those rules* (an implementation)
- **Spring Data JPA** = a *coach who plays the game for you* (a convenience layer)

## 30.2 Comparison table

| Aspect | JPA | Hibernate | Spring Data JPA |
|---|---|---|---|
| **What it is** | Specification | ORM implementation | Abstraction layer |
| **Contains runnable code?** | **No — interfaces only** | **Yes** | Yes |
| **Provided by** | Jakarta EE | Red Hat | Spring (Pivotal/VMware) |
| **Key artifacts** | `@Entity`, `EntityManager`, JPQL | `Session`, `SessionFactory`, HQL, Criteria | `JpaRepository`, query methods |
| **Can be used alone?** | No — needs a provider | **Yes** | No — needs JPA + a provider |
| **Extra features** | — | Caching, `@BatchSize`, filters, multi-tenancy, Envers | Derived queries, paging, auditing, specifications |
| **Vendor lock-in** | None | Hibernate-specific APIs lock you in | None (JPA-based) |

**Alternative JPA providers:** EclipseLink (the reference implementation), OpenJPA. Hibernate is by far the most used, and is what Spring Boot's JPA starter includes.

## 30.3 The same operation at each level

```java
// ---------- Level 1: Pure JPA (EntityManager) ----------
@Repository
public class OrderRepositoryJpa {
    @PersistenceContext
    private EntityManager entityManager;

    public Order findById(Long id) {
        return entityManager.find(Order.class, id);
    }

    public List<Order> findByStatus(OrderStatus status) {
        return entityManager.createQuery(
                    "SELECT o FROM Order o WHERE o.status = :status", Order.class)
                .setParameter("status", status)
                .getResultList();
    }

    @Transactional
    public Order save(Order order) {
        if (order.getId() == null) { entityManager.persist(order); return order; }
        return entityManager.merge(order);
    }
}

// ---------- Level 2: Hibernate-specific (Session) ----------
// Works, but couples you to Hibernate. Only do this for Hibernate-only features.
Session session = entityManager.unwrap(Session.class);
Order order = session.get(Order.class, id);

// ---------- Level 3: Spring Data JPA ----------
public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByStatus(OrderStatus status);        // implementation GENERATED
}
// That's it. No implementation class at all.
```

**The progression is the point:** each level removes code. Spring Data JPA removes essentially all of it for standard operations, while still letting you drop down to `EntityManager` or native SQL when you need to.

## 30.4 What is ORM, and its trade-offs

**ORM (Object-Relational Mapping)** bridges two different worlds:

| Java (objects) | Relational database |
|---|---|
| Objects with references | Rows with foreign keys |
| Inheritance | No native inheritance |
| Collections | Join tables |
| Identity by reference | Identity by primary key |
| Navigate with `.` | Navigate with JOINs |

This mismatch is called the **object-relational impedance mismatch**, and ORM exists to bridge it.

**A balanced view — state both sides to a TL:**

| ORM advantages | ORM disadvantages |
|---|---|
| Far less boilerplate | Hides the generated SQL |
| Database-portable | Performance traps (N+1, over-fetching) |
| Caching, dirty checking | Steep learning curve |
| Type-safe domain model | Complex queries can be awkward |
| Automatic relationship handling | Debugging requires understanding internals |

**The honest position:** *"ORM is excellent for CRUD on a well-modelled domain and poor for complex reporting. We use JPA for the domain and `JdbcTemplate` or native queries for reports."* That answer shows judgement rather than dogma.

## 30.5 Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```
The starter brings Spring Data JPA, **Hibernate**, HikariCP, and `spring-tx`.

```properties
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.open-in-view=false
spring.jpa.properties.hibernate.jdbc.batch_size=50
```

### `ddl-auto` — get this right

| Value | Behaviour | Use in |
|---|---|---|
| `none` | Do nothing | Production (with migrations) |
| `validate` | **Verify the schema matches the entities; fail if not** | **Production** |
| `update` | Alter the schema to match entities | Local dev only |
| `create` | Drop and create at startup | Tests |
| `create-drop` | Create at startup, drop at shutdown | Tests |

**Never use `update` in production.** It never drops columns, can't do renames properly, produces unpredictable DDL, and gives no migration history or rollback. **Use Flyway or Liquibase for schema changes and set `ddl-auto=validate`** so the app refuses to start if the schema and entities disagree — catching a missed migration at deploy time rather than at runtime.

## 30.6 TL Questions

**Q: What's the difference between JPA and Hibernate?**
A: JPA is a specification — just interfaces and annotations, with no executable logic. Hibernate is an implementation of that specification and contains the actual code that generates SQL, manages caching and does dirty checking. We code against JPA so we could swap providers, and Hibernate does the work underneath.

**Q: Where does Spring Data JPA fit?**
A: It's a layer above JPA that generates repository implementations from our interfaces. We declare `findByStatus` and it builds the query. It doesn't replace JPA or Hibernate — it uses both.

**Q: Why not use Hibernate's `Session` directly?**
A: It locks us to Hibernate. The JPA `EntityManager` covers nearly everything, and we can always `unwrap` to `Session` for the rare Hibernate-only feature. Keeping to the standard API keeps our options open at no cost.

**Q: What `ddl-auto` do we use and why?**
A: `validate` in production. Hibernate checks the schema matches our entities and refuses to start otherwise, so a missed migration fails at deploy time. Schema changes go through Flyway, never through `update` — which never drops columns and gives no migration history or rollback.

**Q: When is ORM the wrong tool?**
A: Complex reporting with multi-table aggregation, and bulk operations. There we use native SQL or `JdbcTemplate`, because ORM hides the SQL and it's easy to generate something inefficient without noticing.

## 30.7 Interview Questions

**Beginner — Is Hibernate a framework or a specification?**
An implementation (ORM framework). JPA is the specification.

**Beginner — Which JPA provider does Spring Boot use?**
Hibernate, included in `spring-boot-starter-data-jpa`.

**Intermediate — Can we use Hibernate without JPA?**
Yes — through its native `SessionFactory`/`Session` API, which predates JPA. Modern projects use the JPA API for portability.

**Intermediate — JPQL vs HQL vs native SQL?**
JPQL is the JPA standard query language, operating on **entities and fields**. HQL is Hibernate's superset with extra features. Native SQL is raw, database-specific SQL run through the same API. Prefer JPQL; use native when you need vendor features.

**Advanced — What is the object-relational impedance mismatch?**
The structural differences between object models and relational models: inheritance, collections, associations, and identity semantics have no direct relational equivalents. ORM maps between them, and most ORM complexity and performance traps arise from this mapping.

**Advanced — Why avoid `ddl-auto=update` in production?**
It never removes columns or constraints, handles renames as add+orphan, produces vendor-dependent DDL, has no versioning or rollback, and can lock large tables unpredictably during deployment. Versioned migrations with Flyway or Liquibase plus `validate` is the correct approach.

## 30.8 Quick Revision

1. **JPA = specification. Hibernate = implementation. Spring Data JPA = abstraction on top.**
2. JPA has no runnable code — only interfaces and annotations.
3. Hibernate does SQL generation, caching, dirty checking, lazy loading.
4. Spring Data JPA generates repository implementations from interfaces.
5. Prefer the JPA API; `unwrap` to Hibernate only for provider-specific features.
6. ORM bridges the **object-relational impedance mismatch**.
7. ORM is great for CRUD, weak for complex reporting — mix in `JdbcTemplate`.
8. **`ddl-auto=validate` in production**; Flyway/Liquibase for schema changes.
9. `spring.jpa.open-in-view=false` — always.

## 30.9 TL Explanation (speak this)

> "JPA is just a specification — annotations and interfaces with no logic in them. Hibernate is the implementation that actually generates the SQL and manages caching and dirty checking, and Spring Data JPA sits on top generating our repository implementations from interface method names. We code against the JPA API rather than Hibernate's `Session` so we're not locked to a provider. In production we run `ddl-auto=validate` with Flyway for migrations, so the app refuses to start if the schema and entities disagree — that catches a missed migration at deploy time. And for complex reports we drop to native SQL or `JdbcTemplate`, because ORM is strong for CRUD and weak for aggregation-heavy queries."

---

# 31. Entity and Mapping Annotations

## 31.1 What is an Entity?

**Simple words:**
An entity is a **Java class that maps to a database table**. One object = one row. Each field = one column.

**Technical wording:**
An entity is a lightweight persistent domain object annotated `@Entity`, representing a table, whose instances are managed by the persistence provider and correspond to rows identified by a primary key.

## 31.2 A complete entity

```java
package com.company.orderservice.entity;

@Entity
@Table(name = "orders",
       indexes = {
           @Index(name = "idx_order_customer", columnList = "customer_id"),
           @Index(name = "idx_order_status",   columnList = "status")
       },
       uniqueConstraints = @UniqueConstraint(name = "uk_order_number",
                                             columnNames = "order_number"))
@Getter @Setter
@NoArgsConstructor                 // REQUIRED by JPA
@AllArgsConstructor
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "order_number", nullable = false, length = 20, updatable = false)
    private String orderNumber;

    @Column(name = "customer_id", nullable = false)
    private String customerId;

    @Column(name = "total_amount", nullable = false, precision = 19, scale = 2)
    private BigDecimal totalAmount;        // NEVER use double for money

    @Enumerated(EnumType.STRING)           // store "PENDING", not 0
    @Column(nullable = false, length = 20)
    private OrderStatus status;

    @Column(name = "created_at", nullable = false, updatable = false)
    @CreationTimestamp                     // Hibernate sets this on insert
    private LocalDateTime createdAt;

    @UpdateTimestamp                       // Hibernate sets this on every update
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @Version                               // OPTIMISTIC LOCKING
    private Long version;

    @Lob
    @Column(name = "notes")
    private String notes;

    @Transient                             // NOT persisted — calculated in memory
    private String displayLabel;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}
```

## 31.3 The essential annotations

| Annotation | Purpose |
|---|---|
| `@Entity` | Marks the class as persistent |
| `@Table` | Table name, indexes, unique constraints |
| `@Id` | Primary key |
| `@GeneratedValue` | How the PK is generated |
| `@Column` | Column name, nullability, length, precision |
| `@Enumerated` | How an enum is stored |
| `@Temporal` | Date precision (legacy `Date`/`Calendar` only) |
| `@Lob` | Large object (CLOB/BLOB) |
| `@Transient` | **Do not persist this field** |
| `@Version` | Optimistic locking version column |
| `@Embedded` / `@Embeddable` | Value object mapped into the same table |
| `@CreationTimestamp` / `@UpdateTimestamp` | Hibernate auto-timestamps |

### JPA requirements for an entity class

1. Annotated `@Entity`
2. Has an `@Id`
3. Has a **public or protected no-argument constructor**
4. Class is **not `final`**; persistent fields/methods not `final`
5. Should implement `Serializable` if detached instances travel over the wire

**Why the no-arg constructor?** Hibernate instantiates entities **reflectively** when loading rows. It cannot guess constructor arguments. **Why not `final`?** Hibernate creates **proxy subclasses** for lazy loading — it cannot subclass a `final` class. These two rules cause real, confusing failures when Lombok's `@Builder` is used alone (it removes the no-arg constructor).

## 31.4 `@GeneratedValue` strategies — important

| Strategy | How it works | Batch inserts? | Notes |
|---|---|---|---|
| **`IDENTITY`** | Database auto-increment | **NO** | Simple; **disables JDBC batching** |
| **`SEQUENCE`** | Database sequence object | **YES** | **Best for Postgres/Oracle** |
| `TABLE` | A separate table simulates a sequence | Yes | Slow, contention-prone — avoid |
| `AUTO` | Provider picks | Depends | Unpredictable — be explicit |

**The `IDENTITY` batching trap (genuinely valuable to know):**
With `IDENTITY`, Hibernate must execute each INSERT immediately to learn the generated ID, so it **cannot batch inserts**. Setting `hibernate.jdbc.batch_size=50` has no effect. With `SEQUENCE` and an allocation size, Hibernate pre-fetches IDs and batches inserts properly.

```java
// Optimised sequence generation — 50 IDs per database round trip
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "order_seq")
@SequenceGenerator(name = "order_seq", sequenceName = "order_sequence",
                   allocationSize = 50)     // must match the DB sequence INCREMENT BY
private Long id;
```
> **Caution:** `allocationSize` must match the actual sequence's increment in the database, or you get duplicate-key errors. MySQL has no sequences, so `IDENTITY` is the practical choice there.

## 31.5 `@Enumerated` — a genuine production trap

```java
@Enumerated(EnumType.ORDINAL)     // ❌ stores 0, 1, 2 — THE INDEX
private OrderStatus status;

@Enumerated(EnumType.STRING)      // ✅ stores "PENDING", "SHIPPED"
private OrderStatus status;
```

**Why `ORDINAL` is dangerous:**
```java
enum OrderStatus { PENDING, CONFIRMED, SHIPPED }        // PENDING=0, CONFIRMED=1, SHIPPED=2
```
Later someone inserts a value:
```java
enum OrderStatus { PENDING, CANCELLED, CONFIRMED, SHIPPED }   // CANCELLED=1 now!
```
**Every existing row storing `1` silently changes meaning from CONFIRMED to CANCELLED.** No error, no warning — just corrupted data across the whole table. `ORDINAL` is also unreadable in the database.

**Always use `EnumType.STRING`.** The only cost is a few bytes per row.

## 31.6 `equals()` and `hashCode()` for entities (advanced but important)

A subtle area that causes real bugs.

```java
// ❌ WRONG — Lombok's default uses ALL fields
@Data                       // generates equals/hashCode over every field
@Entity
public class Order { }
```
Problems: the hash code changes when any field changes (breaking `HashSet` membership), and it triggers lazy loading of associations just to compute equality.

```java
// ✅ CORRECT — business key, or ID with a stable hashCode
@Entity
@Getter @Setter
public class Order {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "order_number", nullable = false, unique = true, updatable = false)
    private String orderNumber;         // natural business key

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Order other)) return false;
        return orderNumber != null && orderNumber.equals(other.orderNumber);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();   // constant — stable before and after persist
    }
}
```

**Why a constant `hashCode()`?** Before `persist()`, `id` is `null`. After `persist()`, it has a value. If `hashCode()` depends on `id`, an entity added to a `HashSet` *before* saving becomes unfindable *after* saving, because its bucket changed. A constant hash keeps it locatable; `equals` still distinguishes instances correctly.

**Never use `@Data` on an entity.** It also generates `toString()` over all fields, which triggers lazy loading and can cause infinite recursion on bidirectional relationships. Use `@Getter`/`@Setter` and write `equals`/`hashCode` deliberately.

## 31.7 `@Embeddable` — value objects

```java
@Embeddable
@Getter @Setter @NoArgsConstructor @AllArgsConstructor
public class Address {
    @Column(name = "street")   private String street;
    @Column(name = "city")     private String city;
    @Column(name = "pincode")  private String pincode;
}

@Entity
public class Customer {
    @Id @GeneratedValue
    private Long id;

    @Embedded
    private Address billingAddress;      // → columns street, city, pincode

    @Embedded
    @AttributeOverrides({                // reuse the same type with different columns
        @AttributeOverride(name = "street",  column = @Column(name = "ship_street")),
        @AttributeOverride(name = "city",    column = @Column(name = "ship_city")),
        @AttributeOverride(name = "pincode", column = @Column(name = "ship_pincode"))
    })
    private Address shippingAddress;
}
```
All columns live in the `customer` table — no join. It groups related fields into a reusable, meaningful type instead of six loose fields.

## 31.8 Auditing — who changed what, when

```java
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaAuditingConfig {
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
                .map(SecurityContext::getAuthentication)
                .filter(Authentication::isAuthenticated)
                .map(Authentication::getName)
                .or(() -> Optional.of("SYSTEM"));
    }
}

@MappedSuperclass                       // shared columns, NOT its own table
@EntityListeners(AuditingEntityListener.class)
@Getter @Setter
public abstract class BaseAuditEntity {

    @CreatedDate     @Column(updatable = false) private LocalDateTime createdAt;
    @LastModifiedDate                           private LocalDateTime updatedAt;
    @CreatedBy       @Column(updatable = false) private String createdBy;
    @LastModifiedBy                             private String updatedBy;
}

@Entity
public class Order extends BaseAuditEntity {    // inherits all four audit columns
    @Id @GeneratedValue private Long id;
}
```
**`@MappedSuperclass` vs `@Entity` inheritance:** `@MappedSuperclass` contributes **columns** to subclasses but is not itself queryable and has no table. That's exactly what you want for audit fields.

## 31.9 Common Mistakes

| Mistake | Consequence |
|---|---|
| `@EnumType.ORDINAL` | **Silent data corruption** when the enum changes |
| `double`/`float` for money | Rounding errors — use `BigDecimal` with `precision`/`scale` |
| `@Data` on an entity | Broken `equals`/`hashCode`, lazy loading in `toString`, recursion |
| Missing no-arg constructor (Lombok `@Builder` alone) | Hibernate can't instantiate the entity |
| `final` entity class | Lazy proxies impossible |
| Exposing entities in the API | See Section 21 |
| `@Column(nullable=false)` without a real DB constraint | Only enforced when Hibernate generates the DDL |
| No index on foreign keys / filter columns | Full table scans as data grows |
| Mutable `@Id` | Breaks identity and the persistence context |

## 31.10 TL Questions

**Q: What is an entity?**
A: A Java class mapped to a database table — one instance corresponds to one row, and Hibernate manages its persistent state.

**Q: Why must entities have a no-arg constructor and not be final?**
A: Hibernate instantiates entities reflectively when loading rows, so it needs a no-arg constructor, and it creates proxy subclasses for lazy loading, which is impossible on a final class. This is why Lombok's `@Builder` on its own breaks entities — it removes the default constructor.

**Q: Why `EnumType.STRING`?**
A: `ORDINAL` stores the enum's index. If anyone inserts a new constant in the middle, every existing row silently changes meaning — corrupt data with no error. `STRING` stores the name, so it's stable and readable in the database.

**Q: Why don't we use `@Data` on entities?**
A: It generates `equals`/`hashCode` over all fields, which breaks `HashSet` behaviour when a field changes and triggers lazy loading just to compare. Its `toString` also loads lazy associations and can recurse infinitely on bidirectional relationships.

**Q: How do we track who changed a record?**
A: JPA auditing — a `@MappedSuperclass` with `@CreatedDate`, `@LastModifiedDate`, `@CreatedBy` and `@LastModifiedBy`, populated automatically from the security context via an `AuditorAware` bean.

**Q: Why `BigDecimal` for money?**
A: `double` can't represent decimal fractions exactly, so amounts drift over repeated arithmetic. `BigDecimal` with explicit precision and scale is exact, which is mandatory for financial data.

**Q: What does `@Version` do?**
A: Optimistic locking. Hibernate increments it on each update and includes the old value in the WHERE clause. If another transaction changed the row meanwhile, zero rows match and we get an `OptimisticLockException` instead of silently overwriting their change.

## 31.11 Interview Questions

**Beginner — What does `@Transient` do?**
Excludes a field from persistence — it exists in the object but has no column.

**Beginner — What is `@MappedSuperclass`?**
A class whose mapped fields are inherited as columns by subclasses, but which has no table and cannot be queried itself.

**Intermediate — `IDENTITY` vs `SEQUENCE`?**
`IDENTITY` uses the database's auto-increment and requires an immediate INSERT to obtain the key, which **disables JDBC batch inserts**. `SEQUENCE` pre-fetches IDs from a sequence, allowing batching and better bulk performance. MySQL supports only `IDENTITY` practically; Postgres and Oracle favour `SEQUENCE`.

**Intermediate — Why is a constant `hashCode()` recommended for entities?**
Because the ID is null before persist and populated after. An ID-based hash changes on save, so an entity placed in a `HashSet` before saving cannot be found afterwards. A constant hash keeps bucket placement stable while `equals` still distinguishes instances.

**Advanced — `@Embeddable` vs `@OneToOne`?**
`@Embeddable` maps a value object into the **same table** — no join, no separate identity, lifecycle tied to the owner. `@OneToOne` is a separate table and entity with its own identity and lifecycle. Use embeddable for value semantics like an address, and a relationship for something with independent identity.

**Advanced — What are the entity inheritance strategies?**
`SINGLE_TABLE` (one table plus a discriminator — fast, but subclass columns must be nullable), `JOINED` (one table per class joined by PK — normalised, requires joins), and `TABLE_PER_CLASS` (one table per concrete class — polymorphic queries become UNIONs and perform poorly).

## 31.12 Quick Revision

1. `@Entity` + `@Id` + no-arg constructor + non-final class.
2. **`@Enumerated(EnumType.STRING)` always.**
3. `BigDecimal` with `precision`/`scale` for money.
4. Never `@Data` on entities — write `equals`/`hashCode` deliberately.
5. Constant `hashCode()`; `equals` on a business key.
6. `IDENTITY` blocks batch inserts; `SEQUENCE` with `allocationSize` enables them.
7. `@Version` = optimistic locking.
8. `@MappedSuperclass` + `@EnableJpaAuditing` for audit columns.
9. `@Embeddable` for value objects in the same table.
10. Index foreign keys and frequent filter columns.
11. `@Transient` excludes a field from persistence.

## 31.13 TL Explanation (speak this)

> "Entities are our table mappings — one class per table, managed by Hibernate. A few conventions we hold to: enums are always stored as `STRING` rather than ordinal, because an ordinal silently corrupts every existing row if someone inserts a constant in the middle; money is always `BigDecimal` with explicit precision; and we never put Lombok's `@Data` on an entity because its generated `equals` and `toString` trigger lazy loading and break `HashSet` behaviour. Audit columns come from a shared `@MappedSuperclass` with JPA auditing, so we always know who changed a record and when, and `@Version` gives us optimistic locking so concurrent updates fail loudly instead of overwriting each other."

---

# 32. EntityManager and Persistence Context

## 32.1 What is the Persistence Context?

**Simple words:**
The persistence context is a **workspace in memory where Hibernate keeps the entities it is currently managing**, for the duration of a transaction.

Think of it as Hibernate's **notepad**: it writes down every entity you load or save, remembers what each looked like originally, and at the end compares them to decide what SQL to run.

This one concept explains **first-level cache, dirty checking, lazy loading and entity states** — all four are consequences of the persistence context.

**Technical wording:**
The persistence context is a set of managed entity instances with unique persistent identity, maintained by an `EntityManager`. It acts as a first-level cache, tracks state changes for automatic dirty checking, and guarantees that a given database row maps to exactly one object instance within its scope.

## 32.2 EntityManager

**Simple words:** the `EntityManager` is your **handle to the persistence context** — the API you use to find, save and delete entities.

```java
@Repository
public class OrderRepositoryImpl {

    @PersistenceContext          // NOT @Autowired — see explanation below
    private EntityManager entityManager;
}
```

| Method | What it does |
|---|---|
| `persist(entity)` | Make a new entity managed → INSERT at flush |
| `merge(entity)` | Copy a detached entity's state into a managed one → UPDATE |
| `find(Class, id)` | Look up by PK — **checks the persistence context first** |
| `getReference(Class, id)` | Return a lazy **proxy** without hitting the DB |
| `remove(entity)` | Mark for deletion → DELETE at flush |
| `flush()` | Force pending SQL to the database **now** |
| `clear()` | Detach **all** entities from the context |
| `detach(entity)` | Detach one entity |
| `refresh(entity)` | Reload from the database, discarding in-memory changes |
| `createQuery(jpql)` | Create a JPQL query |

### Why `@PersistenceContext` and not `@Autowired`?

This is an excellent interview question.

`EntityManager` is **not thread-safe** and is bound to a transaction. If Spring injected a single shared instance into your singleton repository, concurrent requests would share one persistence context — a severe correctness bug.

`@PersistenceContext` injects a **thread-safe proxy**. On each call it looks up the `EntityManager` bound to the **current thread's transaction** and delegates to it. So every request transparently gets its own persistence context.

```
Singleton Repository
      │ holds
      ▼
 EntityManager PROXY  (shared, thread-safe)
      │ at call time resolves
      ▼
 The real EntityManager for THIS thread's transaction
```

**In practice with Spring Data JPA you rarely touch either** — but understanding this explains why persistence context scope equals transaction scope.

## 32.3 Persistence context scope = transaction scope

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;

    @Transactional
    public void updateOrderStatus(Long orderId, OrderStatus newStatus) {
        // ─── PERSISTENCE CONTEXT OPENS HERE (transaction begins) ───

        Order order = orderRepository.findById(orderId)     // SELECT; entity now MANAGED
                .orElseThrow(() -> new ResourceNotFoundException("Order", orderId));

        order.setStatus(newStatus);       // just a setter — no SQL yet

        // NO save() CALL NEEDED — see dirty checking below

        // ─── TRANSACTION COMMITS ───
        // Hibernate flushes: compares current state to the snapshot,
        // detects status changed, issues UPDATE, then commits.
        // PERSISTENCE CONTEXT CLOSES — all entities become DETACHED.
    }
}
```

**The most surprising thing for newcomers: you don't call `save()`.** Because the entity is managed, Hibernate detects the change automatically.

## 32.4 Dirty Checking

**Simple words:** when Hibernate loads an entity it takes a **snapshot** of the loaded values. At flush time it compares the current values against that snapshot. Any difference produces an UPDATE automatically.

```
1. findById(1)
        │
        ▼
   SELECT ... FROM orders WHERE id = 1
        │
        ▼
   Entity created and stored in the persistence context
   PLUS a SNAPSHOT of the loaded values:
        snapshot = { status: "PENDING", amount: 500 }
        │
        ▼
2. order.setStatus(SHIPPED)          ← in-memory only
        current  = { status: "SHIPPED", amount: 500 }
        │
        ▼
3. Transaction commit → FLUSH
        compare current vs snapshot → 'status' differs
        │
        ▼
   UPDATE orders SET status = 'SHIPPED', version = version + 1
   WHERE id = 1 AND version = 3
```

**Memory cost note:** that snapshot doubles the memory per managed entity. Loading 100,000 entities in one transaction means 200,000 objects. For bulk reads use a projection or `@Transactional(readOnly = true)`, which lets Hibernate skip snapshots entirely.

## 32.5 Flush modes and ordering

**Flush** = sending pending SQL to the database. It does **not** commit.

Hibernate flushes automatically:
1. Before the transaction commits
2. Before a query that might be affected by pending changes
3. When you call `flush()` explicitly

```java
@Transactional
public void demo() {
    Order order = new Order();
    orderRepository.save(order);        // persist() — NO INSERT yet, just queued

    // Hibernate flushes here, because the query could be affected by the pending insert
    List<Order> all = orderRepository.findAll();   // INSERT runs, then SELECT
}
```

**Why defer writes at all?** Batching (fewer round trips), correct ordering (parent before child), and the chance to cancel work that becomes unnecessary.

**Operation order at flush (fixed, and occasionally surprising):**
```
1. INSERTs      2. UPDATEs      3. Collection deletions
4. Collection updates/inserts   5. Entity DELETEs
```
This is why a "delete then insert with the same unique key" in one transaction fails with a constraint violation — the INSERT runs *before* the DELETE. The fix is an explicit `flush()` between them.

## 32.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Expecting changes to persist without a transaction | Entity is detached; no dirty checking; **changes silently lost** |
| `@Autowired EntityManager` | Thread-safety violation |
| Loading thousands of entities in one transaction | Memory blowout from snapshots |
| Using a detached entity and expecting dirty checking | No updates |
| Calling `save()` on an already-managed entity | Harmless but unnecessary — dirty checking already handles it |
| Delete-then-insert with the same key, no `flush()` | Constraint violation from flush ordering |

## 32.7 TL Questions

**Q: What is the persistence context?**
A: Hibernate's in-memory workspace for the entities it currently manages, scoped to the transaction. It acts as the first-level cache, tracks changes for dirty checking, and guarantees one object instance per database row within its scope.

**Q: Why don't we call `save()` after changing a managed entity?**
A: Because dirty checking handles it. Hibernate snapshots the entity when it's loaded and compares at flush time, issuing an UPDATE for whatever changed. Calling `save()` is harmless but redundant.

**Q: Why `@PersistenceContext` instead of `@Autowired` for `EntityManager`?**
A: `EntityManager` isn't thread-safe and is bound to a transaction. `@PersistenceContext` injects a proxy that resolves the right `EntityManager` for the current thread's transaction, so concurrent requests each get their own persistence context.

**Q: What happens if we modify an entity outside a transaction?**
A: The entity is detached, so there's no dirty checking and the change is silently lost. This is a common bug — no error, just a change that never reaches the database.

**Q: What is flushing and when does it happen?**
A: Flushing sends the pending SQL to the database without committing. Hibernate flushes before commit, before a query whose results could be affected by pending changes, and when we call `flush()` explicitly.

**Q: Why does Hibernate defer writes instead of issuing SQL immediately?**
A: To batch statements into fewer round trips, to order them correctly — parents before children — and to avoid work that later becomes unnecessary within the same transaction.

**Q: What's the memory impact of the persistence context?**
A: Each managed entity costs roughly double, because of the snapshot used for dirty checking. That's why we use `readOnly = true` or projections for large reads — Hibernate can then skip snapshots.

## 32.8 Interview Questions

**Beginner — What is `EntityManager`?**
The JPA interface for interacting with the persistence context — finding, persisting, merging and removing entities, and creating queries.

**Intermediate — `persist()` vs `merge()`?**
`persist()` makes a **new** entity managed and throws if it's already persistent; it makes the passed instance managed. `merge()` copies the state of a **detached** instance into a managed one and **returns the managed instance** — the passed object stays detached. Forgetting to use the returned value from `merge()` is a classic bug.

**Intermediate — `find()` vs `getReference()`?**
`find()` hits the database immediately (unless it's already in the context) and returns the entity or null. `getReference()` returns a lazy **proxy** without a query; the database is hit on first property access, and it throws `EntityNotFoundException` then if the row is missing. `getReference()` is useful for setting a foreign-key association without loading the whole entity.

**Advanced — What guarantee does the persistence context give about identity?**
Within one persistence context, a given database row is represented by exactly **one** object instance, so `==` reference equality holds for two lookups of the same ID. This is the *repeatable read at the object level* guarantee.

**Advanced — Transaction-scoped vs extended persistence context?**
Transaction-scoped (the Spring default) opens and closes with the transaction. An extended persistence context spans multiple transactions, keeping entities managed between them — used in stateful Jakarta EE conversations, and rare in Spring applications.

## 32.9 Quick Revision

1. Persistence context = in-memory workspace of managed entities, **scoped to the transaction**.
2. `EntityManager` is the API handle to it.
3. Use **`@PersistenceContext`**, never `@Autowired` — thread safety.
4. **Dirty checking**: snapshot at load, compare at flush, auto-UPDATE.
5. **No `save()` needed** for an already-managed entity.
6. Flush ≠ commit. Flush sends SQL; commit ends the transaction.
7. Flush triggers: before commit, before an affected query, explicit call.
8. Flush order: INSERT → UPDATE → collection ops → DELETE.
9. One row = one object instance within a context (identity guarantee).
10. Snapshots double memory — use `readOnly` or projections for large reads.
11. Changes outside a transaction are **silently lost**.

## 32.10 TL Explanation (speak this)

> "The persistence context is Hibernate's in-memory workspace for the entities in the current transaction, and almost everything else follows from it. It snapshots each entity when loaded, so at commit time it compares and issues an UPDATE for whatever changed — that's dirty checking, and it's why we don't call `save()` on an entity we loaded inside a transaction. It's also the first-level cache, so repeated lookups of the same ID in one transaction don't re-query. The practical consequences are that modifying an entity outside a transaction silently does nothing, and that loading tens of thousands of entities is expensive because of those snapshots — so bulk reads use `readOnly` or projections."

---

# 33. First Level Cache

## 33.1 What is it?

**Simple words:**
The first-level cache **is** the persistence context, viewed from the caching angle. Within one transaction, if you ask for the same entity twice, the second request comes **from memory** — no second SELECT.

**Technical wording:**
The first-level cache is a mandatory, transaction-scoped cache maintained by the persistence context, keyed by entity type and identifier, guaranteeing object identity and eliminating redundant primary-key lookups within its scope.

## 33.2 Demonstration

```java
@Transactional
public void firstLevelCacheDemo() {

    Order o1 = orderRepository.findById(1L).orElseThrow();
    // SQL:  SELECT * FROM orders WHERE id = 1     ← database hit

    Order o2 = orderRepository.findById(1L).orElseThrow();
    // NO SQL — served from the first-level cache

    System.out.println(o1 == o2);          // true — the SAME object instance
}
```

**Two guarantees here:**
1. **No redundant SELECT** for the same ID in the same transaction.
2. **Object identity** — both references point to the same instance. Change one and the other "sees" it, because they *are* the same object.

## 33.3 Key characteristics

| Property | First Level Cache |
|---|---|
| Scope | **Transaction / persistence context** |
| Enabled by default | **Yes — cannot be disabled** |
| Shared between transactions | **No** |
| Shared between users/threads | **No** |
| Keyed by | Entity class + primary key |
| Cleared when | Transaction ends, or `clear()`/`detach()` |

## 33.4 Important limitation — queries bypass the cache

```java
@Transactional
public void limitation() {

    Order o1 = orderRepository.findById(1L).orElseThrow();   // SELECT — cached

    // A JPQL query ALWAYS goes to the database
    List<Order> orders = orderRepository.findByStatus(OrderStatus.PENDING);
    // SELECT * FROM orders WHERE status = 'PENDING'  ← executes even if id=1 matches

    // BUT: if the result contains id=1, Hibernate returns the CACHED instance
    // rather than a new object — identity is preserved, the query is not avoided.
}
```

**Two-part rule to remember:**
- The first-level cache only avoids a database round trip for **primary-key lookups** (`find`/`getReference`).
- For query results, the **query always runs**, but returned rows already present in the context are **mapped back to the existing instances**, preserving identity.

A subtle consequence: if a row changed in the database since it was cached, a query returning that row gives you the **cached (stale) version**, not the database version — because identity wins. Use `refresh()` if you genuinely need the current database state.

## 33.5 First vs Second Level Cache

| | First Level (L1) | Second Level (L2) |
|---|---|---|
| Scope | Persistence context / transaction | **`SessionFactory` / application-wide** |
| Default | **Always on** | **Off** — must be configured |
| Shared across transactions | No | **Yes** |
| Shared across users | No | **Yes** |
| Provider | Hibernate built-in | Ehcache, Hazelcast, Infinispan, Redis |
| Cleared on | Transaction end | TTL, eviction policy, explicit invalidation |
| Risk | None | **Stale data** in a clustered deployment |

```java
// Enabling L2 (only when justified)
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Country { }
```
```properties
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
```

**When L2 is appropriate:** read-mostly reference data — countries, currencies, product categories, configuration. **When it is dangerous:** frequently updated transactional data, especially across multiple application instances, where one node's update leaves other nodes serving stale data unless you run a distributed cache.

**The honest recommendation to give a TL:** *"L2 adds an invalidation problem. For reference data it's a clear win; for transactional data I'd reach for a query-level cache or an explicit Redis cache with a deliberate TTL, so the staleness window is a conscious decision rather than a side effect."*

## 33.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Expecting L1 to span transactions | It doesn't — new transaction, new empty context |
| Expecting queries to be served from L1 | Only PK lookups skip the database |
| Loading huge result sets in one transaction | L1 grows unbounded → `OutOfMemoryError` |
| Enabling L2 on volatile data in a cluster | Stale reads across nodes |
| Forgetting `clear()` in a long batch loop | Memory grows with every entity processed |

**The batch-processing pattern that matters:**

```java
@Transactional
public void processLargeBatch(List<Long> orderIds) {
    int count = 0;
    for (Long id : orderIds) {
        Order order = orderRepository.findById(id).orElseThrow();
        order.setStatus(OrderStatus.PROCESSED);

        if (++count % 50 == 0) {
            entityManager.flush();    // push pending SQL to the DB
            entityManager.clear();    // EMPTY the persistence context → free memory
        }
    }
}
```
Without `flush()` + `clear()`, processing 100,000 orders keeps 100,000 entities **plus 100,000 snapshots** in memory and the JVM runs out of heap. **This is the standard answer to "how do you handle bulk updates with JPA?"**

## 33.7 TL Questions

**Q: What is the first-level cache?**
A: The persistence context acting as a cache. Within a transaction, repeated lookups of the same entity by ID come from memory instead of the database, and both references point to the same object instance.

**Q: Can we disable it?**
A: No — it's mandatory and fundamental to how JPA works. Dirty checking and the identity guarantee both depend on it.

**Q: Does it help with queries?**
A: Only for primary-key lookups. A JPQL query always hits the database, though any returned rows already in the context are mapped back to the existing instances so identity is preserved.

**Q: What's the risk in batch processing?**
A: The persistence context grows with every entity loaded, each with a change-detection snapshot, so a large batch exhausts heap. We flush and clear every fifty records or so to keep it bounded.

**Q: Should we enable the second-level cache?**
A: Only for read-mostly reference data like countries and currencies. For transactional data across multiple instances it introduces stale reads, so we'd prefer an explicit Redis cache with a deliberate TTL where the staleness window is a conscious choice.

## 33.8 Interview Questions

**Beginner — What is the scope of the first-level cache?**
The persistence context, which in Spring equals the transaction.

**Intermediate — Is it shared between two users?**
No. Each transaction has its own persistence context, so there's no sharing between threads or users.

**Intermediate — How do you clear it?**
`entityManager.clear()` detaches everything; `detach(entity)` removes one. It also clears automatically when the transaction ends.

**Advanced — Why does a query return a cached instance rather than fresh data?**
Because the persistence context guarantees one instance per row. When a query result row matches an entity already in the context, Hibernate returns the existing instance rather than overwriting it, to avoid discarding pending in-memory changes. Use `refresh()` to force a reload.

**Advanced — What is the query cache and how does it relate to L2?**
An optional cache of **query result identifiers**, enabled separately with `hibernate.cache.use_query_cache`. It stores IDs, not entities, so it requires L2 to be enabled to resolve those IDs; without L2 it causes N additional lookups and makes performance worse.

## 33.9 Quick Revision

1. L1 cache = the persistence context; scoped to the transaction.
2. **Always on, cannot be disabled.**
3. Avoids repeat SELECTs for **PK lookups only**.
4. Guarantees one object instance per row (`==` holds).
5. Queries always hit the database; results are mapped to existing instances.
6. Not shared across transactions, threads or users.
7. **Batch processing: `flush()` + `clear()` every ~50 records.**
8. L2 is application-wide, off by default, good for reference data.
9. L2 in a cluster risks stale reads — use deliberately.

## 33.10 TL Explanation (speak this)

> "The first-level cache is just the persistence context seen as a cache — within a transaction, looking up the same entity by ID twice hits the database once, and both references are literally the same object. It's always on and can't be disabled, because dirty checking depends on it. The limitation worth knowing is that it only short-circuits primary-key lookups; a JPQL query always runs, though its results map back to instances already in the context. The practical rule is in batch jobs: we flush and clear every fifty records, otherwise the context keeps every entity plus its snapshot and we run out of heap."

---

# 34. Entity Lifecycle States

## 34.1 The four states

Every entity is in exactly one of four states. Understanding them explains most "why didn't my change save?" problems.

```
                     ┌──────────────┐
                     │  TRANSIENT   │   new Order()
                     │              │   - not in persistence context
                     │  (new)       │   - no DB row
                     └──────┬───────┘   - no ID
                            │
                    persist() / save()
                            │
                            ▼
                     ┌──────────────┐
      find() ───────►│   MANAGED    │◄────── merge()
      query  ───────►│              │
                     │ (persistent) │   - IN the persistence context
                     │              │   - has a DB row (or will at flush)
                     │              │   - ★ DIRTY CHECKING ACTIVE ★
                     └──┬────────┬──┘
                        │        │
              detach()  │        │  remove()
              clear()   │        │
              tx ends   │        │
                        ▼        ▼
              ┌──────────────┐  ┌──────────────┐
              │   DETACHED   │  │   REMOVED    │
              │              │  │              │
              │ - has an ID  │  │ - scheduled  │
              │ - row exists │  │   for DELETE │
              │ - NOT tracked│  │ - still in   │
              │ - NO dirty   │  │   context    │
              │   checking   │  │   until flush│
              └──────────────┘  └──────────────┘
```

## 34.2 Each state explained

### 1. TRANSIENT (New)
```java
Order order = new Order();           // TRANSIENT
order.setCustomerId("CUST-1");
// Not in the persistence context. No database row. ID is null.
// If garbage collected now, nothing is lost — it was never persisted.
```

### 2. MANAGED (Persistent)
```java
@Transactional
public void demo() {
    Order order = orderRepository.save(new Order());   // TRANSIENT → MANAGED
    Order found = orderRepository.findById(1L).get();  // loaded as MANAGED

    found.setStatus(OrderStatus.SHIPPED);   // ★ dirty checking will UPDATE this
}
```
**Only managed entities get dirty checking.** This is the state that matters most.

### 3. DETACHED
```java
@Transactional
public Order load(Long id) {
    return orderRepository.findById(id).orElseThrow();   // MANAGED here
}   // ← transaction ends, persistence context closes

public void caller() {
    Order order = load(1L);            // now DETACHED
    order.setStatus(OrderStatus.SHIPPED);
    // ★ NOTHING HAPPENS. No UPDATE. The change is silently lost.
}
```
**This is the single most common JPA bug.** The object looks fine, the setter works, and nothing reaches the database.

**To persist a detached entity's changes:**
```java
@Transactional
public void update(Order detachedOrder) {
    Order managed = entityManager.merge(detachedOrder);   // returns a MANAGED copy
    // IMPORTANT: detachedOrder itself stays detached.
    // Work with 'managed' from here on.
}
```

### 4. REMOVED
```java
@Transactional
public void delete(Long id) {
    Order order = orderRepository.findById(id).orElseThrow();   // MANAGED
    orderRepository.delete(order);       // → REMOVED (still in the context)
    // DELETE is issued at flush/commit, not immediately.
}
```

## 34.3 State transition reference

| From | Operation | To |
|---|---|---|
| Transient | `persist()` / `save()` | Managed |
| Transient | `merge()` | Managed (a **copy**) |
| Managed | `remove()` | Removed |
| Managed | `detach()` / `clear()` / transaction end | Detached |
| Managed | `refresh()` | Managed (reloaded from DB) |
| Detached | `merge()` | Managed (a **copy** is returned) |
| Removed | `persist()` | Managed (undoes the delete) |

## 34.4 `persist()` vs `merge()` vs `save()`

| | `persist()` | `merge()` | `save()` (Spring Data) |
|---|---|---|---|
| Defined by | JPA | JPA | Spring Data |
| Intended for | **New** entities | **Detached** entities | Both |
| Returns | `void` | **A managed copy** | The saved entity |
| Makes the argument managed | **Yes** | **No** — returns a different instance | Depends |
| If the entity exists | Throws `EntityExistsException` | Updates it | Updates it |
| SELECT before write | No | **Yes** (to load current state) | Yes when merging |

**How `save()` actually works in Spring Data JPA:**

```java
// SimpleJpaRepository.save(), simplified
@Transactional
public <S extends T> S save(S entity) {
    if (entityInformation.isNew(entity)) {     // usually: is the ID null?
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);               // ← note: returns a DIFFERENT instance
    }
}
```

**The trap this creates:**
```java
// ❌ WRONG — for an existing entity, save() returns a different managed instance
orderRepository.save(detachedOrder);
System.out.println(detachedOrder.getVersion());   // stale — not updated

// ✅ CORRECT — always use the returned value
Order saved = orderRepository.save(detachedOrder);
System.out.println(saved.getVersion());
```
**Always use the return value of `save()`.** It costs nothing and avoids a genuinely confusing bug.

## 34.5 Real scenario — the detached-entity bug

```java
// ❌ BROKEN
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;

    public Order getOrder(Long id) {                 // NO @Transactional
        return orderRepository.findById(id).orElseThrow();
    }   // Spring Data opens a transaction for findById and closes it → DETACHED

    public void shipOrder(Long id) {                 // NO @Transactional
        Order order = getOrder(id);
        order.setStatus(OrderStatus.SHIPPED);        // detached → no dirty checking
        // NOTHING IS SAVED. No error. Status unchanged in the database.
    }
}

// ✅ FIXED
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;

    @Transactional                                    // one transaction spans the whole method
    public void shipOrder(Long id) {
        Order order = orderRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Order", id));
        order.setStatus(OrderStatus.SHIPPED);         // MANAGED → dirty checking applies
        // UPDATE issued automatically at commit. No save() needed.
    }
}
```

**The lesson, stated plainly:** `@Transactional` on the service method is what keeps entities managed across the whole unit of work. Without it, every repository call is its own micro-transaction and everything you hold afterwards is detached.

## 34.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| **Modifying a detached entity** | Change silently lost — no error |
| Ignoring `merge()`'s return value | Working with the still-detached instance |
| Ignoring `save()`'s return value | Stale ID/version |
| `persist()` on a detached entity | `EntityExistsException` |
| Accessing a lazy field after detachment | `LazyInitializationException` |
| Missing `@Transactional` on multi-step service methods | Entities detached between calls; no atomicity |

## 34.7 TL Questions

**Q: What are the entity states?**
A: Transient — a new object not known to Hibernate; Managed — tracked in the persistence context with dirty checking active; Detached — was managed but the context closed; and Removed — scheduled for deletion.

**Q: Why do changes to a detached entity get lost?**
A: Dirty checking only applies to managed entities. Once the persistence context closes, Hibernate is no longer tracking that object, so setters just change memory. There's no error, which is what makes it a hard bug to spot.

**Q: How do we reattach a detached entity?**
A: `merge()`, which copies its state into a managed instance and returns that instance. The important detail is that the original object stays detached — we must use the returned one.

**Q: What's the difference between `persist()` and `merge()`?**
A: `persist()` is for new entities and makes the passed object managed. `merge()` is for detached entities, does a SELECT to load current state, and returns a **different** managed instance. Spring Data's `save()` picks between them based on whether the ID is null.

**Q: What happens if we forget `@Transactional` on a service method?**
A: Each repository call runs in its own short transaction, so anything we hold afterwards is detached and modifications are lost. We also lose atomicity — a failure halfway through leaves earlier writes committed.

**Q: Why should we always use `save()`'s return value?**
A: Because for an existing entity it delegates to `merge()`, which returns a different managed instance. The object we passed in keeps the old ID and version, which produces confusing bugs.

## 34.8 Interview Questions

**Beginner — Name the four entity states.**
Transient, Managed (persistent), Detached, Removed.

**Beginner — Which state has dirty checking?**
Managed only.

**Intermediate — What does `merge()` return and why does it matter?**
A managed copy of the entity — a **different object** from the one passed in. The argument remains detached, so subsequent changes to it are ignored. Always use the returned reference.

**Intermediate — Can a removed entity be brought back?**
Yes — calling `persist()` on it before the flush cancels the scheduled deletion and returns it to the managed state.

**Advanced — Why does `merge()` issue a SELECT?**
It must load the current database state into the persistence context before copying the detached values in, so Hibernate can compute the correct UPDATE and enforce optimistic-locking checks. `persist()` needs no SELECT because the row doesn't exist yet.

**Advanced — How does Spring Data decide new vs existing in `save()`?**
Via `EntityInformation.isNew()`. By default it checks whether the `@Id` is null (or zero for primitives). For an entity with an assigned ID this misfires, so you implement `Persistable` and override `isNew()`, or use a `@Version` field, which Spring Data also consults.

## 34.9 Quick Revision

1. Four states: **Transient → Managed → Detached / Removed**.
2. **Only MANAGED entities get dirty checking.**
3. Detached modifications are **silently lost** — the #1 JPA bug.
4. `persist()` for new; `merge()` for detached; `save()` chooses.
5. **`merge()` returns a different instance** — use the return value.
6. **Always use `save()`'s return value.**
7. `@Transactional` on the service keeps entities managed across the unit of work.
8. Accessing lazy fields when detached → `LazyInitializationException`.
9. `remove()` marks for deletion; the DELETE runs at flush.

## 34.10 TL Explanation (speak this)

> "Entities move through four states — transient before we save them, managed while they're in the persistence context, detached once the transaction ends, and removed when scheduled for deletion. The one that causes real bugs is detached: dirty checking only works on managed entities, so setting a field on a detached object changes nothing in the database and throws no error. That's why our service methods carry `@Transactional` — it keeps the entity managed for the whole unit of work, so we can load it, modify it, and let Hibernate write the update at commit without calling `save()` at all. And when we do call `save()` or `merge()` on a detached entity we always use the returned instance, because those return a different object from the one we passed in."

---

# 35. Spring Data Repositories

## 35.1 What is it?

**Simple words:**
You write an **interface** with no implementation, and Spring Data **creates the implementation for you at runtime**. You never write a repository class.

**Technical wording:**
Spring Data JPA generates proxy implementations of repository interfaces at runtime, deriving queries from method names and delegating to `SimpleJpaRepository`, which wraps the `EntityManager`.

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByCustomerId(String customerId);
}
// No implementation class. Spring writes it.
```

## 35.2 The repository hierarchy

```
        Repository<T, ID>                    ← marker interface, no methods
                │
        CrudRepository<T, ID>                ← save, findById, findAll, delete, count
                │
   PagingAndSortingRepository<T, ID>         ← + findAll(Pageable), findAll(Sort)
                │
     ┌──────────┴──────────┐
     │                      │
ListCrudRepository     JpaRepository<T, ID>  ← + flush, saveAndFlush, deleteInBatch,
  (Boot 3+)                                     getReferenceById, JPA-specific methods
```

| Interface | Adds |
|---|---|
| `Repository` | Nothing — just marks the interface for scanning |
| `CrudRepository` | `save`, `saveAll`, `findById`, `existsById`, `findAll`, `count`, `deleteById`, `delete` |
| `PagingAndSortingRepository` | `findAll(Pageable)`, `findAll(Sort)` |
| **`JpaRepository`** | `flush()`, `saveAndFlush()`, `deleteAllInBatch()`, `getReferenceById()`, and returns `List` instead of `Iterable` |

### `JpaRepository` vs `CrudRepository` — common interview question

| Aspect | `CrudRepository` | `JpaRepository` |
|---|---|---|
| Technology | Any Spring Data store (Mongo, Redis, JPA…) | **JPA-specific** |
| `findAll()` returns | `Iterable<T>` | **`List<T>`** (more convenient) |
| Paging/sorting | No | Yes (inherits it) |
| Batch delete | No | `deleteAllInBatch()` |
| Flush control | No | `flush()`, `saveAndFlush()` |
| Portability | Higher | Lower (tied to JPA) |
| **Use when** | Store-agnostic code | **Standard choice for JPA projects** |

**Practical answer:** use `JpaRepository` unless you have a specific reason to stay store-agnostic. The convenience of `List` returns and batch operations outweighs a portability that almost never gets exercised.

**Note on `deleteAllInBatch()`:** it issues a single `DELETE FROM orders` rather than loading entities and deleting one by one — far faster, but it **skips cascade and lifecycle callbacks**, so use it knowingly.

## 35.3 How Spring Data generates the implementation (internal)

```
1. @EnableJpaRepositories (auto-enabled by Boot) scans for Repository sub-interfaces
              │
              ▼
2. For each, JpaRepositoryFactoryBean creates a PROXY
              │
              ▼
3. For every method, decide how to handle it:
              │
    ┌─────────┼─────────────────┬───────────────────────┐
    ▼         ▼                 ▼                       ▼
 Inherited  Has @Query?    Name parseable?        Custom impl
 CRUD       use that       DERIVE the query       (XxxRepositoryImpl)
    │       JPQL/SQL             │                      │
    ▼                            ▼                      ▼
 delegate to             PartTree parses:         delegate to
 SimpleJpaRepository     findBy|CustomerId|And|   your class
                         Status|OrderBy...
              │
              ▼
4. If a method name cannot be parsed → STARTUP FAILURE
   "No property 'custmerId' found for type Order"
```

**Why startup failure is a good thing:** a typo in a derived method name breaks the deployment rather than a live request. Fail fast, again.

## 35.4 Derived query methods

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // ---- Simple ----
    List<Order> findByCustomerId(String customerId);
    Optional<Order> findByOrderNumber(String orderNumber);

    // ---- Multiple conditions ----
    List<Order> findByCustomerIdAndStatus(String customerId, OrderStatus status);
    List<Order> findByStatusOrStatus(OrderStatus s1, OrderStatus s2);

    // ---- Comparison ----
    List<Order> findByTotalAmountGreaterThan(BigDecimal amount);
    List<Order> findByTotalAmountBetween(BigDecimal min, BigDecimal max);
    List<Order> findByCreatedAtAfter(LocalDateTime date);

    // ---- String matching ----
    List<Order> findByCustomerIdContaining(String part);
    List<Order> findByCustomerIdStartingWith(String prefix);
    List<Order> findByCustomerIdIgnoreCase(String customerId);

    // ---- Null checks ----
    List<Order> findByCancelledAtIsNull();
    List<Order> findByCancelledAtIsNotNull();

    // ---- Collections ----
    List<Order> findByStatusIn(List<OrderStatus> statuses);

    // ---- Sorting and limiting ----
    List<Order> findByStatusOrderByCreatedAtDesc(OrderStatus status);
    List<Order> findTop10ByStatusOrderByTotalAmountDesc(OrderStatus status);
    Optional<Order> findFirstByCustomerIdOrderByCreatedAtDesc(String customerId);

    // ---- Counting / existence / deletion ----
    long countByStatus(OrderStatus status);
    boolean existsByOrderNumber(String orderNumber);
    void deleteByStatus(OrderStatus status);

    // ---- Nested property traversal (note the underscore) ----
    List<Order> findByCustomer_Address_City(String city);

    // ---- Paging ----
    Page<Order> findByStatus(OrderStatus status, Pageable pageable);
    Slice<Order> findByCustomerId(String customerId, Pageable pageable);
}
```

**Keyword reference:**

| Keyword | SQL equivalent |
|---|---|
| `And`, `Or` | `AND`, `OR` |
| `Is`, `Equals` | `=` |
| `Between` | `BETWEEN` |
| `LessThan`, `GreaterThan`, `...Equal` | `<`, `>`, `<=`, `>=` |
| `After`, `Before` | `>`, `<` (dates) |
| `IsNull`, `IsNotNull` | `IS NULL`, `IS NOT NULL` |
| `Like`, `NotLike` | `LIKE` |
| `StartingWith`, `EndingWith`, `Containing` | `LIKE 'x%'`, `'%x'`, `'%x%'` |
| `In`, `NotIn` | `IN`, `NOT IN` |
| `True`, `False` | `= true`, `= false` |
| `IgnoreCase` | `UPPER(x) = UPPER(?)` |
| `OrderBy...Asc/Desc` | `ORDER BY` |
| `Top`, `First` | `LIMIT` |
| `Distinct` | `DISTINCT` |

**When to stop using derived queries:** once a method name exceeds ~4 conditions, it becomes unreadable:
```java
// ❌ Technically valid, practically terrible
List<Order> findByCustomerIdAndStatusAndTotalAmountGreaterThanAndCreatedAtBetweenOrderByCreatedAtDesc(...);
```
Switch to `@Query` or a Specification. **Readability is the deciding factor** — a good point to make in a code review discussion.

## 35.5 Custom repository implementation

For logic that can't be expressed declaratively:

```java
// 1. The custom interface
public interface OrderRepositoryCustom {
    List<Order> searchWithDynamicFilters(OrderSearchCriteria criteria);
}

// 2. The implementation — name MUST end with "Impl"
@RequiredArgsConstructor
public class OrderRepositoryCustomImpl implements OrderRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<Order> searchWithDynamicFilters(OrderSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Order> query = cb.createQuery(Order.class);
        Root<Order> root = query.from(Order.class);

        List<Predicate> predicates = new ArrayList<>();
        if (criteria.getStatus() != null) {
            predicates.add(cb.equal(root.get("status"), criteria.getStatus()));
        }
        if (criteria.getMinAmount() != null) {
            predicates.add(cb.greaterThanOrEqualTo(root.get("totalAmount"),
                                                   criteria.getMinAmount()));
        }
        query.where(predicates.toArray(new Predicate[0]))
             .orderBy(cb.desc(root.get("createdAt")));

        return entityManager.createQuery(query).getResultList();
    }
}

// 3. Extend BOTH interfaces
public interface OrderRepository extends JpaRepository<Order, Long>, OrderRepositoryCustom { }
```
**The `Impl` suffix is mandatory** — Spring Data looks for `<CustomInterfaceName>Impl`. Getting the name wrong gives a confusing "no bean found" style failure at startup.

## 35.6 TL Questions

**Q: How does Spring Data create the implementation?**
A: At startup it scans for repository interfaces and creates a proxy for each. Inherited CRUD methods delegate to `SimpleJpaRepository`; methods with `@Query` use that query; and other method names are parsed into queries. If a name can't be parsed, startup fails.

**Q: `JpaRepository` or `CrudRepository`?**
A: `JpaRepository` for our JPA projects — it returns `List` instead of `Iterable`, includes paging and sorting, and adds `flush` and batch deletes. `CrudRepository` only matters if we want the code to be portable across Spring Data stores, which we don't in practice.

**Q: What happens if we misspell a property in a method name?**
A: The application fails to start with a message naming the unknown property. That's deliberate — it catches the typo at deploy time instead of on a live request.

**Q: When do we stop using derived queries?**
A: Around four conditions. Beyond that the method name becomes unreadable, so we switch to `@Query` with JPQL, or a Specification for genuinely dynamic filters.

**Q: How do we write a query that can't be expressed by a method name?**
A: Either `@Query` with JPQL or native SQL, or a custom repository fragment — an interface plus an `Impl` class using the `EntityManager` and the Criteria API, which our main repository interface also extends.

## 35.7 Interview Questions

**Beginner — Do we write repository implementations?**
No — Spring Data generates them at runtime from the interface.

**Intermediate — What is `SimpleJpaRepository`?**
The default implementation backing all standard CRUD methods. It wraps the `EntityManager` and is annotated `@Transactional(readOnly = true)` at class level, with write methods overriding that.

**Intermediate — What does `deleteAllInBatch()` do differently?**
It issues one bulk `DELETE` statement instead of loading each entity and deleting it individually. Much faster, but it bypasses cascading and lifecycle callbacks and doesn't update the persistence context.

**Advanced — How are repository methods transactional by default?**
`SimpleJpaRepository` is annotated `@Transactional(readOnly = true)`, and mutating methods like `save` and `delete` are annotated `@Transactional`. So a single repository call always runs in a transaction — which is exactly why an entity returned from a repository call outside a service transaction comes back detached.

**Advanced — What is the fragment/`Impl` mechanism?**
Spring Data composes a repository from fragments. For each extended interface it looks for a class named `<Interface>Impl` and wires it into the proxy, letting you mix generated methods with hand-written ones in a single repository.

## 35.8 Quick Revision

1. Declare an interface; Spring Data generates the implementation.
2. Hierarchy: `Repository` → `CrudRepository` → `PagingAndSortingRepository` → `JpaRepository`.
3. Use **`JpaRepository`** in JPA projects.
4. Query sources: inherited CRUD, derived names, `@Query`, custom `Impl`.
5. Unparseable method names **fail at startup**.
6. Keep derived names to ~4 conditions; beyond that use `@Query`.
7. Custom fragment class **must** be named `<Interface>Impl`.
8. `deleteAllInBatch()` is fast but skips cascades and callbacks.
9. Repository methods are transactional by default (`readOnly` for reads).

## 35.9 TL Explanation (speak this)

> "We only declare repository interfaces — Spring Data generates the implementations at startup. Standard CRUD comes from `JpaRepository`, and anything else is either a derived query from the method name or an explicit `@Query`. A nice property is that a typo in a derived method name fails the startup rather than a live request. We keep derived names short, about four conditions, and move to `@Query` or a Specification beyond that, because the generated names become unreadable. For genuinely dynamic filtering we add a custom fragment with the Criteria API."

---

# 36. Query Methods, JPQL, @Query and Pagination

## 36.1 `@Query` — JPQL

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // JPQL works on ENTITY names and FIELD names, not tables and columns
    @Query("SELECT o FROM Order o WHERE o.status = :status AND o.totalAmount > :amount")
    List<Order> findHighValueOrders(@Param("status") OrderStatus status,
                                    @Param("amount") BigDecimal amount);

    // Join + fetch join (solves N+1 — see Section 40)
    @Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items WHERE o.status = :status")
    List<Order> findWithItems(@Param("status") OrderStatus status);

    // DTO projection via constructor expression — selects ONLY these columns
    @Query("""
           SELECT new com.company.orderservice.dto.OrderSummaryDto(
                  o.id, o.orderNumber, o.totalAmount, o.status)
           FROM Order o WHERE o.customerId = :customerId
           """)
    List<OrderSummaryDto> findSummaries(@Param("customerId") String customerId);

    // Aggregate
    @Query("SELECT SUM(o.totalAmount) FROM Order o WHERE o.customerId = :customerId")
    BigDecimal getTotalSpend(@Param("customerId") String customerId);

    // Native SQL — when you need vendor features
    @Query(value = "SELECT * FROM orders WHERE MATCH(notes) AGAINST(:text)",
           nativeQuery = true)
    List<Order> fullTextSearch(@Param("text") String text);

    // Modifying query
    @Modifying(clearAutomatically = true, flushAutomatically = true)
    @Transactional
    @Query("UPDATE Order o SET o.status = :status WHERE o.id IN :ids")
    int bulkUpdateStatus(@Param("status") OrderStatus status, @Param("ids") List<Long> ids);
}
```

### JPQL vs SQL — the distinction that matters

| | JPQL | Native SQL |
|---|---|---|
| Operates on | **Entities and fields** | Tables and columns |
| `SELECT o FROM Order o` | `Order` = the **entity class** | `orders` = the table |
| Database portable | **Yes** | No |
| Vendor features | No | **Yes** |
| Validated at startup | **Yes** | No |
| Returns | Managed entities | Entities (with `nativeQuery`) or raw rows |

**JPQL is validated when the application starts** — a syntax error or wrong field name fails the deployment. Native SQL isn't checked until it runs. That's a meaningful argument for preferring JPQL wherever it suffices.

### `@Modifying` — three things that must be right

```java
@Modifying(clearAutomatically = true, flushAutomatically = true)
@Transactional
@Query("UPDATE Order o SET o.status = :status WHERE o.id IN :ids")
int bulkUpdateStatus(...);
```

1. **`@Modifying`** is required for UPDATE/DELETE — without it you get `QueryExecutionRequestException`.
2. **`@Transactional`** is required (better placed on the calling service method).
3. **`clearAutomatically = true`** matters: a bulk UPDATE goes **straight to the database and bypasses the persistence context**, so entities already loaded hold stale values. Clearing the context after forces a reload. `flushAutomatically = true` pushes pending changes *before* the bulk statement so they aren't lost.

**This is a genuine production bug source** — a bulk update followed by reading a previously-loaded entity returns the old value.

## 36.2 Pagination

**Why it is non-negotiable:** `findAll()` on a table with 5 million rows loads 5 million objects into heap and kills the service. Every collection endpoint must be paginated.

```java
// Repository
Page<Order> findByStatus(OrderStatus status, Pageable pageable);

// Service
@Transactional(readOnly = true)
public Page<OrderResponseDto> getOrders(OrderStatus status, Pageable pageable) {
    return orderRepository.findByStatus(status, pageable)
                          .map(orderMapper::toDto);      // Page.map preserves metadata
}

// Controller
@GetMapping
public ResponseEntity<Page<OrderResponseDto>> getOrders(
        @RequestParam(required = false) OrderStatus status,
        @PageableDefault(size = 20, sort = "createdAt",
                         direction = Sort.Direction.DESC) Pageable pageable) {
    return ResponseEntity.ok(orderService.getOrders(status, pageable));
}
// Client calls: GET /api/v1/orders?page=0&size=20&sort=createdAt,desc
```

### `Page` vs `Slice` vs `List`

| | `List` | `Slice` | `Page` |
|---|---|---|---|
| Content | All results | One page | One page |
| Knows if there's a next page | No | **Yes** | Yes |
| Total element count | No | **No** | **Yes** |
| Extra COUNT query | No | **No** | **Yes** |
| Best for | Small fixed sets | **Infinite scroll / "load more"** | **Numbered pagination** |

**Performance point worth raising:** `Page` runs a second `SELECT COUNT(*)` to compute the total. On a large table with a complex WHERE clause that count can be **slower than the data query itself**. If the UI only needs "next", use `Slice` and avoid the count entirely.

### The deep-pagination problem (Additional clarification)

`LIMIT 20 OFFSET 1000000` forces the database to scan and discard a million rows. Page 50,000 is dramatically slower than page 1. For deep pagination use **keyset (cursor) pagination**:

```java
@Query("SELECT o FROM Order o WHERE o.createdAt < :cursor ORDER BY o.createdAt DESC")
List<Order> findNextPage(@Param("cursor") LocalDateTime cursor, Pageable pageable);
// Uses the index directly — constant time regardless of depth
```

### Sorting

```java
Sort sort = Sort.by("createdAt").descending().and(Sort.by("totalAmount"));
Pageable pageable = PageRequest.of(0, 20, sort);
```
**Security caution:** never pass a raw client string into a native-SQL `ORDER BY` — it's an injection vector. With `Sort` on JPQL, Spring Data validates the property against the entity, so an invalid field throws rather than injecting.

## 36.3 Specifications — type-safe dynamic queries

```java
public class OrderSpecifications {

    public static Specification<Order> hasStatus(OrderStatus status) {
        return (root, query, cb) ->
                status == null ? null : cb.equal(root.get("status"), status);
    }

    public static Specification<Order> amountGreaterThan(BigDecimal amount) {
        return (root, query, cb) ->
                amount == null ? null : cb.greaterThan(root.get("totalAmount"), amount);
    }
}

// Repository must extend JpaSpecificationExecutor
public interface OrderRepository extends JpaRepository<Order, Long>,
                                          JpaSpecificationExecutor<Order> { }

// Usage — null specs are ignored, so filters compose cleanly
Specification<Order> spec = Specification
        .where(OrderSpecifications.hasStatus(criteria.getStatus()))
        .and(OrderSpecifications.amountGreaterThan(criteria.getMinAmount()));

Page<Order> results = orderRepository.findAll(spec, pageable);
```
**Why this beats string concatenation:** it's type-safe, composable, injection-proof, and each filter is independently testable. It is the right tool for a search screen with many optional filters.

## 36.4 Common Mistakes

| Mistake | Consequence |
|---|---|
| `findAll()` on a large table | Memory exhaustion |
| Missing `@Modifying` on UPDATE/DELETE | `QueryExecutionRequestException` |
| Missing `clearAutomatically` after a bulk update | Stale entities in the persistence context |
| `Page` when the count query is expensive | Slow endpoint — use `Slice` |
| Deep `OFFSET` pagination | Degrades badly at depth — use keyset |
| Native SQL where JPQL would do | Loses startup validation and portability |
| Client-supplied `ORDER BY` in native SQL | **SQL injection** |
| `JOIN FETCH` combined with pagination | Hibernate warns and paginates **in memory** — see Section 40 |

## 36.5 TL Questions

**Q: When do we use `@Query` instead of a derived method?**
A: When the method name would become unreadable — roughly beyond four conditions — or when we need a join fetch, an aggregate, or a DTO projection. JPQL keeps it explicit and readable.

**Q: JPQL or native SQL?**
A: JPQL by default, because it's validated at startup and portable across databases. Native SQL only when we need vendor-specific features like full-text search.

**Q: Why do bulk updates need `clearAutomatically`?**
A: A bulk UPDATE bypasses the persistence context and goes straight to the database, so any entity already loaded holds stale values. Clearing the context forces a reload on next access. Without it we get a subtle bug where the object disagrees with the database.

**Q: `Page` or `Slice`?**
A: `Page` when the UI shows numbered pages and needs the total. `Slice` for infinite scroll, because it skips the count query — and that count can be slower than the data query on a large filtered table.

**Q: How do we handle a search screen with ten optional filters?**
A: Specifications. Each filter is a small composable `Specification` that returns null when the value is absent, so we build the query dynamically without string concatenation. It's type-safe and injection-proof.

**Q: What's wrong with `page=50000`?**
A: `OFFSET` makes the database scan and discard all preceding rows, so deep pages get progressively slower. For deep pagination we'd use keyset pagination with a cursor on an indexed column, which stays constant-time.

## 36.6 Interview Questions

**Beginner — What is JPQL?**
JPA's query language, operating on entity names and field names rather than tables and columns, making it database-portable.

**Intermediate — What does `@Modifying` do?**
Tells Spring Data the query modifies data rather than selecting, so it calls `executeUpdate()` instead of `getResultList()`. Required for UPDATE and DELETE queries.

**Intermediate — Why does `Page` need two queries?**
One for the page content and a second `SELECT COUNT(*)` to compute total elements and total pages.

**Advanced — What is a constructor expression and why use it?**
`SELECT new com.company.dto.OrderSummaryDto(o.id, o.status) FROM Order o` — it instantiates DTOs directly in the query. The generated SQL selects only those columns, and no entities enter the persistence context, so there's no dirty-check overhead. It's the efficient choice for read-only list endpoints.

**Advanced — What is `JpaSpecificationExecutor`?**
An interface adding `findAll(Specification)`, `count(Specification)` and paged variants. A `Specification` wraps a Criteria API `Predicate`, so filters are composable with `and`/`or` and built type-safely at runtime.

## 36.7 Quick Revision

1. `@Query` for anything beyond a simple derived method.
2. JPQL uses **entity and field names**; it is validated at startup and portable.
3. Native SQL only for vendor-specific features.
4. `@Modifying` + `@Transactional` for UPDATE/DELETE; add `clearAutomatically`.
5. **Bulk updates bypass the persistence context** — entities go stale.
6. Always paginate collection endpoints.
7. `Page` = with count; `Slice` = no count, cheaper; use `Slice` for infinite scroll.
8. Deep `OFFSET` is slow — keyset pagination for depth.
9. Specifications = type-safe, composable dynamic filters.
10. Constructor expressions select only the columns you need.
11. Never interpolate client input into a native `ORDER BY`.

## 36.8 TL Explanation (speak this)

> "Simple lookups use derived method names, and anything more complex uses `@Query` with JPQL — which has the advantage of being validated at application startup, so a bad field name fails the deployment rather than a live request. Every collection endpoint is paginated; we use `Page` where the UI needs a total and `Slice` for infinite scroll, because `Page` runs an extra count query that can be slower than the data query itself. For search screens with many optional filters we use Specifications, which compose type-safely instead of concatenating SQL. The one thing we're careful about is bulk updates — they bypass the persistence context, so we clear it afterwards or previously-loaded entities go stale."

---

# 37. Entity Relationships

## 37.1 The four relationship types

| Annotation | Meaning | Example |
|---|---|---|
| `@OneToOne` | One row relates to exactly one row | User ↔ UserProfile |
| `@OneToMany` | One row relates to many rows | Order → OrderItems |
| `@ManyToOne` | Many rows relate to one row | OrderItem → Order |
| `@ManyToMany` | Many relate to many | Student ↔ Course |

**Key concept — the owning side:**
In a relational database, the **foreign key lives in one table**. The entity holding that foreign key is the **owning side**. The other side is the **inverse side**, marked with `mappedBy`.

> **Only changes to the owning side are written to the database.** Setting the inverse side alone changes nothing — a frequent and confusing bug.

## 37.2 @ManyToOne and @OneToMany (the most common pair)

```java
// ---------- MANY side — OWNS the relationship (holds the FK) ----------
@Entity
@Table(name = "order_items")
@Getter @Setter @NoArgsConstructor
public class OrderItem {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)        // ★ ALWAYS set LAZY explicitly
    @JoinColumn(name = "order_id", nullable = false)   // this is the FK column
    private Order order;                                // ← OWNING SIDE

    private String productName;
    private Integer quantity;
    private BigDecimal price;
}

// ---------- ONE side — INVERSE (no FK column) ----------
@Entity
@Table(name = "orders")
@Getter @Setter @NoArgsConstructor
public class Order {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "order",             // ← the FIELD NAME in OrderItem
               cascade = CascadeType.ALL,
               orphanRemoval = true,
               fetch = FetchType.LAZY)         // default for @OneToMany, but be explicit
    private List<OrderItem> items = new ArrayList<>();   // initialise to avoid NPE

    // ★ HELPER METHODS — keep both sides of the relationship in sync
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);          // CRITICAL: set the owning side too
    }

    public void removeItem(OrderItem item) {
        items.remove(item);
        item.setOrder(null);          // with orphanRemoval=true this deletes the row
    }
}
```

### Why the helper methods matter

```java
// ❌ WRONG — only the inverse side is set
Order order = new Order();
OrderItem item = new OrderItem();
order.getItems().add(item);          // inverse side only
orderRepository.save(order);
// The order_id column in order_items is NULL, or the row isn't inserted at all.
// Why? Hibernate reads the FK from the OWNING side (item.order), which is null.

// ✅ CORRECT
order.addItem(item);                 // sets BOTH sides
orderRepository.save(order);
```

**This is one of the most common JPA bugs in real projects.** Always provide `addX`/`removeX` helpers on the inverse side, and make it a code-review rule.

`mappedBy = "order"` means: *"I am not the owner. The `order` field in `OrderItem` owns this relationship."* Without it, Hibernate assumes both sides are independent and creates an unnecessary **join table**.

## 37.3 @OneToOne

```java
@Entity
public class User {
    @Id @GeneratedValue private Long id;

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL,
              fetch = FetchType.LAZY, orphanRemoval = true)
    private UserProfile profile;                 // inverse side
}

@Entity
public class UserProfile {
    @Id @GeneratedValue private Long id;

    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", unique = true)
    private User user;                           // OWNING side — holds the FK
}
```

**The `@OneToOne` lazy-loading catch (a strong senior-level point):**
Lazy loading on the **inverse** side of a `@OneToOne` **does not work** by default. Hibernate must know whether to create a proxy or set `null`, and the only way to know is to query — so it queries eagerly regardless of the annotation.

Workarounds: make the association **optional = false** (if it's guaranteed to exist), use bytecode enhancement, or **map it as `@ManyToOne` with a unique constraint** — which is often the pragmatic choice.

## 37.4 @ManyToMany

```java
@Entity
public class Student {
    @Id @GeneratedValue private Long id;

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(name = "student_course",                          // join table
               joinColumns = @JoinColumn(name = "student_id"),
               inverseJoinColumns = @JoinColumn(name = "course_id"))
    private Set<Course> courses = new HashSet<>();               // Set, not List
}

@Entity
public class Course {
    @Id @GeneratedValue private Long id;

    @ManyToMany(mappedBy = "courses", fetch = FetchType.LAZY)
    private Set<Student> students = new HashSet<>();
}
```

**Use `Set`, not `List`, for `@ManyToMany`.** With a `List`, removing one element makes Hibernate **delete every join row and re-insert the remainder** — catastrophically inefficient on large collections. A `Set` deletes only the one row.

### The strong recommendation: replace `@ManyToMany` with two `@OneToMany`

```java
// Break the many-to-many into an explicit join ENTITY
@Entity
@Table(name = "student_course")
@Getter @Setter
public class Enrollment {

    @Id @GeneratedValue private Long id;

    @ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name = "student_id")
    private Student student;

    @ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name = "course_id")
    private Course course;

    // ★ Now we can store data ABOUT the relationship — impossible with @ManyToMany
    private LocalDate enrolledOn;
    private String grade;
    private EnrollmentStatus status;
}
```

**Why this is almost always better:** a pure `@ManyToMany` join table can hold *only* the two foreign keys. The moment the business asks "when did they enrol?" or "what grade did they get?" — and it always does — you must refactor to a join entity anyway. Starting with the entity avoids a painful migration later.

**This is an excellent point to raise with a TL**, because it's a design judgement rather than a syntax fact.

## 37.5 Bidirectional vs Unidirectional

| | Unidirectional | Bidirectional |
|---|---|---|
| Navigation | One direction only | Both directions |
| Complexity | Lower | Higher (sync both sides) |
| Jackson recursion risk | No | **Yes** — needs DTOs or `@JsonIgnore` |
| `equals`/`hashCode`/`toString` risk | Lower | **Higher** |
| **Recommendation** | **Prefer this** | Only when you genuinely navigate both ways |

**Advice:** make the association unidirectional unless you actually need to navigate from both ends. Bidirectional adds synchronisation burden and serialization hazards for a convenience you may never use. If you do need it, use DTOs (Section 21), which removes the recursion problem entirely.

## 37.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Setting only the inverse side | FK is null; relationship not persisted |
| Missing `mappedBy` on the inverse side | Unwanted extra join table |
| `EAGER` on collections | N+1 and over-fetching everywhere |
| `List` with `@ManyToMany` | Delete-all-and-reinsert on every change |
| Returning bidirectional entities as JSON | Infinite recursion |
| Uninitialised collection field | `NullPointerException` on `getItems().add(...)` |
| `CascadeType.ALL` on `@ManyToOne` | Deleting an item deletes the parent order |
| `@ManyToMany` where relationship attributes will be needed | Painful later refactor |

## 37.7 TL Questions

**Q: What is the owning side?**
A: The entity holding the foreign key column — normally the `@ManyToOne` side. Only changes to the owning side are written to the database, which is why setting just the inverse collection doesn't persist anything.

**Q: What does `mappedBy` do?**
A: It marks the inverse side and names the field on the owning side that holds the relationship. Without it Hibernate treats the two sides as separate relationships and creates an unnecessary join table.

**Q: Why do we write `addItem`/`removeItem` helpers?**
A: To keep both sides in sync. Adding to the collection alone leaves the foreign key null, so the child row is never linked. The helper sets both references in one call, which makes it hard to get wrong.

**Q: Why do we avoid `@ManyToMany`?**
A: Because a pure join table can only hold the two foreign keys. As soon as the business wants an enrolment date or a status on the relationship, we have to refactor to a join entity. Starting with an explicit join entity with two `@ManyToOne`s avoids that migration and gives us more control.

**Q: What happens if we return bidirectional entities as JSON?**
A: Jackson recurses infinitely — order serializes items, each item serializes its order, and so on until the stack overflows. DTOs solve it properly; `@JsonIgnore` patches it but leaves the entity exposed.

**Q: Why `Set` rather than `List` for many-to-many?**
A: With a `List`, removing one element makes Hibernate delete every join row and re-insert the rest. With a `Set` it deletes just the one row.

## 37.8 Interview Questions

**Beginner — Which side holds the foreign key?**
The owning side — usually `@ManyToOne`.

**Intermediate — What happens without `mappedBy`?**
Hibernate treats each side as its own unidirectional association and creates an extra join table, so the schema and the updates are both wrong.

**Intermediate — Why doesn't lazy loading work on `@OneToOne` inverse sides?**
Hibernate must decide between a proxy and `null`, and the only way to know which is to query. So it queries eagerly regardless of the fetch type. Mapping it as `@ManyToOne` with a unique constraint, or using bytecode enhancement, avoids it.

**Advanced — What's the difference between `orphanRemoval = true` and `CascadeType.REMOVE`?**
`CascadeType.REMOVE` deletes children when the **parent** is deleted. `orphanRemoval = true` additionally deletes a child when it is **removed from the parent's collection**, even if the parent survives. Orphan removal expresses a genuine parent-child ownership relationship.

**Advanced — Why is `@ManyToMany` with `List` inefficient?**
Hibernate cannot identify individual rows in a bag-semantics `List` without an index column, so on any modification it deletes all join rows for the owner and re-inserts the current contents. A `Set` has identity semantics and allows targeted deletes.

## 37.9 Quick Revision

1. Four types: `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`.
2. **Owning side holds the FK**; only its changes are persisted.
3. `mappedBy` marks the inverse side and names the owning field.
4. **Always write `addX`/`removeX` helpers to sync both sides.**
5. Initialise collection fields (`= new ArrayList<>()`).
6. `@ManyToMany` → use `Set`, and prefer a **join entity** instead.
7. Prefer unidirectional unless both directions are genuinely needed.
8. Bidirectional + entity JSON = infinite recursion → use DTOs.
9. `orphanRemoval` deletes children removed from the collection.
10. Never `CascadeType.ALL` on `@ManyToOne`.

## 37.10 TL Explanation (speak this)

> "The owning side of a relationship is whichever entity holds the foreign key — normally the `@ManyToOne` side — and only changes to that side get written. That's why we always have `addItem`/`removeItem` helpers on the parent that set both references; adding to the collection alone leaves the foreign key null and the link is never saved. We generally avoid `@ManyToMany` and model the join as its own entity with two `@ManyToOne`s, because a plain join table can't carry any attributes, and the business always ends up wanting a date or a status on the relationship. And we keep entities out of our JSON responses, which removes the bidirectional recursion problem entirely."

---

# 38. Cascade Types

## 38.1 What is it?

**Simple words:**
Cascading means **operations on a parent automatically apply to its children**. Save the order — its items are saved too. Delete the order — its items are deleted too.

**Technical wording:**
Cascade types define which `EntityManager` operations propagate from a parent entity to its associated entities along a relationship.

## 38.2 The cascade types

| Type | Propagates | Effect |
|---|---|---|
| `PERSIST` | `persist()` | Saving the parent saves new children |
| `MERGE` | `merge()` | Merging the parent merges children |
| `REMOVE` | `remove()` | Deleting the parent deletes children |
| `REFRESH` | `refresh()` | Reloading the parent reloads children |
| `DETACH` | `detach()` | Detaching the parent detaches children |
| **`ALL`** | All of the above | Convenient, but see the warning |

```java
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
private List<OrderItem> items = new ArrayList<>();

// Now:
Order order = new Order();
order.addItem(new OrderItem("Laptop", 1, new BigDecimal("50000")));
order.addItem(new OrderItem("Mouse",  2, new BigDecimal("500")));

orderRepository.save(order);
// ONE call inserts the order AND both items.
// Without cascade, we'd have to save each item separately.
```

## 38.3 `orphanRemoval` vs `CascadeType.REMOVE`

| | `CascadeType.REMOVE` | `orphanRemoval = true` |
|---|---|---|
| Deletes children when the **parent is deleted** | Yes | Yes |
| Deletes a child **removed from the collection** | **No** | **Yes** |
| Meaning | "Delete children with the parent" | "Children cannot exist without the parent" |

```java
order.getItems().remove(item);
// With CascadeType.REMOVE only → the item row survives with a null/stale FK
// With orphanRemoval = true    → DELETE FROM order_items WHERE id = ?
```

**Use `orphanRemoval = true` for true parent-child ownership** — order items, invoice lines, address books. Do **not** use it where the child has independent meaning.

## 38.4 Where cascade is right and where it is dangerous

### ✅ Correct: a genuine parent-child composition

```java
@Entity
public class Order {
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}
```
An `OrderItem` has no meaning without its `Order`. Deleting the order should delete its items. **Correct.**

### ❌ Dangerous: cascading to a shared, independent entity

```java
@Entity
public class Order {
    @ManyToOne(cascade = CascadeType.ALL)     // ❌ CATASTROPHIC
    @JoinColumn(name = "customer_id")
    private Customer customer;
}

// Consequence:
orderRepository.delete(order);
// → DELETES THE CUSTOMER, and cascades to all their other orders.
// One cancelled order wipes out a customer record.
```

**The rule to state plainly:**
> **Never cascade REMOVE (or ALL) from the `@ManyToOne` side.**
> The "many" side references a shared entity it does not own.

```java
// ✅ CORRECT — no cascade on @ManyToOne
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "customer_id", nullable = false)
private Customer customer;
```

### The ownership test

Before adding a cascade, ask: **"Does this child belong exclusively to this parent, and is it meaningless without it?"**

| Relationship | Exclusive? | Cascade? |
|---|---|---|
| Order → OrderItem | Yes | `ALL` + `orphanRemoval` |
| Invoice → InvoiceLine | Yes | `ALL` + `orphanRemoval` |
| User → UserProfile | Yes | `ALL` + `orphanRemoval` |
| Order → Customer | **No** — shared | **None** |
| OrderItem → Product | **No** — shared | **None** |
| Post → Comment | Usually yes | `ALL` + `orphanRemoval` |

## 38.5 Common Mistakes

| Mistake | Consequence |
|---|---|
| `CascadeType.ALL` on `@ManyToOne` | **Deleting a child deletes the shared parent** — data loss |
| Cascading to a shared reference entity (Product, Country) | Deleting one row removes reference data everywhere |
| No cascade where genuinely needed | `TransientObjectException` — "object references an unsaved transient instance" |
| `orphanRemoval` on a non-owned child | Unexpected deletions |
| Relying on cascade for bulk deletes | Hibernate loads every child and deletes one by one — slow |

**The `TransientObjectException` fix:**
```java
// Error: "object references an unsaved transient instance -
//         save the transient instance before flushing"
// Cause: saving a parent with a new, unsaved child and no CascadeType.PERSIST.
// Fix: add cascade = CascadeType.PERSIST (or ALL), or save the child first.
```

**Performance caution on cascaded deletes:** `CascadeType.REMOVE` loads **every** child into memory and issues one DELETE per row. Deleting an order with 10,000 items means 10,001 statements. For large volumes use a bulk `@Modifying` delete or a database-level `ON DELETE CASCADE`, accepting that lifecycle callbacks are then skipped.

## 38.6 TL Questions

**Q: What is cascading?**
A: Propagating an operation from a parent entity to its children — so saving an order also saves its items, and deleting it deletes them.

**Q: When do we use `CascadeType.ALL`?**
A: Only for true parent-child ownership where the child is meaningless without the parent — order items, invoice lines. Never from the `@ManyToOne` side to a shared entity.

**Q: What's the danger of cascade on `@ManyToOne`?**
A: The many side points at a shared entity it doesn't own. Cascading REMOVE from an order to its customer means deleting one order deletes the customer and cascades onward to all their other orders. It's a genuine data-loss scenario.

**Q: `orphanRemoval` versus `CascadeType.REMOVE`?**
A: `REMOVE` deletes children when the parent is deleted. `orphanRemoval` additionally deletes a child when it's removed from the parent's collection. We use orphan removal where the child genuinely can't exist independently.

**Q: What if we forget the cascade?**
A: Saving a parent with new unsaved children throws `TransientObjectException` — "object references an unsaved transient instance". Either add `CascadeType.PERSIST` or save the children first.

**Q: Is cascaded delete efficient?**
A: No, for large collections. Hibernate loads every child and issues one DELETE each, so deleting an order with ten thousand items is ten thousand statements. For that scale we use a bulk delete query or a database-level cascade.

## 38.7 Interview Questions

**Beginner — What does `CascadeType.PERSIST` do?**
Saving the parent also persists its new associated children.

**Intermediate — Why is `CascadeType.ALL` risky on `@ManyToOne`?**
Because it includes REMOVE, and the many side references a shared entity. Deleting one child would delete the shared parent and everything else attached to it.

**Advanced — What is `TransientObjectException` and when does it occur?**
It occurs at flush when a managed entity references an entity that has never been persisted and no `CascadeType.PERSIST` covers the association. Hibernate cannot write a foreign key to a row that doesn't exist.

**Advanced — Cascade vs database `ON DELETE CASCADE`?**
JPA cascade is application-level: Hibernate loads children and deletes them individually, so lifecycle callbacks and the persistence context stay consistent, but it's slow at volume. Database cascade is a single, fast server-side operation, but Hibernate is unaware of it, so the persistence context can hold entities for rows that no longer exist. Choose deliberately, and don't rely on both silently.

## 38.8 Quick Revision

1. Cascade propagates operations from parent to children.
2. Types: PERSIST, MERGE, REMOVE, REFRESH, DETACH, ALL.
3. **`ALL` + `orphanRemoval` only for true parent-child ownership.**
4. **Never cascade REMOVE/ALL from `@ManyToOne`.**
5. `orphanRemoval` also deletes children removed from the collection.
6. Ownership test: *is the child meaningless without this parent?*
7. Missing `PERSIST` cascade → `TransientObjectException`.
8. Cascaded deletes are row-by-row — slow at volume.

## 38.9 TL Explanation (speak this)

> "Cascading propagates operations from parent to child, so saving an order saves its items in one call. We only use `CascadeType.ALL` with `orphanRemoval` where the child genuinely can't exist without the parent — order items, invoice lines. The rule we never break is no cascade from the `@ManyToOne` side, because that side points at a shared entity: cascading remove from an order to its customer would delete the customer and all their other orders. And for very large child collections we don't rely on cascade at all, because Hibernate deletes row by row — we use a bulk delete instead."

---

# 39. Lazy vs Eager Loading

## 39.1 What is it?

**Simple words:**
When you load an `Order`, should Hibernate also load all its `OrderItem`s immediately?

- **EAGER** = yes, load everything now
- **LAZY** = no, load them only if the code actually asks for them

**Technical wording:**
`FetchType` determines whether an association is initialised at load time (EAGER) or deferred until first access via a proxy or collection wrapper (LAZY).

## 39.2 The defaults (memorise this table)

| Relationship | Default fetch | Is the default sensible? |
|---|---|---|
| `@OneToOne` | **EAGER** | ❌ No — override to LAZY |
| `@ManyToOne` | **EAGER** | ❌ No — override to LAZY |
| `@OneToMany` | **LAZY** | ✅ Yes |
| `@ManyToMany` | **LAZY** | ✅ Yes |

**The memory aid:** the annotations ending in **`ToOne` default to EAGER**; those ending in **`ToMany` default to LAZY**.

**The rule to apply everywhere:**
```java
@ManyToOne(fetch = FetchType.LAZY)      // ALWAYS write this explicitly
@OneToOne(fetch = FetchType.LAZY)       // ALWAYS write this explicitly
```

## 39.3 Why EAGER is harmful

```java
@Entity
public class OrderItem {
    @ManyToOne                    // EAGER by default!
    private Order order;
}

@Entity
public class Order {
    @ManyToOne                    // EAGER by default!
    private Customer customer;
}

@Entity
public class Customer {
    @ManyToOne                    // EAGER by default!
    private Address address;
}
```

Loading **one** `OrderItem` now triggers:
```sql
SELECT * FROM order_items WHERE id = 1;
SELECT * FROM orders      WHERE id = ?;    -- because of EAGER
SELECT * FROM customers   WHERE id = ?;    -- because of EAGER
SELECT * FROM addresses   WHERE id = ?;    -- because of EAGER
```
**Four queries to read one row** — and you may have needed only the product name.

**Worse:** EAGER is applied even when the query explicitly doesn't need it. `findAll()` on 100 order items would fire hundreds of extra queries. **EAGER cannot be turned off per query; LAZY can always be overridden with a fetch join.** That asymmetry is the core argument.

**State it this way to a TL:** *"LAZY is the safe default because we can always fetch more when we need it. EAGER is a decision baked into the mapping that every query pays for, whether it needs the data or not."*

## 39.4 How lazy loading works internally

```java
Order order = orderRepository.findById(1L).get();
// SELECT * FROM orders WHERE id = 1
// order.items is NOT a real ArrayList — it's a PersistentBag proxy (uninitialised)

System.out.println(order.getItems().size());   // ← FIRST ACCESS
// NOW: SELECT * FROM order_items WHERE order_id = 1
```

For `@ManyToOne`, Hibernate creates a **proxy subclass** of the target entity with all fields uninitialised except the ID. Touching any other field triggers the SELECT.

```java
OrderItem item = repo.findById(1L).get();
Order order = item.getOrder();      // a proxy — NO query yet
Long id = order.getId();            // still NO query — the ID is already known
String num = order.getOrderNumber();// ← query fires HERE
```
**Useful consequence:** you can set a foreign key without loading the parent row:
```java
item.setOrder(entityManager.getReference(Order.class, orderId));   // no SELECT
```

## 39.5 `LazyInitializationException` — the classic failure

```java
@Service
public class OrderService {

    public Order getOrder(Long id) {          // ❌ no @Transactional
        return orderRepository.findById(id).orElseThrow();
    }   // ← transaction ends, session closes, entity is DETACHED
}

@RestController
public class OrderController {
    @GetMapping("/{id}")
    public Order get(@PathVariable Long id) {
        Order order = orderService.getOrder(id);
        return order;     // Jackson serializes → touches order.getItems()
    }                     // ❌ LazyInitializationException:
}                         //    "could not initialize proxy - no Session"
```

**The cause, in one line:** the lazy proxy needs an open Hibernate session to run its query, and the session closed when the transaction ended.

### The four fixes, ranked

```java
// ✅ FIX 1 — JOIN FETCH (BEST: one query, explicit, works with DTOs)
@Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items WHERE o.id = :id")
Optional<Order> findByIdWithItems(@Param("id") Long id);

// ✅ FIX 2 — @EntityGraph (declarative, reusable)
@EntityGraph(attributePaths = {"items", "customer"})
Optional<Order> findById(Long id);

// ✅ FIX 3 — map to a DTO inside the transaction (our standard approach)
@Transactional(readOnly = true)
public OrderResponseDto getOrder(Long id) {
    Order order = orderRepository.findById(id).orElseThrow();
    return orderMapper.toDto(order);     // lazy fields initialised while session is open
}

// ❌ FIX 4 — open-in-view (the default, and the one to DISABLE)
// spring.jpa.open-in-view=true keeps the session open for the whole HTTP request
```

### Why `open-in-view` should be disabled

Spring Boot enables `spring.jpa.open-in-view=true` **by default**, which keeps the Hibernate session open for the entire request — so lazy loading "just works" in the controller and during serialization.

**Why that is a problem:**

| Issue | Explanation |
|---|---|
| **Hidden N+1** | Serialization silently triggers queries; you never see them in the service |
| **Connection held too long** | The DB connection is held through view rendering, starving the pool |
| **No transaction** | Those queries run outside a transaction — each in auto-commit |
| **Unpredictable performance** | Query count depends on what Jackson touches |
| **Hides real bugs** | Masks missing fetch joins until production load reveals them |

```properties
spring.jpa.open-in-view=false      # set this in EVERY project
```

**What happens when you disable it:** you immediately get `LazyInitializationException` wherever a fetch was missing. That feels like a regression but is actually the point — **it surfaces exactly the places that were doing hidden queries.** Fix each with a fetch join, an entity graph, or DTO mapping inside the transaction.

Boot even logs a warning about this at startup, which most teams ignore. Turning it off is one of the highest-value single-line changes you can propose.

## 39.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Leaving `@ManyToOne` EAGER | Extra queries on every load |
| Relying on `open-in-view` | Hidden N+1, connections held, unpredictable latency |
| Accessing lazy fields outside a transaction | `LazyInitializationException` |
| `JOIN FETCH` on two collections at once | `MultipleBagFetchException` |
| `JOIN FETCH` with pagination | **Hibernate paginates in memory** — loads everything first |
| Serializing entities directly | Triggers lazy loads during serialization |

**`MultipleBagFetchException` — worth knowing the fix:**
```java
// ❌ Fails: cannot fetch two List collections in one query
@Query("SELECT o FROM Order o JOIN FETCH o.items JOIN FETCH o.payments")

// ✅ Fix 1: use Set instead of List for the collections
// ✅ Fix 2: two separate queries — the second reuses the persistence context
// ✅ Fix 3: @BatchSize(size = 20) on the collection
```

**`JOIN FETCH` + pagination — the silent killer:**
Hibernate logs `HHH000104: firstResult/maxResults specified with collection fetch; applying in memory` and then **loads the entire result set into memory** before paginating. On a large table this is an out-of-memory incident. Use `@EntityGraph` with a two-query approach, or `@BatchSize`, when you need both.

## 39.7 TL Questions

**Q: What's the difference between lazy and eager?**
A: Eager loads the association immediately with the parent; lazy defers it until the code actually accesses it, using a proxy.

**Q: What are the defaults and do we keep them?**
A: The `ToOne` associations default to EAGER and the `ToMany` ones to LAZY. We override every `ToOne` to LAZY explicitly, because eager fetching is baked into the mapping and every query pays for it, whereas lazy can always be upgraded with a fetch join when we need the data.

**Q: What causes `LazyInitializationException`?**
A: Touching a lazy association after the transaction has closed. The proxy needs an open session to run its query and there isn't one. We fix it by fetching what we need inside the transaction — a join fetch, an entity graph, or mapping to a DTO before returning.

**Q: Why do we disable `open-in-view`?**
A: Because it keeps the session open for the whole request, so lazy loads happen silently during JSON serialization. That hides N+1 problems, holds a database connection far longer than needed, and runs those queries outside any transaction. Disabling it makes the missing fetches fail loudly in development instead of quietly degrading production.

**Q: What happens internally when we access a lazy field?**
A: For a collection, Hibernate holds a `PersistentBag` proxy that issues a SELECT on first access. For a `ToOne`, it holds a proxy subclass with only the ID populated; touching any other property triggers the query.

**Q: What's the catch with `JOIN FETCH` and pagination?**
A: Hibernate can't apply SQL `LIMIT` correctly when a collection join multiplies the rows, so it loads the entire result set and paginates in memory. It logs a warning most people miss. For paginated queries with collections we use an entity graph or `@BatchSize` instead.

## 39.8 Interview Questions

**Beginner — What's the default fetch type for `@OneToMany`?**
LAZY. `@ManyToOne` and `@OneToOne` default to EAGER.

**Intermediate — How does Hibernate implement lazy loading?**
Through proxies — a CGLIB/ByteBuddy subclass for `ToOne` associations and a `PersistentCollection` wrapper for collections. The query fires on first access to any non-identifier property.

**Intermediate — Why can you set a foreign key without loading the parent?**
Because the proxy already holds the identifier. `getReference()` returns a proxy and calling `getId()` doesn't trigger a query, so you can assign the association without a SELECT.

**Advanced — What is `MultipleBagFetchException`?**
Hibernate cannot fetch two `List`-mapped collections (bags) in a single query, because the cartesian product makes row-to-element attribution ambiguous. Fix by using `Set`, splitting into two queries, or using `@BatchSize`.

**Advanced — Why is in-memory pagination with collection fetch dangerous?**
Because Hibernate must fetch the entire result set before it can paginate, so memory usage scales with the whole table rather than the page size. It's a latent out-of-memory failure that only appears as the data grows.

## 39.9 Quick Revision

1. **`ToOne` defaults EAGER; `ToMany` defaults LAZY.**
2. **Always override `ToOne` to LAZY explicitly.**
3. LAZY can be upgraded per query; EAGER cannot be switched off.
4. Lazy uses proxies; the query fires on first non-ID access.
5. `LazyInitializationException` = lazy access after the session closed.
6. Fixes: `JOIN FETCH`, `@EntityGraph`, DTO mapping inside the transaction.
7. **Set `spring.jpa.open-in-view=false` in every project.**
8. `JOIN FETCH` + pagination → in-memory pagination. Avoid.
9. Two `List` fetch joins → `MultipleBagFetchException`; use `Set` or `@BatchSize`.
10. `getReference()` sets an FK without a SELECT.

## 39.10 TL Explanation (speak this)

> "We set every `ToOne` association to LAZY explicitly, because JPA defaults those to EAGER and that cost is baked into the mapping — every query pays for it whether it needs the data or not. Lazy is the safe default since we can always fetch more with a join fetch or entity graph when a specific use case needs it. We also disable `open-in-view`, which Boot enables by default: it keeps the Hibernate session open through JSON serialization, so lazy loads happen silently and we get hidden N+1 queries and connections held far longer than necessary. Turning it off makes missing fetches fail loudly in development, which is exactly where we want to find them."

---

# 40. The N+1 Select Problem and EntityGraph

## 40.1 What is it?

**Simple words:**
You run **1** query to get a list of orders. Then, for each order, Hibernate runs **1 more query** to get its items.

10 orders → **11 queries**. 1,000 orders → **1,001 queries**.

That's the N+1 problem: **1** query for the parents plus **N** queries for the children.

**Technical wording:**
The N+1 select problem occurs when an initial query retrieves N entities and each associated collection or `ToOne` reference is subsequently initialised with its own query, resulting in N+1 statements where a single join would suffice.

## 40.2 Seeing it happen

```java
@Transactional(readOnly = true)
public List<OrderResponseDto> getAllOrders() {

    List<Order> orders = orderRepository.findAll();
    // QUERY 1:  SELECT * FROM orders                      ← returns 100 orders

    return orders.stream()
            .map(order -> {
                // On EACH iteration, the lazy collection initialises:
                // QUERY 2..101:  SELECT * FROM order_items WHERE order_id = ?
                int itemCount = order.getItems().size();
                return new OrderResponseDto(order.getId(), itemCount);
            })
            .toList();
}
// TOTAL: 101 queries instead of 1 or 2.
```

**Why it's so dangerous:** it **works perfectly in development**. With 10 test rows it's 11 fast queries — nobody notices. In production with 10,000 rows it's 10,001 queries, each with network round-trip latency. A 50ms endpoint becomes 30 seconds. **It is the single most common JPA performance failure.**

### Detecting it

```properties
# Development only
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG

# Best tool: count queries per request and fail the build if a threshold is exceeded
# (datasource-proxy or p6spy)
```
**A practice worth proposing to a TL:** add an integration test that asserts the query count for key endpoints. It turns N+1 from a production surprise into a build failure.

## 40.3 Solution 1 — JOIN FETCH

```java
@Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items WHERE o.status = :status")
List<Order> findByStatusWithItems(@Param("status") OrderStatus status);
```
```sql
-- ONE query
SELECT o.*, i.* FROM orders o
LEFT JOIN order_items i ON i.order_id = o.id
WHERE o.status = 'PENDING';
```

**Why `DISTINCT`:** the join produces one row per item, so an order with 3 items appears 3 times and Hibernate would return the same `Order` object 3 times in the list. `DISTINCT` de-duplicates the object list. (In Hibernate 6 this is automatic for entity queries; in Hibernate 5 you also wanted `hibernate.query.passDistinctThrough=false` so `DISTINCT` wasn't sent to the database unnecessarily.)

**Limitations:** cannot fetch two `List` collections (`MultipleBagFetchException`), and combined with pagination it paginates in memory.

## 40.4 Solution 2 — @EntityGraph (declarative, preferred)

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // Fetch items eagerly for THIS query only
    @EntityGraph(attributePaths = {"items"})
    List<Order> findByStatus(OrderStatus status);

    // Multiple, including nested paths
    @EntityGraph(attributePaths = {"items", "items.product", "customer"})
    Optional<Order> findById(Long id);
}
```

**Why `@EntityGraph` is usually the better choice:**

| | `JOIN FETCH` | `@EntityGraph` |
|---|---|---|
| Written in | The JPQL string | An annotation |
| Works on derived query methods | No | **Yes** |
| Works on inherited `findById` | No | **Yes** |
| Nested paths | Verbose | `"items.product"` |
| Reusable across queries | No | **Yes** (named graphs) |
| Combines with pagination | Poorly | Better |

**Named entity graphs — define once, reuse:**

```java
@Entity
@NamedEntityGraph(
    name = "Order.withItemsAndCustomer",
    attributeNodes = {
        @NamedAttributeNode("customer"),
        @NamedAttributeNode(value = "items", subgraph = "items-subgraph")
    },
    subgraphs = @NamedSubgraph(name = "items-subgraph",
                               attributeNodes = @NamedAttributeNode("product"))
)
public class Order { }

public interface OrderRepository extends JpaRepository<Order, Long> {
    @EntityGraph(value = "Order.withItemsAndCustomer")
    List<Order> findByStatus(OrderStatus status);
}
```

**`FETCH` vs `LOAD` graph type:**
- `EntityGraphType.FETCH` (default): listed attributes are EAGER, **everything else is LAZY**.
- `EntityGraphType.LOAD`: listed attributes are EAGER, everything else keeps its **mapped** fetch type.

`FETCH` gives tighter control and is usually what you want.

## 40.5 Solution 3 — @BatchSize

```java
@Entity
public class Order {
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    @BatchSize(size = 20)                  // Hibernate-specific
    private List<OrderItem> items;
}
```

Instead of one query per order, Hibernate batches IDs:
```sql
SELECT * FROM order_items WHERE order_id IN (1,2,3,...,20);
SELECT * FROM order_items WHERE order_id IN (21,...,40);
```
**100 orders: 101 queries → 6 queries.** Not one, but a massive improvement with **zero query changes**.

```properties
# Apply globally
spring.jpa.properties.hibernate.default_batch_fetch_size=20
```

**Why `@BatchSize` deserves attention:** it is the only solution that **works with pagination**, fixes N+1 across *all* queries at once, and requires no code change. Setting `default_batch_fetch_size` globally is a genuinely high-value one-line configuration — a strong suggestion to bring to a TL.

## 40.6 Solution 4 — DTO projection (best for read-only lists)

```java
@Query("""
       SELECT new com.company.orderservice.dto.OrderSummaryDto(
              o.id, o.orderNumber, o.totalAmount, COUNT(i))
       FROM Order o LEFT JOIN o.items i
       WHERE o.status = :status
       GROUP BY o.id, o.orderNumber, o.totalAmount
       """)
List<OrderSummaryDto> findSummaries(@Param("status") OrderStatus status);
```
One query, only the needed columns, no entities in the persistence context, no dirty-check snapshots. **For a list screen this is the fastest option available.**

## 40.7 Choosing between them

| Situation | Best solution |
|---|---|
| Need the full entity graph, no pagination | `JOIN FETCH` or `@EntityGraph` |
| Derived query method or inherited `findById` | **`@EntityGraph`** |
| **With pagination** | **`@BatchSize`** or `@EntityGraph` (two-query) |
| Read-only list/summary screen | **DTO projection** |
| Global safety net | **`default_batch_fetch_size`** |
| Multiple `List` collections | `Set` + fetch, or `@BatchSize` |

## 40.8 Common Mistakes

| Mistake | Consequence |
|---|---|
| Not noticing N+1 in development | Discovered in production under load |
| "Fixing" it with EAGER | Makes it worse — now *every* query over-fetches |
| `JOIN FETCH` with pagination | In-memory pagination, OOM risk |
| Two `List` fetch joins | `MultipleBagFetchException` |
| Forgetting `DISTINCT` (Hibernate 5) | Duplicate parent objects in results |
| Not enabling `default_batch_fetch_size` | Missing a free, global improvement |

**Why EAGER is not a fix — state this clearly:**
```java
@OneToMany(fetch = FetchType.EAGER)     // ❌ "solves" N+1 by making it universal
```
EAGER still fires a separate query per parent for collections (or one huge cartesian join), and now **every** query loading an `Order` pays the cost, including ones that never touch `items`. It converts a per-query problem into a permanent one.

## 40.9 TL Questions

**Q: What is the N+1 problem?**
A: One query fetches N parents, then each parent's lazy association fires its own query — so N+1 statements where one or two would do. A hundred orders becomes a hundred and one queries.

**Q: Why is it so dangerous?**
A: Because it doesn't show up in development. With ten test rows it's eleven fast queries and nobody notices; with ten thousand production rows it's ten thousand queries and the endpoint times out.

**Q: How do we fix it?**
A: Depends on the case. For a specific query we use `@EntityGraph` or a join fetch. For read-only list screens we use a DTO projection so only the needed columns are selected. And globally we set `default_batch_fetch_size`, which batches the lazy loads and turns a hundred and one queries into about six with no code change.

**Q: Why not just use EAGER?**
A: Because that makes it worse. EAGER still issues the extra queries, and now every query that loads an order pays the cost even if it never touches the items. Lazy plus a targeted fetch keeps the cost where it's actually needed.

**Q: How do we detect it?**
A: SQL logging in development, and ideally an integration test that counts queries per endpoint and fails the build past a threshold — so it's caught in CI rather than production.

**Q: What's the catch with join fetch and pagination?**
A: Hibernate can't apply SQL `LIMIT` when the collection join multiplies rows, so it loads the whole result set and paginates in memory. For paginated endpoints we use `@BatchSize` or an entity graph instead.

**Q: What is `@EntityGraph` and why prefer it over join fetch?**
A: It's a declarative way to say which associations to fetch for a given query. It works on derived query methods and on inherited methods like `findById`, where we can't edit a JPQL string, and it handles nested paths cleanly.

## 40.10 Interview Questions

**Beginner — What causes N+1?**
Lazy associations being initialised one by one after a query returns multiple parent entities.

**Intermediate — Why is `DISTINCT` needed with a collection join fetch?**
The join produces one row per child, so the parent appears multiple times in the result list. `DISTINCT` de-duplicates at the object level. Hibernate 6 does this automatically for entity queries.

**Intermediate — How does `@BatchSize` help?**
Instead of one query per parent, Hibernate collects up to N parent IDs and loads their collections with a single `IN` query, reducing N+1 to roughly N/batchSize + 1 queries.

**Advanced — `@EntityGraph` FETCH vs LOAD?**
`FETCH` makes listed attributes eager and treats all others as lazy regardless of their mapping. `LOAD` makes listed attributes eager while others keep their mapped fetch type. `FETCH` gives more predictable control.

**Advanced — Why can't `JOIN FETCH` be paginated safely?**
SQL `LIMIT` applies to result rows, but a collection join makes rows a multiple of entities, so limiting rows would truncate a parent's children arbitrarily. Hibernate therefore fetches everything and paginates in memory, which defeats the purpose and risks memory exhaustion.

## 40.11 Quick Revision

1. N+1 = 1 parent query + N child queries.
2. **Works fine in dev, fails in production** — always test with realistic data.
3. Fixes: `JOIN FETCH`, `@EntityGraph`, `@BatchSize`, DTO projection.
4. **`@EntityGraph` works on derived and inherited methods** — prefer it.
5. `@BatchSize` / `default_batch_fetch_size` is the **global safety net** and works with pagination.
6. DTO projections are fastest for read-only lists.
7. **EAGER is not a fix** — it makes the cost permanent and universal.
8. `DISTINCT` needed with collection fetch joins (Hibernate 5).
9. Join fetch + pagination → in-memory pagination.
10. Detect with SQL logging; enforce with a query-count test.

## 40.12 TL Explanation (speak this)

> "N+1 is where one query loads a hundred orders and then a hundred more queries load each order's items. The reason it bites is that it looks fine in development with ten rows and only becomes a timeout in production. We fix it per-query with `@EntityGraph` or a join fetch, and for read-only list screens we use a DTO projection so the SQL selects only the columns we actually display. We also set `default_batch_fetch_size` globally, which batches lazy loads into `IN` queries and turns a hundred and one queries into about six with no code change. What we deliberately don't do is switch to EAGER — that just makes every query pay the cost permanently."

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

# 42. Spring Security

## 42.1 What is it?

**Simple words:**
Spring Security is the framework that answers two questions for every request:

1. **Authentication** — *"Who are you?"* (login, token validation)
2. **Authorization** — *"Are you allowed to do this?"* (roles, permissions)

It plugs into the filter chain, so it runs **before** your controller ever sees the request.

**Technical wording:**
Spring Security is a framework providing authentication, authorization and protection against common attacks for Spring applications. It is implemented as a chain of servlet filters registered through a `DelegatingFilterProxy`, backed by a `SecurityContext` bound to the current thread.

## 42.2 Authentication vs Authorization — get this exact

| | Authentication | Authorization |
|---|---|---|
| Question | **Who are you?** | **What can you do?** |
| Happens | First | After authentication |
| Failure status | **401 Unauthorized** | **403 Forbidden** |
| Mechanism | Credentials, JWT, OAuth | Roles, authorities, ACLs |
| Spring artifact | `AuthenticationManager` | `AccessDecisionManager` / `AuthorizationManager` |

**The one-line distinction for interviews:** *"Authentication proves identity; authorization grants permission. A valid token that lacks the required role gets 403, not 401."*

## 42.3 Core architecture

```
   Request
      │
      ▼
 ┌──────────────────────────────────────────────────────────┐
 │  DelegatingFilterProxy  ("springSecurityFilterChain")     │
 │      delegates to ▼                                       │
 │  FilterChainProxy                                         │
 │      selects the matching ▼                               │
 │  SecurityFilterChain — an ordered list of filters:        │
 │                                                            │
 │   1. SecurityContextPersistenceFilter                     │
 │        loads the SecurityContext for this request         │
 │   2. CorsFilter / CsrfFilter                              │
 │   3. UsernamePasswordAuthenticationFilter  (form login)   │
 │      ── or our custom JwtAuthenticationFilter ──          │
 │   4. BearerTokenAuthenticationFilter (OAuth2 resource)    │
 │   5. ExceptionTranslationFilter                           │
 │        catches AuthenticationException  → 401 entry point │
 │        catches AccessDeniedException    → 403 handler     │
 │   6. AuthorizationFilter (was FilterSecurityInterceptor)  │
 │        final check: is this principal allowed here?       │
 └──────────────────────────────────────────────────────────┘
      │ authenticated and authorized
      ▼
 DispatcherServlet → Controller
```

### Key classes

| Class/Interface | Responsibility |
|---|---|
| `SecurityContextHolder` | Holds the `SecurityContext` in a **`ThreadLocal`** |
| `SecurityContext` | Holds the current `Authentication` |
| `Authentication` | Principal + credentials + authorities + authenticated flag |
| `UserDetails` | The user model Spring Security understands |
| `UserDetailsService` | Loads a `UserDetails` by username — **you implement this** |
| `AuthenticationManager` | Entry point for authenticating a request |
| `AuthenticationProvider` | Performs actual authentication (e.g. `DaoAuthenticationProvider`) |
| `PasswordEncoder` | Hashes and verifies passwords |
| `GrantedAuthority` | A single permission/role |

**`SecurityContextHolder` is a `ThreadLocal`.** That's how any service, several layers down, can call `SecurityContextHolder.getContext().getAuthentication()` without it being passed as a parameter. It's also why the security context is **not** available in an `@Async` method unless you propagate it (`DelegatingSecurityContextExecutor` or `MODE_INHERITABLETHREADLOCAL`).

## 42.4 The authentication flow

```
 1. Client POSTs credentials  { "username": "raj", "password": "secret" }
              │
              ▼
 2. AuthenticationFilter builds an unauthenticated
    UsernamePasswordAuthenticationToken
              │
              ▼
 3. AuthenticationManager (ProviderManager) delegates to providers
              │
              ▼
 4. DaoAuthenticationProvider:
       a. userDetailsService.loadUserByUsername("raj")
             → fetch the user + hashed password + roles from the DB
       b. passwordEncoder.matches(rawPassword, storedHash)
             → BCrypt compares; the stored hash is NEVER decrypted
              │
        ┌─────┴─────┐
     match        no match
        │             │
        ▼             ▼
 5. Return an     BadCredentialsException
    AUTHENTICATED       │
    Authentication      ▼
    with authorities   401 via AuthenticationEntryPoint
              │
              ▼
 6. SecurityContextHolder.getContext().setAuthentication(auth)
              │
              ▼
 7. Request proceeds; authorization checks use those authorities
```

## 42.5 Configuration (Spring Security 6 / Boot 3 style)

```java
package com.company.orderservice.config;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity           // enables @PreAuthorize / @PostAuthorize
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;
    private final UserDetailsService userDetailsService;
    private final CustomAuthenticationEntryPoint authenticationEntryPoint;
    private final CustomAccessDeniedHandler accessDeniedHandler;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            // CSRF is not needed for a stateless token-based API (explained below)
            .csrf(AbstractHttpConfigurer::disable)

            .cors(cors -> cors.configurationSource(corsConfigurationSource()))

            // No HTTP session — every request must carry its own token
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))

            .authorizeHttpRequests(auth -> auth
                // Public endpoints
                .requestMatchers("/api/v1/auth/**", "/api/v1/public/**").permitAll()
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/v3/api-docs/**", "/swagger-ui/**").permitAll()

                // Role-based rules — evaluated TOP TO BOTTOM, first match wins
                .requestMatchers(HttpMethod.GET, "/api/v1/orders/**")
                    .hasAnyRole("USER", "ADMIN")
                .requestMatchers(HttpMethod.POST, "/api/v1/orders/**")
                    .hasRole("USER")
                .requestMatchers(HttpMethod.DELETE, "/api/v1/orders/**")
                    .hasRole("ADMIN")
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")

                // ★ Everything else requires authentication — deny by default
                .anyRequest().authenticated()
            )

            // Custom 401 / 403 responses instead of the default HTML error page
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(authenticationEntryPoint)   // 401
                .accessDeniedHandler(accessDeniedHandler)             // 403
            )

            // Our JWT filter runs BEFORE the username/password filter
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)

            .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);      // strength 12
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        // NEVER use "*" in production with credentials
        config.setAllowedOrigins(List.of("https://app.company.com"));
        config.setAllowedMethods(List.of("GET","POST","PUT","PATCH","DELETE","OPTIONS"));
        config.setAllowedHeaders(List.of("Authorization","Content-Type","X-Correlation-Id"));
        config.setExposedHeaders(List.of("X-Correlation-Id"));
        config.setAllowCredentials(true);
        config.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", config);
        return source;
    }
}
```

**Rule ordering matters enormously.** `authorizeHttpRequests` evaluates **top to bottom, first match wins**. Putting `.anyRequest().authenticated()` first would make every rule after it unreachable. Always go from **most specific to least specific**, ending with `anyRequest()`.

## 42.6 Method-level security

```java
@Service
public class OrderService {

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteOrder(Long id) { }

    @PreAuthorize("hasAnyRole('USER','ADMIN')")
    public OrderDto getOrder(Long id) { }

    // SpEL can reference method arguments
    @PreAuthorize("#customerId == authentication.name or hasRole('ADMIN')")
    public List<OrderDto> getCustomerOrders(String customerId) { }

    // Checks the RETURN value after execution
    @PostAuthorize("returnObject.customerId == authentication.name")
    public OrderDto findById(Long id) { }

    @PreAuthorize("hasAuthority('ORDER_DELETE')")   // authority, not role
    public void archive(Long id) { }
}
```

**`hasRole` vs `hasAuthority` — a constant source of confusion:**
`hasRole('ADMIN')` automatically prepends `ROLE_`, so it checks for the authority `ROLE_ADMIN`. `hasAuthority('ADMIN')` checks for exactly `ADMIN`. **If your database stores authorities without the `ROLE_` prefix, `hasRole` silently fails.** Pick one convention and document it.

> **`@PostAuthorize` caution:** the method has already executed when the check runs. If it modified data, that change already happened — the exception only prevents the *return*. Never use `@PostAuthorize` on a method with side effects.

**Note:** `@PreAuthorize` is AOP-based, so **self-invocation bypasses it** — the same limitation as `@Transactional`.

## 42.7 UserDetailsService and password storage

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    @Transactional(readOnly = true)
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        User user = userRepository.findByUsernameWithRoles(username)   // fetch roles eagerly
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
        // NOTE: deliberately a generic message — never reveal whether the
        // username exists (that's a user-enumeration vulnerability).

        return org.springframework.security.core.userdetails.User.builder()
                .username(user.getUsername())
                .password(user.getPassword())          // the BCrypt HASH
                .authorities(user.getRoles().stream()
                        .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
                        .toList())
                .accountLocked(user.isLocked())
                .disabled(!user.isEnabled())
                .build();
    }
}
```

### Why BCrypt

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12);
}
```

| Property | Why it matters |
|---|---|
| **One-way** | Hashes cannot be reversed — a database leak doesn't expose passwords |
| **Salted automatically** | Each hash embeds a random salt, so identical passwords produce different hashes → rainbow tables useless |
| **Deliberately slow** | The work factor makes brute force expensive; increase it as hardware improves |
| **Self-describing** | The hash stores its own algorithm, cost and salt, so verification needs no extra columns |

**Never use MD5 or SHA-256 for passwords** — they're designed to be *fast*, which is exactly wrong for password hashing. Modern GPUs compute billions of SHA-256 hashes per second. BCrypt (or Argon2/scrypt) is the correct family.

## 42.8 CSRF — why we disable it for token APIs

**What CSRF is:** a malicious site causes the victim's browser to send an authenticated request to your site. It works because the browser **automatically attaches cookies** to any request to that domain.

**Why it doesn't apply to a stateless JWT API:** the token lives in the `Authorization` header, and browsers do **not** attach custom headers automatically. The attacker's page cannot read or set that header cross-origin. No automatic credential attachment means no CSRF vector.

**When you MUST keep CSRF enabled:**
- Session-cookie authentication
- **JWT stored in a cookie** — cookies are auto-attached, so the CSRF risk returns

| Token storage | XSS risk | CSRF risk | Notes |
|---|---|---|---|
| `localStorage` | **High** — any injected JS can read it | None | Common, but XSS steals the token |
| `sessionStorage` | High | None | Same, cleared on tab close |
| **`httpOnly` cookie** | **Low** — JS cannot read it | **Yes** — needs CSRF protection | Most secure **if** CSRF protection and `SameSite` are configured |

**The honest answer to give a TL:** *"There's no storage option with zero risk. An `httpOnly`, `Secure`, `SameSite=Strict` cookie plus CSRF tokens is the most defensible choice; `localStorage` is simpler but one XSS away from full account takeover."*

## 42.9 Custom 401 and 403 handlers

Filter-thrown exceptions **cannot** be caught by `@RestControllerAdvice` (Section 23), so security needs its own handlers:

```java
@Component
@RequiredArgsConstructor
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    private final ObjectMapper objectMapper;

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException {
        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        objectMapper.writeValue(response.getOutputStream(), ErrorResponse.builder()
                .timestamp(Instant.now()).status(401).error("Unauthorized")
                .errorCode("AUTH_REQUIRED")
                .message("Authentication required. Provide a valid token.")
                .path(request.getRequestURI())
                .correlationId(MDC.get("correlationId"))
                .build());
    }
}

@Component
@RequiredArgsConstructor
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    private final ObjectMapper objectMapper;

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
                       AccessDeniedException ex) throws IOException {
        response.setStatus(HttpStatus.FORBIDDEN.value());
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        objectMapper.writeValue(response.getOutputStream(), ErrorResponse.builder()
                .timestamp(Instant.now()).status(403).error("Forbidden")
                .errorCode("ACCESS_DENIED")
                .message("You do not have permission to perform this action")
                .path(request.getRequestURI()).build());
    }
}
```

## 42.10 Common Mistakes

| Mistake | Consequence |
|---|---|
| **Storing plain-text passwords** | Catastrophic on any database leak |
| MD5/SHA for passwords | Trivially brute-forced on modern GPUs |
| Rule order wrong in `authorizeHttpRequests` | Broad rules shadow specific ones |
| Forgetting `.anyRequest().authenticated()` | Unlisted endpoints are **unprotected** |
| `hasRole('ROLE_ADMIN')` | Double prefix → checks `ROLE_ROLE_ADMIN`, always fails |
| CORS `allowedOrigins("*")` with credentials | Rejected by browsers; also unsafe |
| Disabling CSRF while using cookie auth | Reintroduces the CSRF vulnerability |
| Revealing "user not found" vs "wrong password" | User enumeration |
| Secrets in `application.yml` | Credentials in version control |
| Expecting `@RestControllerAdvice` to catch security exceptions | Default HTML error page returned |
| Relying on `@PreAuthorize` across self-invocation | Silently skipped |

## 42.11 TL Questions

**Q: How does Spring Security work at a high level?**
A: It's a chain of servlet filters registered ahead of the DispatcherServlet. Each request passes through filters that authenticate it, populate a `SecurityContext` held in a `ThreadLocal`, and then check authorization before the controller runs.

**Q: What's the difference between authentication and authorization in our responses?**
A: Authentication failure is 401 — we don't know who you are. Authorization failure is 403 — we know you but your role doesn't permit this. We have separate handlers for each so clients get a clean JSON body rather than the default HTML page.

**Q: Why do we disable CSRF?**
A: Because our API is stateless and the token travels in the `Authorization` header. CSRF exploits the browser automatically attaching cookies; custom headers aren't attached automatically, so there's no vector. If we ever moved the token into a cookie we'd have to re-enable CSRF.

**Q: How are passwords stored?**
A: BCrypt hashes with a work factor of 12. It's one-way, automatically salted so identical passwords hash differently, and deliberately slow to make brute force expensive. We never store or log the raw password.

**Q: What happens if we forget `anyRequest().authenticated()`?**
A: Any endpoint not explicitly matched by a rule is left unprotected. That line is our deny-by-default backstop, so a newly added controller is secured automatically rather than accidentally public.

**Q: Where do we enforce authorization — the URL rules or the methods?**
A: Both. URL rules give broad coverage at the filter level, and `@PreAuthorize` on service methods handles finer rules like "a user may only read their own orders". Method security also protects the logic if it's ever called from a scheduled job or message listener rather than HTTP.

**Q: Why is the security context unavailable in `@Async` methods?**
A: It's stored in a `ThreadLocal`, and an async method runs on a different pool thread. We either pass the needed values explicitly or configure a `DelegatingSecurityContextExecutor` to propagate it.

## 42.12 Interview Questions

**Beginner — What is `SecurityContextHolder`?**
A holder for the current `SecurityContext`, stored in a `ThreadLocal`, from which any code on that thread can obtain the current `Authentication`.

**Beginner — What does `UserDetailsService` do?**
Loads user data — username, password hash and authorities — by username, for the authentication provider to verify.

**Intermediate — `hasRole` vs `hasAuthority`?**
`hasRole('ADMIN')` implicitly prefixes `ROLE_`, so it matches the authority `ROLE_ADMIN`. `hasAuthority('ADMIN')` matches exactly `ADMIN`. Mismatched conventions cause silent authorization failures.

**Intermediate — Why is BCrypt preferred over SHA-256?**
SHA-256 is fast by design, so an attacker can compute billions of guesses per second. BCrypt has a tunable work factor making each hash deliberately slow, and it salts automatically, defeating rainbow tables.

**Advanced — What is `DelegatingFilterProxy` and why is it needed?**
A container-managed filter that delegates to a Spring bean by name. The servlet container instantiates filters itself, outside the Spring context, so the proxy bridges the two — letting the real filter be a fully injected Spring bean.

**Advanced — `@PreAuthorize` vs `@PostAuthorize`?**
`@PreAuthorize` evaluates before invocation and can prevent execution. `@PostAuthorize` evaluates afterwards against `returnObject`, so it can filter based on the result — but the method has already run, so it must never be used where the method has side effects.

**Advanced — How does the `ExceptionTranslationFilter` work?**
It wraps the rest of the chain and catches two exception types: `AuthenticationException` triggers the `AuthenticationEntryPoint` (401), and `AccessDeniedException` triggers the `AccessDeniedHandler` (403) — or, for an anonymous user, the entry point instead, prompting authentication.

## 42.13 Quick Revision

1. Spring Security is a **filter chain** running before DispatcherServlet.
2. **Authentication = who (401); Authorization = what (403).**
3. `SecurityContextHolder` is a **`ThreadLocal`** — not available in `@Async`.
4. `UserDetailsService` loads the user; `PasswordEncoder` verifies.
5. **BCrypt** — one-way, auto-salted, deliberately slow.
6. Rules are evaluated **top to bottom**; always end with `anyRequest().authenticated()`.
7. `hasRole` prefixes `ROLE_`; `hasAuthority` does not.
8. CSRF disabled only because we're **stateless with header tokens**.
9. Security exceptions need `AuthenticationEntryPoint` / `AccessDeniedHandler`.
10. `@PreAuthorize` is AOP — self-invocation bypasses it.
11. Never reveal whether a username exists.

## 42.14 TL Explanation (speak this)

> "Spring Security runs as a filter chain before the DispatcherServlet. Each request is authenticated, the resulting `Authentication` is stored in a `ThreadLocal` security context, and then authorization rules are checked before our controller is reached. We configure URL rules from most specific to least, ending with `anyRequest().authenticated()` so anything we forget to list is protected by default, and we add `@PreAuthorize` on services for finer rules like a user only reading their own orders. Passwords are BCrypt with a work factor of twelve — one-way and automatically salted. CSRF is off only because we're stateless with the token in the `Authorization` header; if we ever moved it into a cookie we'd have to turn CSRF back on."

---

# 43. JWT (JSON Web Token) Authentication

## 43.1 What is it?

**Simple words:**
A JWT is a **self-contained, digitally signed token** the server gives you after login. You send it with every request, and the server can verify it **without looking anything up** — the token itself carries who you are and what you can do, plus a signature proving it hasn't been tampered with.

**Technical wording:**
A JWT (RFC 7519) is a compact, URL-safe means of representing claims between two parties. It comprises a Base64URL-encoded header, payload and signature, enabling stateless authentication through cryptographic verification.

## 43.2 Structure

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJyYWoiLCJyb2xlcyI6WyJVU0VSIl19.SflKxwRJ...
└────── HEADER ─────┘ └────────── PAYLOAD ──────────────┘ └─ SIGNATURE ─┘
```

```json
// HEADER — algorithm and token type
{ "alg": "HS256", "typ": "JWT" }

// PAYLOAD — the claims
{
  "sub": "raj@company.com",      // subject (who)
  "roles": ["USER", "ADMIN"],    // custom claim
  "iat": 1705315800,             // issued at
  "exp": 1705319400,             // expiry  ← the most important claim
  "iss": "order-service",        // issuer
  "jti": "a1b2c3"                // unique token ID (for revocation lists)
}

// SIGNATURE
HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), SECRET_KEY )
```

### ★ THE MOST IMPORTANT SECURITY FACT ★

**The payload is Base64-ENCODED, not encrypted.** Anyone can decode it — paste any JWT into jwt.io and read every claim.

```
Base64 is ENCODING (reversible, no key) — NOT encryption.
```

**Therefore: NEVER put sensitive data in a JWT payload.** No passwords, no card numbers, no personal identifiers beyond what's necessary. The signature guarantees **integrity** (it hasn't been modified), not **confidentiality**.

**This is the number one JWT interview question, and a real production mistake.** If you need confidentiality, use JWE (encrypted JWT) or keep the data server-side.

## 43.3 Why JWT for microservices

```
SESSION-BASED                          JWT-BASED
─────────────                          ─────────
Client sends session cookie            Client sends JWT in Authorization header
       │                                       │
       ▼                                       ▼
Server looks up the session            Server VERIFIES THE SIGNATURE
  in memory or Redis                     using its secret key
       │                                       │
   NEEDS SHARED STORAGE                  NO LOOKUP, NO STORAGE
   or sticky sessions                    any instance can verify
```

| | Session | JWT |
|---|---|---|
| Server state | **Stored server-side** | **None** |
| Scaling | Needs shared store / sticky sessions | **Any instance handles any request** |
| Revocation | **Easy — delete the session** | **Hard — valid until expiry** |
| Size | Small cookie | Larger (sent on every request) |
| Cross-domain / mobile | Awkward | **Natural** |
| Best for | Traditional server-rendered apps | **Microservices, SPAs, mobile** |

**The honest trade-off to state to a TL:** *"JWT buys us statelessness and easy horizontal scaling, but we give up instant revocation. A stolen token stays valid until it expires — which is exactly why access tokens must be short-lived."*

## 43.4 The complete JWT flow

```
 ═══════════════ LOGIN (once) ═══════════════
 1. POST /api/v1/auth/login  { username, password }
              │
 2. AuthenticationManager → UserDetailsService → BCrypt verify
              │
 3. Valid → generate:
       accessToken  (15 min, contains roles)
       refreshToken (7 days, stored server-side, minimal claims)
              │
 4. Response: { accessToken, refreshToken, expiresIn }
              │
 5. Client stores them

 ═══════════ EVERY SUBSEQUENT REQUEST ═══════════
 6. GET /api/v1/orders
    Authorization: Bearer eyJhbGc...
              │
              ▼
 7. JwtAuthenticationFilter:
       a. read the Authorization header
       b. is it "Bearer ..."?          no → continue unauthenticated
       c. verify the SIGNATURE         invalid → 401
       d. check expiry (exp)           expired → 401
       e. extract the username
       f. load UserDetails (or trust claims)
       g. build an Authentication and put it in SecurityContextHolder
              │
              ▼
 8. AuthorizationFilter checks roles → 403 if insufficient
              │
              ▼
 9. Controller executes

 ═══════════════ TOKEN REFRESH ═══════════════
 10. Access token expires (15 min)
 11. POST /api/v1/auth/refresh { refreshToken }
 12. Server validates it AGAINST THE DATABASE (it is stateful by design)
 13. Issues a new access token (and rotates the refresh token)
```

## 43.5 Implementation

### Dependency and configuration

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version>
</dependency>
<!-- jjwt-impl and jjwt-jackson at runtime scope -->
```

```yaml
app:
  jwt:
    secret: ${JWT_SECRET}                  # from environment — NEVER in the repo
    access-token-expiration-ms: 900000     # 15 minutes
    refresh-token-expiration-ms: 604800000 # 7 days
    issuer: order-service
```

### The token service

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class JwtService {

    private final JwtProperties jwtProperties;

    private SecretKey getSigningKey() {
        // The secret MUST be at least 256 bits for HS256, or JJWT rejects it
        return Keys.hmacShaKeyFor(jwtProperties.getSecret().getBytes(StandardCharsets.UTF_8));
    }

    public String generateAccessToken(UserDetails userDetails) {
        return Jwts.builder()
                .subject(userDetails.getUsername())
                .claim("roles", userDetails.getAuthorities().stream()
                        .map(GrantedAuthority::getAuthority).toList())
                .issuer(jwtProperties.getIssuer())
                .issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis()
                        + jwtProperties.getAccessTokenExpirationMs()))
                .id(UUID.randomUUID().toString())          // jti, for blacklisting
                .signWith(getSigningKey())
                .compact();
    }

    public String extractUsername(String token) {
        return extractAllClaims(token).getSubject();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        try {
            Claims claims = extractAllClaims(token);       // throws if signature invalid
            return claims.getSubject().equals(userDetails.getUsername())
                    && claims.getExpiration().after(new Date());
        } catch (JwtException | IllegalArgumentException e) {
            log.warn("Invalid JWT: {}", e.getMessage());   // do NOT log the token itself
            return false;
        }
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parser()
                .verifyWith(getSigningKey())     // ★ signature verification happens HERE
                .requireIssuer(jwtProperties.getIssuer())
                .build()
                .parseSignedClaims(token)        // ★ parseSigned — NEVER parse unsigned
                .getPayload();
    }
}
```

**Two security-critical lines highlighted above:**
- `verifyWith(...)` + `parseSignedClaims(...)` — this is what actually validates the signature. Using an unsigned parse, or decoding the payload manually, means **an attacker can forge any claims they like**.
- Never log the token. It is a credential; a token in your log aggregator is a leaked credential.

### The authentication filter

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        final String authHeader = request.getHeader(HttpHeaders.AUTHORIZATION);

        // No token → continue unauthenticated. Let the authorization filter decide.
        // Do NOT reject here — public endpoints must still work.
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        try {
            final String jwt = authHeader.substring(7);
            final String username = jwtService.extractUsername(jwt);

            // Only authenticate if not already authenticated on this request
            if (username != null
                    && SecurityContextHolder.getContext().getAuthentication() == null) {

                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                if (jwtService.isTokenValid(jwt, userDetails)) {
                    UsernamePasswordAuthenticationToken authToken =
                            new UsernamePasswordAuthenticationToken(
                                    userDetails, null, userDetails.getAuthorities());
                    authToken.setDetails(
                            new WebAuthenticationDetailsSource().buildDetails(request));

                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        } catch (Exception e) {
            log.warn("JWT authentication failed: {}", e.getMessage());
            SecurityContextHolder.clearContext();
            // Fall through — ExceptionTranslationFilter will produce a proper 401
        }

        filterChain.doFilter(request, response);
    }
}
```

**Design note worth explaining:** the filter does **not** reject requests without a token. It simply doesn't authenticate them. Rejection is the `AuthorizationFilter`'s job, using the URL rules — which is what keeps `/api/v1/auth/login` and other public endpoints working.

### Login endpoint

```java
@RestController
@RequestMapping("/api/v1/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final JwtService jwtService;
    private final RefreshTokenService refreshTokenService;

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@Valid @RequestBody LoginRequest request) {

        // Throws BadCredentialsException → 401 via the entry point
        Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                        request.getUsername(), request.getPassword()));

        UserDetails userDetails = (UserDetails) authentication.getPrincipal();

        return ResponseEntity.ok(AuthResponse.builder()
                .accessToken(jwtService.generateAccessToken(userDetails))
                .refreshToken(refreshTokenService.create(userDetails.getUsername()).getToken())
                .tokenType("Bearer")
                .expiresIn(900)
                .build());
    }
}
```

## 43.6 The revocation problem and refresh tokens

**The core weakness:** a JWT is valid until it expires. If a token is stolen — or a user is deactivated, or their role is revoked — **the server cannot invalidate it**. There's nothing to delete.

### Mitigations

| Strategy | How | Trade-off |
|---|---|---|
| **Short expiry** | 15-minute access tokens | Small exposure window. **The primary defence.** |
| **Refresh tokens** | Long-lived, stored in the DB, revocable | Reintroduces some state — deliberately |
| **Blacklist** | Store revoked `jti` values in Redis until expiry | A lookup per request — partly gives up statelessness |
| **Token version** | A counter on the user row; invalidate all older tokens | One lookup per request |

**The standard production pattern:**

```
ACCESS TOKEN                          REFRESH TOKEN
────────────                          ─────────────
Short: 15 minutes                     Long: 7 days
Stateless — signature only            STORED IN THE DATABASE
Carries roles                         Minimal claims
Sent with every request               Sent only to /auth/refresh
Cannot be revoked                     CAN be revoked (delete the row)
```

**Why this design is defensible:** the access token stays stateless and fast for the 99% of requests. The refresh token is deliberately stateful, so logout and revocation actually work — we accept one database lookup every 15 minutes rather than on every request.

**Refresh token rotation:** issue a new refresh token on each use and invalidate the old one. If an old refresh token is ever reused, that signals theft — revoke the entire family and force re-login. This is the current best practice.

## 43.7 Common Mistakes

| Mistake | Consequence |
|---|---|
| **Sensitive data in the payload** | Anyone can decode it — it is not encrypted |
| **Weak or committed secret** | Attacker forges tokens = full compromise |
| **No expiry (`exp`)** | Token valid forever |
| Very long access-token expiry | Huge window for a stolen token |
| **Not verifying the signature** | Forged tokens accepted |
| Accepting `alg: none` | Classic JWT vulnerability — signature skipped entirely |
| Logging the token | Credential leaked into log aggregation |
| No refresh mechanism | Users forced to log in constantly, or expiry set dangerously long |
| Expecting instant revocation | Doesn't exist without extra state |
| Token in a URL query parameter | Recorded in access logs and browser history |

**The `alg: none` attack, explained:** the JWT spec permits an unsecured token with `"alg": "none"` and an empty signature. If your library is configured to accept the algorithm declared *in the token*, an attacker rewrites the header to `none`, edits the payload to `"roles": ["ADMIN"]`, and sends it with no signature. **Always pin the expected algorithm server-side** — which `verifyWith(key)` + `parseSignedClaims()` does.

## 43.8 TL Questions

**Q: What is a JWT and why do we use it?**
A: A signed, self-contained token carrying the user's identity and roles. Because the server verifies it by checking the signature rather than looking it up, any instance can authenticate any request — which is what lets us scale horizontally without sticky sessions or a shared session store.

**Q: Is the token encrypted?**
A: No. The payload is Base64-encoded and anyone can decode and read it. The signature guarantees it hasn't been tampered with, not that it's secret. So we never put anything sensitive in the claims.

**Q: What happens if someone steals a token?**
A: They can use it until it expires, and we can't invalidate it — that's the fundamental trade-off of statelessness. We mitigate it with a fifteen-minute access-token lifetime, and a longer refresh token that *is* stored in the database and can be revoked.

**Q: How does logout work?**
A: We delete the refresh token server-side, so no new access tokens can be issued. The current access token stays valid for up to fifteen more minutes. If we needed immediate revocation we'd add a Redis blacklist keyed on the token ID, accepting a lookup per request.

**Q: Where does the client store the token?**
A: There's no risk-free option. An `httpOnly`, `Secure`, `SameSite` cookie is the most defensible, because JavaScript can't read it — but then we must re-enable CSRF protection. `localStorage` is simpler but readable by any injected script, so an XSS becomes full account takeover.

**Q: What's the biggest implementation risk?**
A: Not verifying the signature properly, or accepting the algorithm declared inside the token. That allows the `alg: none` attack, where an attacker rewrites the header, edits the claims to give themselves admin, and sends it unsigned. We pin the algorithm and key server-side.

**Q: Why not just use sessions?**
A: Sessions need shared state — sticky sessions or a Redis session store — which adds infrastructure and a failure point. For a microservice architecture where several services must independently validate the same identity, a signed token is simpler. For a single server-rendered app, sessions would be a reasonable choice.

## 43.9 Interview Questions

**Beginner — What are the three parts of a JWT?**
Header, payload and signature, Base64URL-encoded and separated by dots.

**Beginner — Where is the token sent?**
In the `Authorization` header as `Bearer <token>`.

**Intermediate — Is a JWT encrypted?**
No — encoded only. Use JWE if confidentiality is required.

**Intermediate — Why use both an access and a refresh token?**
The access token is short-lived and stateless so most requests need no lookup; the refresh token is long-lived, stored server-side and revocable, so logout and revocation actually work. It balances performance against control.

**Advanced — How can a JWT be revoked?**
Not directly. Options are short expiry (the main defence), a Redis blacklist of `jti` values kept until natural expiry, or a token-version counter on the user record — both of which trade some statelessness for control.

**Advanced — What is the `alg: none` vulnerability?**
The spec allows an unsecured JWT with no signature. If the server trusts the algorithm declared in the token header, an attacker sets it to `none`, modifies the claims freely and omits the signature. Mitigation: always pin the expected algorithm and key server-side.

**Advanced — HS256 vs RS256?**
HS256 is symmetric — the same secret signs and verifies, so every verifying service needs the signing secret and could forge tokens. RS256 is asymmetric — a private key signs and a public key verifies, so downstream services can validate without any ability to issue tokens. **RS256 is the correct choice in a multi-service architecture.**

## 43.10 Quick Revision

1. JWT = header + payload + signature, Base64URL-encoded, dot-separated.
2. **Payload is encoded, NOT encrypted** — never put secrets in it.
3. Signature gives **integrity**, not confidentiality.
4. Stateless → any instance validates → horizontal scaling.
5. **Cannot be revoked** — mitigate with short expiry.
6. Access token ~15 min (stateless); refresh token ~7 days (stored, revocable).
7. Rotate refresh tokens; reuse of an old one signals theft.
8. Always `verifyWith(key)` + `parseSignedClaims` — never trust the token's `alg`.
9. Keep the secret in the environment, at least 256 bits for HS256.
10. **RS256 for multi-service** — verifiers can't forge.
11. Send in the `Authorization` header, never a query parameter.
12. Never log tokens.

## 43.11 TL Explanation (speak this)

> "We use JWT because it's stateless — the token carries the user's identity and roles and is signed, so any service instance can verify it by checking the signature without a session store. The critical thing everyone should know is that the payload is only Base64-encoded, not encrypted, so anyone can read the claims; the signature proves it wasn't tampered with, nothing more. The trade-off we accept is that a token can't be revoked before it expires, which is why access tokens live fifteen minutes and we pair them with a longer refresh token that *is* stored in the database and can be deleted on logout. And we pin the signing algorithm server-side rather than trusting the token header, which is what prevents the `alg: none` forgery attack."

---
---

# PART E — BUILD, CONFIGURATION AND DEPLOYMENT

---

# 44. Maven

## 44.1 What is it?

**Simple words:**
Maven is a **build tool and dependency manager**. You list the libraries you need in one file, and Maven downloads them, compiles your code, runs the tests and packages everything into a JAR.

Before Maven, you downloaded JAR files manually and put them in a `lib` folder — and every library's own dependencies too.

**Technical wording:**
Maven is a build automation and project management tool based on the Project Object Model (POM). It provides declarative dependency management with transitive resolution, a standard build lifecycle, and a plugin-based execution model.

## 44.2 The POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>

    <!-- The Boot parent: manages versions for ~200 libraries -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>

    <!-- COORDINATES — uniquely identify this artifact -->
    <groupId>com.company</groupId>          <!-- organisation -->
    <artifactId>order-service</artifactId>  <!-- project name -->
    <version>1.0.0-SNAPSHOT</version>       <!-- version -->
    <packaging>jar</packaging>

    <properties>
        <java.version>17</java.version>
        <mapstruct.version>1.5.5.Final</mapstruct.version>
    </properties>

    <dependencies>
        <!-- No <version> needed — the parent manages it -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>          <!-- needed at runtime, not compile -->
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>       <!-- not transitive to consumers -->
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>             <!-- test compile + run only -->
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <!-- Repackages the JAR into an EXECUTABLE fat JAR -->
            </plugin>
        </plugins>
    </build>
</project>
```

### GAV coordinates
`groupId:artifactId:version` uniquely identifies any artifact — `com.company:order-service:1.0.0`.

**`SNAPSHOT` versions:** `1.0.0-SNAPSHOT` means "in development". Maven re-checks the remote repository for updates, so builds are **not reproducible**. A release version like `1.0.0` is immutable and cached forever. **Never deploy a SNAPSHOT to production** — you cannot guarantee which build is actually running.

## 44.3 Dependency scopes

| Scope | Compile | Test | Runtime | Packaged | Example |
|---|---|---|---|---|---|
| **`compile`** (default) | Yes | Yes | Yes | Yes | spring-boot-starter-web |
| **`provided`** | Yes | Yes | No | **No** | servlet-api, Tomcat for WAR |
| **`runtime`** | No | Yes | Yes | Yes | **JDBC drivers** |
| **`test`** | No | Yes | No | No | JUnit, Mockito |
| `system` | Yes | Yes | No | No | Deprecated — avoid |
| `import` | — | — | — | — | BOM import in `dependencyManagement` |

**Why a JDBC driver is `runtime`:** your code compiles against the JDBC *interfaces* in `java.sql`, never against MySQL classes. The driver is only needed when the application runs. Marking it `runtime` prevents anyone accidentally importing a vendor class and creating hidden coupling.

## 44.4 Transitive dependencies and conflicts

Maven pulls dependencies of dependencies automatically:
```
your project
 └── spring-boot-starter-web
      ├── spring-web
      ├── spring-webmvc
      ├── spring-boot-starter-tomcat → tomcat-embed-core
      └── spring-boot-starter-json   → jackson-databind → jackson-core
```

**Conflict resolution — the "nearest wins" rule:**
```
A → B → C → jackson 2.13      (depth 3)
A → D → jackson 2.15          (depth 2)   ← WINS: nearer to the root
```
Maven uses the **shortest path**, not the highest version. If two are at equal depth, the **first declared** wins.

**Diagnostic commands:**
```bash
mvn dependency:tree                             # full tree
mvn dependency:tree -Dincludes=com.fasterxml.jackson   # trace one library
mvn dependency:analyze                          # unused / undeclared dependencies
```

**Excluding a transitive dependency:**
```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>some-library</artifactId>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
**Typical real cause:** two libraries pull incompatible versions of the same dependency and you get `NoSuchMethodError` or `ClassNotFoundException` at runtime. `mvn dependency:tree` is the first command to run — it shows exactly which version won and why.

## 44.5 The build lifecycle

```
validate → compile → test → package → verify → install → deploy
```
Running a phase runs **all preceding phases**.

| Phase | Does |
|---|---|
| `validate` | Check the project is correct |
| `compile` | Compile sources to `target/classes` |
| `test` | Run unit tests (Surefire) |
| `package` | Build the JAR/WAR into `target/` |
| `verify` | Run integration tests (Failsafe) and checks |
| `install` | Copy the artifact to the **local** repository (`~/.m2`) |
| `deploy` | Upload to the **remote** repository (Nexus/Artifactory) |

```bash
mvn clean install                 # most common local command
mvn clean package -DskipTests     # skip tests (CI packaging step)
mvn test                          # run tests only
mvn spring-boot:run               # run the app without packaging
mvn versions:display-dependency-updates   # check for newer versions
```

> **`-DskipTests` vs `-Dmaven.test.skip=true`:** the first compiles tests but doesn't run them; the second skips compiling them too. Prefer `-DskipTests` so test code still has to compile — otherwise broken test code goes unnoticed.

## 44.6 Maven vs Gradle

| | Maven | Gradle |
|---|---|---|
| Config | XML (`pom.xml`) | Groovy/Kotlin DSL |
| Style | Declarative, convention-driven | Programmatic, flexible |
| Build speed | Slower | **Faster** (incremental + build cache) |
| Learning curve | **Lower** | Higher |
| Predictability | **High** — hard to do anything unusual | Lower — build logic can be arbitrary code |
| Enterprise adoption | **Very high** | Growing, dominant in Android |

**Balanced view:** Gradle is faster and more flexible; Maven is more predictable because the XML can't contain arbitrary logic. For corporate services where consistency across teams matters more than build speed, Maven remains the common choice.

## 44.7 Common Mistakes

| Mistake | Consequence |
|---|---|
| Specifying versions for Boot starters | Breaks the parent's curated version set |
| JDBC driver at `compile` scope | Encourages coupling to vendor classes |
| **Deploying a SNAPSHOT** | Non-reproducible build; unknown code in production |
| Ignoring `dependency:tree` on a `NoSuchMethodError` | Hours lost to a version conflict |
| `-Dmaven.test.skip=true` in CI | Test code rot goes undetected |
| Secrets in the POM | Committed to version control |
| Not pinning plugin versions | Builds change behaviour without a code change |

## 44.8 TL Questions

**Q: What does Maven do for us?**
A: Dependency management with transitive resolution, and a standard build lifecycle — compile, test, package. We declare what we need and it fetches the libraries and their dependencies.

**Q: Why don't we specify versions for Spring starters?**
A: The `spring-boot-starter-parent` manages them. It declares a tested, mutually compatible set of around two hundred library versions, so we avoid the version conflicts that used to dominate Java projects.

**Q: How does Maven resolve a version conflict?**
A: Nearest-wins — the shortest path from the root, not the highest version. When two are equally deep, the first declared wins. When we hit a `NoSuchMethodError` the first thing we run is `mvn dependency:tree` to see which version was actually selected.

**Q: Why is the JDBC driver `runtime` scope?**
A: Our code compiles only against the `java.sql` interfaces, never MySQL classes. Restricting it to runtime prevents anyone accidentally importing a vendor class and coupling us to that database.

**Q: Why shouldn't we deploy SNAPSHOT versions?**
A: Because a SNAPSHOT is mutable — Maven re-resolves it, so the same version string can mean different code at different times. We can't reproduce a production build or be certain what's running. Releases are immutable.

## 44.9 Interview Questions

**Beginner — What is a POM?**
The Project Object Model — the XML file describing coordinates, dependencies, properties, plugins and build configuration.

**Beginner — What is the local repository?**
`~/.m2/repository`, where Maven caches downloaded artifacts and installs locally built ones.

**Intermediate — Difference between `install` and `deploy`?**
`install` copies the artifact into the local `~/.m2` repository for use by other local projects. `deploy` uploads it to a shared remote repository such as Nexus or Artifactory.

**Intermediate — What does `spring-boot-maven-plugin` do?**
It repackages the plain JAR into an executable fat JAR — nesting dependencies under `BOOT-INF/lib`, adding a `JarLauncher` main class and a nested-JAR-aware classloader, so `java -jar` works.

**Advanced — What is a BOM?**
A Bill of Materials — a POM containing only `dependencyManagement` entries, imported with `<scope>import</scope>`. It centralises version decisions without forcing the dependencies themselves. `spring-boot-dependencies` is a BOM, used when you can't inherit from the Boot parent.

**Advanced — `<optional>true</optional>`?**
Marks a dependency as non-transitive — consumers of your artifact don't inherit it. Lombok uses this because it's compile-time only and consumers don't need it at runtime.

## 44.10 Quick Revision

1. Maven = dependency management + standard build lifecycle.
2. GAV: `groupId:artifactId:version`.
3. `spring-boot-starter-parent` manages ~200 library versions — don't override.
4. Scopes: `compile`, `provided`, `runtime`, `test`.
5. **JDBC drivers = `runtime`.**
6. Conflicts resolved by **nearest wins**, not highest version.
7. `mvn dependency:tree` is the first diagnostic for version problems.
8. Lifecycle: validate → compile → test → package → verify → install → deploy.
9. **Never deploy SNAPSHOT to production.**
10. `spring-boot-maven-plugin` produces the executable fat JAR.

## 44.11 TL Explanation (speak this)

> "Maven handles our dependencies and the build lifecycle. We inherit from `spring-boot-starter-parent`, which pins around two hundred library versions that are tested together — that's why we never specify versions for the starters and why we don't hit the version conflicts Java projects used to suffer from. When we do get a `NoSuchMethodError` at runtime, it's almost always a transitive conflict and `mvn dependency:tree` shows which version won, since Maven picks the nearest in the tree rather than the highest. We also keep releases immutable — no SNAPSHOT ever goes to production, because the same version string could mean different code."

---

# 45. Spring Boot Actuator and Production Readiness

## 45.1 What is it?

**Simple words:**
Actuator adds **ready-made endpoints that tell you how your application is doing** — is it healthy, how much memory is it using, what configuration is active, how many requests has it served.

**Technical wording:**
Spring Boot Actuator provides production-ready features including health indicators, metrics collection, environment introspection and operational endpoints, exposed over HTTP or JMX.

## 45.2 Setup and key endpoints

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus     # ONLY what we need
      base-path: /actuator
  endpoint:
    health:
      show-details: when-authorized                 # not "always" in production
      probes:
        enabled: true                               # /health/liveness, /health/readiness
  metrics:
    tags:
      application: ${spring.application.name}
```

| Endpoint | Shows |
|---|---|
| `/actuator/health` | UP/DOWN, plus per-component status |
| `/actuator/health/liveness` | Is the app alive? (Kubernetes) |
| `/actuator/health/readiness` | Is it ready for traffic? (Kubernetes) |
| `/actuator/info` | Build and git information |
| `/actuator/metrics` | JVM, HTTP, datasource metrics |
| `/actuator/prometheus` | Metrics in Prometheus format |
| `/actuator/env` | **All properties — sensitive** |
| `/actuator/beans` | Every bean in the context |
| `/actuator/conditions` | **Auto-configuration report** (Section 10) |
| `/actuator/loggers` | View **and change** log levels at runtime |
| `/actuator/threaddump` | Thread dump — sensitive |
| `/actuator/heapdump` | **Full heap dump — highly sensitive** |

### ★ Security warning ★

**Only `/health` and `/info` are exposed by default over HTTP** — but many teams set `include: *` for convenience and forget.

`/actuator/env` exposes **every property including database passwords**. `/actuator/heapdump` downloads the entire heap, which contains **tokens, passwords and customer data in memory**. An exposed actuator is a complete system compromise.

**Always:**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus    # explicit allowlist, never "*"
  server:
    port: 9090                             # separate port, not exposed publicly
```
```java
// And secure them
.requestMatchers("/actuator/health", "/actuator/info").permitAll()
.requestMatchers("/actuator/**").hasRole("ADMIN")
```

## 45.3 Liveness vs Readiness (essential for Kubernetes)

| Probe | Question | On failure |
|---|---|---|
| **Liveness** | Is the process alive and not deadlocked? | **Kubernetes RESTARTS the pod** |
| **Readiness** | Can it serve traffic right now? | **Removed from the load balancer** — not restarted |

**Why the distinction matters:** if the database is down, the app is *alive* but *not ready*. Reporting that as a **liveness** failure makes Kubernetes restart every pod in a loop, which does nothing to fix the database and turns a degraded service into a total outage. Database checks belong in **readiness**.

**This is a genuinely valuable point in any deployment discussion.**

## 45.4 Custom health indicator

```java
@Component
@RequiredArgsConstructor
public class PaymentGatewayHealthIndicator implements HealthIndicator {

    private final PaymentGatewayClient client;

    @Override
    public Health health() {
        try {
            if (client.ping()) {
                return Health.up().withDetail("gateway", "reachable").build();
            }
            return Health.down().withDetail("gateway", "unreachable").build();
        } catch (Exception e) {
            return Health.down(e).withDetail("gateway", "error").build();
        }
    }
}
```
**Caution:** any `HealthIndicator` contributes to the overall status, so a non-critical dependency reporting DOWN can mark your whole service DOWN and pull it out of the load balancer. Put non-critical checks in a separate group, or return `Health.up()` with a degraded detail.

## 45.5 Graceful shutdown

```yaml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```
On SIGTERM, Tomcat stops accepting new connections but **finishes in-flight requests** before closing. Without it, a deployment drops every request currently being processed — visible to users as random errors during every release.

## 45.6 TL Questions

**Q: What does Actuator give us?**
A: Production endpoints — health checks for the load balancer and Kubernetes, metrics for Prometheus, and operational tools like runtime log-level changes and the auto-configuration report.

**Q: How do we secure it?**
A: An explicit allowlist rather than `*`, health and info public, everything else behind an admin role, and ideally on a separate management port not exposed publicly. `/env` leaks passwords and `/heapdump` dumps everything in memory, so exposing actuator openly is a full compromise.

**Q: What's the difference between liveness and readiness?**
A: Liveness asks whether the process is alive — failing it restarts the pod. Readiness asks whether it can serve traffic — failing it just removes the pod from the load balancer. Database checks go in readiness, because if the database is down, restarting every pod makes things worse rather than better.

**Q: Why enable graceful shutdown?**
A: So a deployment finishes in-flight requests instead of dropping them. Without it, every release produces a burst of user-visible errors.

## 45.7 Quick Revision

1. Actuator = health, metrics, info, operational endpoints.
2. Only `health` and `info` are exposed by default — **never use `include: *`**.
3. `/env` leaks secrets; `/heapdump` leaks everything in memory.
4. **Liveness failure = restart; readiness failure = remove from LB.**
5. Database checks belong in **readiness**, not liveness.
6. `/actuator/conditions` explains auto-configuration decisions.
7. `/actuator/loggers` changes log levels without a restart.
8. Enable `server.shutdown=graceful`.
9. `/actuator/prometheus` for metrics scraping.

## 45.8 TL Explanation (speak this)

> "Actuator gives us the production endpoints — health for Kubernetes probes and the load balancer, and Prometheus metrics. We expose an explicit allowlist rather than everything, because `/env` returns all our properties including database passwords and `/heapdump` returns the entire heap, so an open actuator is a full compromise. The distinction we're careful about is liveness versus readiness: a database outage should fail readiness so pods leave the load balancer, not liveness, which would restart every pod in a loop and make a degraded service a total outage. We also enable graceful shutdown so deployments don't drop in-flight requests."

---

# 46. Packaging and Deployment

## 46.1 JAR vs WAR

| | Executable JAR | WAR |
|---|---|---|
| Server | **Embedded** (Tomcat inside) | External Tomcat/JBoss |
| Run with | `java -jar app.jar` | Deploy to `webapps/` |
| Server config | Properties | Server-managed |
| Docker/cloud | **Ideal** | Awkward |
| **Recommended** | **Yes — default** | Legacy/mandated environments |

```xml
<!-- WAR only if your environment requires it -->
<packaging>war</packaging>
```
```java
@SpringBootApplication
public class OrderServiceApplication extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(OrderServiceApplication.class);
    }
}
```
Also mark `spring-boot-starter-tomcat` as `provided`, or you ship two servlet containers.

### Fat JAR structure

```
app.jar
├── META-INF/MANIFEST.MF        Main-Class: org.springframework.boot.loader.JarLauncher
│                               Start-Class: com.company.OrderServiceApplication
├── org/springframework/boot/loader/    ← Boot's launcher + nested-JAR classloader
└── BOOT-INF/
    ├── classes/                ← your compiled code
    └── lib/                    ← every dependency JAR, nested
```
Standard Java **cannot** load a JAR inside a JAR, which is why Boot ships its own `LaunchedURLClassLoader`.

## 46.2 Docker

```dockerfile
# ---------- Multi-stage build ----------
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B          # cached layer — deps change rarely
COPY src ./src
RUN mvn clean package -DskipTests

# ---------- Runtime: JRE only, much smaller ----------
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Never run as root
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75.0 -XX:+UseG1GC"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**Three points worth explaining:**
1. **Multi-stage build** — the final image contains only the JRE and the JAR, not Maven and the full JDK. Image size drops from ~700MB to ~180MB.
2. **Non-root user** — a container breakout as root is far more dangerous.
3. **`-XX:MaxRAMPercentage`** — the JVM must respect the *container's* memory limit, not the host's. Without this, the JVM sizes its heap from the host's total RAM and gets OOM-killed by the container runtime. **This is a classic containerised-Java failure.**

## 46.3 Configuration in deployment

```yaml
# Kubernetes — config and secrets injected as environment variables
env:
  - name: SPRING_PROFILES_ACTIVE
    value: "prod"
  - name: SPRING_DATASOURCE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: order-service-secrets
        key: db-password
```
Relaxed binding (Section 13) maps `SPRING_DATASOURCE_PASSWORD` onto `spring.datasource.password` automatically. **One immutable image, configuration from the environment.**

## 46.4 Production checklist

| Area | Setting |
|---|---|
| Profile | `SPRING_PROFILES_ACTIVE=prod` via environment |
| Schema | `ddl-auto=validate` + Flyway/Liquibase |
| SQL logging | **Off** (`show-sql=false`) |
| `open-in-view` | **`false`** |
| Batch fetch | `default_batch_fetch_size=20` |
| Secrets | Environment / secrets manager — never in the image |
| Actuator | Allowlist only; separate port; secured |
| Shutdown | `server.shutdown=graceful` |
| Pool | Sized against DB `max_connections` ÷ instances |
| Timeouts | Connect + read timeouts on **every** HTTP client |
| Logging | JSON structured, with correlation ID |
| Health probes | Liveness and readiness configured correctly |
| Errors | No stack traces in responses |
| JVM | `MaxRAMPercentage` set for containers |

## 46.5 TL Questions

**Q: Why an executable JAR rather than a WAR?**
A: The server is embedded, so we ship one artifact that runs with `java -jar`. There's no Tomcat to install or version-match on the host, which is exactly what makes it work cleanly in a Docker image.

**Q: How does the fat JAR work?**
A: Dependencies are nested under `BOOT-INF/lib` and the manifest points at Boot's `JarLauncher`, which installs a classloader that can read JARs inside a JAR — something standard Java can't do.

**Q: Why a multi-stage Docker build?**
A: The build stage needs Maven and the full JDK; the runtime only needs a JRE and the JAR. Separating them takes the image from around seven hundred megabytes to under two hundred, which means faster pulls and a smaller attack surface.

**Q: Why set `MaxRAMPercentage`?**
A: Because the JVM sizes its heap from the machine's memory, and in a container that's the host's memory, not the container limit. Without it the JVM grows past the limit and gets OOM-killed. It's one of the most common containerised-Java failures.

**Q: How does configuration differ per environment?**
A: The image is identical everywhere. The profile and all secrets come in as environment variables from Kubernetes config maps and secrets, and Boot's relaxed binding maps them onto the properties automatically.

## 46.6 Quick Revision

1. Executable JAR with embedded server is the default.
2. Fat JAR: `BOOT-INF/classes` + `BOOT-INF/lib` + `JarLauncher`.
3. WAR only when the environment mandates it (`provided` Tomcat + `SpringBootServletInitializer`).
4. Multi-stage Docker builds; JRE base image; **non-root user**.
5. **Set `MaxRAMPercentage`** so the JVM respects the container limit.
6. One image, many environments — config from environment variables.
7. Production: `validate`, no SQL logging, `open-in-view=false`, graceful shutdown.
8. Secrets never in the image or the repo.

## 46.7 TL Explanation (speak this)

> "We ship an executable JAR with the server embedded, so deployment is just `java -jar` inside a container — no Tomcat to install or version-match. The Dockerfile is multi-stage, so the final image carries only a JRE and our JAR rather than Maven and the full JDK, and it runs as a non-root user. One setting that matters more than people expect is `MaxRAMPercentage`, because otherwise the JVM sizes its heap from the host's memory rather than the container limit and gets OOM-killed. The image itself is identical across environments — the profile and all secrets arrive as environment variables from Kubernetes."

---
---

# PART F — MASTER FLOWS AND FINAL REVISION

> This part consolidates everything above. If you only revise one section before
> a TL discussion or interview, revise this one.

---

# 47. Complete Spring Boot Architecture Flow

## 47.1 Application startup — what happens when you run the JAR

```
java -jar order-service.jar
        │
        ▼
1. JarLauncher starts, installs the nested-JAR classloader
        │
        ▼
2. main() calls SpringApplication.run(OrderServiceApplication.class, args)
        │
        ▼
3. Deduce application type: SERVLET (spring-webmvc on classpath)
        │
        ▼
4. Prepare the ENVIRONMENT
     - load application.yml, application-{profile}.yml
     - apply property precedence:
         command line > env vars > system props > external file > packaged file
     - activate profiles
        │
        ▼
5. Create the ApplicationContext
     (AnnotationConfigServletWebServerApplicationContext)
        │
        ▼
6. COMPONENT SCANNING from the main class's package downward
     register bean definitions for @Component/@Service/@Repository/
     @Controller/@Configuration
        │
        ▼
7. AUTO-CONFIGURATION (deferred, so USER beans are registered first)
     AutoConfigurationImportSelector reads AutoConfiguration.imports
     → ~150 candidates → evaluate @Conditional on each
     → @ConditionalOnMissingBean makes them BACK OFF where we defined our own
        │
        ▼
8. context.refresh()
     a. BeanFactoryPostProcessors run on bean DEFINITIONS
        (e.g. ${...} placeholder resolution)
     b. BeanPostProcessors registered
     c. Singletons instantiated:
            constructor → dependency injection → aware callbacks
            → @PostConstruct → afterPropertiesSet → initMethod
            → ★ BeanPostProcessor.postProcessAfterInitialization
                  = AOP PROXIES CREATED (@Transactional, @Cacheable, @Async)
     d. onRefresh() → START EMBEDDED TOMCAT
     e. DispatcherServlet registered at "/"
     f. RequestMappingHandlerMapping scans all @Controller beans
        and builds the URL → HandlerMethod registry
        │
        ▼
9. CommandLineRunner / ApplicationRunner beans execute
        │
        ▼
10. ApplicationReadyEvent published
        │
        ▼
    ★ APPLICATION IS SERVING TRAFFIC ★
```

**The one thing to remember from this diagram:** AOP proxies are created at step 8c, *after* initialization callbacks. That single fact explains why `@Transactional` fails in `@PostConstruct` and why self-invocation bypasses it.

## 47.2 The layered runtime architecture

```
                         CLIENT
                           │  HTTP + JSON
   ════════════════════════▼════════════════════════
                    EMBEDDED TOMCAT
                  (thread per request, default max 200)
   ─────────────────────────────────────────────────
                     FILTER CHAIN                        ← servlet level
     CharacterEncoding → CorrelationId → CORS →
     Spring Security (JWT) → RequestLogging
   ═════════════════════════════════════════════════
                  DISPATCHER SERVLET                     ← Spring MVC
     HandlerMapping → Interceptors → HandlerAdapter →
     ArgumentResolvers → MessageConverters
   ─────────────────────────────────────────────────
   ┌─────────────────────────────────────────────────┐
   │  CONTROLLER LAYER        @RestController         │
   │  - bind request, delegate, map status code       │
   │  - DTOs only. No business logic.                 │
   └───────────────────────┬─────────────────────────┘
                           │  DTO
   ┌───────────────────────▼─────────────────────────┐
   │  SERVICE LAYER           @Service                │
   │  - business rules                                │
   │  - ★ @Transactional BOUNDARY IS HERE ★          │
   │  - entity ↔ DTO mapping (inside the transaction) │
   │  - orchestrates repositories and other services  │
   └───────────────────────┬─────────────────────────┘
                           │  Entity
   ┌───────────────────────▼─────────────────────────┐
   │  REPOSITORY LAYER        JpaRepository           │
   │  - data access only, no business logic           │
   └───────────────────────┬─────────────────────────┘
                           │
   ┌───────────────────────▼─────────────────────────┐
   │  JPA / HIBERNATE                                 │
   │  Persistence Context (L1 cache, dirty checking)  │
   └───────────────────────┬─────────────────────────┘
                           │  SQL
   ┌───────────────────────▼─────────────────────────┐
   │  HikariCP  →  JDBC Driver  →  DATABASE           │
   └─────────────────────────────────────────────────┘

   CROSS-CUTTING (applied by AOP proxies at every layer):
     @Transactional · @PreAuthorize · @Cacheable · @Async · custom aspects
```

**Why layering matters — the argument to give a TL:** each layer has one responsibility and depends only on the layer beneath. The controller can be swapped for a Kafka listener without touching business logic; the repository can move from JPA to `JdbcTemplate` without touching the service. It also puts the transaction boundary in exactly one predictable place.

---

# 48. Complete REST API Request/Response Flow

**Scenario:** `POST /api/v1/orders` with a JWT, creating an order.

```
CLIENT
  POST /api/v1/orders
  Authorization: Bearer eyJhbGc...
  Content-Type: application/json
  { "customerId": "CUST-1", "items": [{ "productId": 10, "quantity": 2 }] }
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 1. TOMCAT                                                       ║
 ║    accept connection → take a worker thread → parse HTTP        ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 2. FILTER CHAIN                                                 ║
 ║    CorrelationIdFilter → generate/propagate X-Correlation-Id,   ║
 ║                          put in MDC (all logs now carry it)     ║
 ║    JwtAuthenticationFilter → verify signature, check exp,       ║
 ║                          load UserDetails,                      ║
 ║                          SecurityContextHolder.setAuthentication║
 ║    AuthorizationFilter → URL rules; not permitted → 403         ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 3. DISPATCHER SERVLET — doDispatch()                            ║
 ║                                                                 ║
 ║  3a. HandlerMapping                                             ║
 ║      match POST + /api/v1/orders                                ║
 ║      → HandlerExecutionChain(OrderController.createOrder,        ║
 ║                              [interceptors])                    ║
 ║      no match → 404                                             ║
 ║                                                                 ║
 ║  3b. Interceptor.preHandle()                                    ║
 ║      record start time; returns false → request stops           ║
 ║                                                                 ║
 ║  3c. HandlerAdapter = RequestMappingHandlerAdapter              ║
 ║                                                                 ║
 ║  3d. ARGUMENT RESOLUTION                                        ║
 ║      @RequestBody → RequestResponseBodyMethodProcessor          ║
 ║        → MappingJackson2HttpMessageConverter                    ║
 ║        → JSON deserialized into OrderRequestDto                 ║
 ║        → malformed JSON → HttpMessageNotReadableException → 400  ║
 ║      @Valid → Hibernate Validator runs the constraints          ║
 ║        → violation → MethodArgumentNotValidException → 400       ║
 ║          with a field → message map                             ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │  valid DTO
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 4. CONTROLLER                                                   ║
 ║    orderService.createOrder(dto)   ← one line, no logic here    ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 5. SERVICE  (through the @Transactional AOP proxy)              ║
 ║    ── TRANSACTION BEGINS ──                                     ║
 ║       connection borrowed from HikariCP, autoCommit=false,      ║
 ║       bound to this thread; persistence context opens           ║
 ║                                                                 ║
 ║    @PreAuthorize check (if present)                             ║
 ║    business rules → violation → domain exception                ║
 ║    mapper.toEntity(dto)                                         ║
 ║    repository.save(order)                                       ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 6. REPOSITORY → JPA → HIBERNATE                                 ║
 ║    persist() → entity MANAGED; INSERT queued (not yet sent)     ║
 ║    cascade applies to OrderItems                                ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 7. SERVICE: map entity → DTO                                    ║
 ║    ★ INSIDE the transaction, so lazy associations still load    ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 8. TRANSACTION COMMIT                                           ║
 ║    flush: dirty checking → INSERT/UPDATE statements sent        ║
 ║    commit → connection returned to the pool                     ║
 ║    persistence context closes → entities DETACHED               ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │  DTO (safe — plain object, no proxies)
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 9. RETURN VALUE HANDLING                                        ║
 ║    ResponseEntity.created(location).body(dto)                   ║
 ║    content negotiation → Jackson serializes DTO → JSON          ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 10. Interceptor.postHandle()  → skipped if an exception occurred║
 ║     Interceptor.afterCompletion() → ALWAYS runs (timing/cleanup)║
 ║     Filters unwind → MDC.clear()  ← MUST happen (pooled thread) ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
   HTTP/1.1 201 Created
   Location: /api/v1/orders/1001
   X-Correlation-Id: a1b2c3
   { "id": 1001, "status": "PENDING", "totalAmount": 1500.00 }
```

**How to narrate this in 45 seconds:**
> "Tomcat assigns a thread, filters handle the correlation ID and JWT authentication, then the DispatcherServlet finds the controller method through HandlerMapping. Jackson converts the JSON body to a DTO, validation runs, and the controller calls the service. The service's `@Transactional` proxy opens a transaction, the repository persists via Hibernate, and we map to a DTO *inside* the transaction so lazy fields still load. On commit Hibernate flushes and the connection returns to the pool. The DTO goes back through Jackson as JSON with a 201 and a Location header, interceptors record timing, and the filters clear the MDC."

---

# 49. Complete JPA/Hibernate Flow

```
 SERVICE METHOD (@Transactional)
        │
 ═══════▼══════════════════════════════════════════════════
 1. TRANSACTION BEGINS
      PlatformTransactionManager → HikariCP connection
      → setAutoCommit(false)
      → EntityManager + PERSISTENCE CONTEXT created
      → bound to the thread (TransactionSynchronizationManager)
 ═══════┬══════════════════════════════════════════════════
        │
 ┌──────▼───────────────────────────────────────────────────┐
 │ 2. READ:  orderRepository.findById(1L)                    │
 │      a. check the PERSISTENCE CONTEXT (L1 cache)          │
 │           present → return the SAME instance, NO SQL      │
 │      b. absent → SELECT * FROM orders WHERE id = 1        │
 │      c. build the entity, state = MANAGED                 │
 │      d. ★ take a SNAPSHOT of loaded values (dirty check) ★│
 │      e. lazy associations → proxies, not loaded yet       │
 └──────┬───────────────────────────────────────────────────┘
        │
 ┌──────▼───────────────────────────────────────────────────┐
 │ 3. MODIFY:  order.setStatus(SHIPPED)                      │
 │      in-memory only. NO SQL. NO save() needed.            │
 └──────┬───────────────────────────────────────────────────┘
        │
 ┌──────▼───────────────────────────────────────────────────┐
 │ 4. CREATE:  repository.save(newOrder)                     │
 │      id == null → persist() → MANAGED, INSERT QUEUED      │
 │      id != null → merge() → returns a DIFFERENT instance  │
 │      cascade propagates to children                       │
 └──────┬───────────────────────────────────────────────────┘
        │
 ┌──────▼───────────────────────────────────────────────────┐
 │ 5. LAZY ACCESS:  order.getItems().size()                  │
 │      proxy initialises → SELECT ... WHERE order_id = ?    │
 │      ★ in a loop over N parents → N+1 PROBLEM ★           │
 └──────┬───────────────────────────────────────────────────┘
        │
 ┌──────▼───────────────────────────────────────────────────┐
 │ 6. FLUSH  (before commit, or before an affected query)    │
 │      compare every managed entity to its snapshot         │
 │      generate SQL in fixed order:                         │
 │        INSERT → UPDATE → collection deletes →             │
 │        collection inserts → DELETE                        │
 │      @Version → optimistic lock check in the WHERE clause │
 └──────┬───────────────────────────────────────────────────┘
        │
 ═══════▼══════════════════════════════════════════════════
 7. COMMIT                       │  7'. ROLLBACK (RuntimeException)
      connection.commit()        │       connection.rollback()
      connection → pool          │       connection → pool
      context closes             │       context closes
      entities → DETACHED        │       entities → DETACHED
 ═══════════════════════════════════════════════════════════
        │
        ▼
  ⚠ AFTER THIS POINT:
     - entities are DETACHED → setters do nothing
     - touching a lazy field → LazyInitializationException
     → which is exactly why we map to DTOs INSIDE the transaction
```

**The five facts this diagram encodes:**
1. L1 cache is checked before any SELECT by ID.
2. Dirty checking replaces explicit `save()` calls.
3. Lazy access inside a loop is where N+1 is born.
4. Flush order is fixed — INSERTs before DELETEs.
5. After commit everything is detached — map to DTOs before that.

---

# 50. Complete Spring Security / JWT Flow

```
 ══════════════════ PHASE 1: LOGIN (once) ══════════════════

 POST /api/v1/auth/login  { "username": "raj", "password": "secret" }
        │
        ▼
 AuthController → authenticationManager.authenticate(token)
        │
        ▼
 ProviderManager → DaoAuthenticationProvider
        │
        ├─► userDetailsService.loadUserByUsername("raj")
        │       SELECT u.*, r.* FROM users u JOIN roles r ...
        │       → UserDetails(username, BCRYPT HASH, authorities)
        │
        └─► passwordEncoder.matches(raw, storedHash)
                BCrypt re-hashes the input with the stored salt
                and compares. The hash is NEVER decrypted.
        │
   ┌────┴─────┐
 match      no match
   │            │
   ▼            ▼
 Authenticated   BadCredentialsException
 Authentication      → AuthenticationEntryPoint → 401
   │                  (generic message — no user enumeration)
   ▼
 jwtService.generateAccessToken(userDetails)
     header  { alg: HS256, typ: JWT }
     payload { sub, roles, iat, exp(+15m), iss, jti }
     signature = HMACSHA256(header.payload, SECRET)
   │
   ▼
 refreshTokenService.create()  → row saved in DB (revocable)
   │
   ▼
 200 { accessToken, refreshToken, tokenType: "Bearer", expiresIn: 900 }


 ═════════ PHASE 2: EVERY SUBSEQUENT REQUEST ═════════

 GET /api/v1/orders    Authorization: Bearer eyJhbGc...
        │
        ▼
 JwtAuthenticationFilter (OncePerRequestFilter)
        │
        ├─ no "Bearer " header → continue UNAUTHENTICATED
        │     (public endpoints must still work)
        │
        └─ token present:
              1. verifyWith(key).parseSignedClaims(token)
                   ★ signature verified with the PINNED algorithm ★
                   invalid/forged → exception → 401
              2. check exp → expired → 401
              3. extract subject
              4. load UserDetails
              5. build UsernamePasswordAuthenticationToken
              6. SecurityContextHolder.getContext()
                          .setAuthentication(auth)   ← ThreadLocal
        │
        ▼
 AuthorizationFilter
        - URL rules (hasRole/permitAll/anyRequest().authenticated())
        - not permitted → AccessDeniedException
              → AccessDeniedHandler → 403
        │
        ▼
 DispatcherServlet → Controller
        │
        ▼
 Service  @PreAuthorize("#customerId == authentication.name or hasRole('ADMIN')")
        - fine-grained, SpEL over method arguments
        - fails → AccessDeniedException → 403
        │
        ▼
 Business logic → response


 ═══════════════ PHASE 3: TOKEN REFRESH ═══════════════

 Access token expires after 15 minutes
        │
        ▼
 POST /api/v1/auth/refresh  { refreshToken }
        │
        ▼
 Look up the refresh token IN THE DATABASE   ← deliberately STATEFUL
        - not found / revoked / expired → 401, force re-login
        - found → issue a NEW access token
                → ROTATE the refresh token (invalidate the old one)
        │
        ▼
 Reuse of an already-rotated refresh token
        → signals theft → revoke the whole family → force re-login


 ═══════════════ LOGOUT ═══════════════
 Delete the refresh token row.
 ⚠ The current access token stays valid for up to 15 more minutes.
   Immediate revocation requires a jti blacklist in Redis — which
   trades away some statelessness. That is a deliberate decision.
```

**The two sentences that show you understand JWT:**
> "The payload is Base64-encoded, not encrypted — anyone can read the claims, so nothing sensitive goes in them; the signature provides integrity, not confidentiality. And a JWT can't be revoked before it expires, which is why the access token is short-lived and paired with a database-backed refresh token that *can* be revoked."

---

# 51. Complete Exception Handling Flow

```
 EXCEPTION THROWN — where it happens determines what handles it
 ═══════════════════════════════════════════════════════════════

 ┌─ Thrown in a FILTER ──────────────────────────────────────┐
 │   ✘ @RestControllerAdvice CANNOT catch it                 │
 │     (filters run before DispatcherServlet in the stack)   │
 │   → handle inside the filter: write JSON manually         │
 │   → or Spring Security's AuthenticationEntryPoint (401)   │
 │                              AccessDeniedHandler    (403) │
 └───────────────────────────────────────────────────────────┘

 ┌─ Thrown in INTERCEPTOR / CONTROLLER / SERVICE / REPOSITORY ┐
 │   ✔ caught by DispatcherServlet                            │
 └────────────────────────┬───────────────────────────────────┘
                          ▼
              processHandlerException()
                          │
                          ▼
         HandlerExceptionResolver chain, in order:
                          │
    ┌─────────────────────▼──────────────────────────────┐
    │ 1. ExceptionHandlerExceptionResolver               │
    │      search @ExceptionHandler:                     │
    │        a) in the SAME controller  (wins)           │
    │        b) in @RestControllerAdvice                 │
    │      choose the MOST SPECIFIC exception type       │
    │      (by inheritance distance, NOT declaration     │
    │       order)                                        │
    │      found → invoke → build response → DONE        │
    └─────────────────────┬──────────────────────────────┘
                          │ not found
    ┌─────────────────────▼──────────────────────────────┐
    │ 2. ResponseStatusExceptionResolver                 │
    │      @ResponseStatus on the exception class,       │
    │      or ResponseStatusException                    │
    └─────────────────────┬──────────────────────────────┘
                          │ not handled
    ┌─────────────────────▼──────────────────────────────┐
    │ 3. DefaultHandlerExceptionResolver                 │
    │      HttpRequestMethodNotSupported     → 405       │
    │      HttpMediaTypeNotSupported         → 415       │
    │      MethodArgumentNotValid            → 400       │
    │      NoHandlerFound                    → 404       │
    └─────────────────────┬──────────────────────────────┘
                          │ still nothing
                          ▼
              forward to /error (BasicErrorController)
                    → default 500 JSON
```

## Standard exception → status mapping

| Exception | Status | Meaning |
|---|---|---|
| `MethodArgumentNotValidException` | 400 | `@Valid` on the body failed |
| `ConstraintViolationException` | 400 | `@Validated` on params failed |
| `HttpMessageNotReadableException` | 400 | Malformed JSON |
| `MethodArgumentTypeMismatchException` | 400 | Wrong parameter type |
| `AuthenticationException` | 401 | Not authenticated |
| `AccessDeniedException` | 403 | Authenticated, not permitted |
| `ResourceNotFoundException` (ours) | 404 | Missing resource |
| `HttpRequestMethodNotSupportedException` | 405 | Wrong HTTP method |
| `DuplicateResourceException` / `DataIntegrityViolationException` | 409 | Conflict |
| `OptimisticLockingFailureException` | 409 | Concurrent modification |
| `HttpMediaTypeNotSupportedException` | 415 | Wrong Content-Type |
| Business rule violation | 422 | Semantically invalid |
| `Exception` (catch-all) | 500 | **Bug — log with stack trace** |

## The rules

1. **Business exceptions extend `RuntimeException`** — checked exceptions do not roll back transactions.
2. **Never swallow** — catching without rethrowing commits a failed transaction.
3. **Never expose** stack traces, SQL or constraint names.
4. **Log by severity** — client mistakes at WARN, unexpected exceptions at ERROR with the trace.
5. **Always include a correlation ID** in the error response.
6. **Always have a catch-all** `@ExceptionHandler(Exception.class)`.

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

# 53. Filter vs Interceptor vs AOP — Complete Comparison

```
 REQUEST
    │
    ▼
 ┌───────────────────────────────────────────────────────────┐
 │ FILTER            (Servlet API — outside Spring MVC)      │
 │  sees: raw HttpServletRequest / HttpServletResponse       │
 │  ┌───────────────────────────────────────────────────────┐│
 │  │ DISPATCHER SERVLET                                     ││
 │  │  ┌───────────────────────────────────────────────────┐ ││
 │  │  │ INTERCEPTOR    (Spring MVC — knows HandlerMethod) │ ││
 │  │  │  ┌───────────────────────────────────────────────┐│ ││
 │  │  │  │ CONTROLLER                                     ││ ││
 │  │  │  │   ┌─────────────────────────────────────────┐ ││ ││
 │  │  │  │   │ AOP ASPECT (any Spring bean, any layer) │ ││ ││
 │  │  │  │   │   ┌───────────────────────────────────┐ │ ││ ││
 │  │  │  │   │   │ SERVICE / REPOSITORY METHOD       │ │ ││ ││
 │  │  │  │   │   └───────────────────────────────────┘ │ ││ ││
 │  │  │  │   └─────────────────────────────────────────┘ ││ ││
 │  │  │  └───────────────────────────────────────────────┘│ ││
 │  │  └───────────────────────────────────────────────────┘ ││
 │  └───────────────────────────────────────────────────────┘│
 └───────────────────────────────────────────────────────────┘
```

| Aspect | **Filter** | **Interceptor** | **AOP** |
|---|---|---|---|
| Specification | Jakarta Servlet | Spring MVC | Spring AOP / AspectJ |
| Scope | Every HTTP request | Mapped requests only | **Any Spring bean method** |
| Runs relative to others | **First** | After filters | Innermost |
| Layer | Servlet container | Web (DispatcherServlet) | **All layers** |
| Sees raw request/response | **Yes** | Yes (read-only use) | **No** |
| Can wrap/modify request | **Yes** | No | No |
| Knows the handler method | **No** | **Yes** (`HandlerMethod`) | Yes (`JoinPoint`) |
| Reads method annotations | No | **Yes** | **Yes** |
| Sees method arguments | No | No | **Yes** |
| Sees the return value | No | Via `ModelAndView` | **Yes** |
| Can prevent execution | Yes (skip `doFilter`) | Yes (`preHandle` false) | Yes (`@Around`, skip `proceed`) |
| Applies to static resources | **Yes** | Only if mapped | No |
| Applies when no handler matches | **Yes** | No | No |
| Works outside HTTP (jobs, Kafka) | No | No | **Yes** |
| Exception caught by `@ControllerAdvice` | **No** | Yes | Yes |
| Self-invocation limitation | No | No | **Yes** |
| Registration | `@Component` / `FilterRegistrationBean` | `WebMvcConfigurer.addInterceptors` | `@Aspect` + `@Component` |
| Ordering | `@Order` / `setOrder()` | `.order(n)` | `@Order` |

## Choosing — the decision rule

```
Do you need the raw request/response, or must it run for
EVERY request including unmapped URLs and static resources?
        │ YES → FILTER
        │        (auth, CORS, encoding, correlation ID, compression)
        ▼ NO
Does the logic depend on WHICH CONTROLLER METHOD is running,
and is it HTTP-specific?
        │ YES → INTERCEPTOR
        │        (per-endpoint auditing, request timing, annotation checks)
        ▼ NO
Does it apply to service/repository methods, or must it also work
outside HTTP (scheduled jobs, message listeners)?
        │ YES → AOP
                 (transactions, caching, retry, business auditing)
```

**Real-world assignment in a typical service:**

| Concern | Use |
|---|---|
| JWT authentication | **Filter** (Spring Security) |
| CORS | **Filter** |
| Correlation ID / MDC | **Filter** |
| Request/response logging | **Filter** |
| Request timing, slow-request alerts | **Interceptor** |
| Annotation-driven endpoint auditing | **Interceptor** |
| `@Transactional` | **AOP** |
| `@Cacheable`, `@Async`, `@Retryable` | **AOP** |
| Business audit trail (`@Auditable`) | **AOP** |
| Method-level security (`@PreAuthorize`) | **AOP** |

---

# 54. Most Important Spring Boot Annotations

## Core / Configuration

| Annotation | What it does |
|---|---|
| `@SpringBootApplication` | `@SpringBootConfiguration` + `@EnableAutoConfiguration` + `@ComponentScan` |
| `@Configuration` | Class declares `@Bean` methods; CGLIB-proxied by default |
| `@Bean` | Method's return value becomes a managed bean |
| `@ComponentScan` | Which packages to scan |
| `@EnableAutoConfiguration` | Turn on conditional auto-configuration |
| `@Import` | Pull in another configuration class |
| `@Profile` | Register a bean only for given profiles |
| `@Conditional...` | `OnClass`, `OnMissingBean`, `OnProperty` — conditional registration |

## Stereotypes and DI

| Annotation | What it does |
|---|---|
| `@Component` | Generic managed bean |
| `@Service` | Business logic layer (semantic; = `@Component`) |
| `@Repository` | Data layer **+ exception translation** |
| `@Controller` | Web layer returning view names |
| `@RestController` | `@Controller` + `@ResponseBody` — REST APIs |
| `@Autowired` | Inject a dependency (optional on a single constructor) |
| `@Qualifier` | Choose among multiple candidate beans |
| `@Primary` | Default candidate when several match |
| `@Value` | Inject a single property or SpEL value |
| `@ConfigurationProperties` | **Type-safe, validated, relaxed-binding group of properties** |
| `@Scope` | singleton / prototype / request / session |
| `@Lazy` | Defer creation until first use |
| `@PostConstruct` / `@PreDestroy` | Lifecycle callbacks |

## Web / REST

| Annotation | What it does |
|---|---|
| `@RequestMapping` | Map requests (path, method, headers, consumes, produces) |
| `@GetMapping` … `@DeleteMapping` | Method-specific shortcuts |
| `@PathVariable` | Bind a URL path segment |
| `@RequestParam` | Bind a query parameter or form field |
| `@RequestBody` | Deserialize the body (one per method) |
| `@RequestHeader` / `@CookieValue` | Bind a header / cookie |
| `@ResponseBody` | Serialize the return value |
| `@ResponseStatus` | Fixed status code |
| `@RestControllerAdvice` | **Global exception handling** |
| `@ExceptionHandler` | Handle a specific exception type |
| `@CrossOrigin` | Per-controller CORS |

## Validation

| Annotation | What it does |
|---|---|
| `@Valid` | Trigger validation; **cascades** into nested objects |
| `@Validated` | Spring's variant: class-level param validation + **groups** |
| `@NotNull` / `@NotEmpty` / `@NotBlank` | Null / empty / blank checks |
| `@Size`, `@Min`, `@Max`, `@DecimalMin`, `@Digits` | Bounds |
| `@Email`, `@Pattern` | Format |
| `@Past`, `@Future` | Temporal |

## JPA / Persistence

| Annotation | What it does |
|---|---|
| `@Entity` / `@Table` | Map a class to a table |
| `@Id` / `@GeneratedValue` | Primary key and generation strategy |
| `@Column` | Column mapping and constraints |
| `@Enumerated(EnumType.STRING)` | **Always STRING, never ORDINAL** |
| `@Transient` | Exclude from persistence |
| `@Version` | **Optimistic locking** |
| `@OneToMany` / `@ManyToOne` / `@OneToOne` / `@ManyToMany` | Relationships |
| `@JoinColumn` / `@JoinTable` / `mappedBy` | FK column / join table / inverse side |
| `@MappedSuperclass` | Share columns without a table |
| `@Embeddable` / `@Embedded` | Value object in the same table |
| `@Query` | JPQL or native query |
| `@Modifying` | Required for UPDATE/DELETE queries |
| `@EntityGraph` | **Declarative fetch plan — the N+1 fix** |
| `@Transactional` | **Declarative transaction boundary** |
| `@Lock` | Pessimistic locking |
| `@CreationTimestamp` / `@CreatedDate` | Auditing timestamps |

## AOP / Security / Misc

| Annotation | What it does |
|---|---|
| `@Aspect` | Declares an aspect class |
| `@Before` / `@After` / `@AfterReturning` / `@AfterThrowing` / `@Around` | Advice types |
| `@Pointcut` | Named, reusable pointcut expression |
| `@EnableWebSecurity` | Enable Spring Security web configuration |
| `@EnableMethodSecurity` | Enable `@PreAuthorize` / `@PostAuthorize` |
| `@PreAuthorize` / `@PostAuthorize` | Method-level authorization (SpEL) |
| `@Async` / `@EnableAsync` | Run on a separate thread |
| `@Scheduled` / `@EnableScheduling` | Scheduled execution |
| `@Cacheable` / `@CacheEvict` | Caching |
| `@EventListener` / `@TransactionalEventListener` | Application events |
| `@SpringBootTest` / `@DataJpaTest` / `@WebMvcTest` | Test slices |

## The five annotations that are all AOP proxies

**`@Transactional`, `@Async`, `@Cacheable`, `@PreAuthorize`, `@Validated`** — all proxy-based, therefore **all** broken by self-invocation, non-public methods, and `new`-ed objects. Understanding that one mechanism explains five separate "why isn't this working?" problems at once.

---

# 55. Most Important TL / Senior-Level Questions

These are the questions that distinguish someone who has *used* Spring Boot from someone who *understands* it. Each answer is written to be spoken aloud.

**1. Walk me through what happens when a request hits our API.**
> Tomcat takes a worker thread and the filter chain runs — correlation ID, then JWT authentication which populates the security context. The DispatcherServlet asks HandlerMapping which controller method matches, a HandlerAdapter invokes it, and argument resolvers with Jackson turn the JSON body into our DTO, with validation running right after binding. The controller calls the service, whose `@Transactional` proxy opens a transaction, and the repository goes through Hibernate to the database. We map the entity to a DTO inside the transaction, commit, and the DTO is serialized back to JSON. Interceptors handle timing and the filters clear the MDC.

**2. Why doesn't `@Transactional` work sometimes?**
> Four reasons, all from it being proxy-based. Self-invocation — an internal call goes through `this` and skips the proxy. A non-public method, which CGLIB can't proxy. A checked exception, because Spring only rolls back on unchecked ones by default. And catching an exception without rethrowing, which lets the method return normally so the transaction commits. The same limitations apply to `@Cacheable`, `@Async` and `@PreAuthorize`, because they're all proxies too.

**3. What is the N+1 problem and how do we handle it?**
> One query loads N parents and then each parent's lazy association fires its own query. The reason it's dangerous is that it looks fine with ten rows in development and becomes a timeout with ten thousand in production. We fix it per-query with `@EntityGraph` or a join fetch, use DTO projections for read-only list screens, and set `default_batch_fetch_size` globally as a safety net. What we don't do is switch to EAGER — that makes every query pay the cost permanently.

**4. Why do we use DTOs instead of returning entities?**
> Security, because entities contain fields like password hashes that Jackson would happily serialize. Decoupling, so a column rename isn't a breaking API change. And stability — serializing a lazy association after the transaction closes throws `LazyInitializationException`, and bidirectional relationships recurse infinitely in Jackson. We map inside the service, inside the transaction, so the entity never leaves that layer.

**5. Explain auto-configuration.**
> `@EnableAutoConfiguration` imports a selector that reads the `AutoConfiguration.imports` files from every jar, giving around a hundred and fifty candidate configuration classes. Each is evaluated against its `@Conditional` annotations — is this class on the classpath, does this property exist, does this bean already exist. The key one is `@ConditionalOnMissingBean`: the moment we define our own bean of that type, the auto-configuration backs off. That's what makes Boot opinionated but not restrictive.

**6. How do we secure the API?**
> Spring Security's filter chain runs before the DispatcherServlet. A JWT filter verifies the signature and expiry and populates the security context, then URL rules check authorization, ending with `anyRequest().authenticated()` so anything unlisted is protected by default. We add `@PreAuthorize` on services for finer rules. Passwords are BCrypt. Security exceptions are handled by an `AuthenticationEntryPoint` for 401 and an `AccessDeniedHandler` for 403, because `@RestControllerAdvice` can't catch filter exceptions.

**7. What's the biggest performance risk in our data layer?**
> Two things. N+1 queries from lazy loading in a loop, and long transactions. Transactions hold a pooled database connection for their whole duration, so if we make an external HTTP call inside one and that service is slow, every in-flight request holds a connection until the pool drains and we stop responding — brought down by someone else's outage. So we keep transactions short and free of network I/O.

**8. How would you debug a production issue in this service?**
> Start from the correlation ID. Every request gets one in a filter, it goes into the MDC so every log line carries it, and it's returned in the response and error bodies. With that one ID we can trace the request across every service. Beyond that, Actuator metrics — especially the Hikari pending-connections gauge, which is the earliest sign of pool pressure — and the conditions report if something isn't configured as expected.

**9. What would you change about our current setup?**
> Three things I'd check first. Whether `spring.jpa.open-in-view` is disabled — it's on by default and hides N+1 queries while holding connections through serialization. Whether `default_batch_fetch_size` is set, which is a one-line global improvement. And whether Actuator is exposing more than health and info, because `/env` returns our passwords and `/heapdump` returns everything in memory.

**10. How do you decide between a filter, an interceptor and AOP?**
> Filter if it needs the raw request or must cover everything including unmapped URLs — authentication, CORS, correlation IDs. Interceptor if the logic depends on which controller method is running and is HTTP-specific. AOP if it applies to service or repository methods, or must also work outside HTTP, like in a scheduled job or a Kafka listener.

**11. Singleton beans and thread safety — what's the risk?**
> Every controller, service and repository is a singleton shared across all request threads. So any mutable instance field is shared concurrent state, and under load one request's data leaks into another's. We keep them stateless with all per-request data in method parameters. Where we genuinely need per-request state we use a request-scoped bean, which Spring injects as a proxy that resolves per thread.

**12. Why do our business exceptions extend `RuntimeException`?**
> Two reasons. Checked exceptions force `throws` declarations through every layer. More importantly, Spring rolls back automatically on unchecked exceptions but commits on checked ones — so a checked business exception would let a half-completed transaction commit and leave inconsistent data with no error anywhere.

**13. What's the trade-off with JWT?**
> We get statelessness, so any instance can validate any request and we scale horizontally without sticky sessions. What we give up is revocation — a token stays valid until it expires and we can't invalidate it. That's why access tokens live fifteen minutes and we pair them with a database-backed refresh token that can be revoked on logout. And the payload is only encoded, not encrypted, so nothing sensitive goes in the claims.

**14. How do we handle schema changes?**
> Flyway migrations with `ddl-auto=validate`. Hibernate verifies the schema matches our entities and refuses to start if it doesn't, so a missed migration fails the deployment rather than surfacing as a runtime error. We never use `ddl-auto=update` in production — it never drops columns, gives no migration history and no rollback path.

**15. How do you size the connection pool?**
> Small and deliberate, roughly twice the database server's core count, which lands near Hikari's default of ten. Bigger pools usually reduce throughput because the database context-switches and contends more. The number people forget is that total connections equal pool size times instance count — ten pods at twenty each already exceeds MySQL's default limit of a hundred and fifty-one.

---

# 56. One-Page Spring Boot Quick Revision Sheet

## The mental model
```
Spring Framework  = IoC container + DI + AOP
Spring Boot       = Spring + auto-config + starters + embedded server
JPA               = specification (interfaces only)
Hibernate         = the implementation that does the work
Spring Data JPA   = generates repository implementations
```

## Request flow (say this in one breath)
```
Tomcat → Filters → DispatcherServlet → HandlerMapping → Interceptor.preHandle
→ HandlerAdapter → ArgumentResolvers + Jackson → @Valid → CONTROLLER
→ SERVICE (@Transactional) → REPOSITORY → Hibernate → DB
→ map to DTO (inside tx) → COMMIT → Jackson → postHandle → afterCompletion
→ Response
```

## The ten rules that prevent most production bugs
1. `@Transactional` on the **service**, never the controller. Keep it short. **No HTTP calls inside.**
2. **Never return entities** — return DTOs, mapped inside the transaction.
3. Set every `ToOne` association to **LAZY** explicitly.
4. `spring.jpa.open-in-view=false` and `default_batch_fetch_size=20`.
5. `ddl-auto=validate` in production, with Flyway migrations.
6. **Constructor injection** with `final` fields — never field injection.
7. Singletons are **stateless** — no mutable instance fields.
8. Business exceptions extend **`RuntimeException`**; never swallow them.
9. `@Enumerated(EnumType.STRING)`; `BigDecimal` for money.
10. Actuator: **explicit allowlist**, never `include: *`.

## Things that silently do nothing
| It looks right, but... | Why |
|---|---|
| `@Transactional` on a self-invoked method | Proxy bypassed |
| `@Transactional` in `@PostConstruct` | Proxy not created yet |
| `@Transactional` with a checked exception | No rollback |
| Modifying a **detached** entity | No dirty checking |
| `@Valid` without `spring-boot-starter-validation` | Validator absent |
| `@Valid` missing on a nested object | No cascade |
| `@NotNull` on a String | `""` passes |
| Interceptor not added in `addInterceptors` | Never registered |
| Filter missing `chain.doFilter()` | Request hangs |
| Prototype injected into a singleton | Injected once |
| `@Autowired` on a static field | Ignored |
| `merge()`/`save()` return value ignored | Working with a detached copy |

## Status codes
`200` OK · `201` Created (+`Location`) · `204` No Content
`400` validation · **`401` not authenticated** · **`403` not permitted** · `404` missing
`409` conflict · `415` wrong Content-Type · `422` business rule · `429` rate limit
`500` our bug · `503` unavailable
> **4xx = client's problem. 5xx = our problem. Never 500 for validation.**

## Comparison one-liners
| Pair | The distinction |
|---|---|
| IoC vs DI | Principle vs the pattern implementing it |
| `@Controller` vs `@RestController` | View name vs response body |
| `@Valid` vs `@Validated` | Jakarta + cascade vs Spring + groups + param validation |
| `@PathVariable` vs `@RequestParam` vs `@RequestBody` | Which resource / how to fetch / the data |
| `@Component` vs `@Bean` | My class vs a third-party class |
| `CrudRepository` vs `JpaRepository` | Store-agnostic vs JPA-specific with `List` + batch |
| JPA vs Hibernate | Specification vs implementation |
| L1 vs L2 cache | Transaction-scoped, always on vs app-wide, opt-in |
| LAZY vs EAGER | Deferred, upgradeable vs always paid |
| Optimistic vs pessimistic lock | Version check at commit vs row lock upfront |
| `REQUIRED` vs `REQUIRES_NEW` | Join vs suspend + new connection |
| Filter vs Interceptor vs AOP | Servlet / knows handler / any bean any layer |
| 401 vs 403 | Don't know you vs know you, not allowed |
| Session vs JWT | Server state, revocable vs stateless, not revocable |

## Bean lifecycle
```
instantiate → inject → aware → BPP.before → @PostConstruct
→ afterPropertiesSet → initMethod → ★ BPP.after = AOP PROXY CREATED ★
→ IN USE → @PreDestroy → destroy() → destroyMethod
```

## Entity states
```
TRANSIENT --persist--> MANAGED --tx ends--> DETACHED --merge--> MANAGED
                          │
                       remove()
                          ▼
                       REMOVED
```
**Only MANAGED entities get dirty checking. Detached changes are silently lost.**

## Five annotations that are all AOP proxies
`@Transactional` · `@Async` · `@Cacheable` · `@PreAuthorize` · `@Validated`
→ all broken by **self-invocation**, non-public methods, and `new`-ed objects.

## Production configuration
```yaml
spring:
  jpa:
    open-in-view: false
    hibernate.ddl-auto: validate
    show-sql: false
    properties.hibernate.jdbc.batch_size: 50
    properties.hibernate.default_batch_fetch_size: 20
  datasource:
    hikari:
      maximum-pool-size: 10        # × instances must be < DB max_connections
      max-lifetime: 1700000        # BELOW the firewall/DB idle timeout
server:
  shutdown: graceful
management:
  endpoints.web.exposure.include: health,info,prometheus   # never "*"
```

---

## Final note

This document was built from the topic curriculum you supplied, not from a video
transcript — the workspace was empty when it was generated. When you send your
playlist parts, each one will be merged into this same document: sections will be
deepened where your instructor goes further, new sections added where your
playlist covers something not here, and anything that contradicts what is written
above will be flagged explicitly rather than silently overwritten.

Anything marked **Additional clarification** is context added beyond a standard
tutorial treatment — useful for TL discussions, but worth confirming against your
own project's conventions before quoting it as our team's practice.
