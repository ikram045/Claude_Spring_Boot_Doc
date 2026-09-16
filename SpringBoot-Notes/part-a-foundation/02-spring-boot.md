[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

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

[← 1. Spring Framework](01-spring-framework.md) | [Contents](../README.md) | [3. Inversion of Control (IoC) →](03-inversion-of-control-ioc.md)
