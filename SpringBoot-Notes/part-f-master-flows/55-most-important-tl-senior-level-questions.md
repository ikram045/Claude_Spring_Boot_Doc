[← Back to Contents](../README.md) · Part F — Master Flows and Final Revision

---

# 55. Most Important TL / Senior-Level Questions

These are the questions that distinguish someone who has *used* Spring Boot from someone who *understands* it. Each answer is written to be spoken aloud.

**1. Walk me through what happens when a request hits our API.**
> Tomcat takes a worker thread and the filter chain runs — correlation ID, then JWT authentication which populates the security context. The DispatcherServlet asks HandlerMapping which controller method matches, a HandlerAdapter invokes it, and argument resolvers with Jackson turn the JSON body into our DTO, with validation running right after binding. The controller calls the service, whose `@Transactional` proxy opens a transaction, and the repository goes through Hibernate to the database. We map the entity to a DTO inside the transaction, commit, and the DTO is serialized back to JSON. Interceptors handle timing and the filters clear the MDC.

**2. Why doesn't `@Transactional` work sometimes?**
> Four reasons, all from it being proxy-based. Self-invocation — an internal call goes through `this` and skips the proxy. A non-public method, which CGLIB can't proxy. A checked exception, because Spring only rolls back on unchecked ones by default. And catching an exception without rethrowing, which lets the method return normally so the transaction commits. The same limitations apply to `@Cacheable`, `@Async` and `@PreAuthorize`, because they're all proxies too.

**3. What is the N+1 problem and how do we handle it?**
> One query loads N parents and then each parent's lazy association fires its own query. The reason it's dangerous is that it looks fine with ten rows in development and becomes a timeout with ten thousand in production. We fix it per-query with `@EntityGraph` or a join fetch, use DTO projections for read-only list screens, and set `default_batch_fetch_size` globally as a safety net. What we don't do is switch to EAGER — that makes every query pay the cost permanently.

**4. Why do we use DTOs instead of returning entities?**
> Security, because entities contain fields like password hashes that Jackson would happily serialize. Decoupling, so a column rename isn't a breaking API change. And stability — serializing a lazy association after the transaction closes throws `LazyInitializationException`, and bidirectional relationships recurse infinitely in Jackson. We map inside the service, inside the transaction, so the entity never leaves that layer.

**5. Explain auto-configuration.**
> `@EnableAutoConfiguration` imports a selector that reads the `AutoConfiguration.imports` files from every jar, giving around a hundred and fifty candidate configuration classes. Each is evaluated against its `@Conditional` annotations — is this class on the classpath, does this property exist, does this bean already exist. The key one is `@ConditionalOnMissingBean`: the moment we define our own bean of that type, the auto-configuration backs off. That's what makes Boot opinionated but not restrictive.

**6. How do we secure the API?**
> Spring Security's filter chain runs before the DispatcherServlet. A JWT filter verifies the signature and expiry and populates the security context, then URL rules check authorization, ending with `anyRequest().authenticated()` so anything unlisted is protected by default. We add `@PreAuthorize` on services for finer rules. Passwords are BCrypt. Security exceptions are handled by an `AuthenticationEntryPoint` for 401 and an `AccessDeniedHandler` for 403, because `@RestControllerAdvice` can't catch filter exceptions.

**7. What's the biggest performance risk in our data layer?**
> Two things. N+1 queries from lazy loading in a loop, and long transactions. Transactions hold a pooled database connection for their whole duration, so if we make an external HTTP call inside one and that service is slow, every in-flight request holds a connection until the pool drains and we stop responding — brought down by someone else's outage. So we keep transactions short and free of network I/O.

**8. How would you debug a production issue in this service?**
> Start from the correlation ID. Every request gets one in a filter, it goes into the MDC so every log line carries it, and it's returned in the response and error bodies. With that one ID we can trace the request across every service. Beyond that, Actuator metrics — especially the Hikari pending-connections gauge, which is the earliest sign of pool pressure — and the conditions report if something isn't configured as expected.

**9. What would you change about our current setup?**
> Three things I'd check first. Whether `spring.jpa.open-in-view` is disabled — it's on by default and hides N+1 queries while holding connections through serialization. Whether `default_batch_fetch_size` is set, which is a one-line global improvement. And whether Actuator is exposing more than health and info, because `/env` returns our passwords and `/heapdump` returns everything in memory.

**10. How do you decide between a filter, an interceptor and AOP?**
> Filter if it needs the raw request or must cover everything including unmapped URLs — authentication, CORS, correlation IDs. Interceptor if the logic depends on which controller method is running and is HTTP-specific. AOP if it applies to service or repository methods, or must also work outside HTTP, like in a scheduled job or a Kafka listener.

**11. Singleton beans and thread safety — what's the risk?**
> Every controller, service and repository is a singleton shared across all request threads. So any mutable instance field is shared concurrent state, and under load one request's data leaks into another's. We keep them stateless with all per-request data in method parameters. Where we genuinely need per-request state we use a request-scoped bean, which Spring injects as a proxy that resolves per thread.

**12. Why do our business exceptions extend `RuntimeException`?**
> Two reasons. Checked exceptions force `throws` declarations through every layer. More importantly, Spring rolls back automatically on unchecked exceptions but commits on checked ones — so a checked business exception would let a half-completed transaction commit and leave inconsistent data with no error anywhere.

**13. What's the trade-off with JWT?**
> We get statelessness, so any instance can validate any request and we scale horizontally without sticky sessions. What we give up is revocation — a token stays valid until it expires and we can't invalidate it. That's why access tokens live fifteen minutes and we pair them with a database-backed refresh token that can be revoked on logout. And the payload is only encoded, not encrypted, so nothing sensitive goes in the claims.

**14. How do we handle schema changes?**
> Flyway migrations with `ddl-auto=validate`. Hibernate verifies the schema matches our entities and refuses to start if it doesn't, so a missed migration fails the deployment rather than surfacing as a runtime error. We never use `ddl-auto=update` in production — it never drops columns, gives no migration history and no rollback path.

**15. How do you size the connection pool?**
> Small and deliberate, roughly twice the database server's core count, which lands near Hikari's default of ten. Bigger pools usually reduce throughput because the database context-switches and contends more. The number people forget is that total connections equal pool size times instance count — ten pods at twenty each already exceeds MySQL's default limit of a hundred and fifty-one.

---

[← 54. Most Important Spring Boot Annotations](54-most-important-spring-boot-annotations.md) | [Contents](../README.md) | [56. One-Page Spring Boot Quick Revision Sheet →](56-one-page-spring-boot-quick-revision-sheet.md)
