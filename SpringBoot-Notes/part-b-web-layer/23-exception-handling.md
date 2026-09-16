[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 23. Exception Handling

## 23.1 What is it?

**Simple words:**
Exception handling means deciding **what the client sees when something goes wrong**. Instead of a stack trace or a bare 500, the client should get a clear, consistent error message with the right status code.

Spring lets you write this **once, in one class**, instead of try/catch in every controller.

**Technical wording:**
Spring MVC resolves exceptions through the `HandlerExceptionResolver` chain. `@ExceptionHandler` methods — declared locally in a controller or globally in a `@RestControllerAdvice` — are discovered by `ExceptionHandlerExceptionResolver` and invoked to produce the error response.

## 23.2 Why centralise it?

**Without central handling:**

```java
@GetMapping("/{id}")
public ResponseEntity<?> getOrder(@PathVariable Long id) {
    try {
        return ResponseEntity.ok(orderService.getOrderById(id));
    } catch (OrderNotFoundException e) {
        return ResponseEntity.status(404).body(Map.of("error", e.getMessage()));
    } catch (Exception e) {
        return ResponseEntity.status(500).body(Map.of("error", "Something went wrong"));
    }
}
```
Repeated in **every** method. Inconsistent formats. Business logic buried in error handling.

**With central handling:**

```java
@GetMapping("/{id}")
public ResponseEntity<OrderDto> getOrder(@PathVariable Long id) {
    return ResponseEntity.ok(orderService.getOrderById(id));   // clean
}
```
The exception propagates, and the advice turns it into a proper response. **Consistent format across the whole API, in one place.**

## 23.3 The complete exception-handling flow

```
    Controller / Service / Repository throws an exception
                       │
                       ▼
        Propagates up to DispatcherServlet
                       │
                       ▼
        processHandlerException()
                       │
                       ▼
        HandlerExceptionResolver chain (in order):
                       │
        ┌──────────────┴──────────────────────────────────┐
        │                                                  │
        ▼                                                  │
  1. ExceptionHandlerExceptionResolver                     │
       - looks for @ExceptionHandler in the SAME controller│
       - then in @ControllerAdvice / @RestControllerAdvice │
       - picks the MOST SPECIFIC matching type             │
       - found? → invoke it → build the response → DONE ───┤
       - not found? ↓                                      │
        │                                                  │
        ▼                                                  │
  2. ResponseStatusExceptionResolver                       │
       - handles @ResponseStatus-annotated exceptions      │
         and ResponseStatusException             ──────────┤
        │                                                  │
        ▼                                                  │
  3. DefaultHandlerExceptionResolver                       │
       - maps standard Spring MVC exceptions:              │
         HttpRequestMethodNotSupportedException → 405      │
         HttpMediaTypeNotSupportedException     → 415      │
         MethodArgumentNotValidException        → 400      │
         NoHandlerFoundException                → 404 ─────┤
        │                                                  │
        ▼                                                  │
  4. Nothing handled it                                    │
       → forwarded to /error (BasicErrorController)        │
       → default 500 JSON response                         │
        └─────────────────────────────────────────────────┘
                       │
                       ▼
              HTTP error response to client
```

## 23.4 The building blocks

### Custom exceptions

```java
// package com.company.orderservice.exception

// Base class — one root for all our business exceptions
public abstract class BusinessException extends RuntimeException {
    private final String errorCode;
    protected BusinessException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
    public String getErrorCode() { return errorCode; }
}

public class ResourceNotFoundException extends BusinessException {
    public ResourceNotFoundException(String resource, Object id) {
        super("RESOURCE_NOT_FOUND", String.format("%s not found with id: %s", resource, id));
    }
}

public class InsufficientBalanceException extends BusinessException {
    public InsufficientBalanceException(BigDecimal required, BigDecimal available) {
        super("INSUFFICIENT_BALANCE",
              String.format("Required %s but available %s", required, available));
    }
}

public class DuplicateResourceException extends BusinessException {
    public DuplicateResourceException(String field, String value) {
        super("DUPLICATE_RESOURCE", String.format("%s '%s' already exists", field, value));
    }
}
```

**Why extend `RuntimeException` and not `Exception`?**
This is important and frequently asked. A **checked** exception forces every caller to `throws` or catch it, polluting every signature up the stack. More critically, **Spring's declarative transaction management rolls back on `RuntimeException` by default but NOT on checked exceptions.** A checked business exception would let a partially completed transaction **commit**. Always use unchecked exceptions for business errors.

### The error response DTO

```java
@Getter @Builder
@JsonInclude(JsonInclude.Include.NON_NULL)     // omit null fields
public class ErrorResponse {
    private Instant timestamp;
    private int status;
    private String error;              // "Not Found"
    private String errorCode;          // "RESOURCE_NOT_FOUND" — machine-readable
    private String message;            // human-readable
    private String path;               // which endpoint
    private String correlationId;      // ties to our logs
    private Map<String, String> fieldErrors;   // only for validation failures
}
```

**Why include both `errorCode` and `message`:** the client's code branches on the stable `errorCode`; the `message` is for humans and can be reworded or localised without breaking clients.

**Why include `correlationId`:** the client reports "I got this error with correlation ID abc-123", and you find that exact request across every microservice log. This is the single most useful field in a production error response.

### The global handler

```java
package com.company.orderservice.exception;

@RestControllerAdvice           // = @ControllerAdvice + @ResponseBody
@Slf4j
@RequiredArgsConstructor
public class GlobalExceptionHandler {

    // ---------- 404 ----------
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex,
                                                        HttpServletRequest request) {
        log.warn("Resource not found: {}", ex.getMessage());       // WARN, not ERROR
        return build(HttpStatus.NOT_FOUND, ex.getErrorCode(), ex.getMessage(), request, null);
    }

    // ---------- 422 business rule violation ----------
    @ExceptionHandler(InsufficientBalanceException.class)
    public ResponseEntity<ErrorResponse> handleInsufficientBalance(
            InsufficientBalanceException ex, HttpServletRequest request) {
        log.warn("Business rule violation: {}", ex.getMessage());
        return build(HttpStatus.UNPROCESSABLE_ENTITY, ex.getErrorCode(),
                     ex.getMessage(), request, null);
    }

    // ---------- 409 duplicate ----------
    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ErrorResponse> handleDuplicate(DuplicateResourceException ex,
                                                         HttpServletRequest request) {
        return build(HttpStatus.CONFLICT, ex.getErrorCode(), ex.getMessage(), request, null);
    }

    // ---------- 400 validation (@Valid on @RequestBody) ----------
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex,
                                                          HttpServletRequest request) {
        Map<String, String> fieldErrors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(e -> fieldErrors.put(e.getField(), e.getDefaultMessage()));
        ex.getBindingResult().getGlobalErrors()
          .forEach(e -> fieldErrors.put(e.getObjectName(), e.getDefaultMessage()));
        return build(HttpStatus.BAD_REQUEST, "VALIDATION_FAILED",
                     "Request contains invalid fields", request, fieldErrors);
    }

    // ---------- 400 validation (@Validated on params) ----------
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ErrorResponse> handleConstraintViolation(
            ConstraintViolationException ex, HttpServletRequest request) {
        Map<String, String> errors = ex.getConstraintViolations().stream()
                .collect(Collectors.toMap(v -> v.getPropertyPath().toString(),
                                          ConstraintViolation::getMessage, (a, b) -> a));
        return build(HttpStatus.BAD_REQUEST, "VALIDATION_FAILED",
                     "Request parameters are invalid", request, errors);
    }

    // ---------- 400 malformed JSON ----------
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<ErrorResponse> handleUnreadable(HttpMessageNotReadableException ex,
                                                          HttpServletRequest request) {
        // Deliberately do NOT echo ex.getMessage() — it can leak class names and structure
        return build(HttpStatus.BAD_REQUEST, "MALFORMED_REQUEST",
                     "Request body is malformed or unreadable", request, null);
    }

    // ---------- 400 wrong parameter type ----------
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ErrorResponse> handleTypeMismatch(
            MethodArgumentTypeMismatchException ex, HttpServletRequest request) {
        String message = String.format("Parameter '%s' has invalid value '%s'",
                                       ex.getName(), ex.getValue());
        return build(HttpStatus.BAD_REQUEST, "INVALID_PARAMETER", message, request, null);
    }

    // ---------- 409 database constraint ----------
    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<ErrorResponse> handleDataIntegrity(DataIntegrityViolationException ex,
                                                             HttpServletRequest request) {
        log.error("Data integrity violation", ex);
        // Never expose the raw SQL/constraint name to the client
        return build(HttpStatus.CONFLICT, "DATA_CONFLICT",
                     "Operation conflicts with existing data", request, null);
    }

    // ---------- 403 ----------
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex,
                                                            HttpServletRequest request) {
        return build(HttpStatus.FORBIDDEN, "ACCESS_DENIED",
                     "You do not have permission to perform this action", request, null);
    }

    // ---------- 500 catch-all — MUST BE LAST ----------
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAll(Exception ex, HttpServletRequest request) {
        // ERROR level + full stack trace → this is a bug we must investigate
        log.error("Unhandled exception on {} {}", request.getMethod(),
                  request.getRequestURI(), ex);
        // Generic message to the client — NEVER leak internals
        return build(HttpStatus.INTERNAL_SERVER_ERROR, "INTERNAL_ERROR",
                     "An unexpected error occurred. Please contact support.", request, null);
    }

    private ResponseEntity<ErrorResponse> build(HttpStatus status, String code, String message,
                                                HttpServletRequest request,
                                                Map<String, String> fieldErrors) {
        ErrorResponse body = ErrorResponse.builder()
                .timestamp(Instant.now())
                .status(status.value())
                .error(status.getReasonPhrase())
                .errorCode(code)
                .message(message)
                .path(request.getRequestURI())
                .correlationId(MDC.get("correlationId"))   // ties response to logs
                .fieldErrors(fieldErrors)
                .build();
        return ResponseEntity.status(status).body(body);
    }
}
```

**Three practices in that code worth explaining to a senior:**

1. **Log levels differ by cause.** Client mistakes (404, validation) are `WARN` — they're expected. Unhandled exceptions are `ERROR` with a stack trace — they're bugs. If you log 404s at ERROR, your alerting becomes noise and real incidents get missed.
2. **Never leak internals.** No stack traces, SQL, constraint names or class names in responses. That's reconnaissance material for an attacker.
3. **The catch-all is a safety net, not a design.** Every *expected* failure should have its own handler with a meaningful status code.

## 23.5 Exception handler resolution order

```java
@ExceptionHandler(ResourceNotFoundException.class)   // most specific
@ExceptionHandler(BusinessException.class)           // parent
@ExceptionHandler(RuntimeException.class)            // grandparent
@ExceptionHandler(Exception.class)                   // catch-all
```
Spring picks the **most specific match by class hierarchy distance**, not declaration order. So a `ResourceNotFoundException` hits the first handler even if `Exception.class` is declared earlier in the file.

**Local vs global precedence:** an `@ExceptionHandler` inside the controller **wins over** one in `@ControllerAdvice`. That lets one controller override the global behaviour for a specific case.

**Ordering multiple advices:** if you have several `@RestControllerAdvice` classes, control precedence with `@Order`. A common pattern is a general advice plus a higher-priority one for a specific package:

```java
@RestControllerAdvice(basePackages = "com.company.orderservice.admin")
@Order(Ordered.HIGHEST_PRECEDENCE)
public class AdminExceptionHandler { }
```

## 23.6 `@ControllerAdvice` vs `@RestControllerAdvice`

| | `@ControllerAdvice` | `@RestControllerAdvice` |
|---|---|---|
| Composition | `@Component` | **`@ControllerAdvice` + `@ResponseBody`** |
| Return value | View name unless `@ResponseBody` | Serialized to JSON |
| Use for | MVC apps with HTML views | **REST APIs** |

`@ControllerAdvice` can also scope itself:
```java
@RestControllerAdvice(basePackages = "com.company.orderservice.controller")
@RestControllerAdvice(assignableTypes = {OrderController.class})
@RestControllerAdvice(annotations = RestController.class)
```

## 23.7 Other mechanisms

```java
// @ResponseStatus directly on an exception class — quick, but less flexible
@ResponseStatus(HttpStatus.NOT_FOUND)
public class OrderNotFoundException extends RuntimeException { }

// ResponseStatusException — throw a status inline without a custom class
throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Order not found");

// ProblemDetail (RFC 7807) — the standardised error format, Spring 6 / Boot 3
@ExceptionHandler(ResourceNotFoundException.class)
public ProblemDetail handle(ResourceNotFoundException ex) {
    ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    pd.setTitle("Resource Not Found");
    pd.setType(URI.create("https://api.company.com/errors/not-found"));
    pd.setProperty("errorCode", ex.getErrorCode());
    return pd;
}
```
```properties
# Enable RFC 7807 responses for Spring's built-in exceptions
spring.mvc.problemdetails.enabled=true
```

**Additional clarification:** `ProblemDetail` is the modern, standards-based approach in Boot 3. If you're starting a new service, it's worth proposing — clients get a predictable `type`/`title`/`status`/`detail`/`instance` structure defined by RFC 7807, rather than a bespoke shape per team.

## 23.8 What `@RestControllerAdvice` CANNOT catch

This is a genuinely important limitation, and a strong interview answer.

```
Request
   │
   ▼
FILTERS  ◄── exceptions HERE are NOT caught by @RestControllerAdvice
   │            (filters run BEFORE DispatcherServlet exists in the call stack)
   ▼
DispatcherServlet
   │
   ▼
Interceptors, Controller, Service, Repository  ◄── these ARE caught
```

**Consequence:** a JWT authentication filter that throws produces the container's default HTML error page, not your clean JSON. **This surprises a lot of developers.**

**Solutions:**
1. Catch inside the filter and write the JSON response yourself with an `ObjectMapper`.
2. Use Spring Security's `AuthenticationEntryPoint` and `AccessDeniedHandler` (the correct approach for security filters — see Section 42).
3. Add a `@Component` `ErrorController` to customise the `/error` fallback.

```java
// Writing a clean JSON error from inside a filter
private void writeError(HttpServletResponse response, HttpStatus status, String message)
        throws IOException {
    response.setStatus(status.value());
    response.setContentType(MediaType.APPLICATION_JSON_VALUE);
    ErrorResponse body = ErrorResponse.builder()
            .timestamp(Instant.now()).status(status.value())
            .error(status.getReasonPhrase()).message(message).build();
    objectMapper.writeValue(response.getOutputStream(), body);
}
```

## 23.9 Common Mistakes

| Mistake | Consequence |
|---|---|
| try/catch in every controller | Duplication, inconsistency |
| **Swallowing exceptions** (`catch (Exception e) { }`) | Silent failure — the worst bug class |
| Exposing stack traces | Security exposure |
| Everything returns 500 | Clients can't react appropriately |
| Checked exceptions for business errors | **Transaction does not roll back** |
| Logging 404s at ERROR | Alert fatigue; real incidents get missed |
| Catching an exception and rethrowing without the cause | Stack trace lost — undebuggable |
| Expecting advice to catch filter exceptions | Ugly HTML error instead of JSON |
| No catch-all handler | Unexpected exceptions leak Spring's default error body |
| Returning the raw DB message | Leaks schema and constraint names |

## 23.10 TL Questions

**Q: How do we handle exceptions?**
A: Centrally, with a `@RestControllerAdvice`. Controllers and services throw domain exceptions; the advice maps each type to a status code and a consistent error body, so every endpoint returns errors in the same shape.

**Q: Why not try/catch in controllers?**
A: It duplicates the same code everywhere, makes formats drift between endpoints, and mixes error handling into business flow. One advice class gives us consistency and a single place to change.

**Q: Why do our business exceptions extend `RuntimeException`?**
A: Two reasons. Checked exceptions force `throws` declarations through every layer, and more importantly Spring rolls back on `RuntimeException` by default but **not** on checked exceptions — a checked business exception would let a partially completed transaction commit.

**Q: What happens internally when an exception is thrown?**
A: It propagates to DispatcherServlet, which runs the `HandlerExceptionResolver` chain. `ExceptionHandlerExceptionResolver` looks for a matching `@ExceptionHandler` first in the controller, then in the advice classes, choosing the most specific type match. If nothing matches, the request forwards to `/error`.

**Q: What if an exception is thrown in a filter?**
A: The advice can't catch it, because filters run before DispatcherServlet. We handle it inside the filter by writing the JSON response directly, and for security filters we use Spring Security's `AuthenticationEntryPoint` and `AccessDeniedHandler`.

**Q: What do we return for an unexpected exception?**
A: A generic 500 with a correlation ID and no internal detail. The full stack trace goes to the logs at ERROR level. Exposing stack traces would leak our internal structure to an attacker.

**Q: How do we make production errors debuggable?**
A: Every request gets a correlation ID from a filter, stored in the MDC so it appears on every log line and is returned in the error response. When a user reports an error we search that one ID and see the whole request across all services.

**Q: What alternative can we use?**
A: `ProblemDetail` (RFC 7807), standard in Boot 3, which gives a standardised error format instead of a custom shape. For new services that's worth adopting.

## 23.11 Interview Questions

**Beginner — What is `@ControllerAdvice`?**
A specialised component that applies `@ExceptionHandler`, `@InitBinder` and `@ModelAttribute` methods globally across controllers.

**Beginner — `@ControllerAdvice` vs `@RestControllerAdvice`?**
`@RestControllerAdvice` adds `@ResponseBody`, so handler return values are serialized to JSON rather than treated as view names.

**Intermediate — How does Spring choose among multiple handlers?**
By the most specific match in the exception class hierarchy — measured by inheritance distance, not declaration order. Handlers in the controller itself take precedence over those in advice classes.

**Intermediate — `@ResponseStatus` vs `@ExceptionHandler`?**
`@ResponseStatus` on an exception class gives a fixed status with no custom body. `@ExceptionHandler` builds a full custom response, can inspect the exception, add headers and log. Use `@ExceptionHandler` for a real API.

**Intermediate — Why don't checked exceptions roll back transactions?**
Spring's default rollback rule is `RuntimeException` and `Error` only, following the EJB convention that checked exceptions are recoverable business conditions. Override with `@Transactional(rollbackFor = Exception.class)`.

**Advanced — What is the `HandlerExceptionResolver` chain?**
Ordered resolvers: `ExceptionHandlerExceptionResolver` (your `@ExceptionHandler`s), `ResponseStatusExceptionResolver` (`@ResponseStatus` and `ResponseStatusException`), and `DefaultHandlerExceptionResolver` (standard MVC exceptions → 405, 415, 400). The first to return a non-null `ModelAndView` wins.

**Advanced — Why can't advice catch filter exceptions?**
Filters execute in the servlet container's chain before `DispatcherServlet.doDispatch()` is entered, so there is no dispatcher frame to catch and resolve the exception. Handle it in the filter or via Spring Security's entry point.

**Advanced — What is `ProblemDetail`?**
Spring 6's implementation of RFC 7807 "Problem Details for HTTP APIs" — a standard error body with `type`, `title`, `status`, `detail` and `instance`, plus custom properties. Enabled for built-in exceptions via `spring.mvc.problemdetails.enabled=true`.

## 23.12 Quick Revision

1. Centralise in `@RestControllerAdvice` — never try/catch per controller.
2. Business exceptions extend **`RuntimeException`** so transactions roll back.
3. Consistent `ErrorResponse`: timestamp, status, errorCode, message, path, correlationId.
4. Map each exception to a **meaningful** status: 404, 409, 422, 400, 403.
5. Always include a **catch-all** `Exception` handler as a safety net.
6. Resolution = most specific type; controller-local beats global advice.
7. **Never expose stack traces, SQL or constraint names.**
8. Log client errors at WARN, unexpected errors at ERROR with the stack trace.
9. Advice **cannot** catch filter exceptions — handle those in the filter.
10. `ProblemDetail` (RFC 7807) is the modern standard format in Boot 3.
11. Include a correlation ID in every error — it makes production debugging possible.

## 23.13 TL Explanation (speak this)

> "All our error handling is centralised in a `@RestControllerAdvice`. Controllers and services just throw domain exceptions — `ResourceNotFoundException`, `InsufficientBalanceException` — and the advice maps each to the right status code and a consistent error body with a timestamp, error code, message and correlation ID. There's no try/catch in the controllers at all. Our business exceptions extend `RuntimeException` deliberately, because Spring only rolls back automatically on unchecked exceptions. We never return stack traces to clients, we log client mistakes at WARN and genuine bugs at ERROR, and the correlation ID in the response lets us find that exact request in the logs across every service. The one gap is that filters run before the DispatcherServlet, so exceptions there are handled in the filter itself or through Spring Security's entry point."

---

[← 22. Validation](22-validation.md) | [Contents](../README.md) | [24. Filters →](24-filters.md)
