# Spring Boot — Complete Study Notes

Structured study notes for corporate Spring Boot development, written to be
read chapter by chapter and used for TL discussions and interviews.

> **Start here:** [About this document](00-about.md) — how to use these notes,
> and where the content came from.

---

## Contents

### [Part A — Foundation (Core Spring)](part-a-foundation/README.md)

Core Spring concepts everything else is built on: the container, dependency injection, bean lifecycle, and how a Boot application starts.

| # | Chapter |
|---|---|
| 1 | [Spring Framework](part-a-foundation/01-spring-framework.md) |
| 2 | [Spring Boot](part-a-foundation/02-spring-boot.md) |
| 3 | [Inversion of Control (IoC)](part-a-foundation/03-inversion-of-control-ioc.md) |
| 4 | [Dependency Injection (DI)](part-a-foundation/04-dependency-injection-di.md) |
| 5 | [Beans](part-a-foundation/05-beans.md) |
| 6 | [Bean Lifecycle](part-a-foundation/06-bean-lifecycle.md) |
| 7 | [Bean Scopes](part-a-foundation/07-bean-scopes.md) |
| 8 | [ApplicationContext and BeanFactory](part-a-foundation/08-applicationcontext-and-beanfactory.md) |
| 9 | [@SpringBootApplication](part-a-foundation/09-springbootapplication.md) |
| 10 | [Auto-Configuration](part-a-foundation/10-auto-configuration.md) |
| 11 | [Component Scanning](part-a-foundation/11-component-scanning.md) |
| 12 | [Java Configuration — @Configuration and @Bean](part-a-foundation/12-java-configuration-configuration-and-bean.md) |
| 13 | [application.properties and application.yml](part-a-foundation/13-application-properties-and-application-yml.md) |
| 14 | [Profiles](part-a-foundation/14-profiles.md) |

### [Part B — Web Layer (Spring MVC / REST)](part-b-web-layer/README.md)

How an HTTP request becomes a Java method call and comes back as JSON, plus the cross-cutting layers around it.

| # | Chapter |
|---|---|
| 15 | [Spring MVC and the DispatcherServlet](part-b-web-layer/15-spring-mvc-and-the-dispatcherservlet.md) |
| 16 | [HandlerMapping and HandlerAdapter](part-b-web-layer/16-handlermapping-and-handleradapter.md) |
| 17 | [REST API Fundamentals](part-b-web-layer/17-rest-api-fundamentals.md) |
| 18 | [Controllers](part-b-web-layer/18-controllers.md) |
| 19 | [Reading Request Data — @PathVariable, @RequestParam, @RequestBody](part-b-web-layer/19-reading-request-data-pathvariable-requestparam-requestbody.md) |
| 20 | [Sending Responses — ResponseEntity and Status Codes](part-b-web-layer/20-sending-responses-responseentity-and-status-codes.md) |
| 21 | [DTO Pattern](part-b-web-layer/21-dto-pattern.md) |
| 22 | [Validation](part-b-web-layer/22-validation.md) |
| 23 | [Exception Handling](part-b-web-layer/23-exception-handling.md) |
| 24 | [Filters](part-b-web-layer/24-filters.md) |
| 25 | [Interceptors](part-b-web-layer/25-interceptors.md) |
| 26 | [AOP (Aspect Oriented Programming)](part-b-web-layer/26-aop-aspect-oriented-programming.md) |

### [Part C — Data Layer (JDBC / JPA / Hibernate)](part-c-data-layer/README.md)

Talking to the database: connection pooling, ORM, the persistence context, relationships, and transactions.

| # | Chapter |
|---|---|
| 27 | [JDBC and DataSource](part-c-data-layer/27-jdbc-and-datasource.md) |
| 28 | [HikariCP (Connection Pooling)](part-c-data-layer/28-hikaricp-connection-pooling.md) |
| 29 | [JdbcTemplate](part-c-data-layer/29-jdbctemplate.md) |
| 30 | [JPA vs Hibernate vs Spring Data JPA](part-c-data-layer/30-jpa-vs-hibernate-vs-spring-data-jpa.md) |
| 31 | [Entity and Mapping Annotations](part-c-data-layer/31-entity-and-mapping-annotations.md) |
| 32 | [EntityManager and Persistence Context](part-c-data-layer/32-entitymanager-and-persistence-context.md) |
| 33 | [First Level Cache](part-c-data-layer/33-first-level-cache.md) |
| 34 | [Entity Lifecycle States](part-c-data-layer/34-entity-lifecycle-states.md) |
| 35 | [Spring Data Repositories](part-c-data-layer/35-spring-data-repositories.md) |
| 36 | [Query Methods, JPQL, @Query and Pagination](part-c-data-layer/36-query-methods-jpql-query-and-pagination.md) |
| 37 | [Entity Relationships](part-c-data-layer/37-entity-relationships.md) |
| 38 | [Cascade Types](part-c-data-layer/38-cascade-types.md) |
| 39 | [Lazy vs Eager Loading](part-c-data-layer/39-lazy-vs-eager-loading.md) |
| 40 | [The N+1 Select Problem and EntityGraph](part-c-data-layer/40-the-n-1-select-problem-and-entitygraph.md) |
| 41 | [Transactions and @Transactional](part-c-data-layer/41-transactions-and-transactional.md) |

### [Part D — Security](part-d-security/README.md)

Authentication and authorization with Spring Security, and stateless tokens with JWT.

| # | Chapter |
|---|---|
| 42 | [Spring Security](part-d-security/42-spring-security.md) |
| 43 | [JWT (JSON Web Token) Authentication](part-d-security/43-jwt-json-web-token-authentication.md) |

### [Part E — Build, Configuration and Deployment](part-e-build-deployment/README.md)

Turning the code into a running, observable, deployable service.

| # | Chapter |
|---|---|
| 44 | [Maven](part-e-build-deployment/44-maven.md) |
| 45 | [Spring Boot Actuator and Production Readiness](part-e-build-deployment/45-spring-boot-actuator-and-production-readiness.md) |
| 46 | [Packaging and Deployment](part-e-build-deployment/46-packaging-and-deployment.md) |

### [Part F — Master Flows and Final Revision](part-f-master-flows/README.md)

Consolidation. End-to-end flows and revision material — read this part before a TL discussion or interview.

| # | Chapter |
|---|---|
| 47 | [Complete Spring Boot Architecture Flow](part-f-master-flows/47-complete-spring-boot-architecture-flow.md) |
| 48 | [Complete REST API Request/Response Flow](part-f-master-flows/48-complete-rest-api-request-response-flow.md) |
| 49 | [Complete JPA/Hibernate Flow](part-f-master-flows/49-complete-jpa-hibernate-flow.md) |
| 50 | [Complete Spring Security / JWT Flow](part-f-master-flows/50-complete-spring-security-jwt-flow.md) |
| 51 | [Complete Exception Handling Flow](part-f-master-flows/51-complete-exception-handling-flow.md) |
| 52 | [Complete Transaction Flow](part-f-master-flows/52-complete-transaction-flow.md) |
| 53 | [Filter vs Interceptor vs AOP — Complete Comparison](part-f-master-flows/53-filter-vs-interceptor-vs-aop-complete-comparison.md) |
| 54 | [Most Important Spring Boot Annotations](part-f-master-flows/54-most-important-spring-boot-annotations.md) |
| 55 | [Most Important TL / Senior-Level Questions](part-f-master-flows/55-most-important-tl-senior-level-questions.md) |
| 56 | [One-Page Spring Boot Quick Revision Sheet](part-f-master-flows/56-one-page-spring-boot-quick-revision-sheet.md) |

---

## How each chapter is structured

Most chapters follow the same shape, so you can navigate them predictably:

| Section | Purpose |
|---|---|
| **What is it? / Why do we use it?** | Plain English first, then the technical wording |
| **How does it work?** | Internals and request/data flow, with ASCII diagrams |
| **Code example** | Realistic, layered, with the important lines explained |
| **Common Mistakes** | What actually breaks in production |
| **TL Questions** | Practical questions with answers you can speak aloud |
| **Interview Questions** | Beginner → intermediate → advanced |
| **Quick Revision** | 5–10 points for last-minute review |
| **TL Explanation** | A 3–5 line answer to speak directly to your TL/senior |

---

## Suggested reading order

**If you are new to Spring Boot:** Parts A → B → C in order. Do not skip Part A —
Parts B and C assume the container and bean lifecycle are understood.

**If you are preparing for a discussion tomorrow:** go straight to
[Part F](part-f-master-flows/README.md), then read the *TL Explanation* box at the
end of each chapter in the topic you will be asked about.

**The three highest-value chapters,** because each explains several behaviours at once:

- [26. AOP](part-b-web-layer/26-aop-aspect-oriented-programming.md) — why `@Transactional`, `@Cacheable`, `@Async`, `@PreAuthorize` and `@Validated` *all* fail on self-invocation
- [32. EntityManager and Persistence Context](part-c-data-layer/32-entitymanager-and-persistence-context.md) — first-level cache, dirty checking, lazy loading and entity states all follow from this
- [56. Quick Revision Sheet](part-f-master-flows/56-one-page-spring-boot-quick-revision-sheet.md) — includes a *things that silently do nothing* table

---

## Provenance

These notes were generated from a **topic curriculum**, not from a video transcript —
no playlist content had been supplied when they were written. Standard, widely-accepted
Spring Boot behaviour is used throughout, and anything beyond a normal tutorial treatment
is marked **Additional clarification**. See [About this document](00-about.md) for detail.
