[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 24. Filters

## 24.1 What is it?

**Simple words:**
A filter is code that runs **before and after every HTTP request**, at the servlet container level — *before* Spring MVC even gets involved. It's the outermost layer of your application.

Think of it as security at the building entrance: everyone passes through it, before they reach any particular office.

**Technical wording:**
A `Filter` is a Jakarta Servlet API component (`jakarta.servlet.Filter`) that intercepts requests and responses at the container level. Filters form a chain executed before the request reaches `DispatcherServlet` and after the response leaves it.

## 24.2 Where filters sit

```
 CLIENT
   │
   ▼
 TOMCAT
   │
   ▼
┌──────────────────────────────────────────────┐
│ FILTER 1  (order 1)                          │
│   code before chain.doFilter()               │
│  ┌────────────────────────────────────────┐  │
│  │ FILTER 2  (order 2)                    │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │ FILTER 3 (Spring Security)       │  │  │
│  │  │  ┌────────────────────────────┐  │  │  │
│  │  │  │  DISPATCHER SERVLET        │  │  │  │
│  │  │  │    Interceptors            │  │  │  │
│  │  │  │      Controller            │  │  │  │
│  │  │  │        Service             │  │  │  │
│  │  │  │          Repository        │  │  │  │
│  │  │  └────────────────────────────┘  │  │  │
│  │  │   code after chain.doFilter()    │  │  │
│  │  └──────────────────────────────────┘  │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
   │
   ▼
 RESPONSE
```

Filters nest like layers of an onion: outermost in, then unwind outermost out.

## 24.3 Why use filters?

- Runs for **every** request, including static resources and error dispatches
- Works even when **no controller matches** (404s still pass through)
- Has access to the raw `HttpServletRequest`/`HttpServletResponse`
- Can **modify or wrap** the request and response
- Can **stop** the request entirely (authentication rejection)

**Typical uses:** authentication/JWT validation, CORS, correlation IDs, request/response logging, character encoding, compression, rate limiting, XSS sanitisation.

## 24.4 Creating a filter

### Approach 1 — `OncePerRequestFilter` (RECOMMENDED)

```java
package com.company.orderservice.filter;

@Component
@Order(1)                                          // lower value = runs earlier
@Slf4j
public class CorrelationIdFilter extends OncePerRequestFilter {

    private static final String CORRELATION_ID_HEADER = "X-Correlation-Id";
    private static final String MDC_KEY = "correlationId";

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        // Reuse the caller's ID if present, so a trace spans microservices
        String correlationId = Optional.ofNullable(request.getHeader(CORRELATION_ID_HEADER))
                                       .filter(s -> !s.isBlank())
                                       .orElse(UUID.randomUUID().toString());
        try {
            MDC.put(MDC_KEY, correlationId);              // every log line now carries it
            response.setHeader(CORRELATION_ID_HEADER, correlationId);  // return it to the client

            filterChain.doFilter(request, response);      // ← CONTINUE THE CHAIN

        } finally {
            MDC.clear();     // MUST clear: Tomcat reuses threads, values would leak
        }
    }

    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) {
        return request.getRequestURI().startsWith("/actuator");   // skip health checks
    }
}
```

**Why `OncePerRequestFilter` rather than implementing `Filter` directly?**
A plain `Filter` can execute **multiple times per request** — on `FORWARD`, `INCLUDE` and `ERROR` dispatches. That means duplicate log entries, double-counted metrics, and authentication running twice. `OncePerRequestFilter` guarantees exactly one execution per request. **Always use it** unless you have a specific reason not to.

**The `finally` block is not optional.** Tomcat pools threads, so a `ThreadLocal`/MDC value left behind will appear on a completely unrelated later request — leaking one user's correlation ID (or worse, identity) into another's logs.

### Approach 2 — `FilterRegistrationBean` (precise control)

```java
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<RequestLoggingFilter> requestLoggingFilter() {
        FilterRegistrationBean<RequestLoggingFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new RequestLoggingFilter());
        registration.addUrlPatterns("/api/*");      // only these URLs — @Component can't do this
        registration.setOrder(2);
        registration.setName("requestLoggingFilter");
        return registration;
    }
}
```

**Use this when** you need URL-pattern scoping, or you must register a third-party filter you can't annotate.

> **Gotcha:** if a filter is both a `@Component` **and** registered via `FilterRegistrationBean`, it gets registered **twice**. Either drop `@Component`, or call `registration.setEnabled(false)` on the auto-registration.

## 24.5 Real example — request/response logging with a wrapper

A filter can only read the request body once — unless you wrap it.

```java
@Component
@Order(2)
@Slf4j
public class RequestLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {

        // Wrappers CACHE the body so it can be read here AND by the controller
        ContentCachingRequestWrapper  wrappedReq  = new ContentCachingRequestWrapper(request);
        ContentCachingResponseWrapper wrappedResp = new ContentCachingResponseWrapper(response);

        long start = System.currentTimeMillis();
        try {
            chain.doFilter(wrappedReq, wrappedResp);
        } finally {
            long duration = System.currentTimeMillis() - start;

            log.info("{} {} -> {} ({}ms)",
                     request.getMethod(), request.getRequestURI(),
                     wrappedResp.getStatus(), duration);

            if (log.isDebugEnabled()) {
                // Mask secrets before logging — never log raw passwords or tokens
                log.debug("Request body: {}", mask(new String(wrappedReq.getContentAsByteArray())));
            }

            // CRITICAL: the wrapper buffered the response — copy it out or the
            // client receives an EMPTY body.
            wrappedResp.copyBodyToResponse();
        }
    }

    private String mask(String body) {
        return body.replaceAll("(\"password\"\\s*:\\s*\")[^\"]*(\")", "$1****$2")
                   .replaceAll("(\"token\"\\s*:\\s*\")[^\"]*(\")", "$1****$2");
    }
}
```

**Two things that bite people here:**
1. Forgetting `copyBodyToResponse()` → the client gets an empty response body. Very confusing to debug.
2. Logging request bodies **without masking** → passwords and tokens land in your log aggregator, which is a compliance incident. Always mask.

## 24.6 Filter ordering

```java
@Order(Ordered.HIGHEST_PRECEDENCE)      // Integer.MIN_VALUE — runs first
@Order(1)
@Order(Ordered.LOWEST_PRECEDENCE)       // runs last
```

**Sensible ordering in a real service:**

```
1  CharacterEncodingFilter        (Boot registers this automatically)
2  CorrelationIdFilter            (so everything downstream can log the ID)
3  CorsFilter                     (must precede security — preflight has no token)
4  RequestLoggingFilter
5  Spring Security filter chain   (default order -100, configurable)
6  RateLimitingFilter
```

**Practical note:** Spring Security's chain runs at order `-100` by default (`SecurityProperties.DEFAULT_FILTER_ORDER`), so a filter with `@Order(1)` actually runs **after** security. If you need something before authentication — like a correlation ID — give it a negative order lower than `-100`, or register it with `addFilterBefore` in the security config.

## 24.7 Stopping the chain

```java
@Component
public class ApiKeyFilter extends OncePerRequestFilter {

    private final ObjectMapper objectMapper;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {

        String apiKey = request.getHeader("X-Api-Key");

        if (!isValid(apiKey)) {
            // Do NOT call chain.doFilter() → the request stops here.
            // Remember: @RestControllerAdvice cannot catch this, so we write JSON ourselves.
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            objectMapper.writeValue(response.getOutputStream(),
                    Map.of("status", 401, "error", "Unauthorized",
                           "message", "Invalid or missing API key"));
            return;                          // ← request does not proceed
        }
        chain.doFilter(request, response);
    }
}
```

## 24.8 Common Mistakes

| Mistake | Consequence |
|---|---|
| Forgetting `chain.doFilter()` | **The request silently hangs / returns empty.** The #1 filter bug. |
| Implementing `Filter` instead of `OncePerRequestFilter` | Runs multiple times per request |
| Not clearing `MDC`/`ThreadLocal` in `finally` | Values leak into other users' requests |
| Reading the body without a caching wrapper | Controller gets an empty body — stream already consumed |
| Forgetting `copyBodyToResponse()` | Client receives an empty response |
| Logging unmasked bodies | Passwords and tokens in logs — compliance breach |
| Registering both as `@Component` and via `FilterRegistrationBean` | Filter runs twice |
| Expecting `@RestControllerAdvice` to catch filter exceptions | Ugly HTML error page instead of JSON |
| Heavy work (DB calls) in a filter | Adds latency to **every** request including static resources |

## 24.9 TL Questions

**Q: What is a filter and where does it run?**
A: A servlet-level component that wraps every request, running before the DispatcherServlet and after the response is produced. It's the outermost layer, so it sees requests even when no controller matches.

**Q: Why do we use filters in our project?**
A: For correlation IDs so we can trace a request across services, request/response logging with timing, JWT authentication through the Spring Security chain, and CORS.

**Q: Why `OncePerRequestFilter`?**
A: A plain `Filter` can run multiple times per request on forward, include and error dispatches, which would double-log and re-authenticate. `OncePerRequestFilter` guarantees a single execution.

**Q: What happens if we forget `chain.doFilter()`?**
A: The request stops there — the controller is never reached and the client gets an empty response. It's the most common filter bug and it's silent, which is what makes it painful.

**Q: What happens when a filter throws an exception?**
A: Our `@RestControllerAdvice` can't catch it, because filters run outside the DispatcherServlet. We handle it inside the filter and write the JSON error ourselves, and for security filters we use `AuthenticationEntryPoint`.

**Q: How do we control filter order?**
A: `@Order` or `FilterRegistrationBean.setOrder()`. One detail to know is that Spring Security's chain sits at order -100, so anything that must run before authentication needs a lower order than that.

**Q: Why clear the MDC in a finally block?**
A: Tomcat reuses worker threads from a pool. If we leave a value in a ThreadLocal, the next unrelated request on that thread inherits it — so one user's correlation ID or identity shows up in another user's logs.

## 24.10 Interview Questions

**Beginner — What is a servlet filter?**
A Jakarta Servlet API component that intercepts requests and responses before and after the servlet handles them.

**Beginner — How do you register a filter in Spring Boot?**
Annotate it `@Component` (auto-registered for all URLs), or register a `FilterRegistrationBean` for URL patterns and explicit ordering.

**Intermediate — Filter vs Interceptor?**
Filters are servlet-level, run outside Spring MVC, work on raw request/response, apply to all requests including static resources, and have no knowledge of the handler. Interceptors are Spring MVC-level, run inside DispatcherServlet, know which handler method will run, and only apply to mapped requests. Full table in Section 53.

**Intermediate — How do you read the request body in a filter without breaking the controller?**
Wrap the request in `ContentCachingRequestWrapper` (or a custom wrapper), which caches the bytes so both the filter and the message converter can read them. The raw `ServletInputStream` can only be consumed once.

**Advanced — How does Spring Security fit into the filter chain?**
Boot registers a single `DelegatingFilterProxy` named `springSecurityFilterChain`, which delegates to a `FilterChainProxy`. That proxy holds one or more `SecurityFilterChain`s, each a list of security filters (`SecurityContextPersistenceFilter`, `UsernamePasswordAuthenticationFilter`, `ExceptionTranslationFilter`, `FilterSecurityInterceptor`, plus any custom JWT filter). It runs at order -100 within the container's chain.

**Advanced — What is `DelegatingFilterProxy` for?**
It bridges the servlet container and the Spring context. The container instantiates the proxy, which looks up the real filter **as a Spring bean** by name and delegates to it — so the actual filter can use dependency injection and the full Spring lifecycle, which a container-instantiated filter could not.

## 24.11 Quick Revision

1. Filters are **servlet-level**, running before and after DispatcherServlet.
2. Extend **`OncePerRequestFilter`** — guarantees one execution per request.
3. **Always call `chain.doFilter()`** unless deliberately blocking the request.
4. Register with `@Component` (all URLs) or `FilterRegistrationBean` (patterns + order).
5. `@Order` controls sequence; Spring Security is at **-100**.
6. Always clear `MDC`/`ThreadLocal` in `finally` — threads are pooled.
7. Use `ContentCachingRequestWrapper`/`ResponseWrapper` to read bodies; call `copyBodyToResponse()`.
8. **`@RestControllerAdvice` cannot catch filter exceptions** — handle them in the filter.
9. Mask secrets before logging bodies.
10. Keep filters lightweight — they run on every single request.

## 24.12 TL Explanation (speak this)

> "Filters are the outermost layer — servlet-level, so they run before the DispatcherServlet and see every request, even ones that don't match a controller. We use them for correlation IDs, request logging with timing, and JWT authentication through Spring Security's chain. We always extend `OncePerRequestFilter` so they don't re-run on forwards and error dispatches, and we always clear the MDC in a `finally` block because Tomcat reuses threads and values would leak into other users' requests. The one thing to be aware of is that our `@RestControllerAdvice` can't catch exceptions thrown in a filter, so filters write their own JSON error responses."

---

[← 23. Exception Handling](23-exception-handling.md) | [Contents](../README.md) | [25. Interceptors →](25-interceptors.md)
