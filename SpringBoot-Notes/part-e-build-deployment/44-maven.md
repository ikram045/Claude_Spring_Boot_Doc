[← Back to Contents](../README.md) · Part E — Build, Configuration and Deployment

---

# 44. Maven

## 44.1 What is it?

**Simple words:**
Maven is a **build tool and dependency manager**. You list the libraries you need in one file, and Maven downloads them, compiles your code, runs the tests and packages everything into a JAR.

Before Maven, you downloaded JAR files manually and put them in a `lib` folder — and every library's own dependencies too.

**Technical wording:**
Maven is a build automation and project management tool based on the Project Object Model (POM). It provides declarative dependency management with transitive resolution, a standard build lifecycle, and a plugin-based execution model.

## 44.2 The POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>

    <!-- The Boot parent: manages versions for ~200 libraries -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>

    <!-- COORDINATES — uniquely identify this artifact -->
    <groupId>com.company</groupId>          <!-- organisation -->
    <artifactId>order-service</artifactId>  <!-- project name -->
    <version>1.0.0-SNAPSHOT</version>       <!-- version -->
    <packaging>jar</packaging>

    <properties>
        <java.version>17</java.version>
        <mapstruct.version>1.5.5.Final</mapstruct.version>
    </properties>

    <dependencies>
        <!-- No <version> needed — the parent manages it -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>          <!-- needed at runtime, not compile -->
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>       <!-- not transitive to consumers -->
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>             <!-- test compile + run only -->
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <!-- Repackages the JAR into an EXECUTABLE fat JAR -->
            </plugin>
        </plugins>
    </build>
</project>
```

### GAV coordinates
`groupId:artifactId:version` uniquely identifies any artifact — `com.company:order-service:1.0.0`.

**`SNAPSHOT` versions:** `1.0.0-SNAPSHOT` means "in development". Maven re-checks the remote repository for updates, so builds are **not reproducible**. A release version like `1.0.0` is immutable and cached forever. **Never deploy a SNAPSHOT to production** — you cannot guarantee which build is actually running.

## 44.3 Dependency scopes

| Scope | Compile | Test | Runtime | Packaged | Example |
|---|---|---|---|---|---|
| **`compile`** (default) | Yes | Yes | Yes | Yes | spring-boot-starter-web |
| **`provided`** | Yes | Yes | No | **No** | servlet-api, Tomcat for WAR |
| **`runtime`** | No | Yes | Yes | Yes | **JDBC drivers** |
| **`test`** | No | Yes | No | No | JUnit, Mockito |
| `system` | Yes | Yes | No | No | Deprecated — avoid |
| `import` | — | — | — | — | BOM import in `dependencyManagement` |

**Why a JDBC driver is `runtime`:** your code compiles against the JDBC *interfaces* in `java.sql`, never against MySQL classes. The driver is only needed when the application runs. Marking it `runtime` prevents anyone accidentally importing a vendor class and creating hidden coupling.

## 44.4 Transitive dependencies and conflicts

Maven pulls dependencies of dependencies automatically:
```
your project
 └── spring-boot-starter-web
      ├── spring-web
      ├── spring-webmvc
      ├── spring-boot-starter-tomcat → tomcat-embed-core
      └── spring-boot-starter-json   → jackson-databind → jackson-core
```

**Conflict resolution — the "nearest wins" rule:**
```
A → B → C → jackson 2.13      (depth 3)
A → D → jackson 2.15          (depth 2)   ← WINS: nearer to the root
```
Maven uses the **shortest path**, not the highest version. If two are at equal depth, the **first declared** wins.

**Diagnostic commands:**
```bash
mvn dependency:tree                             # full tree
mvn dependency:tree -Dincludes=com.fasterxml.jackson   # trace one library
mvn dependency:analyze                          # unused / undeclared dependencies
```

**Excluding a transitive dependency:**
```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>some-library</artifactId>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
**Typical real cause:** two libraries pull incompatible versions of the same dependency and you get `NoSuchMethodError` or `ClassNotFoundException` at runtime. `mvn dependency:tree` is the first command to run — it shows exactly which version won and why.

## 44.5 The build lifecycle

```
validate → compile → test → package → verify → install → deploy
```
Running a phase runs **all preceding phases**.

| Phase | Does |
|---|---|
| `validate` | Check the project is correct |
| `compile` | Compile sources to `target/classes` |
| `test` | Run unit tests (Surefire) |
| `package` | Build the JAR/WAR into `target/` |
| `verify` | Run integration tests (Failsafe) and checks |
| `install` | Copy the artifact to the **local** repository (`~/.m2`) |
| `deploy` | Upload to the **remote** repository (Nexus/Artifactory) |

```bash
mvn clean install                 # most common local command
mvn clean package -DskipTests     # skip tests (CI packaging step)
mvn test                          # run tests only
mvn spring-boot:run               # run the app without packaging
mvn versions:display-dependency-updates   # check for newer versions
```

> **`-DskipTests` vs `-Dmaven.test.skip=true`:** the first compiles tests but doesn't run them; the second skips compiling them too. Prefer `-DskipTests` so test code still has to compile — otherwise broken test code goes unnoticed.

## 44.6 Maven vs Gradle

| | Maven | Gradle |
|---|---|---|
| Config | XML (`pom.xml`) | Groovy/Kotlin DSL |
| Style | Declarative, convention-driven | Programmatic, flexible |
| Build speed | Slower | **Faster** (incremental + build cache) |
| Learning curve | **Lower** | Higher |
| Predictability | **High** — hard to do anything unusual | Lower — build logic can be arbitrary code |
| Enterprise adoption | **Very high** | Growing, dominant in Android |

**Balanced view:** Gradle is faster and more flexible; Maven is more predictable because the XML can't contain arbitrary logic. For corporate services where consistency across teams matters more than build speed, Maven remains the common choice.

## 44.7 Common Mistakes

| Mistake | Consequence |
|---|---|
| Specifying versions for Boot starters | Breaks the parent's curated version set |
| JDBC driver at `compile` scope | Encourages coupling to vendor classes |
| **Deploying a SNAPSHOT** | Non-reproducible build; unknown code in production |
| Ignoring `dependency:tree` on a `NoSuchMethodError` | Hours lost to a version conflict |
| `-Dmaven.test.skip=true` in CI | Test code rot goes undetected |
| Secrets in the POM | Committed to version control |
| Not pinning plugin versions | Builds change behaviour without a code change |

## 44.8 TL Questions

**Q: What does Maven do for us?**
A: Dependency management with transitive resolution, and a standard build lifecycle — compile, test, package. We declare what we need and it fetches the libraries and their dependencies.

**Q: Why don't we specify versions for Spring starters?**
A: The `spring-boot-starter-parent` manages them. It declares a tested, mutually compatible set of around two hundred library versions, so we avoid the version conflicts that used to dominate Java projects.

**Q: How does Maven resolve a version conflict?**
A: Nearest-wins — the shortest path from the root, not the highest version. When two are equally deep, the first declared wins. When we hit a `NoSuchMethodError` the first thing we run is `mvn dependency:tree` to see which version was actually selected.

**Q: Why is the JDBC driver `runtime` scope?**
A: Our code compiles only against the `java.sql` interfaces, never MySQL classes. Restricting it to runtime prevents anyone accidentally importing a vendor class and coupling us to that database.

**Q: Why shouldn't we deploy SNAPSHOT versions?**
A: Because a SNAPSHOT is mutable — Maven re-resolves it, so the same version string can mean different code at different times. We can't reproduce a production build or be certain what's running. Releases are immutable.

## 44.9 Interview Questions

**Beginner — What is a POM?**
The Project Object Model — the XML file describing coordinates, dependencies, properties, plugins and build configuration.

**Beginner — What is the local repository?**
`~/.m2/repository`, where Maven caches downloaded artifacts and installs locally built ones.

**Intermediate — Difference between `install` and `deploy`?**
`install` copies the artifact into the local `~/.m2` repository for use by other local projects. `deploy` uploads it to a shared remote repository such as Nexus or Artifactory.

**Intermediate — What does `spring-boot-maven-plugin` do?**
It repackages the plain JAR into an executable fat JAR — nesting dependencies under `BOOT-INF/lib`, adding a `JarLauncher` main class and a nested-JAR-aware classloader, so `java -jar` works.

**Advanced — What is a BOM?**
A Bill of Materials — a POM containing only `dependencyManagement` entries, imported with `<scope>import</scope>`. It centralises version decisions without forcing the dependencies themselves. `spring-boot-dependencies` is a BOM, used when you can't inherit from the Boot parent.

**Advanced — `<optional>true</optional>`?**
Marks a dependency as non-transitive — consumers of your artifact don't inherit it. Lombok uses this because it's compile-time only and consumers don't need it at runtime.

## 44.10 Quick Revision

1. Maven = dependency management + standard build lifecycle.
2. GAV: `groupId:artifactId:version`.
3. `spring-boot-starter-parent` manages ~200 library versions — don't override.
4. Scopes: `compile`, `provided`, `runtime`, `test`.
5. **JDBC drivers = `runtime`.**
6. Conflicts resolved by **nearest wins**, not highest version.
7. `mvn dependency:tree` is the first diagnostic for version problems.
8. Lifecycle: validate → compile → test → package → verify → install → deploy.
9. **Never deploy SNAPSHOT to production.**
10. `spring-boot-maven-plugin` produces the executable fat JAR.

## 44.11 TL Explanation (speak this)

> "Maven handles our dependencies and the build lifecycle. We inherit from `spring-boot-starter-parent`, which pins around two hundred library versions that are tested together — that's why we never specify versions for the starters and why we don't hit the version conflicts Java projects used to suffer from. When we do get a `NoSuchMethodError` at runtime, it's almost always a transitive conflict and `mvn dependency:tree` shows which version won, since Maven picks the nearest in the tree rather than the highest. We also keep releases immutable — no SNAPSHOT ever goes to production, because the same version string could mean different code."

---

[← 43. JWT (JSON Web Token) Authentication](../part-d-security/43-jwt-json-web-token-authentication.md) | [Contents](../README.md) | [45. Spring Boot Actuator and Production Readiness →](45-spring-boot-actuator-and-production-readiness.md)
