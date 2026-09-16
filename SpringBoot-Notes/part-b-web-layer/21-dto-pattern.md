[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 21. DTO Pattern

## 21.1 What is it?

**Simple words:**
A DTO (Data Transfer Object) is a **simple class used only to carry data between your API and the outside world**. It is not your database entity — it's the shape you agreed to expose to clients.

Think of it as the difference between what's in your warehouse (entity) and what's on the price tag the customer sees (DTO).

**Technical wording:**
A DTO is a flat, serializable object used to transfer data across architectural boundaries, decoupling the external API contract from the internal domain/persistence model.

## 21.2 Why do we use DTOs? (Seven concrete reasons)

### Reason 1 — Security: don't leak sensitive fields

```java
@Entity
public class User {
    private Long id;
    private String username;
    private String password;      // BCrypt hash
    private String ssn;
    private boolean internalFlag;
}
```
Return this entity from a controller and Jackson serializes **every** field — including the password hash and SSN. A DTO exposes only what's safe.

### Reason 2 — Decoupling: schema changes don't break clients
Rename a column, split a table, or switch to a different persistence model — the DTO keeps the API contract stable. Without DTOs, **every database refactor becomes a breaking API change**.

### Reason 3 — Avoid `LazyInitializationException`
Serializing an entity with a `LAZY` association after the transaction has closed throws, because Jackson triggers the proxy outside the persistence context. DTOs are plain objects built while the session is open — no proxies, no surprise.

### Reason 4 — Avoid infinite recursion
A bidirectional relationship (`Order` → `List<OrderItem>` → `Order` → …) makes Jackson loop forever and blow the stack. DTOs are unidirectional by construction.

### Reason 5 — Shape data for the client
One screen may need order + customer name + item count — data from three tables. A DTO models the *response*, not a table.

### Reason 6 — Different shapes for input and output
The create request has no `id` and no `createdAt`; the response has both and no password. One entity can't represent both correctly.

### Reason 7 — Validation belongs on the DTO
Validation rules are API concerns (`@NotBlank`, `@Email`). Putting them on the entity mixes API validation with persistence constraints.

## 21.3 Entity vs DTO

| Aspect | Entity | DTO |
|---|---|---|
| Purpose | Map to a DB table | Transfer data over the API |
| Annotations | `@Entity`, `@Table`, `@Column`, `@OneToMany` | Validation + Jackson only |
| Contains | Persistence state, relationships, lazy proxies | Plain, flat fields |
| Layer | Repository / persistence | Controller / API boundary |
| Managed by | Hibernate persistence context | Nobody — plain object |
| Changes when | The schema changes | The API contract changes |
| Exposed to clients | **Never** | Yes |

## 21.4 Real example — request and response DTOs

```java
// ---------- dto/OrderRequestDto.java (what the client SENDS) ----------
@Getter @Setter
@NoArgsConstructor @AllArgsConstructor     // no-arg needed by Jackson
public class OrderRequestDto {

    @NotBlank(message = "Customer ID is required")
    private String customerId;

    @NotEmpty(message = "Order must contain at least one item")
    @Valid                                  // cascade validation into each item
    private List<OrderItemDto> items;

    @NotNull(message = "Payment mode is required")
    private PaymentMode paymentMode;

    // NOTE: no id, no createdAt, no status — the SERVER decides those.
    // Accepting them from the client is a security hole (mass-assignment).
}

// ---------- dto/OrderResponseDto.java (what the client RECEIVES) ----------
@Getter @Builder
public class OrderResponseDto {

    private Long id;
    private String customerId;
    private String customerName;             // joined from another table
    private BigDecimal totalAmount;
    private OrderStatus status;

    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    private LocalDateTime createdAt;

    private List<OrderItemResponseDto> items;

    // NOTE: no internal audit fields, no version column, no soft-delete flag
}
```

**Mass assignment — a real security issue:** if you bind the request straight onto the entity, a client can send `{"status":"PAID","totalAmount":0}` and set fields they should never control. A request DTO that simply doesn't contain those fields makes the attack impossible.

## 21.5 Mapping between entity and DTO

### Option 1 — Manual mapper (no dependency, fully explicit)

```java
@Component
public class OrderMapper {

    public Order toEntity(OrderRequestDto dto) {
        Order order = new Order();
        order.setCustomerId(dto.getCustomerId());
        order.setPaymentMode(dto.getPaymentMode());
        order.setStatus(OrderStatus.PENDING);      // server-controlled
        return order;
    }

    public OrderResponseDto toDto(Order order) {
        return OrderResponseDto.builder()
                .id(order.getId())
                .customerId(order.getCustomerId())
                .totalAmount(order.getTotalAmount())
                .status(order.getStatus())
                .createdAt(order.getCreatedAt())
                .items(order.getItems().stream()
                            .map(this::toItemDto)
                            .toList())
                .build();
    }
}
```

### Option 2 — MapStruct (compile-time, recommended for large projects)

```java
@Mapper(componentModel = "spring")          // generates a Spring bean
public interface OrderMapper {

    OrderResponseDto toDto(Order order);

    @Mapping(target = "id",     ignore = true)
    @Mapping(target = "status", constant = "PENDING")
    Order toEntity(OrderRequestDto dto);
}
```
MapStruct **generates the implementation at compile time** — so it's as fast as hand-written code, and a field mismatch is a compile error rather than a runtime surprise.

### Mapping approaches compared

| | Manual | MapStruct | ModelMapper |
|---|---|---|---|
| Speed | Fastest | Fastest (generated) | Slower (reflection) |
| Errors caught | Compile time | **Compile time** | **Runtime** |
| Boilerplate | High | Very low | Very low |
| Debuggable | Yes | Yes (readable generated code) | Harder |
| **Recommendation** | Small projects | **Large projects** | Prototypes |

**Warning about ModelMapper:** it maps by name at runtime using reflection. A field rename silently stops mapping and you ship a `null`. For that reason many teams standardise on MapStruct.

## 21.6 Where should mapping happen?

**Recommended: in the service layer.**

```java
@Service
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final OrderRepository orderRepository;
    private final OrderMapper orderMapper;

    @Override
    @Transactional
    public OrderResponseDto createOrder(OrderRequestDto request) {
        Order order = orderMapper.toEntity(request);          // DTO → entity
        order.setTotalAmount(calculateTotal(request.getItems()));
        Order saved = orderRepository.save(order);
        return orderMapper.toDto(saved);                       // entity → DTO
    }
}
```

**Why the service and not the controller?** Because mapping the entity to a DTO must happen **while the transaction is still open**, so lazy associations can still be loaded. If the controller did the mapping, the session would already be closed and you'd get `LazyInitializationException` (unless `open-in-view` is on — which you should disable; see Section 39).

**Key rule: the entity must never escape the service layer.**

```
Controller  ←── DTO ──►  Service  ←── Entity ──►  Repository  ←──►  DB
                            ▲
                    mapping happens HERE,
                    inside the transaction
```

## 21.7 Projections — a lighter alternative

For read-only endpoints you can skip the mapper entirely by having the query return the DTO directly:

```java
// Interface projection — Spring Data implements it
public interface OrderSummary {
    Long getId();
    String getCustomerId();
    BigDecimal getTotalAmount();
}

public interface OrderRepository extends JpaRepository<Order, Long> {
    List<OrderSummary> findByStatus(OrderStatus status);
}

// Class-based (DTO) projection via JPQL constructor expression
@Query("SELECT new com.company.dto.OrderSummaryDto(o.id, o.customerId, o.totalAmount) " +
       "FROM Order o WHERE o.status = :status")
List<OrderSummaryDto> findSummaries(@Param("status") OrderStatus status);
```

**Why this is a performance win:** the generated SQL selects **only those three columns** instead of `SELECT *`, and no entities enter the persistence context, so there's no dirty-checking overhead. For list screens on wide tables this is a measurable improvement — a strong point to raise with a senior.

## 21.8 Common Mistakes

| Mistake | Consequence |
|---|---|
| Returning entities from controllers | Schema leak, lazy exceptions, recursion, security exposure |
| One DTO for both request and response | Exposes server-controlled fields; weakens validation |
| Accepting `id`/`status` in a create request | Mass-assignment vulnerability |
| Mapping in the controller | `LazyInitializationException` |
| Putting business logic in a DTO | DTOs are dumb data carriers |
| JPA annotations on a DTO | Confuses the layers |
| Too many near-identical DTOs | Maintenance burden — consolidate sensibly |

## 21.9 TL Questions

**Q: Why do we use DTOs instead of returning entities?**
A: Three main reasons — security, because entities contain fields like password hashes we must never serialize; decoupling, so a schema change isn't a breaking API change; and stability, because serializing lazy associations outside the transaction throws `LazyInitializationException` and bidirectional relationships cause infinite recursion in Jackson.

**Q: Where do we do the mapping?**
A: In the service layer, inside the transaction, so lazy associations can still be initialised. The entity never leaves the service — the controller only ever sees DTOs.

**Q: Why separate request and response DTOs?**
A: They have genuinely different shapes. The request has no `id`, `status` or `createdAt` because the server owns those; accepting them from the client would be a mass-assignment vulnerability. The response has them but excludes internal audit fields.

**Q: What mapping library do we use and why?**
A: MapStruct. It generates the mapping code at compile time, so it's as fast as hand-written code and a field mismatch is a compile error. Reflection-based mappers like ModelMapper fail silently at runtime when a field is renamed.

**Q: What alternative is there for read-heavy endpoints?**
A: Projections — either an interface projection or a JPQL constructor expression returning the DTO directly. The SQL then selects only the needed columns and nothing enters the persistence context, which is noticeably faster for list screens.

**Q: Isn't this extra boilerplate?**
A: Some, yes, and MapStruct removes most of it. The trade is worth it: without DTOs every database change risks breaking clients, and we'd be one careless `return entity` away from serializing a password hash.

## 21.10 Interview Questions

**Beginner — What is a DTO?**
A plain object used to carry data between layers or across the API boundary, decoupled from the persistence model.

**Intermediate — Name three concrete problems DTOs prevent.**
Leaking sensitive entity fields, `LazyInitializationException` when serializing lazy associations, and infinite recursion from bidirectional relationships.

**Intermediate — Can a DTO be a Java `record`?**
Yes, and it's an excellent fit for **response** DTOs — immutable and concise. Jackson supports records natively. For **request** DTOs it works too, but validation annotations must go on the record components, and some frameworks still expect a no-arg constructor.

**Advanced — DTO vs Value Object vs Entity?**
A DTO is a boundary data carrier with no behaviour. A Value Object (DDD) is immutable, has no identity, is compared by value, and *can* hold domain behaviour (e.g. `Money`). An Entity has identity and lifecycle.

**Advanced — When is it acceptable to skip DTOs?**
Internal tools, throwaway prototypes, or a service with a single trusted consumer and no sensitive fields. Even then, expect the API to outlive that assumption. For anything public or long-lived, DTOs are not optional.

## 21.11 Quick Revision

1. DTO = plain data carrier at the API boundary. Not an entity.
2. Prevents: sensitive-field leaks, schema coupling, lazy exceptions, recursion, mass assignment.
3. **Separate request and response DTOs.**
4. Never accept server-controlled fields (`id`, `status`) in a create request.
5. Map in the **service layer, inside the transaction**.
6. **The entity must never leave the service layer.**
7. MapStruct = compile-time, safe. ModelMapper = runtime reflection, risky.
8. Projections give DTOs straight from the query — fewer columns, no persistence context.
9. DTOs hold validation and Jackson annotations, never business logic or JPA annotations.

## 21.12 TL Explanation (speak this)

> "DTOs are how we keep the API contract separate from the database schema. Entities never leave the service layer — the controller only sees DTOs. That protects us three ways: we can't accidentally serialize a password hash, a column rename doesn't break our clients, and we avoid `LazyInitializationException` and Jackson recursion from bidirectional relationships. We keep request and response DTOs separate so the client can't set server-owned fields like status, and we map with MapStruct inside the transaction so field mismatches are caught at compile time. For read-heavy list screens we skip entities entirely and use a projection, so the SQL selects only the columns we need."

---

[← 20. Sending Responses — ResponseEntity and Status Codes](20-sending-responses-responseentity-and-status-codes.md) | [Contents](../README.md) | [22. Validation →](22-validation.md)
