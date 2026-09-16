[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 19. Reading Request Data — @PathVariable, @RequestParam, @RequestBody

## 19.1 The three ways data arrives

```
POST /api/v1/orders/42/items?includeTax=true&currency=INR
Content-Type: application/json
Authorization: Bearer eyJhbGc...

{ "productId": 100, "quantity": 2 }

      │              │                    │              │
      │              │                    │              └─► HEADER  → @RequestHeader
      │              │                    └─► BODY          → @RequestBody
      │              └─► QUERY PARAMS  ?includeTax=true    → @RequestParam
      └─► PATH VARIABLE  /orders/{orderId}/items            → @PathVariable
```

## 19.2 The comparison table (very common interview question)

| Feature | `@PathVariable` | `@RequestParam` | `@RequestBody` |
|---|---|---|---|
| Reads from | URL **path** | URL **query string** or form data | Request **body** |
| Example | `/orders/42` | `/orders?status=NEW` | `{"amount":500}` |
| Typical use | **Identify** a specific resource | **Filter / sort / paginate** | **Send complex data** |
| Data type | Simple (String, Long, enum) | Simple, or List | Complex object (DTO) |
| How many per method | Many | Many | **Only one** |
| Optional? | `required=false` (needs a second mapping) | `required=false` or `defaultValue` | `required=false` |
| Conversion by | `WebDataBinder` / `Converter` | `WebDataBinder` / `Converter` | **`HttpMessageConverter` (Jackson)** |
| Used with | GET, PUT, DELETE, PATCH | Mostly GET | POST, PUT, PATCH |
| Visible in logs/history | Yes | Yes — **never put secrets here** | No (body isn't usually logged) |

**Rule of thumb:**
- Identifies **which** resource → `@PathVariable`
- Modifies **how** you retrieve it → `@RequestParam`
- **Is** the data → `@RequestBody`

## 19.3 `@PathVariable`

```java
// Simple case — parameter name matches {id}
@GetMapping("/orders/{id}")
public OrderDto getOrder(@PathVariable Long id) { ... }

// Explicit name — needed when names differ
@GetMapping("/orders/{orderId}")
public OrderDto getOrder(@PathVariable("orderId") Long id) { ... }

// Multiple path variables — nested resources
@GetMapping("/customers/{customerId}/orders/{orderId}")
public OrderDto get(@PathVariable Long customerId,
                    @PathVariable Long orderId) { ... }

// All at once
@GetMapping("/customers/{customerId}/orders/{orderId}")
public OrderDto get(@PathVariable Map<String, String> vars) {
    String customerId = vars.get("customerId");
}
```

> **Compilation note:** relying on the parameter name requires the class to be compiled with debug symbols (`-parameters`). Spring Boot's Maven plugin enables this by default; if you ever see *"Name for argument of type [Long] not specified"*, add the explicit name.

## 19.4 `@RequestParam`

```java
// Required (default)
@GetMapping("/orders")
public List<OrderDto> get(@RequestParam String status) { }
// Missing → 400 Bad Request: "Required request parameter 'status' is not present"

// Optional with default — the common production pattern
@GetMapping("/orders")
public Page<OrderDto> get(
        @RequestParam(defaultValue = "0")  int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(required = false) OrderStatus status) { }

// Optional wrapper
@RequestParam Optional<String> status

// List:  ?ids=1&ids=2&ids=3   OR   ?ids=1,2,3
@GetMapping("/orders")
public List<OrderDto> byIds(@RequestParam List<Long> ids) { }

// All params as a map
@RequestParam Map<String, String> allParams
```

**Important:** `@RequestParam` also reads `application/x-www-form-urlencoded` form bodies, not just query strings. That's why HTML form posts bind with `@RequestParam`, not `@RequestBody`.

### Better: bind many params into an object

When a search endpoint has 6+ filters, stop listing parameters:

```java
@Getter @Setter
public class OrderSearchCriteria {
    private OrderStatus status;
    private LocalDate fromDate;
    private LocalDate toDate;
    private BigDecimal minAmount;
    @Min(0) private int page = 0;
    @Min(1) @Max(100) private int size = 20;
}

@GetMapping("/orders")
public Page<OrderDto> search(@Valid OrderSearchCriteria criteria) { ... }
// NOTE: no annotation needed — Spring binds query params onto the object fields.
// This also gets you validation for free.
```

## 19.5 `@RequestBody`

```java
@PostMapping("/orders")
public ResponseEntity<OrderResponseDto> create(
        @Valid @RequestBody OrderRequestDto request) { ... }
```

**What happens internally, step by step:**

```
1. RequestResponseBodyMethodProcessor handles the parameter
                │
2. Reads the Content-Type header  →  application/json
                │
3. Selects an HttpMessageConverter that can read JSON
   → MappingJackson2HttpMessageConverter
                │
4. Jackson's ObjectMapper deserializes the body into OrderRequestDto
   - matches JSON field names to setters/fields
   - converts types (string → BigDecimal, string → LocalDate)
   - unknown field → FAIL by default in Boot? No: Boot sets
     FAIL_ON_UNKNOWN_PROPERTIES=false, so extra fields are ignored
                │
5. @Valid present → run Bean Validation
   - violation → MethodArgumentNotValidException → 400
                │
6. The populated, validated DTO is passed to your method
```

**Requirements for Jackson deserialization:**
- A **no-argument constructor** (or `@JsonCreator` / a `record` / a Lombok `@NoArgsConstructor`)
- **Setters**, or field access
- **Field names matching the JSON** (or `@JsonProperty("json_name")`)

**Common failure:** using Lombok's `@Builder` alone removes the no-arg constructor, causing *"Cannot construct instance ... no Creators"*. Fix: add `@NoArgsConstructor @AllArgsConstructor` alongside `@Builder`.

**Only one `@RequestBody` per method** — there's a single body to read, and the stream can be consumed once.

## 19.6 Other useful binding annotations

```java
@GetMapping("/orders")
public List<OrderDto> get(
        @RequestHeader("Authorization") String authHeader,
        @RequestHeader(value = "X-Correlation-Id", required = false) String correlationId,
        @CookieValue(value = "sessionId", required = false) String sessionId,
        HttpServletRequest request,          // raw servlet request
        Principal principal) { }             // authenticated user (Spring Security)

// File upload
@PostMapping(value = "/orders/{id}/invoice", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public ResponseEntity<Void> upload(@PathVariable Long id,
                                   @RequestPart("file") MultipartFile file) { }
```

## 19.7 Type conversion and a real gotcha

Spring converts `String` → target type automatically for primitives, wrappers, enums and common types. For dates you usually need a format hint:

```java
// Without this, "2024-01-15" often fails to bind to LocalDate
@GetMapping("/orders")
public List<OrderDto> get(
    @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate fromDate) { }
```

**Enum binding:** `?status=PENDING` binds to `OrderStatus.PENDING`. An invalid value throws `MethodArgumentTypeMismatchException` → handle it in your advice to return a clean 400 listing the allowed values, instead of a raw 500.

## 19.8 Common Mistakes

| Mistake | Consequence |
|---|---|
| Two `@RequestBody` parameters | `HttpMessageNotReadableException` — body already consumed |
| Sensitive data in query params | Logged in access logs, browser history, proxies. **Use the body.** |
| No `defaultValue` on an optional param | 400 when the client omits it |
| DTO without a no-arg constructor | Jackson can't instantiate it |
| Mismatch between `{id}` and the parameter name without an explicit name | Binding failure |
| `@RequestBody` on a GET | GET bodies are not reliably supported by clients/proxies |
| Using `@RequestParam` for a JSON body | Body won't bind — 400 |

## 19.9 TL Questions

**Q: When do we use `@PathVariable` versus `@RequestParam`?**
A: `@PathVariable` identifies which resource — the ID is part of the resource's address. `@RequestParam` modifies how we retrieve it — filters, sorting, pagination. So `/orders/42?includeItems=true`.

**Q: Why can there be only one `@RequestBody`?**
A: There's a single HTTP body and the input stream can only be read once. If we need several pieces of data, they belong in one DTO.

**Q: How does JSON become our DTO internally?**
A: `RequestResponseBodyMethodProcessor` picks an `HttpMessageConverter` based on `Content-Type`; for JSON that's Jackson, which deserializes the body into the DTO. Then `@Valid` runs Bean Validation before our method is called.

**Q: What happens if a client sends a field we don't have in the DTO?**
A: Spring Boot configures Jackson with `FAIL_ON_UNKNOWN_PROPERTIES` disabled, so unknown fields are ignored rather than causing an error. That's usually what we want for forward compatibility, but it does mean a typo in a field name fails silently — which is why validation on required fields matters.

**Q: Why don't we put the token in a query parameter?**
A: Query strings are recorded in access logs, browser history and proxy logs. Credentials belong in the `Authorization` header or the body, never the URL.

**Q: What happens if the client sends an invalid enum value?**
A: Spring throws `MethodArgumentTypeMismatchException`. We handle it in `@RestControllerAdvice` and return a 400 listing the valid values, rather than letting it become a 500.

## 19.10 Interview Questions

**Beginner — Which annotation reads the JSON body?**
`@RequestBody`.

**Beginner — How do you give a query param a default?**
`@RequestParam(defaultValue = "20") int size`.

**Intermediate — `@RequestParam` vs `@RequestPart`?**
`@RequestParam` handles simple values and can handle a `MultipartFile`, converting via `WebDataBinder`. `@RequestPart` handles a named part of a multipart request and applies `HttpMessageConverter`s — so it's what you use when a part is itself JSON.

**Intermediate — How do you bind many query params cleanly?**
Declare a POJO parameter with no annotation. Spring binds matching query params onto its fields, and `@Valid` works on it. Much cleaner than eight `@RequestParam`s.

**Advanced — What is `WebDataBinder` and how do you customise conversion?**
It performs binding and type conversion for non-body parameters. Customise it per controller with `@InitBinder`, or globally by registering a `Converter`/`Formatter` in `WebMvcConfigurer.addFormatters` — the right place for a project-wide custom date format.

**Advanced — Why is `@RequestBody` on a GET request a bad idea?**
The HTTP spec assigns no semantics to a GET body; many proxies, caches and client libraries strip or reject it, and it breaks caching. Use query params, or make it a POST search endpoint if the criteria are genuinely complex.

## 19.11 Quick Revision

1. `@PathVariable` = which resource. `@RequestParam` = how to retrieve. `@RequestBody` = the data.
2. Only **one** `@RequestBody` per method.
3. `@RequestParam` supports `defaultValue`, `required`, `List`, `Map`, and form bodies.
4. `@RequestBody` uses `HttpMessageConverter`/Jackson; the others use `WebDataBinder`.
5. Bind many query params into a POJO — cleaner and validatable.
6. Never put secrets in the URL.
7. Use `@DateTimeFormat` for date parameters.
8. DTOs need a no-arg constructor for Jackson.
9. `@RequestHeader`, `@CookieValue`, `@RequestPart` cover the rest.

## 19.12 TL Explanation (speak this)

> "We use `@PathVariable` for the resource identifier, `@RequestParam` for filters and paging, and `@RequestBody` for the payload — so a create is a POST with a DTO body, and a search is a GET with query params. There's only one `@RequestBody` per method because the request stream is read once. Internally `@RequestBody` goes through Jackson via an `HttpMessageConverter`, and `@Valid` runs right after binding, before our method is called. We never put tokens or sensitive values in query strings because they end up in access logs."

---

[← 18. Controllers](18-controllers.md) | [Contents](../README.md) | [20. Sending Responses — ResponseEntity and Status Codes →](20-sending-responses-responseentity-and-status-codes.md)
