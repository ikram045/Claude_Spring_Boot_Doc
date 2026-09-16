[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

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

[← 12. Java Configuration — @Configuration and @Bean](12-java-configuration-configuration-and-bean.md) | [Contents](../README.md) | [14. Profiles →](14-profiles.md)
