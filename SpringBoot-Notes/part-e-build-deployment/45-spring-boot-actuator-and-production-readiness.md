[← Back to Contents](../README.md) · Part E — Build, Configuration and Deployment

---

# 45. Spring Boot Actuator and Production Readiness

## 45.1 What is it?

**Simple words:**
Actuator adds **ready-made endpoints that tell you how your application is doing** — is it healthy, how much memory is it using, what configuration is active, how many requests has it served.

**Technical wording:**
Spring Boot Actuator provides production-ready features including health indicators, metrics collection, environment introspection and operational endpoints, exposed over HTTP or JMX.

## 45.2 Setup and key endpoints

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus     # ONLY what we need
      base-path: /actuator
  endpoint:
    health:
      show-details: when-authorized                 # not "always" in production
      probes:
        enabled: true                               # /health/liveness, /health/readiness
  metrics:
    tags:
      application: ${spring.application.name}
```

| Endpoint | Shows |
|---|---|
| `/actuator/health` | UP/DOWN, plus per-component status |
| `/actuator/health/liveness` | Is the app alive? (Kubernetes) |
| `/actuator/health/readiness` | Is it ready for traffic? (Kubernetes) |
| `/actuator/info` | Build and git information |
| `/actuator/metrics` | JVM, HTTP, datasource metrics |
| `/actuator/prometheus` | Metrics in Prometheus format |
| `/actuator/env` | **All properties — sensitive** |
| `/actuator/beans` | Every bean in the context |
| `/actuator/conditions` | **Auto-configuration report** (Section 10) |
| `/actuator/loggers` | View **and change** log levels at runtime |
| `/actuator/threaddump` | Thread dump — sensitive |
| `/actuator/heapdump` | **Full heap dump — highly sensitive** |

### ★ Security warning ★

**Only `/health` and `/info` are exposed by default over HTTP** — but many teams set `include: *` for convenience and forget.

`/actuator/env` exposes **every property including database passwords**. `/actuator/heapdump` downloads the entire heap, which contains **tokens, passwords and customer data in memory**. An exposed actuator is a complete system compromise.

**Always:**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus    # explicit allowlist, never "*"
  server:
    port: 9090                             # separate port, not exposed publicly
```
```java
// And secure them
.requestMatchers("/actuator/health", "/actuator/info").permitAll()
.requestMatchers("/actuator/**").hasRole("ADMIN")
```

## 45.3 Liveness vs Readiness (essential for Kubernetes)

| Probe | Question | On failure |
|---|---|---|
| **Liveness** | Is the process alive and not deadlocked? | **Kubernetes RESTARTS the pod** |
| **Readiness** | Can it serve traffic right now? | **Removed from the load balancer** — not restarted |

**Why the distinction matters:** if the database is down, the app is *alive* but *not ready*. Reporting that as a **liveness** failure makes Kubernetes restart every pod in a loop, which does nothing to fix the database and turns a degraded service into a total outage. Database checks belong in **readiness**.

**This is a genuinely valuable point in any deployment discussion.**

## 45.4 Custom health indicator

```java
@Component
@RequiredArgsConstructor
public class PaymentGatewayHealthIndicator implements HealthIndicator {

    private final PaymentGatewayClient client;

    @Override
    public Health health() {
        try {
            if (client.ping()) {
                return Health.up().withDetail("gateway", "reachable").build();
            }
            return Health.down().withDetail("gateway", "unreachable").build();
        } catch (Exception e) {
            return Health.down(e).withDetail("gateway", "error").build();
        }
    }
}
```
**Caution:** any `HealthIndicator` contributes to the overall status, so a non-critical dependency reporting DOWN can mark your whole service DOWN and pull it out of the load balancer. Put non-critical checks in a separate group, or return `Health.up()` with a degraded detail.

## 45.5 Graceful shutdown

```yaml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```
On SIGTERM, Tomcat stops accepting new connections but **finishes in-flight requests** before closing. Without it, a deployment drops every request currently being processed — visible to users as random errors during every release.

## 45.6 TL Questions

**Q: What does Actuator give us?**
A: Production endpoints — health checks for the load balancer and Kubernetes, metrics for Prometheus, and operational tools like runtime log-level changes and the auto-configuration report.

**Q: How do we secure it?**
A: An explicit allowlist rather than `*`, health and info public, everything else behind an admin role, and ideally on a separate management port not exposed publicly. `/env` leaks passwords and `/heapdump` dumps everything in memory, so exposing actuator openly is a full compromise.

**Q: What's the difference between liveness and readiness?**
A: Liveness asks whether the process is alive — failing it restarts the pod. Readiness asks whether it can serve traffic — failing it just removes the pod from the load balancer. Database checks go in readiness, because if the database is down, restarting every pod makes things worse rather than better.

**Q: Why enable graceful shutdown?**
A: So a deployment finishes in-flight requests instead of dropping them. Without it, every release produces a burst of user-visible errors.

## 45.7 Quick Revision

1. Actuator = health, metrics, info, operational endpoints.
2. Only `health` and `info` are exposed by default — **never use `include: *`**.
3. `/env` leaks secrets; `/heapdump` leaks everything in memory.
4. **Liveness failure = restart; readiness failure = remove from LB.**
5. Database checks belong in **readiness**, not liveness.
6. `/actuator/conditions` explains auto-configuration decisions.
7. `/actuator/loggers` changes log levels without a restart.
8. Enable `server.shutdown=graceful`.
9. `/actuator/prometheus` for metrics scraping.

## 45.8 TL Explanation (speak this)

> "Actuator gives us the production endpoints — health for Kubernetes probes and the load balancer, and Prometheus metrics. We expose an explicit allowlist rather than everything, because `/env` returns all our properties including database passwords and `/heapdump` returns the entire heap, so an open actuator is a full compromise. The distinction we're careful about is liveness versus readiness: a database outage should fail readiness so pods leave the load balancer, not liveness, which would restart every pod in a loop and make a degraded service a total outage. We also enable graceful shutdown so deployments don't drop in-flight requests."

---

[← 44. Maven](44-maven.md) | [Contents](../README.md) | [46. Packaging and Deployment →](46-packaging-and-deployment.md)
