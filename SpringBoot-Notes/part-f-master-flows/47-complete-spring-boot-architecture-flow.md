[← Back to Contents](../README.md) · Part F — Master Flows and Final Revision

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

[← 46. Packaging and Deployment](../part-e-build-deployment/46-packaging-and-deployment.md) | [Contents](../README.md) | [48. Complete REST API Request/Response Flow →](48-complete-rest-api-request-response-flow.md)
