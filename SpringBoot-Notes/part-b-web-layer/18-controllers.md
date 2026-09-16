[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 18. Controllers

## 18.1 What is it?

**Simple words:**
A controller is the class that **receives web requests**. Each method maps to a URL. Its only job is: take the request, pass it to the service, return the result. **No business logic.**

**Technical wording:**
A controller is a `@Controller`-annotated bean whose `@RequestMapping` methods are registered as handlers by `RequestMappingHandlerMapping`. `@RestController` additionally applies `@ResponseBody` to every method, so return values are serialized directly into the response body.

## 18.2 `@Controller` vs `@RestController` — key comparison

| | `@Controller` | `@RestController` |
|---|---|---|
| Composed of | `@Component` | **`@Controller` + `@ResponseBody`** |
| Return value meaning | **View name** (resolved to an HTML template) | **Response body** (serialized to JSON) |
| Needs `@ResponseBody` per method | Yes, for data responses | No — applied to all methods |
| Use for | Server-rendered pages (Thymeleaf, JSP) | **REST APIs** |
| Typical return | `"order-list"` → `order-list.html` | `OrderDto` → `{"id":1,...}` |

```java
// @Controller — returns a VIEW NAME
@Controller
public class OrderViewController {

    @GetMapping("/orders")
    public String list(Model model) {
        model.addAttribute("orders", orderService.findAll());
        return "order-list";        // → resolves to templates/order-list.html
    }

    @GetMapping("/orders/data")
    @ResponseBody                   // must add this for JSON
    public List<OrderDto> data() {
        return orderService.findAll();
    }
}

// @RestController — returns DATA. This is what you use for APIs.
@RestController
public class OrderApiController {

    @GetMapping("/api/orders")
    public List<OrderDto> list() {
        return orderService.findAll();    // automatically serialized to JSON
    }
}
```

**If you use `@Controller` and return `"order-list"` from a REST endpoint by mistake**, Spring tries to resolve a *view* called "order-list", fails, and returns a 404 or 500 — a confusing bug with an obvious cause once you know this distinction.

## 18.3 Mapping annotations

```java
@RequestMapping(value = "/orders", method = RequestMethod.GET)   // generic form
@GetMapping("/orders")        // shortcut — preferred
@PostMapping("/orders")
@PutMapping("/orders/{id}")
@PatchMapping("/orders/{id}")
@DeleteMapping("/orders/{id}")
```

The shortcuts are **composed annotations** — `@GetMapping` is `@RequestMapping(method = GET)`. Prefer them: they're shorter and make the HTTP method impossible to miss when reading the code.

**Class-level mapping** sets a common prefix:

```java
@RestController
@RequestMapping("/api/v1/orders")     // all methods inherit this prefix
public class OrderController {

    @GetMapping                       // → GET /api/v1/orders
    @GetMapping("/{id}")              // → GET /api/v1/orders/{id}
}
```

**Narrowing conditions:**

```java
@PostMapping(
    value    = "/orders",
    consumes = MediaType.APPLICATION_JSON_VALUE,   // only accept JSON requests
    produces = MediaType.APPLICATION_JSON_VALUE    // declare JSON responses
)
// Wrong Content-Type → 415 Unsupported Media Type
// Unacceptable Accept header → 406 Not Acceptable
```

## 18.4 A complete, production-shaped controller

```java
package com.company.orderservice.controller;

@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
@Validated                                   // enables validation on @PathVariable/@RequestParam
@Slf4j
public class OrderController {

    private final OrderService orderService;   // ONLY dependency — no repository here

    // ---------- CREATE ----------
    @PostMapping
    public ResponseEntity<OrderResponseDto> createOrder(
            @Valid @RequestBody OrderRequestDto request) {

        OrderResponseDto created = orderService.createOrder(request);

        // 201 + Location header pointing at the new resource
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(created.getId())
                .toUri();

        return ResponseEntity.created(location).body(created);
    }

    // ---------- READ ONE ----------
    @GetMapping("/{id}")
    public ResponseEntity<OrderResponseDto> getOrder(@PathVariable Long id) {
        // Service throws ResourceNotFoundException → handled by @RestControllerAdvice → 404
        return ResponseEntity.ok(orderService.getOrderById(id));
    }

    // ---------- READ MANY (paginated + filtered) ----------
    @GetMapping
    public ResponseEntity<Page<OrderResponseDto>> getOrders(
            @RequestParam(required = false) OrderStatus status,
            @RequestParam(defaultValue = "0")  @Min(0)  int page,
            @RequestParam(defaultValue = "20") @Min(1) @Max(100) int size) {

        // size is CAPPED at 100 — never let a client request 1,000,000 rows
        Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
        return ResponseEntity.ok(orderService.getOrders(status, pageable));
    }

    // ---------- FULL UPDATE ----------
    @PutMapping("/{id}")
    public ResponseEntity<OrderResponseDto> updateOrder(
            @PathVariable Long id,
            @Valid @RequestBody OrderRequestDto request) {
        return ResponseEntity.ok(orderService.updateOrder(id, request));
    }

    // ---------- PARTIAL UPDATE ----------
    @PatchMapping("/{id}/status")
    public ResponseEntity<OrderResponseDto> updateStatus(
            @PathVariable Long id,
            @Valid @RequestBody StatusUpdateDto request) {
        return ResponseEntity.ok(orderService.updateStatus(id, request.getStatus()));
    }

    // ---------- DELETE ----------
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteOrder(@PathVariable Long id) {
        orderService.deleteOrder(id);
        return ResponseEntity.noContent().build();      // 204
    }
}
```

**What makes this controller "correct" — point these out to a TL:**

1. **It is thin.** Every method is 1–5 lines. All logic is in the service.
2. **It depends only on the service**, never on the repository.
3. **It uses DTOs**, never entities.
4. **`@Valid` on every request body.**
5. **Page size is capped** — protects the database from a hostile or careless client.
6. **No try/catch** — exceptions are handled centrally.
7. **Correct status codes**: 201 + Location, 204 for delete.
8. **Versioned URL.**

## 18.5 Common Mistakes

| Mistake | Why it's wrong |
|---|---|
| **Business logic in the controller** | Untestable, unreusable, outside transaction boundaries |
| **Injecting the repository into the controller** | Skips the service layer and the transaction boundary |
| **Returning entities instead of DTOs** | Leaks the DB schema; causes lazy-loading serialization errors |
| **try/catch in every method** | Duplicated; use `@RestControllerAdvice` |
| **Always returning 200** | Clients can't distinguish create from update from failure |
| **Missing `@Valid`** | Invalid data reaches the service and the database |
| **Uncapped page size** | One request can pull the whole table into memory |
| **`@Autowired` field injection** | See Section 4 |
| **Storing state in controller fields** | Controllers are singletons — race conditions |

## 18.6 TL Questions

**Q: What should a controller contain?**
A: Only request/response handling — bind the input, call one service method, map the result to a status code. No business rules, no database access, no transaction management.

**Q: Why not inject the repository directly into the controller?**
A: It bypasses the service layer, which is where our transaction boundary and business rules live. It also makes the logic unreusable, since another caller — a scheduled job or a message listener — would have to duplicate it.

**Q: `@Controller` or `@RestController`?**
A: `@RestController` for our APIs. It's `@Controller` plus `@ResponseBody`, so return values are serialized to JSON instead of being treated as view names.

**Q: What happens if we remove `@RestController` and use `@Controller`?**
A: Return values are interpreted as view names. Returning an `OrderDto` would make Spring look for a view, and we'd get a 404 or 500 instead of JSON — unless we add `@ResponseBody` to every method.

**Q: Why are controllers singletons, and what's the risk?**
A: Spring creates one instance shared across all requests for efficiency. The risk is that any mutable instance field would be shared across concurrent requests, so we keep controllers stateless and hold all request data in method parameters.

**Q: What happens when an exception is thrown in a controller?**
A: It propagates to DispatcherServlet, which passes it to the `HandlerExceptionResolver` chain. `ExceptionHandlerExceptionResolver` finds the matching `@ExceptionHandler` in our `@RestControllerAdvice` and builds a consistent error response. That's why we have no try/catch in controllers.

## 18.7 Interview Questions

**Beginner — Difference between `@Controller` and `@RestController`?**
`@RestController` = `@Controller` + `@ResponseBody`. It returns data, not view names.

**Beginner — What is `@RequestMapping`?**
It maps HTTP requests to handler methods, optionally narrowing by method, params, headers, consumes and produces.

**Intermediate — Can a controller method return `void`?**
Yes. Combined with `@ResponseStatus`, or by writing to the `HttpServletResponse` directly. Spring treats the response as already handled. `ResponseEntity<Void>` is usually clearer.

**Intermediate — What does `produces` actually do?**
It restricts the mapping to requests whose `Accept` header is compatible, and sets the response `Content-Type`. A mismatch produces 406 Not Acceptable.

**Advanced — How would you inject the authenticated user into a controller method?**
Spring Security supports `@AuthenticationPrincipal UserDetails user` or a `Principal` parameter. For anything custom, implement a `HandlerMethodArgumentResolver` and register it via `WebMvcConfigurer.addArgumentResolvers` — that's the extension point argument binding is built on.

**Advanced — Are controller methods thread-safe?**
The method invocation is, because local variables and parameters are per-thread. The *bean* is not, because it's a shared singleton — so instance fields must never hold request state.

## 18.8 Quick Revision

1. `@RestController` = `@Controller` + `@ResponseBody`; use it for APIs.
2. Controllers are **thin**: bind → delegate → map to status code.
3. Inject the **service**, never the repository.
4. Return **DTOs**, never entities.
5. Use `@GetMapping`/`@PostMapping` shortcuts; class-level `@RequestMapping` for the prefix.
6. Always `@Valid` the request body; always cap page size.
7. No try/catch — central `@RestControllerAdvice`.
8. Correct codes: 201+Location on create, 204 on delete.
9. Controllers are singletons — keep them stateless.

## 18.9 TL Explanation (speak this)

> "Our controllers are deliberately thin — they bind the request, call a single service method and map the result to the right status code, and that's all. Business logic and the transaction boundary sit in the service, so a scheduled job or a Kafka listener can reuse the same logic. We use `@RestController`, so returns are serialized to JSON, we validate every request body with `@Valid`, and we always return DTOs rather than entities so we're not leaking the database schema. There's no try/catch in the controllers because exceptions go to a central `@RestControllerAdvice`."

---

[← 17. REST API Fundamentals](17-rest-api-fundamentals.md) | [Contents](../README.md) | [19. Reading Request Data — @PathVariable, @RequestParam, @RequestBody →](19-reading-request-data-pathvariable-requestparam-requestbody.md)
