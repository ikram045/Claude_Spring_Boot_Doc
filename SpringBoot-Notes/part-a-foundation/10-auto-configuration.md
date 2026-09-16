[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

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

[← 9. @SpringBootApplication](09-springbootapplication.md) | [Contents](../README.md) | [11. Component Scanning →](11-component-scanning.md)
