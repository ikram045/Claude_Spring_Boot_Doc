[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 20. Sending Responses — ResponseEntity and Status Codes

## 20.1 Three ways to return a response

```java
// 1. Return the object directly — always 200 OK
@GetMapping("/{id}")
public OrderDto getOrder(@PathVariable Long id) {
    return orderService.getOrderById(id);
}

// 2. Object + @ResponseStatus — fixed, non-200 status
@PostMapping
@ResponseStatus(HttpStatus.CREATED)          // always 201
public OrderDto create(@Valid @RequestBody OrderRequestDto req) {
    return orderService.createOrder(req);
}

// 3. ResponseEntity — FULL control over status, headers and body (PREFERRED)
@PostMapping
public ResponseEntity<OrderDto> create(@Valid @RequestBody OrderRequestDto req) {
    OrderDto created = orderService.createOrder(req);
    return ResponseEntity
            .status(HttpStatus.CREATED)
            .header("X-Order-Id", created.getId().toString())
            .body(created);
}
```

| | Direct return | `@ResponseStatus` | `ResponseEntity` |
|---|---|---|---|
| Status control | Always 200 | Fixed at compile time | **Dynamic at runtime** |
| Custom headers | No | No | **Yes** |
| Conditional status | No | No | **Yes** |
| Verbosity | Lowest | Low | Higher |
| **Recommended for APIs** | Simple GETs | Simple creates | **Yes — default choice** |

## 20.2 ResponseEntity builder methods

```java
ResponseEntity.ok(dto)                                  // 200 + body
ResponseEntity.ok().build()                             // 200, no body
ResponseEntity.created(location).body(dto)              // 201 + Location header
ResponseEntity.accepted().body(dto)                     // 202
ResponseEntity.noContent().build()                      // 204
ResponseEntity.badRequest().body(errorDto)              // 400
ResponseEntity.notFound().build()                       // 404
ResponseEntity.status(HttpStatus.CONFLICT).body(err)    // any status
ResponseEntity.status(409).header("Retry-After","30").body(err)
```

**Creating with a Location header — the correct REST pattern:**

```java
@PostMapping
public ResponseEntity<OrderDto> create(@Valid @RequestBody OrderRequestDto req) {
    OrderDto created = orderService.createOrder(req);

    URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();                       // → http://host/api/v1/orders/42

    return ResponseEntity.created(location).body(created);
}
```
The client now knows where the new resource lives without guessing the URL pattern.

## 20.3 How the object becomes JSON

```
Controller returns OrderDto
        │
        ▼
HandlerMethodReturnValueHandler  (RequestResponseBodyMethodProcessor)
        │
        ▼
Content negotiation: what does the client's Accept header allow?
        │
        ▼
Select HttpMessageConverter → MappingJackson2HttpMessageConverter
        │
        ▼
Jackson ObjectMapper serializes:
   - getters become JSON fields  (getCustomerId() → "customerId")
   - @JsonProperty renames
   - @JsonIgnore omits
   - null handling per configuration
        │
        ▼
JSON bytes written to the response output stream
```

**Useful Jackson annotations on DTOs:**

```java
public class OrderResponseDto {

    private Long id;

    @JsonProperty("customer_id")                       // rename in JSON
    private String customerId;

    @JsonIgnore                                        // never serialize
    private String internalNotes;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")       // date format
    private LocalDateTime createdAt;

    @JsonInclude(JsonInclude.Include.NON_NULL)         // omit if null
    private String cancelReason;
}
```

```properties
# Global: omit all null fields from every response
spring.jackson.default-property-inclusion=non_null
# Global: ISO dates instead of timestamps
spring.jackson.serialization.write-dates-as-timestamps=false
```

## 20.4 Standard response envelope (a common corporate pattern)

Many teams wrap every response in a consistent shape so clients can parse uniformly:

```java
@Getter @AllArgsConstructor
public class ApiResponse<T> {
    private boolean success;
    private String message;
    private T data;
    private Instant timestamp;

    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, "Success", data, Instant.now());
    }
}

@GetMapping("/{id}")
public ResponseEntity<ApiResponse<OrderDto>> get(@PathVariable Long id) {
    return ResponseEntity.ok(ApiResponse.success(orderService.getOrderById(id)));
}
```

**Balanced view to give a TL:** an envelope gives clients one parsing shape and a place for metadata. The counter-argument is that HTTP *already* has a status code and headers for this, so the envelope duplicates them and can encourage the anti-pattern of returning 200 with `success: false`. **If you use an envelope, still set the correct HTTP status code.** Never return 200 for an error.

## 20.5 Common Mistakes

| Mistake | Consequence |
|---|---|
| **200 with an error body** | Clients, retry logic, monitoring and load balancers all treat it as success |
| Returning entities directly | Schema leak + `LazyInitializationException` |
| Forgetting the `Location` header on 201 | Client must construct the URL itself |
| Returning `null` from a controller | 200 with an empty body — use 404 |
| Exposing stack traces in responses | **Security risk** — leaks internals |
| Returning an unbounded list | Memory blowout — paginate |

## 20.6 TL Questions

**Q: Why do we use `ResponseEntity`?**
A: It gives us runtime control over the status code, headers and body. We can return 201 with a `Location` header on create and 204 on delete, instead of always returning 200.

**Q: How does our object become JSON?**
A: The return value handler performs content negotiation against the `Accept` header, selects the Jackson message converter, and Jackson serializes the object's getters into JSON on the response stream.

**Q: Why not return the entity directly?**
A: It couples the API contract to the database schema, so a column rename becomes a breaking API change. It also exposes fields we don't want public, and serializing a lazy association outside the persistence context throws `LazyInitializationException`.

**Q: Should errors return 200 with a flag?**
A: No. The HTTP status code is the contract that infrastructure understands — load balancers, retry logic, monitoring and the client library all key off it. An error must be 4xx or 5xx.

**Q: What happens if a controller returns null?**
A: The client gets a 200 with an empty body, which is misleading. If a resource doesn't exist we throw a `ResourceNotFoundException` so the advice returns a proper 404.

## 20.7 Interview Questions

**Beginner — What is `ResponseEntity`?**
A wrapper representing the full HTTP response: status code, headers and body.

**Intermediate — `@ResponseStatus` vs `ResponseEntity`?**
`@ResponseStatus` fixes the status at compile time; `ResponseEntity` decides it at runtime and also allows custom headers. `@ResponseStatus` is convenient on exception classes.

**Intermediate — What is content negotiation?**
Choosing the response representation based on the client's `Accept` header, the method's `produces` attribute, and the registered converters. With only Jackson on the classpath, it effectively always resolves to JSON.

**Advanced — How do you add a custom `HttpMessageConverter`?**
Implement `WebMvcConfigurer` and override `extendMessageConverters` (to adjust the defaults) or `configureMessageConverters` (to replace them entirely). Order matters — the first converter that can handle the media type wins.

**Advanced — How do you stream a large response?**
Return `StreamingResponseBody`, or a `ResponseEntity<Resource>` for files, so data is written incrementally rather than buffered fully in memory. For long-lived pushes, `SseEmitter` provides server-sent events.

## 20.8 Quick Revision

1. Prefer `ResponseEntity` — full control of status, headers, body.
2. 201 + `Location` on create; 204 on delete; 200 for reads and updates.
3. Never return 200 for an error.
4. Never return entities — return DTOs.
5. Jackson serializes via `HttpMessageConverter` after content negotiation.
6. `@JsonIgnore`, `@JsonProperty`, `@JsonFormat`, `@JsonInclude` shape the JSON.
7. Never expose stack traces to clients.
8. Always paginate collection endpoints.

## 20.9 TL Explanation (speak this)

> "We return `ResponseEntity` from our controllers so we control the status code and headers at runtime — 201 with a `Location` header when we create something, 204 on delete, 200 on reads. The body is always a DTO, never an entity, so the API contract doesn't move every time the schema changes and we avoid lazy-loading serialization errors. Jackson does the serialization through a message converter after content negotiation. The rule we hold firmly is that errors get a real 4xx or 5xx status — never a 200 with a success flag set to false, because all our infrastructure keys off the status code."

---

[← 19. Reading Request Data — @PathVariable, @RequestParam, @RequestBody](19-reading-request-data-pathvariable-requestparam-requestbody.md) | [Contents](../README.md) | [21. DTO Pattern →](21-dto-pattern.md)
