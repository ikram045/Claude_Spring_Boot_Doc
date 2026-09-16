[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 16. HandlerMapping and HandlerAdapter

## 16.1 What are they?

**Simple words:**
- **HandlerMapping** answers: *"Which method should handle this URL?"*
- **HandlerAdapter** answers: *"How do I actually call that method?"*

They are separated because a handler is not necessarily an annotated controller method — it could be an older `Controller` interface implementation or a functional route. The adapter hides that difference from the dispatcher.

## 16.2 How HandlerMapping builds its registry (internal)

At startup:

```
1. RequestMappingHandlerMapping implements InitializingBean
              │
              ▼
2. afterPropertiesSet() → scans EVERY bean in the context
              │
              ▼
3. Is the bean annotated @Controller or @RequestMapping?
              │ yes
              ▼
4. For each method, read its @RequestMapping/@GetMapping/etc.
   and build a RequestMappingInfo:
       - path patterns      /api/orders/{id}
       - HTTP methods       GET
       - params / headers   conditions
       - consumes / produces  media types
              │
              ▼
5. Register into a MultiValueMap:  RequestMappingInfo → HandlerMethod
              │
              ▼
6. At request time, match the incoming request against all
   RequestMappingInfos, sort by specificity, pick the best match
```

**Why this matters:** the registry is built **once at startup**, not per request. Lookup at runtime is a fast map match, not reflection scanning. It is also why an ambiguous mapping fails at startup, not on the first call.

**Matching specificity:** exact path beats a path variable, which beats a single wildcard `*`, which beats `**`. So `/api/orders/latest` wins over `/api/orders/{id}` — a useful fact when designing URLs.

## 16.3 The main implementations

| HandlerMapping | Handles |
|---|---|
| `RequestMappingHandlerMapping` | `@RequestMapping` methods — **the one you use** |
| `RouterFunctionMapping` | Functional endpoints (WebFlux style) |
| `SimpleUrlHandlerMapping` | Explicit URL → bean maps |
| `BeanNameUrlHandlerMapping` | Bean names starting with `/` (legacy) |

| HandlerAdapter | Invokes |
|---|---|
| `RequestMappingHandlerAdapter` | `@RequestMapping` methods — **the one you use** |
| `HttpRequestHandlerAdapter` | `HttpRequestHandler` implementations (e.g. static resources) |
| `SimpleControllerHandlerAdapter` | Legacy `Controller` interface |

## 16.4 TL Questions

**Q: What does HandlerMapping do?**
A: It maps an incoming request to the controller method that should handle it, based on path, HTTP method, headers, and content types. It builds that registry once at startup by scanning all `@Controller` beans.

**Q: Why do we need a separate HandlerAdapter?**
A: Because a handler can be different things — an annotated method, a legacy `Controller` implementation, a resource handler. The adapter knows how to invoke each type, so DispatcherServlet stays generic.

**Q: What happens internally when no mapping matches?**
A: DispatcherServlet either throws `NoHandlerFoundException` or, by default in Boot, forwards to `/error`, producing a 404. You can turn the exception on with `spring.mvc.throw-exception-if-no-handler-found=true` to handle it in `@RestControllerAdvice`.

**Q: How does Spring choose between `/api/orders/latest` and `/api/orders/{id}`?**
A: By specificity — a literal path segment always beats a path variable, so `/latest` wins. That's why we can safely have both.

## 16.5 Interview Questions

**Beginner — Which HandlerMapping is used for annotated controllers?**
`RequestMappingHandlerMapping`.

**Intermediate — When is the mapping registry built?**
At startup, in `afterPropertiesSet()`, by scanning all beans. Request-time lookup is then a fast map match.

**Advanced — What is `RequestMappingInfo`?**
The composite matching condition for a handler method: path patterns, HTTP methods, params, headers, consumable and producible media types. Requests are matched against it and results sorted by specificity.

**Advanced — What is `PathPatternParser`?**
The modern, pre-parsed path matching implementation (default since Spring 5.3/Boot 2.6), replacing `AntPathMatcher`. It is significantly faster and stricter — notably it does not support `**` in the middle of a pattern.

## 16.6 Quick Revision

1. HandlerMapping: request → handler method. HandlerAdapter: how to invoke it.
2. `RequestMappingHandlerMapping` + `RequestMappingHandlerAdapter` are the pair you use.
3. Registry built **once at startup**, so ambiguous mappings fail fast.
4. Matching considers path, method, headers, params, consumes, produces.
5. Literal paths beat path variables beat wildcards.
6. No match → 404 (or `NoHandlerFoundException` if enabled).

## 16.7 TL Explanation (speak this)

> "HandlerMapping decides which controller method handles a request and HandlerAdapter knows how to invoke it. At startup Spring scans every `@Controller` bean and builds a registry keyed on path, HTTP method, headers and content type, so runtime lookup is just a fast match rather than reflection. That's also why two methods mapped to the same path fail the startup instead of failing on a live request."

---

[← 15. Spring MVC and the DispatcherServlet](15-spring-mvc-and-the-dispatcherservlet.md) | [Contents](../README.md) | [17. REST API Fundamentals →](17-rest-api-fundamentals.md)
