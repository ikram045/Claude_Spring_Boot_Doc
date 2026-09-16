[← Back to Contents](../README.md) · Part D — Security

---

# 42. Spring Security

## 42.1 What is it?

**Simple words:**
Spring Security is the framework that answers two questions for every request:

1. **Authentication** — *"Who are you?"* (login, token validation)
2. **Authorization** — *"Are you allowed to do this?"* (roles, permissions)

It plugs into the filter chain, so it runs **before** your controller ever sees the request.

**Technical wording:**
Spring Security is a framework providing authentication, authorization and protection against common attacks for Spring applications. It is implemented as a chain of servlet filters registered through a `DelegatingFilterProxy`, backed by a `SecurityContext` bound to the current thread.

## 42.2 Authentication vs Authorization — get this exact

| | Authentication | Authorization |
|---|---|---|
| Question | **Who are you?** | **What can you do?** |
| Happens | First | After authentication |
| Failure status | **401 Unauthorized** | **403 Forbidden** |
| Mechanism | Credentials, JWT, OAuth | Roles, authorities, ACLs |
| Spring artifact | `AuthenticationManager` | `AccessDecisionManager` / `AuthorizationManager` |

**The one-line distinction for interviews:** *"Authentication proves identity; authorization grants permission. A valid token that lacks the required role gets 403, not 401."*

## 42.3 Core architecture

```
   Request
      │
      ▼
 ┌──────────────────────────────────────────────────────────┐
 │  DelegatingFilterProxy  ("springSecurityFilterChain")     │
 │      delegates to ▼                                       │
 │  FilterChainProxy                                         │
 │      selects the matching ▼                               │
 │  SecurityFilterChain — an ordered list of filters:        │
 │                                                            │
 │   1. SecurityContextPersistenceFilter                     │
 │        loads the SecurityContext for this request         │
 │   2. CorsFilter / CsrfFilter                              │
 │   3. UsernamePasswordAuthenticationFilter  (form login)   │
 │      ── or our custom JwtAuthenticationFilter ──          │
 │   4. BearerTokenAuthenticationFilter (OAuth2 resource)    │
 │   5. ExceptionTranslationFilter                           │
 │        catches AuthenticationException  → 401 entry point │
 │        catches AccessDeniedException    → 403 handler     │
 │   6. AuthorizationFilter (was FilterSecurityInterceptor)  │
 │        final check: is this principal allowed here?       │
 └──────────────────────────────────────────────────────────┘
      │ authenticated and authorized
      ▼
 DispatcherServlet → Controller
```

### Key classes

| Class/Interface | Responsibility |
|---|---|
| `SecurityContextHolder` | Holds the `SecurityContext` in a **`ThreadLocal`** |
| `SecurityContext` | Holds the current `Authentication` |
| `Authentication` | Principal + credentials + authorities + authenticated flag |
| `UserDetails` | The user model Spring Security understands |
| `UserDetailsService` | Loads a `UserDetails` by username — **you implement this** |
| `AuthenticationManager` | Entry point for authenticating a request |
| `AuthenticationProvider` | Performs actual authentication (e.g. `DaoAuthenticationProvider`) |
| `PasswordEncoder` | Hashes and verifies passwords |
| `GrantedAuthority` | A single permission/role |

**`SecurityContextHolder` is a `ThreadLocal`.** That's how any service, several layers down, can call `SecurityContextHolder.getContext().getAuthentication()` without it being passed as a parameter. It's also why the security context is **not** available in an `@Async` method unless you propagate it (`DelegatingSecurityContextExecutor` or `MODE_INHERITABLETHREADLOCAL`).

## 42.4 The authentication flow

```
 1. Client POSTs credentials  { "username": "raj", "password": "secret" }
              │
              ▼
 2. AuthenticationFilter builds an unauthenticated
    UsernamePasswordAuthenticationToken
              │
              ▼
 3. AuthenticationManager (ProviderManager) delegates to providers
              │
              ▼
 4. DaoAuthenticationProvider:
       a. userDetailsService.loadUserByUsername("raj")
             → fetch the user + hashed password + roles from the DB
       b. passwordEncoder.matches(rawPassword, storedHash)
             → BCrypt compares; the stored hash is NEVER decrypted
              │
        ┌─────┴─────┐
     match        no match
        │             │
        ▼             ▼
 5. Return an     BadCredentialsException
    AUTHENTICATED       │
    Authentication      ▼
    with authorities   401 via AuthenticationEntryPoint
              │
              ▼
 6. SecurityContextHolder.getContext().setAuthentication(auth)
              │
              ▼
 7. Request proceeds; authorization checks use those authorities
```

## 42.5 Configuration (Spring Security 6 / Boot 3 style)

```java
package com.company.orderservice.config;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity           // enables @PreAuthorize / @PostAuthorize
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;
    private final UserDetailsService userDetailsService;
    private final CustomAuthenticationEntryPoint authenticationEntryPoint;
    private final CustomAccessDeniedHandler accessDeniedHandler;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            // CSRF is not needed for a stateless token-based API (explained below)
            .csrf(AbstractHttpConfigurer::disable)

            .cors(cors -> cors.configurationSource(corsConfigurationSource()))

            // No HTTP session — every request must carry its own token
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))

            .authorizeHttpRequests(auth -> auth
                // Public endpoints
                .requestMatchers("/api/v1/auth/**", "/api/v1/public/**").permitAll()
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/v3/api-docs/**", "/swagger-ui/**").permitAll()

                // Role-based rules — evaluated TOP TO BOTTOM, first match wins
                .requestMatchers(HttpMethod.GET, "/api/v1/orders/**")
                    .hasAnyRole("USER", "ADMIN")
                .requestMatchers(HttpMethod.POST, "/api/v1/orders/**")
                    .hasRole("USER")
                .requestMatchers(HttpMethod.DELETE, "/api/v1/orders/**")
                    .hasRole("ADMIN")
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")

                // ★ Everything else requires authentication — deny by default
                .anyRequest().authenticated()
            )

            // Custom 401 / 403 responses instead of the default HTML error page
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(authenticationEntryPoint)   // 401
                .accessDeniedHandler(accessDeniedHandler)             // 403
            )

            // Our JWT filter runs BEFORE the username/password filter
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)

            .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);      // strength 12
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        // NEVER use "*" in production with credentials
        config.setAllowedOrigins(List.of("https://app.company.com"));
        config.setAllowedMethods(List.of("GET","POST","PUT","PATCH","DELETE","OPTIONS"));
        config.setAllowedHeaders(List.of("Authorization","Content-Type","X-Correlation-Id"));
        config.setExposedHeaders(List.of("X-Correlation-Id"));
        config.setAllowCredentials(true);
        config.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", config);
        return source;
    }
}
```

**Rule ordering matters enormously.** `authorizeHttpRequests` evaluates **top to bottom, first match wins**. Putting `.anyRequest().authenticated()` first would make every rule after it unreachable. Always go from **most specific to least specific**, ending with `anyRequest()`.

## 42.6 Method-level security

```java
@Service
public class OrderService {

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteOrder(Long id) { }

    @PreAuthorize("hasAnyRole('USER','ADMIN')")
    public OrderDto getOrder(Long id) { }

    // SpEL can reference method arguments
    @PreAuthorize("#customerId == authentication.name or hasRole('ADMIN')")
    public List<OrderDto> getCustomerOrders(String customerId) { }

    // Checks the RETURN value after execution
    @PostAuthorize("returnObject.customerId == authentication.name")
    public OrderDto findById(Long id) { }

    @PreAuthorize("hasAuthority('ORDER_DELETE')")   // authority, not role
    public void archive(Long id) { }
}
```

**`hasRole` vs `hasAuthority` — a constant source of confusion:**
`hasRole('ADMIN')` automatically prepends `ROLE_`, so it checks for the authority `ROLE_ADMIN`. `hasAuthority('ADMIN')` checks for exactly `ADMIN`. **If your database stores authorities without the `ROLE_` prefix, `hasRole` silently fails.** Pick one convention and document it.

> **`@PostAuthorize` caution:** the method has already executed when the check runs. If it modified data, that change already happened — the exception only prevents the *return*. Never use `@PostAuthorize` on a method with side effects.

**Note:** `@PreAuthorize` is AOP-based, so **self-invocation bypasses it** — the same limitation as `@Transactional`.

## 42.7 UserDetailsService and password storage

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    @Transactional(readOnly = true)
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        User user = userRepository.findByUsernameWithRoles(username)   // fetch roles eagerly
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
        // NOTE: deliberately a generic message — never reveal whether the
        // username exists (that's a user-enumeration vulnerability).

        return org.springframework.security.core.userdetails.User.builder()
                .username(user.getUsername())
                .password(user.getPassword())          // the BCrypt HASH
                .authorities(user.getRoles().stream()
                        .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
                        .toList())
                .accountLocked(user.isLocked())
                .disabled(!user.isEnabled())
                .build();
    }
}
```

### Why BCrypt

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12);
}
```

| Property | Why it matters |
|---|---|
| **One-way** | Hashes cannot be reversed — a database leak doesn't expose passwords |
| **Salted automatically** | Each hash embeds a random salt, so identical passwords produce different hashes → rainbow tables useless |
| **Deliberately slow** | The work factor makes brute force expensive; increase it as hardware improves |
| **Self-describing** | The hash stores its own algorithm, cost and salt, so verification needs no extra columns |

**Never use MD5 or SHA-256 for passwords** — they're designed to be *fast*, which is exactly wrong for password hashing. Modern GPUs compute billions of SHA-256 hashes per second. BCrypt (or Argon2/scrypt) is the correct family.

## 42.8 CSRF — why we disable it for token APIs

**What CSRF is:** a malicious site causes the victim's browser to send an authenticated request to your site. It works because the browser **automatically attaches cookies** to any request to that domain.

**Why it doesn't apply to a stateless JWT API:** the token lives in the `Authorization` header, and browsers do **not** attach custom headers automatically. The attacker's page cannot read or set that header cross-origin. No automatic credential attachment means no CSRF vector.

**When you MUST keep CSRF enabled:**
- Session-cookie authentication
- **JWT stored in a cookie** — cookies are auto-attached, so the CSRF risk returns

| Token storage | XSS risk | CSRF risk | Notes |
|---|---|---|---|
| `localStorage` | **High** — any injected JS can read it | None | Common, but XSS steals the token |
| `sessionStorage` | High | None | Same, cleared on tab close |
| **`httpOnly` cookie** | **Low** — JS cannot read it | **Yes** — needs CSRF protection | Most secure **if** CSRF protection and `SameSite` are configured |

**The honest answer to give a TL:** *"There's no storage option with zero risk. An `httpOnly`, `Secure`, `SameSite=Strict` cookie plus CSRF tokens is the most defensible choice; `localStorage` is simpler but one XSS away from full account takeover."*

## 42.9 Custom 401 and 403 handlers

Filter-thrown exceptions **cannot** be caught by `@RestControllerAdvice` (Section 23), so security needs its own handlers:

```java
@Component
@RequiredArgsConstructor
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    private final ObjectMapper objectMapper;

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException {
        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        objectMapper.writeValue(response.getOutputStream(), ErrorResponse.builder()
                .timestamp(Instant.now()).status(401).error("Unauthorized")
                .errorCode("AUTH_REQUIRED")
                .message("Authentication required. Provide a valid token.")
                .path(request.getRequestURI())
                .correlationId(MDC.get("correlationId"))
                .build());
    }
}

@Component
@RequiredArgsConstructor
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    private final ObjectMapper objectMapper;

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
                       AccessDeniedException ex) throws IOException {
        response.setStatus(HttpStatus.FORBIDDEN.value());
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        objectMapper.writeValue(response.getOutputStream(), ErrorResponse.builder()
                .timestamp(Instant.now()).status(403).error("Forbidden")
                .errorCode("ACCESS_DENIED")
                .message("You do not have permission to perform this action")
                .path(request.getRequestURI()).build());
    }
}
```

## 42.10 Common Mistakes

| Mistake | Consequence |
|---|---|
| **Storing plain-text passwords** | Catastrophic on any database leak |
| MD5/SHA for passwords | Trivially brute-forced on modern GPUs |
| Rule order wrong in `authorizeHttpRequests` | Broad rules shadow specific ones |
| Forgetting `.anyRequest().authenticated()` | Unlisted endpoints are **unprotected** |
| `hasRole('ROLE_ADMIN')` | Double prefix → checks `ROLE_ROLE_ADMIN`, always fails |
| CORS `allowedOrigins("*")` with credentials | Rejected by browsers; also unsafe |
| Disabling CSRF while using cookie auth | Reintroduces the CSRF vulnerability |
| Revealing "user not found" vs "wrong password" | User enumeration |
| Secrets in `application.yml` | Credentials in version control |
| Expecting `@RestControllerAdvice` to catch security exceptions | Default HTML error page returned |
| Relying on `@PreAuthorize` across self-invocation | Silently skipped |

## 42.11 TL Questions

**Q: How does Spring Security work at a high level?**
A: It's a chain of servlet filters registered ahead of the DispatcherServlet. Each request passes through filters that authenticate it, populate a `SecurityContext` held in a `ThreadLocal`, and then check authorization before the controller runs.

**Q: What's the difference between authentication and authorization in our responses?**
A: Authentication failure is 401 — we don't know who you are. Authorization failure is 403 — we know you but your role doesn't permit this. We have separate handlers for each so clients get a clean JSON body rather than the default HTML page.

**Q: Why do we disable CSRF?**
A: Because our API is stateless and the token travels in the `Authorization` header. CSRF exploits the browser automatically attaching cookies; custom headers aren't attached automatically, so there's no vector. If we ever moved the token into a cookie we'd have to re-enable CSRF.

**Q: How are passwords stored?**
A: BCrypt hashes with a work factor of 12. It's one-way, automatically salted so identical passwords hash differently, and deliberately slow to make brute force expensive. We never store or log the raw password.

**Q: What happens if we forget `anyRequest().authenticated()`?**
A: Any endpoint not explicitly matched by a rule is left unprotected. That line is our deny-by-default backstop, so a newly added controller is secured automatically rather than accidentally public.

**Q: Where do we enforce authorization — the URL rules or the methods?**
A: Both. URL rules give broad coverage at the filter level, and `@PreAuthorize` on service methods handles finer rules like "a user may only read their own orders". Method security also protects the logic if it's ever called from a scheduled job or message listener rather than HTTP.

**Q: Why is the security context unavailable in `@Async` methods?**
A: It's stored in a `ThreadLocal`, and an async method runs on a different pool thread. We either pass the needed values explicitly or configure a `DelegatingSecurityContextExecutor` to propagate it.

## 42.12 Interview Questions

**Beginner — What is `SecurityContextHolder`?**
A holder for the current `SecurityContext`, stored in a `ThreadLocal`, from which any code on that thread can obtain the current `Authentication`.

**Beginner — What does `UserDetailsService` do?**
Loads user data — username, password hash and authorities — by username, for the authentication provider to verify.

**Intermediate — `hasRole` vs `hasAuthority`?**
`hasRole('ADMIN')` implicitly prefixes `ROLE_`, so it matches the authority `ROLE_ADMIN`. `hasAuthority('ADMIN')` matches exactly `ADMIN`. Mismatched conventions cause silent authorization failures.

**Intermediate — Why is BCrypt preferred over SHA-256?**
SHA-256 is fast by design, so an attacker can compute billions of guesses per second. BCrypt has a tunable work factor making each hash deliberately slow, and it salts automatically, defeating rainbow tables.

**Advanced — What is `DelegatingFilterProxy` and why is it needed?**
A container-managed filter that delegates to a Spring bean by name. The servlet container instantiates filters itself, outside the Spring context, so the proxy bridges the two — letting the real filter be a fully injected Spring bean.

**Advanced — `@PreAuthorize` vs `@PostAuthorize`?**
`@PreAuthorize` evaluates before invocation and can prevent execution. `@PostAuthorize` evaluates afterwards against `returnObject`, so it can filter based on the result — but the method has already run, so it must never be used where the method has side effects.

**Advanced — How does the `ExceptionTranslationFilter` work?**
It wraps the rest of the chain and catches two exception types: `AuthenticationException` triggers the `AuthenticationEntryPoint` (401), and `AccessDeniedException` triggers the `AccessDeniedHandler` (403) — or, for an anonymous user, the entry point instead, prompting authentication.

## 42.13 Quick Revision

1. Spring Security is a **filter chain** running before DispatcherServlet.
2. **Authentication = who (401); Authorization = what (403).**
3. `SecurityContextHolder` is a **`ThreadLocal`** — not available in `@Async`.
4. `UserDetailsService` loads the user; `PasswordEncoder` verifies.
5. **BCrypt** — one-way, auto-salted, deliberately slow.
6. Rules are evaluated **top to bottom**; always end with `anyRequest().authenticated()`.
7. `hasRole` prefixes `ROLE_`; `hasAuthority` does not.
8. CSRF disabled only because we're **stateless with header tokens**.
9. Security exceptions need `AuthenticationEntryPoint` / `AccessDeniedHandler`.
10. `@PreAuthorize` is AOP — self-invocation bypasses it.
11. Never reveal whether a username exists.

## 42.14 TL Explanation (speak this)

> "Spring Security runs as a filter chain before the DispatcherServlet. Each request is authenticated, the resulting `Authentication` is stored in a `ThreadLocal` security context, and then authorization rules are checked before our controller is reached. We configure URL rules from most specific to least, ending with `anyRequest().authenticated()` so anything we forget to list is protected by default, and we add `@PreAuthorize` on services for finer rules like a user only reading their own orders. Passwords are BCrypt with a work factor of twelve — one-way and automatically salted. CSRF is off only because we're stateless with the token in the `Authorization` header; if we ever moved it into a cookie we'd have to turn CSRF back on."

---

[← 41. Transactions and @Transactional](../part-c-data-layer/41-transactions-and-transactional.md) | [Contents](../README.md) | [43. JWT (JSON Web Token) Authentication →](43-jwt-json-web-token-authentication.md)
