[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

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

&nbsp; | [Contents](../README.md) | [2. Spring Boot →](02-spring-boot.md)
