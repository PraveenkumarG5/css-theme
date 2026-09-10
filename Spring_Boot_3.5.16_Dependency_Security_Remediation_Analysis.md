# Dependency Security Remediation Analysis

## Sonatype IQ Policy Violations – Spring Boot 3.5.16

**Constraint:** Spring Boot must remain at **3.5.16** and must not be upgraded as part of this remediation.  
**Analysis date:** 9 September 2026

---

## 1. Executive Summary

The provided Sonatype IQ Application Composition Report shows **109 policy violations affecting 53 components**, including Threat 10 (Security-Critical) and Threat 9 (Security-High) findings. Several entries are duplicate occurrences of the same dependency family.

Because the application must remain on **Spring Boot 3.5.16**, recommendations should prioritize dependency versions from the same compatible generation. The earlier recommendation to move to Spring Framework 7.x, Spring Batch 6.x, Spring Integration 7.x, or Reactor 3.8.x should **not** be used as the default remediation.

### Important limitation

For **Spring Framework, Spring Batch, Spring Integration, and Reactor 3.7**, the security-fixed versions on the same compatible generation may require **Spring Enterprise support**. Without that support, Sonatype IQ may continue to report those findings and a formal waiver/risk acceptance may be necessary until the application can move to a newer supported Spring Boot generation.

---

## 2. Recommended Target Versions

| Threat | Dependency | Current | Recommended | Boot 3.5.16 Compatible? | Notes |
|---|---|---:|---:|---|---|
| **10** | `io.netty:netty-handler` | 4.1.135.Final | **4.1.137.Final** | Yes | Align entire Netty family |
| **10** | `org.apache.logging.log4j:log4j-api` | 2.24.3 | **2.26.1** | Yes | Align Log4j family/BOM |
| **10** | `org.apache.tomcat.embed:tomcat-embed-core` | 10.1.57 | **10.1.59** | Yes | Security/patch update |
| **10** | `org.springframework.batch:spring-batch-core` | 5.2.6 | **5.2.7*** | Yes | Same Batch generation; Enterprise support |
| **9** | `ch.qos.logback:logback-core` | 1.5.34 | **1.5.38** | Yes | Stay on Logback 1.5 generation |
| **9** | `com.fasterxml.jackson.core:jackson-databind` | 2.21.4 | **2.21.6** | Yes | Prefer Jackson BOM |
| **9** | `io.netty:netty-codec` | 4.1.135.Final | **4.1.137.Final** | Yes | Align Netty family |
| **9** | `io.netty:netty-codec-http` | 4.1.135.Final | **4.1.137.Final** | Yes | Align Netty family |
| **9** | `io.netty:netty-codec-http2` | 4.1.135.Final | **4.1.137.Final** | Yes | Align Netty family |
| **9** | `io.netty:netty-codec-socks` | 4.1.135.Final | **4.1.137.Final** | Yes | Align Netty family |
| **9** | `io.netty:netty-handler` | 4.1.135.Final | **4.1.137.Final** | Yes | Align Netty family |
| **9** | `io.projectreactor:reactor-core` | 3.7.19 | **3.7.20*** | Yes | Same Reactor generation; Enterprise support |
| **9** | `org.apache.sshd:sshd-core` | 2.15.0 | **2.19.0** | Yes | Stable 2.x target |
| **9** | `org.apache.tomcat.embed:tomcat-embed-core` | 10.1.57 | **10.1.59** | Yes | Align Tomcat family |
| **9** | `org.springframework:spring-expression` | 6.2.19 | **6.2.20*** | Yes | Align entire Spring Framework family |
| **9** | `org.springframework:spring-webmvc` | 6.2.19 | **6.2.20*** | Yes | Align entire Spring Framework family |
| **9** | `org.springframework.batch:spring-batch-infrastructure` | 5.2.6 | **5.2.7*** | Yes | Align Batch modules |
| **9** | `org.springframework.integration:spring-integration-core` | 6.5.10 | **6.5.11*** | Yes | Align Integration modules |

> **\*** Same-generation security-fixed target may require Spring Enterprise support.

---

## 3. Detailed Remediation Analysis

### 3.1 Netty – 4.1.137.Final

The report contains multiple Netty artifacts at `4.1.135.Final`. Treat them as one family and upgrade them together.

**Recommended target: `4.1.137.Final`**

Recommended alignment:

- `netty-handler` = `4.1.137.Final`
- `netty-codec` = `4.1.137.Final`
- `netty-codec-http` = `4.1.137.Final`
- `netty-codec-http2` = `4.1.137.Final`
- `netty-codec-socks` = `4.1.137.Final`
- Other resolved Netty modules should also be checked and aligned.

**Do not mix `4.1.135.Final` and `4.1.137.Final` across the dependency tree.**

---

### 3.2 Apache Log4j – 2.26.1

The report shows:

```text
org.apache.logging.log4j:log4j-api:2.24.3
```

**Recommended target: `2.26.1`**

Keep the Log4j family aligned, preferably through the Log4j BOM.

Recommended where applicable:

- `log4j-api` = `2.26.1`
- `log4j-core` = `2.26.1`
- `log4j-slf4j2-impl` = `2.26.1`
- Other Log4j modules = `2.26.1`

---

### 3.3 Apache Tomcat – 10.1.59

The report repeatedly shows:

```text
org.apache.tomcat.embed:tomcat-embed-core:10.1.57
```

**Recommended target: `10.1.59`**

Keep embedded Tomcat modules aligned:

- `tomcat-embed-core` = `10.1.59`
- `tomcat-embed-el` = `10.1.59`, if used
- `tomcat-embed-websocket` = `10.1.59`, if used
- `tomcat-annotations-api` = `10.1.59`, if used

This stays within Tomcat 10.1 and is preferable to a major-generation change.

---

### 3.4 Spring Batch – 5.2.7

The report shows:

```text
spring-batch-core:5.2.6
spring-batch-infrastructure:5.2.6
```

**Recommended target: `5.2.7`**

Keep the complete Spring Batch family aligned:

- `spring-batch-core` = `5.2.7`
- `spring-batch-infrastructure` = `5.2.7`
- `spring-batch-integration` = `5.2.7`, if used
- `spring-batch-test` = `5.2.7`, if used

This same-generation security-fixed version may require Spring Enterprise support.

**Do not move to Batch 6.0.x merely to remove the IQ finding when Boot must remain at 3.5.16.**

---

### 3.5 Logback – 1.5.38

The report shows:

```text
ch.qos.logback:logback-core:1.5.34
```

For Boot 3.5.16 compatibility, use **Logback 1.5.38** rather than moving to 1.6.x.

Recommended:

- `logback-core` = `1.5.38`
- `logback-classic` = `1.5.38`, if used

Verify SLF4J compatibility after the change.

---

### 3.6 Jackson – 2.21.6

The report shows:

```text
com.fasterxml.jackson.core:jackson-databind:2.21.4
```

For a Boot 3.5.16 application, use **Jackson 2.21.6** to stay in the same Jackson generation.

Prefer the Jackson BOM and align:

- `jackson-core` = `2.21.6`
- `jackson-databind` = `2.21.6`
- `jackson-annotations` = `2.21.6`
- Applicable `jackson-datatype-*`
- Applicable `jackson-module-*`
- Applicable `jackson-dataformat-*`

---

### 3.7 Reactor Core – 3.7.20

The report shows:

```text
io.projectreactor:reactor-core:3.7.19
```

The compatible same-generation security-fixed target is **3.7.20**, which may require Spring Enterprise support.

**Do not blindly move to Reactor 3.8.x** solely to remove the IQ finding because that changes the Reactor generation used by Boot 3.5.16.

Verify the Reactor BOM and all resolved Reactor modules after the change.

---

### 3.8 Apache MINA SSHD – 2.19.0

The report shows:

```text
org.apache.sshd:sshd-core:2.15.0
```

**Recommended target: `2.19.0`**

Prefer the stable 2.x release rather than a milestone/pre-release 3.x version.

Also check and align other SSHD modules such as `sshd-common` if present.

---

### 3.9 Spring Framework – 6.2.20

The report shows:

```text
spring-expression:6.2.19
spring-webmvc:6.2.19
```

Because Boot must remain at **3.5.16**, the compatible target is **Spring Framework 6.2.20**, not Spring Framework 7.x.

The entire Spring Framework family should be aligned, including where applicable:

- `spring-core`
- `spring-beans`
- `spring-context`
- `spring-expression`
- `spring-jcl`
- `spring-aop`
- `spring-tx`
- `spring-jdbc`
- `spring-web`
- `spring-webmvc`
- `spring-webflux`
- `spring-websocket`

The same-generation security-fixed version may require Spring Enterprise support.

If Enterprise support is unavailable, Sonatype IQ may continue to report Spring Framework findings while the application remains on Boot 3.5.16.

---

### 3.10 Spring Integration – 6.5.11

The report shows:

```text
org.springframework.integration:spring-integration-core:6.5.10
```

**Recommended target: `6.5.11`**

Keep all Spring Integration modules aligned at `6.5.11` where used.

This same-generation security-fixed version may require Spring Enterprise support.

**Do not jump to Integration 7.x solely to eliminate the IQ finding** without assessing the broader Spring/Boot compatibility impact.

---

## 4. Priority Classification

### Lower-risk / comparatively straightforward

1. Netty `4.1.135.Final` → **`4.1.137.Final`**
2. Tomcat `10.1.57` → **`10.1.59`**
3. Log4j `2.24.3` → **`2.26.1`**
4. Jackson `2.21.4` → **`2.21.6`**
5. Apache MINA SSHD `2.15.0` → **`2.19.0`**

### Medium-risk / compatibility testing

1. Logback `1.5.34` → **`1.5.38`**
2. Reactor Core `3.7.19` → **`3.7.20`**

### High-impact framework/security updates

1. Spring Framework `6.2.19` → **`6.2.20`**
2. Spring Batch `5.2.6` → **`5.2.7`**
3. Spring Integration `6.5.10` → **`6.5.11`**

---

## 5. Maven / Dependency Management Strategy

Use BOMs and dependency management rather than overriding isolated transitive artifacts.

Recommended:

- **Netty BOM** – align all Netty modules
- **Jackson BOM** – align all Jackson modules
- **Log4j BOM** – align all Log4j modules
- **Reactor BOM** – align Reactor modules
- **Spring dependency management** – align all Spring Framework modules

### Maven

```bash
mvn dependency:tree
```

### Gradle

```bash
./gradlew dependencies
```

After upgrading, inspect the complete resolved dependency tree and confirm that old vulnerable versions are not being pulled transitively.

---

## 6. Verification Checklist

Before closing the Sonatype IQ defect:

- [ ] Confirm every direct dependency has the intended target version.
- [ ] Run the complete resolved dependency tree.
- [ ] Identify and eliminate old transitive versions.
- [ ] Confirm no old Netty `4.1.135.Final` artifacts remain.
- [ ] Confirm no old Tomcat `10.1.57` artifacts remain.
- [ ] Confirm no old Log4j `2.24.3` artifacts remain.
- [ ] Confirm no old Logback `1.5.34` artifacts remain.
- [ ] Confirm no old Jackson `2.21.4` artifacts remain.
- [ ] Confirm no old SSHD `2.15.0` artifacts remain.
- [ ] Confirm Spring Framework modules are aligned rather than overriding only `spring-expression`/`spring-webmvc`.
- [ ] Confirm Spring Batch modules are aligned rather than overriding only `spring-batch-core`.
- [ ] Confirm Spring Integration modules are aligned rather than overriding only `spring-integration-core`.
- [ ] Run unit tests.
- [ ] Run integration tests.
- [ ] Run application regression testing.
- [ ] Run a fresh Sonatype IQ scan.
- [ ] Acceptance criterion: **no unwaived Threat 10 / Threat 9 findings attributable to these dependencies.**

---

## 7. Final Recommended Baseline

| Component | Target |
|---|---:|
| **Spring Boot** | **3.5.16** |
| Netty | **4.1.137.Final** |
| Tomcat | **10.1.59** |
| Log4j | **2.26.1** |
| Logback | **1.5.38** |
| Jackson | **2.21.6** |
| Reactor Core | **3.7.20*** |
| Spring Framework | **6.2.20*** |
| Spring Batch | **5.2.7*** |
| Spring Integration | **6.5.11*** |
| Apache MINA SSHD | **2.19.0** |

> **\*** Same-generation security-fixed target may require Spring Enterprise support.

---

## 8. Key Conclusion

For a Spring Boot 3.5.16 application, the preferred security-remediation strategy is to stay within the compatible dependency generations wherever possible.

### Recommended baseline

```text
Spring Boot                 3.5.16

Netty                       4.1.137.Final
Tomcat                      10.1.59
Log4j                       2.26.1
Logback                     1.5.38
Jackson                     2.21.6
Reactor Core                3.7.20
Spring Framework            6.2.20
Spring Batch                5.2.7
Spring Integration          6.5.11
Apache MINA SSHD            2.19.0
```

Do **not** use Spring Framework 7.x, Spring Batch 6.x, Spring Integration 7.x, or Reactor 3.8.x solely to eliminate Sonatype findings while Boot is constrained to 3.5.16. Those changes cross dependency generations and can introduce avoidable application compatibility risk.

---

## 9. Important Caveat

A dependency version having no known vulnerability at the time of analysis is **not a guarantee against future CVE disclosure**.

The definitive closure check is a **fresh Sonatype IQ scan against the exact dependency tree produced by the build**.

Also review the IQ report's **Waived** column separately. A waived finding should not be confused with a resolved finding.

The analysis is based on the provided Sonatype IQ screenshots and vendor/security information reviewed for the stated analysis date.
