[← Back to Contents](../README.md) · Part F — Master Flows and Final Revision

---

# 53. Filter vs Interceptor vs AOP — Complete Comparison

```
 REQUEST
    │
    ▼
 ┌───────────────────────────────────────────────────────────┐
 │ FILTER            (Servlet API — outside Spring MVC)      │
 │  sees: raw HttpServletRequest / HttpServletResponse       │
 │  ┌───────────────────────────────────────────────────────┐│
 │  │ DISPATCHER SERVLET                                     ││
 │  │  ┌───────────────────────────────────────────────────┐ ││
 │  │  │ INTERCEPTOR    (Spring MVC — knows HandlerMethod) │ ││
 │  │  │  ┌───────────────────────────────────────────────┐│ ││
 │  │  │  │ CONTROLLER                                     ││ ││
 │  │  │  │   ┌─────────────────────────────────────────┐ ││ ││
 │  │  │  │   │ AOP ASPECT (any Spring bean, any layer) │ ││ ││
 │  │  │  │   │   ┌───────────────────────────────────┐ │ ││ ││
 │  │  │  │   │   │ SERVICE / REPOSITORY METHOD       │ │ ││ ││
 │  │  │  │   │   └───────────────────────────────────┘ │ ││ ││
 │  │  │  │   └─────────────────────────────────────────┘ ││ ││
 │  │  │  └───────────────────────────────────────────────┘│ ││
 │  │  └───────────────────────────────────────────────────┘ ││
 │  └───────────────────────────────────────────────────────┘│
 └───────────────────────────────────────────────────────────┘
```

| Aspect | **Filter** | **Interceptor** | **AOP** |
|---|---|---|---|
| Specification | Jakarta Servlet | Spring MVC | Spring AOP / AspectJ |
| Scope | Every HTTP request | Mapped requests only | **Any Spring bean method** |
| Runs relative to others | **First** | After filters | Innermost |
| Layer | Servlet container | Web (DispatcherServlet) | **All layers** |
| Sees raw request/response | **Yes** | Yes (read-only use) | **No** |
| Can wrap/modify request | **Yes** | No | No |
| Knows the handler method | **No** | **Yes** (`HandlerMethod`) | Yes (`JoinPoint`) |
| Reads method annotations | No | **Yes** | **Yes** |
| Sees method arguments | No | No | **Yes** |
| Sees the return value | No | Via `ModelAndView` | **Yes** |
| Can prevent execution | Yes (skip `doFilter`) | Yes (`preHandle` false) | Yes (`@Around`, skip `proceed`) |
| Applies to static resources | **Yes** | Only if mapped | No |
| Applies when no handler matches | **Yes** | No | No |
| Works outside HTTP (jobs, Kafka) | No | No | **Yes** |
| Exception caught by `@ControllerAdvice` | **No** | Yes | Yes |
| Self-invocation limitation | No | No | **Yes** |
| Registration | `@Component` / `FilterRegistrationBean` | `WebMvcConfigurer.addInterceptors` | `@Aspect` + `@Component` |
| Ordering | `@Order` / `setOrder()` | `.order(n)` | `@Order` |

## Choosing — the decision rule

```
Do you need the raw request/response, or must it run for
EVERY request including unmapped URLs and static resources?
        │ YES → FILTER
        │        (auth, CORS, encoding, correlation ID, compression)
        ▼ NO
Does the logic depend on WHICH CONTROLLER METHOD is running,
and is it HTTP-specific?
        │ YES → INTERCEPTOR
        │        (per-endpoint auditing, request timing, annotation checks)
        ▼ NO
Does it apply to service/repository methods, or must it also work
outside HTTP (scheduled jobs, message listeners)?
        │ YES → AOP
                 (transactions, caching, retry, business auditing)
```

**Real-world assignment in a typical service:**

| Concern | Use |
|---|---|
| JWT authentication | **Filter** (Spring Security) |
| CORS | **Filter** |
| Correlation ID / MDC | **Filter** |
| Request/response logging | **Filter** |
| Request timing, slow-request alerts | **Interceptor** |
| Annotation-driven endpoint auditing | **Interceptor** |
| `@Transactional` | **AOP** |
| `@Cacheable`, `@Async`, `@Retryable` | **AOP** |
| Business audit trail (`@Auditable`) | **AOP** |
| Method-level security (`@PreAuthorize`) | **AOP** |

---

[← 52. Complete Transaction Flow](52-complete-transaction-flow.md) | [Contents](../README.md) | [54. Most Important Spring Boot Annotations →](54-most-important-spring-boot-annotations.md)
