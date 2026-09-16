[← Back to Contents](../README.md) · Part F — Master Flows and Final Revision

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

---

[← 55. Most Important TL / Senior-Level Questions](55-most-important-tl-senior-level-questions.md) | [Contents](../README.md) | &nbsp;
