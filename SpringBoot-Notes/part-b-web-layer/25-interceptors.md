[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 25. Interceptors

## 25.1 What is it?

**Simple words:**
An interceptor is like a filter, but it lives **inside Spring MVC**. It runs after the DispatcherServlet has already worked out *which controller method* will handle the request — so unlike a filter, it **knows the target method**.

**Technical wording:**
A `HandlerInterceptor` is a Spring MVC component invoked by `DispatcherServlet` around handler execution, with access to the resolved `HandlerMethod`, the `ModelAndView`, and any exception raised.

## 25.2 The three methods

```java
@Component
@Slf4j
public class PerformanceInterceptor implements HandlerInterceptor {

    private static final String START_TIME = "startTime";

    // 1. BEFORE the controller method
    //    return false → request STOPS here, controller never runs
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) {
        request.setAttribute(START_TIME, System.currentTimeMillis());

        // We KNOW the target method here — a filter cannot do this
        if (handler instanceof HandlerMethod handlerMethod) {
            log.debug("Invoking {}.{}",
                      handlerMethod.getBeanType().getSimpleName(),
                      handlerMethod.getMethod().getName());
        }
        return true;                          // continue
    }

    // 2. AFTER the controller, BEFORE the view is rendered
    //    NOT called if the controller threw an exception
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) {
        log.debug("Controller completed for {}", request.getRequestURI());
    }

    // 3. ALWAYS runs — after completion, even on exception. Cleanup goes here.
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                                Object handler, Exception ex) {
        long duration = System.currentTimeMillis() - (long) request.getAttribute(START_TIME);

        if (duration > 1000) {
            log.warn("SLOW REQUEST: {} {} took {}ms",
                     request.getMethod(), request.getRequestURI(), duration);
        }
        if (ex != null) {
            log.error("Request failed: {}", request.getRequestURI(), ex);
        }
    }
}
```

| Method | When | Skipped if... | Use for |
|---|---|---|---|
| `preHandle` | Before the controller | — | Auth checks, logging, timing start, rate limiting |
| `postHandle` | After controller, before view render | **Controller threw an exception** | Adding model attributes, response headers |
| `afterCompletion` | After everything | Never (if `preHandle` returned true) | Cleanup, timing end, resource release |

**The critical distinction:** `postHandle` is **skipped on exception**; `afterCompletion` **always runs**. So cleanup and metrics belong in `afterCompletion`, not `postHandle`. Getting this wrong means your timing metrics silently exclude every failed request — exactly the requests you most want to measure.

## 25.3 Registration

```java
@Configuration
@RequiredArgsConstructor
public class WebConfig implements WebMvcConfigurer {

    private final PerformanceInterceptor performanceInterceptor;
    private final AuthorizationInterceptor authorizationInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        registry.addInterceptor(performanceInterceptor)
                .addPathPatterns("/**")
                .order(1);

        registry.addInterceptor(authorizationInterceptor)
                .addPathPatterns("/api/**")                  // apply to these
                .excludePathPatterns("/api/v1/auth/**",      // except these
                                     "/api/v1/public/**")
                .order(2);
    }
}
```

**Note:** unlike filters, `@Component` alone does **not** register an interceptor. You must add it in `addInterceptors`. This catches people out.

**Path-pattern support is a real advantage** over filters, where you'd need a `FilterRegistrationBean` and coarser URL patterns.

## 25.4 Real example — method-aware authorization

This shows what interceptors can do that filters cannot:

```java
// Custom annotation on controller methods
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequiresRole {
    String[] value();
}

@RestController
public class AdminController {
    @DeleteMapping("/api/v1/orders/{id}")
    @RequiresRole({"ADMIN", "SUPER_ADMIN"})        // ← read by the interceptor
    public ResponseEntity<Void> delete(@PathVariable Long id) { ... }
}

@Component
@RequiredArgsConstructor
public class AuthorizationInterceptor implements HandlerInterceptor {

    private final CurrentUserService currentUserService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws IOException {

        if (!(handler instanceof HandlerMethod handlerMethod)) {
            return true;                      // static resource, not a controller method
        }

        // THIS is the interceptor advantage: we can read the method's annotations
        RequiresRole requiresRole = handlerMethod.getMethodAnnotation(RequiresRole.class);
        if (requiresRole == null) {
            return true;                      // endpoint is not role-restricted
        }

        Set<String> userRoles = currentUserService.getCurrentUserRoles();
        boolean allowed = Arrays.stream(requiresRole.value()).anyMatch(userRoles::contains);

        if (!allowed) {
            response.setStatus(HttpStatus.FORBIDDEN.value());
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            response.getWriter().write("{\"status\":403,\"message\":\"Insufficient role\"}");
            return false;                     // ← STOP: controller is never invoked
        }
        return true;
    }
}
```

A filter **could not** do this — at filter time, Spring hasn't yet resolved which method will handle the request, so the `@RequiresRole` annotation is invisible.

> **Additional clarification:** in a project already using Spring Security, prefer `@PreAuthorize("hasRole('ADMIN')")` over a custom interceptor like this. The example above is the clearest way to demonstrate the handler-awareness capability, but re-implementing authorization when Spring Security already provides it is duplicated effort and a security risk.

## 25.5 Filter vs Interceptor

| Aspect | Filter | Interceptor |
|---|---|---|
| Specification | Jakarta Servlet API | Spring MVC |
| Runs | **Outside** DispatcherServlet | **Inside** DispatcherServlet |
| Order relative to each other | **Always first** | After all filters |
| Knows the handler method | **No** | **Yes** (`HandlerMethod`) |
| Can read method annotations | No | **Yes** |
| Applies to static resources | Yes | Only if mapped |
| Applies when no handler matches | Yes | No |
| Can modify request/response objects | **Yes** (wrapping) | No |
| Access to `ModelAndView` | No | **Yes** (`postHandle`) |
| Path patterns | Coarse (URL patterns) | **Fine-grained include/exclude** |
| Spring DI | Yes (as a bean) | Yes |
| Exception caught by `@ControllerAdvice` | **No** | **Yes** |
| Registration | `@Component` / `FilterRegistrationBean` | `WebMvcConfigurer.addInterceptors` |
| Best for | Auth, CORS, encoding, correlation ID, compression | Handler-aware logic, per-endpoint auditing, timing |

**How to choose, in one line:**
> *"If it needs raw request/response manipulation or must apply to everything, use a filter. If it needs to know which controller method is running, use an interceptor."*

## 25.6 Common Mistakes

| Mistake | Consequence |
|---|---|
| Forgetting to register in `addInterceptors` | `@Component` alone does nothing — silently never runs |
| Returning `false` without writing a response | Client gets an empty 200 |
| Cleanup in `postHandle` | Skipped whenever an exception occurs — resource leak |
| Using a mutable field on the interceptor | Singleton → thread-safety bug. Use request attributes. |
| Assuming `handler` is always a `HandlerMethod` | `ClassCastException` on static resources — always type-check |
| Heavy DB work in `preHandle` | Latency on every request |
| Re-implementing Spring Security in an interceptor | Duplicated, weaker security |

## 25.7 TL Questions

**Q: What is an interceptor?**
A: A Spring MVC component that runs around the controller method. Unlike a filter, it runs inside the DispatcherServlet and knows which handler method is about to execute, so it can read that method's annotations.

**Q: Filter or interceptor — how do we decide?**
A: Filter when we need the raw request/response or need to cover everything including unmapped URLs — authentication, CORS, correlation IDs. Interceptor when the logic depends on which controller method is running — per-endpoint auditing or annotation-driven checks.

**Q: What's the difference between `postHandle` and `afterCompletion`?**
A: `postHandle` is skipped if the controller throws, `afterCompletion` always runs. So all cleanup and timing goes in `afterCompletion` — otherwise our metrics would silently exclude every failed request.

**Q: What happens if `preHandle` returns false?**
A: The chain stops — the controller is never invoked and `postHandle`/`afterCompletion` are skipped for interceptors that haven't run yet. We must write the response ourselves, or the client gets an empty 200.

**Q: How do we register one?**
A: Through `WebMvcConfigurer.addInterceptors`, where we can also specify include and exclude path patterns. Just annotating it `@Component` isn't enough — a mistake that's easy to miss because it fails silently.

**Q: Can `@RestControllerAdvice` catch exceptions from an interceptor?**
A: Yes for `preHandle` and `postHandle`, because they run inside the DispatcherServlet. That's a real advantage over filters.

## 25.8 Interview Questions

**Beginner — Name the three interceptor methods.**
`preHandle`, `postHandle`, `afterCompletion`.

**Beginner — How do you register an interceptor?**
Implement `WebMvcConfigurer` and add it in `addInterceptors`.

**Intermediate — Which runs first, a filter or an interceptor?**
A filter, always. Filters wrap the entire DispatcherServlet, and interceptors run inside it.

**Intermediate — Why type-check the `handler` parameter?**
It's declared `Object` and is only a `HandlerMethod` for annotated controller methods. For static resources it's a `ResourceHttpRequestHandler`, so an unchecked cast throws.

**Advanced — Interceptor vs AOP for cross-cutting logic?**
Interceptors are web-layer only and see HTTP request/response. AOP works on any Spring bean at any layer and sees method arguments and return values. Use an interceptor for HTTP concerns, AOP for service-layer concerns like transactions, caching and business auditing.

**Advanced — Are interceptors thread-safe?**
The bean is a shared singleton, so instance fields are not safe. Per-request state belongs in `HttpServletRequest` attributes (as in the timing example) or a properly cleared `ThreadLocal`.

## 25.9 Quick Revision

1. Interceptors are **Spring MVC**-level, running inside DispatcherServlet.
2. Three methods: `preHandle` (before), `postHandle` (after controller, skipped on exception), `afterCompletion` (always).
3. **Cleanup and metrics go in `afterCompletion`.**
4. `preHandle` returning `false` stops the request — write the response yourself.
5. Register via `WebMvcConfigurer.addInterceptors`, not `@Component` alone.
6. Supports fine-grained include/exclude path patterns.
7. **Key advantage: knows the `HandlerMethod`** and can read its annotations.
8. Filters always run before interceptors.
9. Interceptors are singletons — use request attributes for per-request state.
10. Exceptions from interceptors **can** be caught by `@ControllerAdvice`.

## 25.10 TL Explanation (speak this)

> "Interceptors sit inside Spring MVC, so unlike filters they run after the DispatcherServlet has resolved which controller method will handle the request — which means they can read that method's annotations. We use one for request timing and slow-request warnings. The important detail is that `postHandle` is skipped when the controller throws, so anything that must always run — cleanup, metrics — goes in `afterCompletion`. And they need registering in `WebMvcConfigurer.addInterceptors`; just marking them `@Component` silently does nothing."

---

[← 24. Filters](24-filters.md) | [Contents](../README.md) | [26. AOP (Aspect Oriented Programming) →](26-aop-aspect-oriented-programming.md)
