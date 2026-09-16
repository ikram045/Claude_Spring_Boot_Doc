[← Back to Contents](../README.md) · Part F — Master Flows and Final Revision

---

# 51. Complete Exception Handling Flow

```
 EXCEPTION THROWN — where it happens determines what handles it
 ═══════════════════════════════════════════════════════════════

 ┌─ Thrown in a FILTER ──────────────────────────────────────┐
 │   ✘ @RestControllerAdvice CANNOT catch it                 │
 │     (filters run before DispatcherServlet in the stack)   │
 │   → handle inside the filter: write JSON manually         │
 │   → or Spring Security's AuthenticationEntryPoint (401)   │
 │                              AccessDeniedHandler    (403) │
 └───────────────────────────────────────────────────────────┘

 ┌─ Thrown in INTERCEPTOR / CONTROLLER / SERVICE / REPOSITORY ┐
 │   ✔ caught by DispatcherServlet                            │
 └────────────────────────┬───────────────────────────────────┘
                          ▼
              processHandlerException()
                          │
                          ▼
         HandlerExceptionResolver chain, in order:
                          │
    ┌─────────────────────▼──────────────────────────────┐
    │ 1. ExceptionHandlerExceptionResolver               │
    │      search @ExceptionHandler:                     │
    │        a) in the SAME controller  (wins)           │
    │        b) in @RestControllerAdvice                 │
    │      choose the MOST SPECIFIC exception type       │
    │      (by inheritance distance, NOT declaration     │
    │       order)                                        │
    │      found → invoke → build response → DONE        │
    └─────────────────────┬──────────────────────────────┘
                          │ not found
    ┌─────────────────────▼──────────────────────────────┐
    │ 2. ResponseStatusExceptionResolver                 │
    │      @ResponseStatus on the exception class,       │
    │      or ResponseStatusException                    │
    └─────────────────────┬──────────────────────────────┘
                          │ not handled
    ┌─────────────────────▼──────────────────────────────┐
    │ 3. DefaultHandlerExceptionResolver                 │
    │      HttpRequestMethodNotSupported     → 405       │
    │      HttpMediaTypeNotSupported         → 415       │
    │      MethodArgumentNotValid            → 400       │
    │      NoHandlerFound                    → 404       │
    └─────────────────────┬──────────────────────────────┘
                          │ still nothing
                          ▼
              forward to /error (BasicErrorController)
                    → default 500 JSON
```

## Standard exception → status mapping

| Exception | Status | Meaning |
|---|---|---|
| `MethodArgumentNotValidException` | 400 | `@Valid` on the body failed |
| `ConstraintViolationException` | 400 | `@Validated` on params failed |
| `HttpMessageNotReadableException` | 400 | Malformed JSON |
| `MethodArgumentTypeMismatchException` | 400 | Wrong parameter type |
| `AuthenticationException` | 401 | Not authenticated |
| `AccessDeniedException` | 403 | Authenticated, not permitted |
| `ResourceNotFoundException` (ours) | 404 | Missing resource |
| `HttpRequestMethodNotSupportedException` | 405 | Wrong HTTP method |
| `DuplicateResourceException` / `DataIntegrityViolationException` | 409 | Conflict |
| `OptimisticLockingFailureException` | 409 | Concurrent modification |
| `HttpMediaTypeNotSupportedException` | 415 | Wrong Content-Type |
| Business rule violation | 422 | Semantically invalid |
| `Exception` (catch-all) | 500 | **Bug — log with stack trace** |

## The rules

1. **Business exceptions extend `RuntimeException`** — checked exceptions do not roll back transactions.
2. **Never swallow** — catching without rethrowing commits a failed transaction.
3. **Never expose** stack traces, SQL or constraint names.
4. **Log by severity** — client mistakes at WARN, unexpected exceptions at ERROR with the trace.
5. **Always include a correlation ID** in the error response.
6. **Always have a catch-all** `@ExceptionHandler(Exception.class)`.

---

[← 50. Complete Spring Security / JWT Flow](50-complete-spring-security-jwt-flow.md) | [Contents](../README.md) | [52. Complete Transaction Flow →](52-complete-transaction-flow.md)
