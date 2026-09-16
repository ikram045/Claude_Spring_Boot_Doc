[← Back to Contents](../README.md) · Part F — Master Flows and Final Revision

---

# 48. Complete REST API Request/Response Flow

**Scenario:** `POST /api/v1/orders` with a JWT, creating an order.

```
CLIENT
  POST /api/v1/orders
  Authorization: Bearer eyJhbGc...
  Content-Type: application/json
  { "customerId": "CUST-1", "items": [{ "productId": 10, "quantity": 2 }] }
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 1. TOMCAT                                                       ║
 ║    accept connection → take a worker thread → parse HTTP        ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 2. FILTER CHAIN                                                 ║
 ║    CorrelationIdFilter → generate/propagate X-Correlation-Id,   ║
 ║                          put in MDC (all logs now carry it)     ║
 ║    JwtAuthenticationFilter → verify signature, check exp,       ║
 ║                          load UserDetails,                      ║
 ║                          SecurityContextHolder.setAuthentication║
 ║    AuthorizationFilter → URL rules; not permitted → 403         ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 3. DISPATCHER SERVLET — doDispatch()                            ║
 ║                                                                 ║
 ║  3a. HandlerMapping                                             ║
 ║      match POST + /api/v1/orders                                ║
 ║      → HandlerExecutionChain(OrderController.createOrder,        ║
 ║                              [interceptors])                    ║
 ║      no match → 404                                             ║
 ║                                                                 ║
 ║  3b. Interceptor.preHandle()                                    ║
 ║      record start time; returns false → request stops           ║
 ║                                                                 ║
 ║  3c. HandlerAdapter = RequestMappingHandlerAdapter              ║
 ║                                                                 ║
 ║  3d. ARGUMENT RESOLUTION                                        ║
 ║      @RequestBody → RequestResponseBodyMethodProcessor          ║
 ║        → MappingJackson2HttpMessageConverter                    ║
 ║        → JSON deserialized into OrderRequestDto                 ║
 ║        → malformed JSON → HttpMessageNotReadableException → 400  ║
 ║      @Valid → Hibernate Validator runs the constraints          ║
 ║        → violation → MethodArgumentNotValidException → 400       ║
 ║          with a field → message map                             ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │  valid DTO
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 4. CONTROLLER                                                   ║
 ║    orderService.createOrder(dto)   ← one line, no logic here    ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 5. SERVICE  (through the @Transactional AOP proxy)              ║
 ║    ── TRANSACTION BEGINS ──                                     ║
 ║       connection borrowed from HikariCP, autoCommit=false,      ║
 ║       bound to this thread; persistence context opens           ║
 ║                                                                 ║
 ║    @PreAuthorize check (if present)                             ║
 ║    business rules → violation → domain exception                ║
 ║    mapper.toEntity(dto)                                         ║
 ║    repository.save(order)                                       ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 6. REPOSITORY → JPA → HIBERNATE                                 ║
 ║    persist() → entity MANAGED; INSERT queued (not yet sent)     ║
 ║    cascade applies to OrderItems                                ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 7. SERVICE: map entity → DTO                                    ║
 ║    ★ INSIDE the transaction, so lazy associations still load    ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 8. TRANSACTION COMMIT                                           ║
 ║    flush: dirty checking → INSERT/UPDATE statements sent        ║
 ║    commit → connection returned to the pool                     ║
 ║    persistence context closes → entities DETACHED               ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │  DTO (safe — plain object, no proxies)
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 9. RETURN VALUE HANDLING                                        ║
 ║    ResponseEntity.created(location).body(dto)                   ║
 ║    content negotiation → Jackson serializes DTO → JSON          ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
 ╔═══▼════════════════════════════════════════════════════════════╗
 ║ 10. Interceptor.postHandle()  → skipped if an exception occurred║
 ║     Interceptor.afterCompletion() → ALWAYS runs (timing/cleanup)║
 ║     Filters unwind → MDC.clear()  ← MUST happen (pooled thread) ║
 ╚═══┬════════════════════════════════════════════════════════════╝
     │
   HTTP/1.1 201 Created
   Location: /api/v1/orders/1001
   X-Correlation-Id: a1b2c3
   { "id": 1001, "status": "PENDING", "totalAmount": 1500.00 }
```

**How to narrate this in 45 seconds:**
> "Tomcat assigns a thread, filters handle the correlation ID and JWT authentication, then the DispatcherServlet finds the controller method through HandlerMapping. Jackson converts the JSON body to a DTO, validation runs, and the controller calls the service. The service's `@Transactional` proxy opens a transaction, the repository persists via Hibernate, and we map to a DTO *inside* the transaction so lazy fields still load. On commit Hibernate flushes and the connection returns to the pool. The DTO goes back through Jackson as JSON with a 201 and a Location header, interceptors record timing, and the filters clear the MDC."

---

[← 47. Complete Spring Boot Architecture Flow](47-complete-spring-boot-architecture-flow.md) | [Contents](../README.md) | [49. Complete JPA/Hibernate Flow →](49-complete-jpa-hibernate-flow.md)
