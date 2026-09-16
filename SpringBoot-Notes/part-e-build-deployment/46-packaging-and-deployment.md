[← Back to Contents](../README.md) · Part E — Build, Configuration and Deployment

---

# 46. Packaging and Deployment

## 46.1 JAR vs WAR

| | Executable JAR | WAR |
|---|---|---|
| Server | **Embedded** (Tomcat inside) | External Tomcat/JBoss |
| Run with | `java -jar app.jar` | Deploy to `webapps/` |
| Server config | Properties | Server-managed |
| Docker/cloud | **Ideal** | Awkward |
| **Recommended** | **Yes — default** | Legacy/mandated environments |

```xml
<!-- WAR only if your environment requires it -->
<packaging>war</packaging>
```
```java
@SpringBootApplication
public class OrderServiceApplication extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(OrderServiceApplication.class);
    }
}
```
Also mark `spring-boot-starter-tomcat` as `provided`, or you ship two servlet containers.

### Fat JAR structure

```
app.jar
├── META-INF/MANIFEST.MF        Main-Class: org.springframework.boot.loader.JarLauncher
│                               Start-Class: com.company.OrderServiceApplication
├── org/springframework/boot/loader/    ← Boot's launcher + nested-JAR classloader
└── BOOT-INF/
    ├── classes/                ← your compiled code
    └── lib/                    ← every dependency JAR, nested
```
Standard Java **cannot** load a JAR inside a JAR, which is why Boot ships its own `LaunchedURLClassLoader`.

## 46.2 Docker

```dockerfile
# ---------- Multi-stage build ----------
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B          # cached layer — deps change rarely
COPY src ./src
RUN mvn clean package -DskipTests

# ---------- Runtime: JRE only, much smaller ----------
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Never run as root
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75.0 -XX:+UseG1GC"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**Three points worth explaining:**
1. **Multi-stage build** — the final image contains only the JRE and the JAR, not Maven and the full JDK. Image size drops from ~700MB to ~180MB.
2. **Non-root user** — a container breakout as root is far more dangerous.
3. **`-XX:MaxRAMPercentage`** — the JVM must respect the *container's* memory limit, not the host's. Without this, the JVM sizes its heap from the host's total RAM and gets OOM-killed by the container runtime. **This is a classic containerised-Java failure.**

## 46.3 Configuration in deployment

```yaml
# Kubernetes — config and secrets injected as environment variables
env:
  - name: SPRING_PROFILES_ACTIVE
    value: "prod"
  - name: SPRING_DATASOURCE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: order-service-secrets
        key: db-password
```
Relaxed binding (Section 13) maps `SPRING_DATASOURCE_PASSWORD` onto `spring.datasource.password` automatically. **One immutable image, configuration from the environment.**

## 46.4 Production checklist

| Area | Setting |
|---|---|
| Profile | `SPRING_PROFILES_ACTIVE=prod` via environment |
| Schema | `ddl-auto=validate` + Flyway/Liquibase |
| SQL logging | **Off** (`show-sql=false`) |
| `open-in-view` | **`false`** |
| Batch fetch | `default_batch_fetch_size=20` |
| Secrets | Environment / secrets manager — never in the image |
| Actuator | Allowlist only; separate port; secured |
| Shutdown | `server.shutdown=graceful` |
| Pool | Sized against DB `max_connections` ÷ instances |
| Timeouts | Connect + read timeouts on **every** HTTP client |
| Logging | JSON structured, with correlation ID |
| Health probes | Liveness and readiness configured correctly |
| Errors | No stack traces in responses |
| JVM | `MaxRAMPercentage` set for containers |

## 46.5 TL Questions

**Q: Why an executable JAR rather than a WAR?**
A: The server is embedded, so we ship one artifact that runs with `java -jar`. There's no Tomcat to install or version-match on the host, which is exactly what makes it work cleanly in a Docker image.

**Q: How does the fat JAR work?**
A: Dependencies are nested under `BOOT-INF/lib` and the manifest points at Boot's `JarLauncher`, which installs a classloader that can read JARs inside a JAR — something standard Java can't do.

**Q: Why a multi-stage Docker build?**
A: The build stage needs Maven and the full JDK; the runtime only needs a JRE and the JAR. Separating them takes the image from around seven hundred megabytes to under two hundred, which means faster pulls and a smaller attack surface.

**Q: Why set `MaxRAMPercentage`?**
A: Because the JVM sizes its heap from the machine's memory, and in a container that's the host's memory, not the container limit. Without it the JVM grows past the limit and gets OOM-killed. It's one of the most common containerised-Java failures.

**Q: How does configuration differ per environment?**
A: The image is identical everywhere. The profile and all secrets come in as environment variables from Kubernetes config maps and secrets, and Boot's relaxed binding maps them onto the properties automatically.

## 46.6 Quick Revision

1. Executable JAR with embedded server is the default.
2. Fat JAR: `BOOT-INF/classes` + `BOOT-INF/lib` + `JarLauncher`.
3. WAR only when the environment mandates it (`provided` Tomcat + `SpringBootServletInitializer`).
4. Multi-stage Docker builds; JRE base image; **non-root user**.
5. **Set `MaxRAMPercentage`** so the JVM respects the container limit.
6. One image, many environments — config from environment variables.
7. Production: `validate`, no SQL logging, `open-in-view=false`, graceful shutdown.
8. Secrets never in the image or the repo.

## 46.7 TL Explanation (speak this)

> "We ship an executable JAR with the server embedded, so deployment is just `java -jar` inside a container — no Tomcat to install or version-match. The Dockerfile is multi-stage, so the final image carries only a JRE and our JAR rather than Maven and the full JDK, and it runs as a non-root user. One setting that matters more than people expect is `MaxRAMPercentage`, because otherwise the JVM sizes its heap from the host's memory rather than the container limit and gets OOM-killed. The image itself is identical across environments — the profile and all secrets arrive as environment variables from Kubernetes."

---
---

# PART F — MASTER FLOWS AND FINAL REVISION

> This part consolidates everything above. If you only revise one section before
> a TL discussion or interview, revise this one.

---

[← 45. Spring Boot Actuator and Production Readiness](45-spring-boot-actuator-and-production-readiness.md) | [Contents](../README.md) | [47. Complete Spring Boot Architecture Flow →](../part-f-master-flows/47-complete-spring-boot-architecture-flow.md)
