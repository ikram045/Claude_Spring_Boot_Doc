[← Back to Contents](../README.md) · Part B — Web Layer (Spring MVC / REST)

---

# 22. Validation

## 22.1 What is it?

**Simple words:**
Validation means **checking that the data the client sent is acceptable before you use it** — is the email really an email, is the amount positive, is the required field actually present.

Spring lets you declare these rules as annotations on the DTO instead of writing `if` checks everywhere.

**Technical wording:**
Spring integrates Jakarta Bean Validation (JSR-380), implemented by Hibernate Validator. Constraint annotations on DTO fields are evaluated by a `Validator` when `@Valid`/`@Validated` is present, producing `ConstraintViolation`s that Spring converts into `MethodArgumentNotValidException`.

## 22.2 Why validate at the API boundary?

- **Fail fast** — reject bad data before it reaches business logic or the database
- **Security** — the first line of defence against malformed and malicious input
- **Clear errors** — the client learns exactly which field is wrong
- **No duplication** — declared once on the DTO instead of repeated `if` blocks
- **Data integrity** — the database never receives invalid rows

## 22.3 Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```
**Important:** since Spring Boot 2.3 this starter is **no longer included** in `spring-boot-starter-web`. If your `@Valid` annotations appear to do nothing, this missing dependency is the most common cause.

## 22.4 The constraint annotations

| Annotation | Applies to | Checks |
|---|---|---|
| `@NotNull` | Any | Not null (**empty string passes!**) |
| `@NotEmpty` | String, Collection, Map, Array | Not null **and** size > 0 (`" "` passes) |
| `@NotBlank` | String | Not null, and contains non-whitespace |
| `@Size(min, max)` | String, Collection | Length/size in range |
| `@Min` / `@Max` | Numbers | Numeric bounds |
| `@Positive` / `@PositiveOrZero` | Numbers | > 0 / >= 0 |
| `@Negative` / `@NegativeOrZero` | Numbers | < 0 / <= 0 |
| `@DecimalMin` / `@DecimalMax` | BigDecimal | Bounds with precision |
| `@Digits(integer, fraction)` | Numbers | Digit counts — ideal for money |
| `@Email` | String | Email format |
| `@Pattern(regexp)` | String | Regex match |
| `@Past` / `@PastOrPresent` | Dates | In the past |
| `@Future` / `@FutureOrPresent` | Dates | In the future |
| `@AssertTrue` / `@AssertFalse` | boolean | Must be true/false |
| `@Valid` | Object / Collection | **Cascade** validation into nested objects |

### The `@NotNull` vs `@NotEmpty` vs `@NotBlank` trap

This is asked in almost every interview:

| Input | `@NotNull` | `@NotEmpty` | `@NotBlank` |
|---|---|---|---|
| `null` | FAIL | FAIL | FAIL |
| `""` | **PASS** | FAIL | FAIL |
| `"   "` | **PASS** | **PASS** | FAIL |
| `"abc"` | PASS | PASS | PASS |

**For a String field the client must actually fill in, use `@NotBlank`.** Using `@NotNull` on a String is a common bug — an empty string sails straight through into the database.

## 22.5 Example DTO with validation

```java
@Getter @Setter
@NoArgsConstructor
public class CustomerRequestDto {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between {min} and {max} characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Email format is invalid")
    private String email;

    @NotBlank(message = "Mobile number is required")
    @Pattern(regexp = "^[6-9]\\d{9}$", message = "Mobile must be a valid 10-digit number")
    private String mobile;

    @NotNull(message = "Date of birth is required")
    @Past(message = "Date of birth must be in the past")
    private LocalDate dateOfBirth;

    @NotNull(message = "Amount is required")
    @DecimalMin(value = "0.01", message = "Amount must be greater than zero")
    @Digits(integer = 10, fraction = 2, message = "Amount can have at most 2 decimal places")
    private BigDecimal creditLimit;

    @NotEmpty(message = "At least one address is required")
    @Valid                               // ← CASCADES validation into each AddressDto
    private List<AddressDto> addresses;
}
```

**Note the `{min}`/`{max}` placeholders** — they interpolate the constraint's own values, so the message stays correct if you change the bounds.

**The `@Valid` on the list is essential.** Without it, the constraints *inside* `AddressDto` are never checked. Nested validation does not happen automatically — a very common oversight.

## 22.6 Triggering validation

```java
// Request body — the most common case
@PostMapping
public ResponseEntity<CustomerDto> create(@Valid @RequestBody CustomerRequestDto request) { }
// Violation → MethodArgumentNotValidException → 400

// Query params / path variables bound to a POJO
@GetMapping
public Page<OrderDto> search(@Valid OrderSearchCriteria criteria) { }

// Individual params — needs @Validated on the CLASS
@RestController
@Validated                                          // ← REQUIRED for the line below
public class OrderController {
    @GetMapping("/{id}")
    public OrderDto get(@PathVariable @Min(1) Long id) { }
}
// Violation → ConstraintViolationException (NOT MethodArgumentNotValidException!)

// Service-layer method validation
@Service
@Validated
public class OrderService {
    public void process(@Valid OrderRequestDto dto, @NotBlank String userId) { }
}
```

## 22.7 `@Valid` vs `@Validated` — the key comparison

| Feature | `@Valid` | `@Validated` |
|---|---|---|
| Defined by | Jakarta Bean Validation (JSR-380) | **Spring** |
| Placed on | Method parameters, fields | **Classes**, methods, parameters |
| Validation groups | **No** | **Yes** |
| Nested/cascade validation | **Yes** | **No** (not on its own) |
| Method-level validation on params | No | **Yes** (via AOP proxy) |
| Exception thrown | `MethodArgumentNotValidException` | `ConstraintViolationException` |
| Typical use | `@RequestBody` DTOs, nested objects | Class-level to enable param validation; groups |

**The two must be understood together:** use `@Valid` on the request body and on nested fields; use `@Validated` on the class when you want constraints on individual parameters or need validation groups. **They throw different exceptions, so your exception handler must cover both.**

## 22.8 Validation Groups — different rules for create vs update

```java
public interface OnCreate { }
public interface OnUpdate { }

@Getter @Setter
public class CustomerDto {

    @Null(groups = OnCreate.class,    message = "ID must not be provided when creating")
    @NotNull(groups = OnUpdate.class, message = "ID is required when updating")
    private Long id;

    @NotBlank(groups = {OnCreate.class, OnUpdate.class})
    private String name;

    @NotBlank(groups = OnCreate.class)     // required on create, optional on update
    private String password;
}

@RestController
public class CustomerController {

    @PostMapping
    public CustomerDto create(@Validated(OnCreate.class) @RequestBody CustomerDto dto) { }

    @PutMapping("/{id}")
    public CustomerDto update(@Validated(OnUpdate.class) @RequestBody CustomerDto dto) { }
}
```
This lets **one DTO** serve both operations with correctly different rules — the main practical reason `@Validated` exists.

## 22.9 Custom validator (real project need)

Scenario: order dates must be valid business dates and not on a company holiday.

```java
// 1. The annotation
@Documented
@Constraint(validatedBy = ValidBusinessDateValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidBusinessDate {
    String message() default "Date must be a working day";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. The validator — note it CAN be dependency-injected
@Component
@RequiredArgsConstructor
public class ValidBusinessDateValidator
        implements ConstraintValidator<ValidBusinessDate, LocalDate> {

    private final HolidayService holidayService;      // DI works in validators

    @Override
    public boolean isValid(LocalDate value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;       // null-handling is @NotNull's job, not ours.
        }                      // Returning true here keeps constraints composable.
        boolean weekend = value.getDayOfWeek() == DayOfWeek.SATURDAY
                       || value.getDayOfWeek() == DayOfWeek.SUNDAY;
        return !weekend && !holidayService.isHoliday(value);
    }
}

// 3. Use it
public class OrderRequestDto {
    @NotNull
    @ValidBusinessDate
    private LocalDate deliveryDate;
}
```

**Two teaching points:**
1. A `ConstraintValidator` is a Spring bean, so it can inject services and hit the database.
2. **Always return `true` for `null`** and let `@NotNull` handle nullability separately. Otherwise the two constraints fight and you get confusing double errors.

### Class-level validator (cross-field validation)

When a rule spans two fields — "endDate must be after startDate" — the constraint goes on the **class**:

```java
@Constraint(validatedBy = DateRangeValidator.class)
@Target(ElementType.TYPE)                    // ← on the TYPE, not a field
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidDateRange {
    String message() default "End date must be after start date";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class DateRangeValidator implements ConstraintValidator<ValidDateRange, ReportRequestDto> {
    @Override
    public boolean isValid(ReportRequestDto dto, ConstraintValidatorContext ctx) {
        if (dto.getStartDate() == null || dto.getEndDate() == null) return true;
        boolean valid = dto.getEndDate().isAfter(dto.getStartDate());
        if (!valid) {
            ctx.disableDefaultConstraintViolation();
            ctx.buildConstraintViolationWithTemplate("End date must be after start date")
               .addPropertyNode("endDate")        // attach the error to a specific field
               .addConstraintViolation();
        }
        return valid;
    }
}

@ValidDateRange                              // applied to the class
public class ReportRequestDto {
    private LocalDate startDate;
    private LocalDate endDate;
}
```

## 22.10 Handling validation errors properly

By default Spring returns a verbose, unhelpful error blob. Convert it into a clean, field-keyed response:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Triggered by @Valid on @RequestBody
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex,
                                          HttpServletRequest request) {

        Map<String, String> fieldErrors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(error -> fieldErrors.put(error.getField(), error.getDefaultMessage()));

        // Class-level (cross-field) errors have no field name
        ex.getBindingResult().getGlobalErrors()
          .forEach(error -> fieldErrors.put(error.getObjectName(), error.getDefaultMessage()));

        return ErrorResponse.builder()
                .timestamp(Instant.now())
                .status(400)
                .error("Validation Failed")
                .message("Request contains invalid fields")
                .path(request.getRequestURI())
                .fieldErrors(fieldErrors)
                .build();
    }

    // Triggered by @Validated on params — a DIFFERENT exception. Don't forget it.
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleConstraintViolation(ConstraintViolationException ex,
                                                   HttpServletRequest request) {
        Map<String, String> errors = ex.getConstraintViolations().stream()
                .collect(Collectors.toMap(
                        v -> v.getPropertyPath().toString(),
                        ConstraintViolation::getMessage,
                        (a, b) -> a));
        return ErrorResponse.builder()
                .timestamp(Instant.now()).status(400).error("Validation Failed")
                .message("Request parameters are invalid")
                .path(request.getRequestURI()).fieldErrors(errors).build();
    }
}
```

**The response the client actually wants:**

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "status": 400,
  "error": "Validation Failed",
  "message": "Request contains invalid fields",
  "path": "/api/v1/customers",
  "fieldErrors": {
    "name": "Name is required",
    "email": "Email format is invalid",
    "creditLimit": "Amount must be greater than zero"
  }
}
```
A frontend can now highlight exactly the wrong inputs. **This is the difference between a usable API and a frustrating one** — a good point to make to a TL.

## 22.11 The layers of validation (Additional clarification)

Validation is not only at the API boundary. Know all four layers:

| Layer | Mechanism | Catches |
|---|---|---|
| **1. API / DTO** | `@Valid` + constraints | Malformed input, missing fields |
| **2. Business rules** | Explicit code in the service | "Order total exceeds credit limit", "Cannot cancel a shipped order" |
| **3. Entity constraints** | `@Column(nullable=false, length=50)` | Last-line schema integrity |
| **4. Database constraints** | `NOT NULL`, `UNIQUE`, `CHECK`, FK | Absolute guarantee, including other writers |

**Point to make to a senior:** DTO validation handles *format*; the service handles *business rules*; the database is the final authority. A unique-email check in the service is a race condition under concurrency — two simultaneous requests can both pass the check. The **database unique constraint** is what actually guarantees it, and you catch `DataIntegrityViolationException` to return a clean 409.

## 22.12 Common Mistakes

| Mistake | Consequence |
|---|---|
| Missing `spring-boot-starter-validation` | `@Valid` silently does nothing |
| `@NotNull` on a String instead of `@NotBlank` | Empty strings accepted |
| Forgetting `@Valid` on a nested object/list | Nested constraints never run |
| Forgetting `@Validated` on the class for param validation | `@Min` on `@PathVariable` ignored |
| Only handling `MethodArgumentNotValidException` | `ConstraintViolationException` returns an ugly 500 |
| `isValid` returning `false` for null | Fights with `@NotNull`; confusing duplicate messages |
| Business rules as custom constraints | Better in the service, where you have full context and transactions |
| Relying on service-level uniqueness checks | Race condition — use a DB unique constraint |
| Returning 500 for validation failures | Pollutes error dashboards with client mistakes |

## 22.13 TL Questions

**Q: How do we validate incoming requests?**
A: Constraint annotations on the request DTOs, triggered by `@Valid` on the `@RequestBody`. Failures throw `MethodArgumentNotValidException`, which our `@RestControllerAdvice` converts into a 400 with a field-to-message map.

**Q: What's the difference between `@Valid` and `@Validated`?**
A: `@Valid` is the Jakarta standard — we use it on request bodies and on nested objects to cascade validation. `@Validated` is Spring's, added at class level to enable validation of individual method parameters, and it's the only one supporting validation groups. They throw different exceptions, so we handle both.

**Q: Why `@NotBlank` instead of `@NotNull` for strings?**
A: `@NotNull` accepts an empty string and a string of spaces. `@NotBlank` requires actual content. For any field the user must fill in, `@NotBlank` is correct.

**Q: What happens internally when validation fails?**
A: The argument resolver binds the JSON to the DTO, then invokes Hibernate Validator. Violations are collected into a `BindingResult`; if it has errors, Spring throws `MethodArgumentNotValidException` **before** our controller method is called, so invalid data never reaches the service.

**Q: Where do business rules go?**
A: In the service, not in constraint annotations. Constraints are for format and structure — is this a valid email, is the amount positive. Rules like "cannot cancel a shipped order" need entity state and the transaction, so they belong in the service and throw domain exceptions.

**Q: How do we guarantee a unique email?**
A: A unique constraint on the database column. A service-level check is a race condition — two concurrent requests can both see "not taken" and both insert. We catch `DataIntegrityViolationException` and return 409 Conflict.

**Q: What if we remove `@Valid` from the controller?**
A: The constraints are never evaluated. Invalid data flows into the service and probably into the database, and we'd only find out via a database constraint error surfacing as a 500.

## 22.14 Interview Questions

**Beginner — Which dependency enables validation?**
`spring-boot-starter-validation` (Hibernate Validator). Not included in the web starter since Boot 2.3.

**Beginner — `@NotNull` vs `@NotEmpty` vs `@NotBlank`?**
`@NotNull`: not null only. `@NotEmpty`: not null and size > 0. `@NotBlank`: not null and has non-whitespace characters. For strings, use `@NotBlank`.

**Intermediate — How do you validate a nested object?**
Put `@Valid` on the nested field or collection. Cascade is not automatic.

**Intermediate — Which exception does each mechanism throw?**
`@Valid` on `@RequestBody` → `MethodArgumentNotValidException`. `@Validated` on class + constrained params → `ConstraintViolationException`. Binding failures on non-body params → `BindException` / `MethodArgumentTypeMismatchException`.

**Intermediate — How do you write a custom constraint?**
Create an annotation meta-annotated with `@Constraint(validatedBy = X.class)` declaring `message()`, `groups()` and `payload()`, then implement `ConstraintValidator<A, T>`. The validator is a Spring bean, so it can inject dependencies.

**Advanced — How does method-level validation work internally?**
`@Validated` on a class causes `MethodValidationPostProcessor` to create an AOP proxy. The interceptor calls `ExecutableValidator.validateParameters()` before the method and `validateReturnValue()` after. Because it's proxy-based, **self-invocation bypasses it** — the same limitation as `@Transactional`.

**Advanced — What are validation groups and why do they exist?**
Interfaces used as markers to activate subsets of constraints, so one DTO can enforce different rules for create versus update — for example `id` must be null on create and non-null on update. Activated with `@Validated(OnCreate.class)`.

**Advanced — Where should validation live in a layered architecture?**
Format validation at the API boundary on DTOs; business-rule validation in the service where entity state and the transaction are available; structural constraints on the entity; and absolute guarantees as database constraints. Each layer catches what the one before it cannot.

## 22.15 Quick Revision

1. Add `spring-boot-starter-validation` — not bundled with the web starter.
2. `@Valid` on `@RequestBody` triggers DTO validation.
3. `@NotBlank` for strings; `@NotNull` lets `""` through.
4. `@Valid` on nested fields/collections to cascade — not automatic.
5. `@Validated` on the class enables parameter validation and groups.
6. Two exceptions: `MethodArgumentNotValidException` and `ConstraintViolationException` — **handle both**.
7. Custom constraint = annotation + `ConstraintValidator`; validators support DI.
8. Return `true` for `null` in custom validators; let `@NotNull` own nullability.
9. Cross-field rules = class-level constraint.
10. Business rules go in the service; uniqueness is guaranteed by the **database**.
11. Always return 400 with a field-to-message map.

## 22.16 TL Explanation (speak this)

> "We validate at the API boundary with Jakarta Bean Validation annotations on the request DTOs and `@Valid` on the request body, so bad input is rejected before it ever reaches the service. Validation failures are caught in our `@RestControllerAdvice` and returned as a 400 with a field-to-message map, so the frontend can highlight exactly which inputs are wrong. Two details we're careful about: `@NotBlank` rather than `@NotNull` for strings, and handling both `MethodArgumentNotValidException` and `ConstraintViolationException`, because `@Valid` and `@Validated` throw different ones. Business rules stay in the service, and genuine uniqueness is enforced by a database constraint rather than a service check, which would be a race condition."

---

[← 21. DTO Pattern](21-dto-pattern.md) | [Contents](../README.md) | [23. Exception Handling →](23-exception-handling.md)
