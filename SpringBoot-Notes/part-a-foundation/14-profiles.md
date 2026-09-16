[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

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

[← 13. application.properties and application.yml](13-application-properties-and-application-yml.md) | [Contents](../README.md) | [15. Spring MVC and the DispatcherServlet →](../part-b-web-layer/15-spring-mvc-and-the-dispatcherservlet.md)
