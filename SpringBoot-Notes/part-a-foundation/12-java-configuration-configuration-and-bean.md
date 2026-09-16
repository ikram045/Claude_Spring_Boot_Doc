[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

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

[← 11. Component Scanning](11-component-scanning.md) | [Contents](../README.md) | [13. application.properties and application.yml →](13-application-properties-and-application-yml.md)
