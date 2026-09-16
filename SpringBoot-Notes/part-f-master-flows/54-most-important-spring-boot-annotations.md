[← Back to Contents](../README.md) · Part F — Master Flows and Final Revision

---

# 54. Most Important Spring Boot Annotations

## Core / Configuration

| Annotation | What it does |
|---|---|
| `@SpringBootApplication` | `@SpringBootConfiguration` + `@EnableAutoConfiguration` + `@ComponentScan` |
| `@Configuration` | Class declares `@Bean` methods; CGLIB-proxied by default |
| `@Bean` | Method's return value becomes a managed bean |
| `@ComponentScan` | Which packages to scan |
| `@EnableAutoConfiguration` | Turn on conditional auto-configuration |
| `@Import` | Pull in another configuration class |
| `@Profile` | Register a bean only for given profiles |
| `@Conditional...` | `OnClass`, `OnMissingBean`, `OnProperty` — conditional registration |

## Stereotypes and DI

| Annotation | What it does |
|---|---|
| `@Component` | Generic managed bean |
| `@Service` | Business logic layer (semantic; = `@Component`) |
| `@Repository` | Data layer **+ exception translation** |
| `@Controller` | Web layer returning view names |
| `@RestController` | `@Controller` + `@ResponseBody` — REST APIs |
| `@Autowired` | Inject a dependency (optional on a single constructor) |
| `@Qualifier` | Choose among multiple candidate beans |
| `@Primary` | Default candidate when several match |
| `@Value` | Inject a single property or SpEL value |
| `@ConfigurationProperties` | **Type-safe, validated, relaxed-binding group of properties** |
| `@Scope` | singleton / prototype / request / session |
| `@Lazy` | Defer creation until first use |
| `@PostConstruct` / `@PreDestroy` | Lifecycle callbacks |

## Web / REST

| Annotation | What it does |
|---|---|
| `@RequestMapping` | Map requests (path, method, headers, consumes, produces) |
| `@GetMapping` … `@DeleteMapping` | Method-specific shortcuts |
| `@PathVariable` | Bind a URL path segment |
| `@RequestParam` | Bind a query parameter or form field |
| `@RequestBody` | Deserialize the body (one per method) |
| `@RequestHeader` / `@CookieValue` | Bind a header / cookie |
| `@ResponseBody` | Serialize the return value |
| `@ResponseStatus` | Fixed status code |
| `@RestControllerAdvice` | **Global exception handling** |
| `@ExceptionHandler` | Handle a specific exception type |
| `@CrossOrigin` | Per-controller CORS |

## Validation

| Annotation | What it does |
|---|---|
| `@Valid` | Trigger validation; **cascades** into nested objects |
| `@Validated` | Spring's variant: class-level param validation + **groups** |
| `@NotNull` / `@NotEmpty` / `@NotBlank` | Null / empty / blank checks |
| `@Size`, `@Min`, `@Max`, `@DecimalMin`, `@Digits` | Bounds |
| `@Email`, `@Pattern` | Format |
| `@Past`, `@Future` | Temporal |

## JPA / Persistence

| Annotation | What it does |
|---|---|
| `@Entity` / `@Table` | Map a class to a table |
| `@Id` / `@GeneratedValue` | Primary key and generation strategy |
| `@Column` | Column mapping and constraints |
| `@Enumerated(EnumType.STRING)` | **Always STRING, never ORDINAL** |
| `@Transient` | Exclude from persistence |
| `@Version` | **Optimistic locking** |
| `@OneToMany` / `@ManyToOne` / `@OneToOne` / `@ManyToMany` | Relationships |
| `@JoinColumn` / `@JoinTable` / `mappedBy` | FK column / join table / inverse side |
| `@MappedSuperclass` | Share columns without a table |
| `@Embeddable` / `@Embedded` | Value object in the same table |
| `@Query` | JPQL or native query |
| `@Modifying` | Required for UPDATE/DELETE queries |
| `@EntityGraph` | **Declarative fetch plan — the N+1 fix** |
| `@Transactional` | **Declarative transaction boundary** |
| `@Lock` | Pessimistic locking |
| `@CreationTimestamp` / `@CreatedDate` | Auditing timestamps |

## AOP / Security / Misc

| Annotation | What it does |
|---|---|
| `@Aspect` | Declares an aspect class |
| `@Before` / `@After` / `@AfterReturning` / `@AfterThrowing` / `@Around` | Advice types |
| `@Pointcut` | Named, reusable pointcut expression |
| `@EnableWebSecurity` | Enable Spring Security web configuration |
| `@EnableMethodSecurity` | Enable `@PreAuthorize` / `@PostAuthorize` |
| `@PreAuthorize` / `@PostAuthorize` | Method-level authorization (SpEL) |
| `@Async` / `@EnableAsync` | Run on a separate thread |
| `@Scheduled` / `@EnableScheduling` | Scheduled execution |
| `@Cacheable` / `@CacheEvict` | Caching |
| `@EventListener` / `@TransactionalEventListener` | Application events |
| `@SpringBootTest` / `@DataJpaTest` / `@WebMvcTest` | Test slices |

## The five annotations that are all AOP proxies

**`@Transactional`, `@Async`, `@Cacheable`, `@PreAuthorize`, `@Validated`** — all proxy-based, therefore **all** broken by self-invocation, non-public methods, and `new`-ed objects. Understanding that one mechanism explains five separate "why isn't this working?" problems at once.

---

[← 53. Filter vs Interceptor vs AOP — Complete Comparison](53-filter-vs-interceptor-vs-aop-complete-comparison.md) | [Contents](../README.md) | [55. Most Important TL / Senior-Level Questions →](55-most-important-tl-senior-level-questions.md)
