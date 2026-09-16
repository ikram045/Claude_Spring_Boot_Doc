[← Back to Contents](../README.md) · Part A — Foundation (Core Spring)

---

# 11. Component Scanning

## 11.1 What is it?

**Simple words:**
Component scanning is Spring **walking through your packages, looking at every class, and asking "does this have `@Component`, `@Service`, `@Repository`, `@Controller` or `@Configuration` on it?"** Every class that does becomes a bean.

**Technical wording:**
Component scanning is the process by which the container discovers candidate components by scanning the classpath under specified base packages, reading class metadata (via ASM, without loading the class), and registering matching classes as bean definitions.

## 11.2 How it works internally

```
1. @ComponentScan declares base packages
        (default = the package of the annotated class)
                    │
                    ▼
2. ClassPathBeanDefinitionScanner walks those packages
                    │
                    ▼
3. For each .class file, read annotation metadata with ASM
   (bytecode is inspected — the class is NOT loaded yet; this keeps startup fast)
                    │
                    ▼
4. Apply include filters (default: any @Component meta-annotation)
   then apply exclude filters
                    │
                    ▼
5. Matching classes → ScannedGenericBeanDefinition registered
                    │
                    ▼
6. Bean name generated: class simple name, first letter lowercased
```

## 11.3 Controlling the scan

```java
// Default — package of the annotated class and below
@SpringBootApplication
public class App { }

// Explicit packages
@ComponentScan(basePackages = {"com.company.orderservice", "com.company.shared"})

// Type-safe: uses the package of the given class (survives refactoring/renaming)
@ComponentScan(basePackageClasses = {OrderService.class, SharedMarker.class})

// Exclude by annotation
@ComponentScan(excludeFilters =
    @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Deprecated.class))

// Exclude by regex
@ComponentScan(excludeFilters =
    @ComponentScan.Filter(type = FilterType.REGEX, pattern = "com\\.company\\.legacy\\..*"))
```

**Best practice:** prefer `basePackageClasses` over `basePackages` — a `String` package name silently breaks when someone renames a package; a class reference is checked by the compiler.

## 11.4 Common Mistakes

| Mistake | Consequence |
|---|---|
| Bean class outside the scanned tree | `NoSuchBeanDefinitionException` |
| Adding `@ComponentScan` with `basePackages` on the main class | **Replaces** the default scan — your own package is no longer scanned unless you list it |
| Scanning a very broad package (e.g. `com`) | Slow startup; may pick up third-party components unintentionally |
| Expecting interfaces or abstract classes to be scanned | They are not instantiable, so they are not registered (Spring Data repository interfaces are a special case — registered by a different mechanism) |

## 11.5 TL Questions

**Q: What is component scanning?**
A: Spring scanning the configured base packages for classes annotated with stereotype annotations and registering each one as a bean definition.

**Q: What packages does it scan by default?**
A: The package of the `@SpringBootApplication` class and all its sub-packages. That is the reason the main class must live in the root package.

**Q: What happens internally — does it load every class?**
A: No. It reads bytecode metadata with ASM, so it only checks annotations without initialising classes. That keeps startup fast even in large codebases.

**Q: What happens if a class has no stereotype annotation?**
A: It is ignored entirely, so injecting it fails at startup with `NoSuchBeanDefinitionException`. If we can't annotate it — a third-party class — we register it with an `@Bean` method instead.

**Q: How do we include beans from a shared library module?**
A: Either add its package via `scanBasePackages`, or better, have the library ship its own auto-configuration so consuming services get the beans automatically without knowing its package structure.

## 11.6 Interview Questions

**Beginner — Which annotations are picked up by scanning?**
`@Component` and everything meta-annotated with it: `@Service`, `@Repository`, `@Controller`, `@RestController`, `@Configuration`.

**Intermediate — `basePackages` vs `basePackageClasses`?**
`basePackages` takes `String` names — not refactor-safe. `basePackageClasses` takes class references and uses their packages — compile-time checked and refactor-safe. Prefer the latter.

**Advanced — Does scanning hurt startup time in large applications?**
It can. Mitigations: narrow the base packages, use `spring-context-indexer` (which generates a compile-time `META-INF/spring.components` index so the scanner reads a file instead of walking the classpath), or use explicit `@Bean` configuration in extreme cases.

## 11.7 Quick Revision

1. Scanning finds stereotype-annotated classes and registers them as beans.
2. Default base package = package of the `@SpringBootApplication` class, downward only.
3. Metadata is read via ASM — no class loading, fast.
4. Bean name = class simple name, lowercase first letter.
5. `basePackageClasses` is safer than `basePackages`.
6. Declaring `@ComponentScan` explicitly **replaces** the default behaviour.
7. Third-party classes cannot be scanned — use `@Bean`.

## 11.8 TL Explanation (speak this)

> "Component scanning is how Spring finds our beans — it walks the package tree under the main class and registers every class carrying `@Service`, `@Repository`, `@Controller` or `@Component`. It reads annotations straight from bytecode rather than loading classes, so it's fast. The practical consequences are that the main class has to sit in the root package, and that anything outside that tree — a shared library, say — either needs `scanBasePackages` or should ship its own auto-configuration."

---

[← 10. Auto-Configuration](10-auto-configuration.md) | [Contents](../README.md) | [12. Java Configuration — @Configuration and @Bean →](12-java-configuration-configuration-and-bean.md)
