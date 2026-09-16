[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 15. Spring MVC and the DispatcherServlet

## 15.1 What is it?

**Simple words:**
When a request arrives at your application, **something** has to decide which of your methods should handle it. That something is the **DispatcherServlet**.

It is the single entry door for every HTTP request. It receives the request, works out which controller method matches the URL, calls it, takes the returned value, converts it to JSON, and writes the response.

It is often called the **Front Controller**, because *every* request goes through this one object first.

**Technical wording:**
Spring MVC is Spring's servlet-based web framework implementing the Front Controller pattern. The `DispatcherServlet` is the central dispatcher that receives all matching requests and delegates to specialised strategy components — `HandlerMapping`, `HandlerAdapter`, `HandlerExceptionResolver`, `ViewResolver` and `HttpMessageConverter` — to produce a response.

## 15.2 Why do we use a Front Controller?

Without it, every servlet would duplicate: authentication checks, logging, character encoding, exception handling, JSON conversion.

With one front controller, all of that lives in **one place**, and your controllers contain only business-facing code.

| Benefit | Explanation |
|---|---|
| Centralised request handling | One place for cross-cutting concerns |
| Consistent behaviour | Every endpoint behaves the same way |
| Pluggable strategies | Swap the JSON converter, add argument resolvers, add interceptors |
| Clean controllers | Your methods just take parameters and return objects |

## 15.3 THE COMPLETE REQUEST FLOW (memorise this)

This is the diagram to draw on a whiteboard when asked "explain the Spring Boot request flow".

```
  CLIENT (browser / Postman / another microservice)
     │
     │  HTTP POST /api/orders   { "customerId": "C1", "amount": 500 }
     ▼
┌─────────────────────────────────────────────────────────────────┐
│  EMBEDDED TOMCAT (servlet container)                            │
│  - accepts the TCP connection                                   │
│  - assigns a worker thread from its thread pool                 │
│  - parses the raw HTTP into an HttpServletRequest               │
└─────────────────────────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────────────┐
│  FILTER CHAIN  (servlet-level, before Spring MVC)               │
│  CharacterEncodingFilter → Spring Security filters →            │
│  your custom filters (correlation id, request logging)          │
└─────────────────────────────────────────────────────────────────┘
     │
     ▼
╔═════════════════════════════════════════════════════════════════╗
║  DISPATCHER SERVLET   (the Front Controller)                    ║
╚═════════════════════════════════════════════════════════════════╝
     │
     │ STEP 1 ─────────────────────────────────────────────────────
     │ Ask each HandlerMapping: "who handles POST /api/orders?"
     │ RequestMappingHandlerMapping searches its registry of
     │ @RequestMapping methods and returns a HandlerExecutionChain
     │   = the handler method + the matching interceptors
     ▼
     │ STEP 2 ─────────────────────────────────────────────────────
     │ INTERCEPTOR preHandle()  ← runs BEFORE the controller
     │   returns false → request stops here
     ▼
     │ STEP 3 ─────────────────────────────────────────────────────
     │ Pick a HandlerAdapter that knows how to invoke this handler
     │ (RequestMappingHandlerAdapter for annotated controllers)
     ▼
     │ STEP 4 ─────────────────────────────────────────────────────
     │ ARGUMENT RESOLVERS build each method parameter:
     │   @PathVariable      → PathVariableMethodArgumentResolver
     │   @RequestParam      → RequestParamMethodArgumentResolver
     │   @RequestBody       → RequestResponseBodyMethodProcessor
     │                          └─► HttpMessageConverter (Jackson)
     │                              reads JSON → Java DTO object
     │   @Valid present?    → run Bean Validation NOW
     │                          └─► fails → MethodArgumentNotValidException
     ▼
     │ STEP 5 ─────────────────────────────────────────────────────
     │ ★ YOUR CONTROLLER METHOD IS FINALLY CALLED ★
     │
     │      Controller  →  Service  (@Transactional starts here)
     │                        │
     │                        ▼
     │                    Repository
     │                        │
     │                        ▼
     │                 JPA / Hibernate
     │                        │
     │                        ▼
     │                     DATABASE
     │                        │
     │                   result returns back up
     ▼
     │ STEP 6 ─────────────────────────────────────────────────────
     │ RETURN VALUE HANDLER processes what you returned
     │   @ResponseBody / ResponseEntity
     │        └─► HttpMessageConverter (Jackson)
     │            writes Java object → JSON into the response body
     │   (a plain String view name → ViewResolver → HTML template)
     ▼
     │ STEP 7 ─────────────────────────────────────────────────────
     │ INTERCEPTOR postHandle()   ← after controller, before response committed
     ▼
     │ STEP 8 ─────────────────────────────────────────────────────
     │ INTERCEPTOR afterCompletion()  ← always runs, even on exception
     ▼
┌─────────────────────────────────────────────────────────────────┐
│  FILTER CHAIN unwinds (code after chain.doFilter)               │
└─────────────────────────────────────────────────────────────────┘
     │
     ▼
  HTTP RESPONSE  201 Created   { "orderId": 1001, "status": "CONFIRMED" }
     │
     ▼
  CLIENT


  ══ IF AN EXCEPTION IS THROWN ANYWHERE IN STEPS 4–6 ══
     │
     ▼
  HandlerExceptionResolver chain
     └─► ExceptionHandlerExceptionResolver
            └─► finds a matching @ExceptionHandler
                (in the controller, or in @RestControllerAdvice)
                   └─► builds the error response
     └─► if none matches → DefaultHandlerExceptionResolver
            → falls back to /error → 500 Internal Server Error
```

**How to summarise this in an interview in 30 seconds:**
> "Tomcat hands the request to the filter chain, then to the DispatcherServlet. It uses HandlerMapping to find the controller method, a HandlerAdapter to invoke it, argument resolvers and message converters to build the parameters from JSON, runs my controller, then converts the return value back to JSON with a message converter. Interceptors wrap the controller call, and any exception goes to a HandlerExceptionResolver, which is how `@RestControllerAdvice` works."

## 15.4 The strategy components inside DispatcherServlet

| Component | Responsibility | Default implementation |
|---|---|---|
| **`HandlerMapping`** | URL + method → handler | `RequestMappingHandlerMapping` |
| **`HandlerAdapter`** | Knows how to invoke that handler | `RequestMappingHandlerAdapter` |
| **`HandlerMethodArgumentResolver`** | Builds each method parameter | ~30 built-in resolvers |
| **`HttpMessageConverter`** | JSON/XML ↔ Java object | `MappingJackson2HttpMessageConverter` |
| **`HandlerExceptionResolver`** | Turns exceptions into responses | `ExceptionHandlerExceptionResolver` |
| **`ViewResolver`** | View name → template (not used in pure REST) | `ThymeleafViewResolver` |
| **`LocaleResolver`** | Determines the request locale | `AcceptHeaderLocaleResolver` |
| **`MultipartResolver`** | File uploads | `StandardServletMultipartResolver` |

**Key insight:** DispatcherServlet itself contains **almost no logic**. It orchestrates these pluggable strategies. That is why Spring MVC is so extensible — you add a custom argument resolver or message converter without touching the dispatcher.

## 15.5 How does DispatcherServlet get registered?

In old Spring, you wrote it in `web.xml`:

```xml
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

In Spring Boot, `DispatcherServletAutoConfiguration` does it automatically and maps it to `/`. You can change the path:

```properties
spring.mvc.servlet.path=/api      # DispatcherServlet now handles /api/**
```

## 15.6 Spring MVC vs Spring WebFlux (Additional clarification)

You will hear WebFlux mentioned; know the distinction.

| | Spring MVC | Spring WebFlux |
|---|---|---|
| Model | **Blocking**, one thread per request | **Non-blocking**, event loop |
| Stack | Servlet API | Reactive Streams |
| Server | Tomcat/Jetty/Undertow | Netty (default) |
| Return types | Objects, `ResponseEntity` | `Mono<T>`, `Flux<T>` |
| Best for | Standard CRUD, JDBC/JPA | High concurrency, streaming, non-blocking clients |
| Learning curve | Low | High |
| **Use when** | **Almost always in corporate CRUD systems** | Very high concurrency with reactive data access end-to-end |

**Important honesty point:** WebFlux only helps if the *whole* chain is non-blocking. Using WebFlux with blocking JDBC/JPA gives you the complexity with none of the benefit. Most corporate Spring Boot projects correctly use MVC.

## 15.7 Common Mistakes

| Mistake | Consequence |
|---|---|
| Thinking you must configure DispatcherServlet in Boot | It's automatic |
| Two methods mapped to the same URL + method | `IllegalStateException: Ambiguous mapping` at startup |
| Blocking I/O in a controller with no timeout | Tomcat worker threads exhausted → whole service unresponsive |
| Confusing filter order with interceptor order | Filters are servlet-level and always run **before** any interceptor |
| Expecting `@ExceptionHandler` to catch filter exceptions | It cannot — filters run outside DispatcherServlet (see Section 24) |

## 15.8 TL Questions

**Q: What is the DispatcherServlet?**
A: The front controller — the single servlet that receives every HTTP request and orchestrates handler lookup, parameter binding, controller invocation, response conversion and exception resolution.

**Q: Walk me through what happens when a request hits our API.**
A: Tomcat accepts it and assigns a worker thread, the filter chain runs, then DispatcherServlet asks HandlerMapping which controller method matches. A HandlerAdapter invokes it, argument resolvers and Jackson build the parameters from the JSON body, validation runs if `@Valid` is present, our controller calls the service and repository, and the return value goes back through Jackson to become the JSON response. Interceptors wrap the controller call and any exception is routed to our `@RestControllerAdvice`.

**Q: Why is it called a front controller?**
A: Because all requests enter through it. Cross-cutting concerns live there instead of being duplicated in every endpoint.

**Q: What happens if two methods map to the same path?**
A: The application fails to start with an ambiguous mapping error. That's deliberate — it's a configuration error caught at deploy time rather than at runtime.

**Q: How does JSON become our DTO object?**
A: `@RequestBody` triggers `RequestResponseBodyMethodProcessor`, which picks an `HttpMessageConverter` based on the `Content-Type` header. For `application/json` that's Jackson's converter, which deserializes the body into our DTO class.

**Q: What alternative can we use?**
A: Spring WebFlux for non-blocking handling, but it only pays off when the entire chain including data access is reactive. With JPA and JDBC, MVC is the correct choice for us.

## 15.9 Interview Questions

**Beginner — What design pattern does DispatcherServlet implement?**
Front Controller.

**Beginner — Who registers it in Spring Boot?**
`DispatcherServletAutoConfiguration`, mapped to `/` by default.

**Intermediate — What is a `HandlerExecutionChain`?**
The object returned by `HandlerMapping`. It contains the handler method plus the ordered list of `HandlerInterceptor`s applicable to that request.

**Intermediate — How are method parameters populated?**
By `HandlerMethodArgumentResolver` implementations. Each resolver declares which parameter types it supports (`supportsParameter`) and builds the value (`resolveArgument`). You can register custom ones for things like injecting the authenticated user.

**Advanced — What is `doDispatch()`?**
The core method of `DispatcherServlet`: checks for multipart, resolves the handler, resolves the adapter, handles conditional-GET/`Last-Modified`, runs `preHandle`, invokes the handler, applies the default view name, runs `postHandle`, then `processDispatchResult` which handles the exception or renders the view, and finally `afterCompletion`.

**Advanced — Can there be multiple DispatcherServlets?**
Yes — an older pattern mapped different servlets to different URL spaces, each with its own child `WebApplicationContext` sharing a common root context. Rare today; microservices solve the same problem by separation of services.

**Advanced — What is the thread model?**
One thread per request, taken from Tomcat's pool (default max 200). The thread is held for the entire request, including blocking database and HTTP calls. This is exactly why timeouts and connection pool sizing matter — a slow downstream service can consume all 200 threads and take the whole application down.

## 15.10 Quick Revision

1. `DispatcherServlet` = front controller, single entry point for all requests.
2. Flow: **Tomcat → Filters → DispatcherServlet → HandlerMapping → Interceptor preHandle → HandlerAdapter → Argument Resolvers → Controller → Service → Repository → DB → Return Value Handler → Message Converter → postHandle → afterCompletion → Response.**
3. `HandlerMapping` finds the handler; `HandlerAdapter` invokes it.
4. `HttpMessageConverter` (Jackson) does JSON ↔ Java in **both** directions.
5. Exceptions go to `HandlerExceptionResolver` → `@ExceptionHandler` / `@RestControllerAdvice`.
6. Boot auto-registers the servlet at `/`; change with `spring.mvc.servlet.path`.
7. DispatcherServlet delegates everything — it is pure orchestration.
8. Thread-per-request model: default 200 Tomcat threads; blocking calls need timeouts.
9. Filters run **outside** DispatcherServlet; interceptors **inside** it.

## 15.11 TL Explanation (speak this)

> "The DispatcherServlet is the front controller — every request goes through it. It asks HandlerMapping which controller method matches the URL and HTTP method, uses a HandlerAdapter to invoke it, and argument resolvers plus Jackson's message converter to turn the JSON body into our DTO. Our controller then calls the service and repository, and the return value goes back through Jackson to become the JSON response. Interceptors wrap the controller call and exceptions are routed to a HandlerExceptionResolver, which is the mechanism behind our `@RestControllerAdvice`. Boot registers all of this automatically."

---

[← 14. Profiles](../part-a-foundation/14-profiles.md) | [Contents](../README.md) | [16. HandlerMapping and HandlerAdapter →](16-handlermapping-and-handleradapter.md)
