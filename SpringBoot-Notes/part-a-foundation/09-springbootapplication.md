[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

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

[← 8. ApplicationContext and BeanFactory](08-applicationcontext-and-beanfactory.md) | [Contents](../README.md) | [10. Auto-Configuration →](10-auto-configuration.md)
