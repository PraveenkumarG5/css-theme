# Spring Boot 3.5.16 – Latest Sonatype IQ Remediation Analysis

**Application:** `bce-modernization`  
**Scan date:** 11 September 2026  
**Constraint:** Spring Boot must remain at **3.5.16**

## Executive summary

The latest Sonatype IQ scan shows **73 violations affecting 55
components**: - **5 Critical** - **65 High** - **3 Medium** - **0
Legacy**

This is an improvement from the previous **109 violations**, but the
remaining findings include security, component-identification, and
license-policy issues.

## Recommended secure baseline

| Component          | Current scan         | Recommended       | Action                  |
|--------------------|----------------------|-------------------|-------------------------|
| Spring Boot        | 3.5.16               | **3.5.16**        | Keep                    |
| Netty              | 4.1.137.Final        | **4.1.138.Final** | **Upgrade immediately** |
| Tomcat             | Boot-managed / older | **10.1.59**       | Upgrade/override        |
| Log4j API          | 2.26.1               | **2.26.1**        | Investigate IQ finding  |
| Logback            | 1.5.38               | **1.6.3**         | Recommended upgrade     |
| Jackson            | 2.21.4               | **2.22.2**        | Recommended upgrade     |
| Micrometer         | 1.15.12              | **1.15.13**\*     | Upgrade                 |
| Reactor Core       | 3.7.20               | **3.7.20**\*      | Keep                    |
| Reactor Netty      | 1.2.19               | **1.2.19**\*      | Keep                    |
| Spring Framework   | 6.2.20               | **6.2.20**\*      | Keep                    |
| Spring Batch       | 5.2.7                | **5.2.7**\*       | Keep                    |
| Spring Integration | 6.5.11               | **6.5.11**\*      | Keep                    |
| Spring Data JPA    | 3.5.13               | **3.5.14**\*      | Upgrade                 |
| Flyway             | 11.7.2               | **11.20.3**       | Recommended upgrade     |
| H2                 | 2.3.232              | **2.5.250**       | Upgrade                 |
| PostgreSQL JDBC    | 42.7.11              | **42.7.13**       | Upgrade                 |
| Apache MINA SSHD   | 2.15.0 / older       | **2.19.0**        | Upgrade                 |

`*` Same-generation Enterprise-supported maintenance release may be
required.

## 1. Threat 10 – Security Critical

### Log4j API 2.26.1

The scan reports `org.apache.logging.log4j:log4j-api:2.26.1` as Threat
10 / Security-Critical.

Do **not** blindly change this dependency. Current Apache security
information and public vulnerability databases indicate that **2.26.1 is
a security-fixed/current version** for the recent Log4j issue affecting
earlier releases.

The exact Sonatype IQ **CVE ID, policy ID, affected range, and fixed
version** are required before changing it. If IQ is using stale
intelligence or an organization-specific policy, the correct remediation
may be a policy/intelligence refresh or approved exception rather than a
dependency downgrade.

## 2. Threat 9 – Security High

### Netty 4.1.137.Final

The latest scan contains multiple Netty modules at `4.1.137.Final`,
including HTTP, HTTP/2, and handler components.

**Required target: `4.1.138.Final`.**

Netty 4.1.138.Final was released on 9 September 2026 with security
fixes. CVE-2026-89044 affects Netty 4.1.133.Final through 4.1.137.Final.

Align the complete Netty family rather than upgrading one artifact:

``` xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-bom</artifactId>
            <version>4.1.138.Final</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Verify with:

``` bash
mvn dependency:tree -Dincludes=io.netty
```

There should be no `4.1.137.Final` artifact left in the tree.

### Micrometer 1.15.12

Upgrade `io.micrometer:micrometer-core` from **1.15.12 to 1.15.13**.
August 2026 security advisories affect 1.15.0–1.15.12; 1.15.13 is the
same-generation fixed release.

Because this maintenance line may require Spring Enterprise support,
confirm entitlement before implementation.

## 3. Security-fixed dependencies that should remain

### Reactor Core

`reactor-core:3.7.20` is already the same-generation fixed version.
**Keep 3.7.20**.

### Reactor Netty

`reactor-netty-core:1.2.19` and `reactor-netty-http:1.2.19` are already
the same-generation fixed versions. **Keep 1.2.19**.

### Spring Framework

The scan contains Spring Framework 6.2.20 modules including
`spring-core`, `spring-beans`, `spring-context`, `spring-expression`,
`spring-web`, and `spring-webmvc`.

**Keep 6.2.20.** It is the appropriate fixed same-generation version
under the Spring Boot 3.5.16 constraint.

### Spring Batch

`spring-batch-core:5.2.7` and `spring-batch-infrastructure:5.2.7` are
already on the appropriate fixed same-generation release.

**Keep 5.2.7.**

### Spring Integration

`spring-integration-core`, `spring-integration-file`, and
`spring-integration-sftp` are at **6.5.11**.

**Keep 6.5.11.**

These Spring/Reactor findings may appear as Threat 7 /
Component-Unknown. That classification does not by itself mean the
versions are vulnerable.

## 4. Spring Data JPA

Current: `3.5.13`

Recommended: **3.5.14**

Spring Data JPA 3.5.13 is affected by CVE-2026-47834; 3.5.14 is the
same-generation fixed release. Confirm Enterprise support availability.

Do not jump to Spring Data 4.x simply to remove this finding while Boot
remains fixed at 3.5.16.

## 5. Tomcat

Recommended target: **10.1.59**.

Tomcat 10.1.59 contains security fixes for earlier 10.1.x versions.
Verify the actual runtime dependency with:

``` bash
mvn dependency:tree -Dincludes=org.apache.tomcat
```

## 6. Logback

Current: `1.5.38`

Recommended: **1.6.3**

Current public vulnerability information reports no direct known
vulnerabilities for 1.6.3. Because the IQ report still flags Logback,
upgrading to 1.6.3 is recommended, followed by regression testing of
`logback-spring.xml`, custom appenders, structured logging, and any
conditional/Janino configuration.

## 7. Jackson

Current Boot-managed baseline: `2.21.4`

Recommended: **2.22.2**

Use the Jackson BOM if the application can be regression-tested against
this minor generation:

``` xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson</groupId>
            <artifactId>jackson-bom</artifactId>
            <version>2.22.2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Pay particular attention to REST serialization, date/time handling,
custom modules, polymorphic types, and message serialization.

## 8. H2

Current: `2.3.232`

Recommended: **2.5.250**

H2 2.5.250 is the current non-vulnerable baseline. Run the complete test
suite after the upgrade, particularly if H2 is used for integration
tests.

## 9. PostgreSQL JDBC

Current: `42.7.11`

Recommended: **42.7.13**

42.7.12 addressed a security issue affecting older versions including
42.7.11, and 42.7.13 is the current non-vulnerable baseline.

## 10. Flyway

Current: `11.7.2`

Recommended: **11.20.3**

11.20.3 is the latest 11.x release and is preferable to a
major-generation change while Boot remains fixed at 3.5.16.

After upgrading, inspect Flyway’s transitive dependencies, especially
Jackson and Log4j:

``` bash
mvn dependency:tree -Dincludes=org.flywaydb
mvn dependency:tree -Dincludes=com.fasterxml.jackson
mvn dependency:tree -Dincludes=org.apache.logging.log4j
```

If an unwanted transitive version is introduced, align it with the
application’s dependency management after compatibility testing.

## 11. Apache MINA SSHD

Recommended: **2.19.0**.

Verify:

``` bash
mvn dependency:tree -Dincludes=org.apache.sshd
```

## 12. Threat 7 – Component Unknown

The report identifies several artifacts as Threat 7 / Component-Unknown,
including:

- `bce-modernization-0.0.7-SNAPSHOT.jar`
- `reactor-core-3.7.20.jar`
- `reactor-netty-core-1.2.19.jar`
- `reactor-netty-http-1.2.19.jar`
- Spring Framework 6.2.20 modules
- Spring Batch 5.2.7 modules
- Spring Integration 6.5.11 modules

**Component-Unknown is not the same as a vulnerability.**

The preferred remediation is to improve Sonatype component
identification using Maven coordinates, SBOM/PURL information, or
component mapping.

For example:

``` text
pkg:maven/org.springframework/spring-core@6.2.20
```

The internal `bce-modernization-0.0.7-SNAPSHOT.jar` should be identified
as an internal application component or handled under an approved IQ
policy/waiver.

## 13. Threat 7 / Threat 5 – License Findings

The report also contains license findings for components such as:

- `com.github.mwiede:jsch:0.2.18`
- `com.sun.istack:istack-commons-runtime:4.1.2`
- `jakarta.activation:jakarta.activation-api:2.1.4`
- `jakarta.persistence:jakarta.persistence-api:3.1.0`
- `org.eclipse.angus:angus-activation:2.0.3`
- `org.eclipse.angus:angus-mail:2.0.5`
- JAXB 4.0.9 modules
- `org.hibernate.orm:hibernate-core:6.6.53.Final`
- `org.reactivestreams:reactive-streams:1.0.4`
- Logback
- H2

These should not automatically be treated as CVE remediation.

Recommended process: 1. Identify the license. 2. Compare it with the
organization’s approved-license policy. 3. Request legal/open-source
approval where applicable. 4. Add approved components/licenses to the IQ
policy. 5. Replace a dependency only when the license cannot be
approved.

## 14. Recommended remediation order

### Immediate

1.  **Netty 4.1.137 → 4.1.138.Final**
2.  **Micrometer 1.15.12 → 1.15.13** (subject to Enterprise support)
3.  **Spring Data JPA 3.5.13 → 3.5.14** (subject to Enterprise support)
4.  **PostgreSQL JDBC 42.7.11 → 42.7.13**
5.  **H2 2.3.232 → 2.5.250**

### Next

6.  Tomcat → **10.1.59**
7.  Logback → **1.6.3**
8.  Jackson → **2.22.2**
9.  Flyway → **11.20.3**
10. MINA SSHD → **2.19.0**

### Keep unless IQ provides a specific reason

- Spring Boot **3.5.16**
- Spring Framework **6.2.20**
- Spring Batch **5.2.7**
- Spring Integration **6.5.11**
- Reactor Core **3.7.20**
- Reactor Netty **1.2.19**
- Log4j **2.26.1**, pending exact IQ CVE/policy investigation

## 15. Final target matrix

``` text
Spring Boot                  3.5.16
Netty                        4.1.138.Final
Tomcat                       10.1.59
Log4j API                    2.26.1 (investigate IQ finding)
Logback                      1.6.3
Jackson                      2.22.2
Micrometer                   1.15.13*
Reactor Core                 3.7.20*
Reactor Netty                1.2.19*
Spring Framework             6.2.20*
Spring Batch                 5.2.7*
Spring Integration           6.5.11*
Spring Data JPA              3.5.14*
Flyway                       11.20.3
H2                           2.5.250
PostgreSQL JDBC              42.7.13
Apache MINA SSHD             2.19.0
```

`*` Enterprise-supported maintenance release may be required.

## 16. Verification checklist

After changes:

``` bash
mvn clean verify
mvn dependency:tree
mvn dependency:tree -Dincludes=io.netty
mvn dependency:tree -Dincludes=org.springframework
mvn dependency:tree -Dincludes=com.fasterxml.jackson
mvn dependency:tree -Dincludes=org.apache.logging.log4j
mvn dependency:tree -Dincludes=io.projectreactor
mvn dependency:tree -Dincludes=io.micrometer
mvn dependency:tree -Dincludes=org.apache.tomcat
mvn dependency:tree -Dincludes=org.postgresql
mvn dependency:tree -Dincludes=com.h2database
```

Before the next IQ scan, confirm that no `4.1.137.Final`, `1.15.12`,
`3.5.13`, or `42.7.11` artifacts remain unless they are intentionally
retained and documented.

## Conclusion

The latest scan is materially better than the previous scan, dropping
from **109 to 73 violations**.

The highest-priority technical correction is **Netty 4.1.138.Final**
because 4.1.137.Final is now within a known vulnerable range.

Several other dependencies are already on the correct same-generation
security-fixed releases. Component-Unknown and license findings should
be addressed through Sonatype IQ mapping and policy processes rather
than indiscriminate dependency upgrades.

The **Log4j 2.26.1 Threat 10** finding is the major exception: obtain
the exact IQ CVE/policy ID before changing the version, because current
public security information indicates that 2.26.1 is already fixed.

**Spring Boot remains fixed at 3.5.16 throughout this remediation
plan.**
