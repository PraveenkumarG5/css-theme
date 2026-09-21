# Spring Boot 3.5.16 — Sonatype IQ Security & Policy Remediation

## 1. Executive Summary

This document consolidates the latest Sonatype IQ findings for the `bce-modernization` Spring Boot application and the recommended remediation approach.

### Hard constraint

- **Spring Boot must remain exactly `3.5.16`.**
- Do **not** upgrade Spring Boot to 4.x or another 3.x release as part of this remediation.
- Dependency upgrades must remain compatible with Spring Boot `3.5.16`.

### Latest Sonatype IQ report

Build report:

- Build: `bce-modernization_aa38512-bce_396176`
- Report time: `2026-09-19 18:06:00 UTC+0530`
- Violations: **69**
- Components affected: **58**
- Components identified: **137**
- Identification rate: **79%**
- Legacy violations: **0**
- Critical: **2**
- High: **64**
- Medium: **3**
- Application Risk Score: **448**

The findings fall into several different categories:

1. Genuine security vulnerabilities with a known patched version.
2. Sonatype proprietary advisories requiring applicability/reachability analysis.
3. Security findings where the upstream fixed release is not yet available.
4. License-policy findings.
5. Organization-specific CTO warning policies.
6. Component-Unknown findings that are not themselves vulnerabilities.

Do **not** treat every Sonatype IQ violation as a reason to change a dependency version.

---

# 2. Recommended Immediate Actions

| Component | Current version | Current status | Recommended action |
|---|---:|---|---|
| Spring Boot | `3.5.16` | Fixed project constraint | **Keep exactly 3.5.16** |
| Hibernate ORM | `6.6.53.Final` | CVE-2026-77874 | **Upgrade to 6.6.57.Final** |
| Log4j API | `2.26.1` | Sonatype advisory `sonatype-2026-006746` | **Keep version; audit deserialization path; request waiver if not applicable** |
| Flyway | `11.20.3` | Sonatype advisory `sonatype-2026-002508` | **Do not guess a version; obtain IQ advisory details first** |
| H2 | `2.5.250` | Sonatype advisory `sonatype-2018-0613` | **Keep pending applicability/fix analysis; ensure test scope if possible** |
| Jackson Databind | `2.22.2` | CVE-2026-91776, CVE-2026-91777 | **Do not use unreleased 2.22.3; temporary waiver/monitor until fixed release** |
| Netty | `4.1.138.Final` | IQ still reports finding | **Keep 4.1.138; verify no 4.1.137 remains and investigate IQ mapping** |
| HdrHistogram | `2.2.2` | Low severity finding | **Investigate exact advisory; avoid random downgrade** |
| LatencyUtils | `2.0.3` | Low severity finding | **Investigate exact advisory; avoid random downgrade** |

---

# 3. Hibernate ORM

## Current finding

Component:

```text
org.hibernate.orm:hibernate-core:6.6.53.Final
```

Finding:

```text
CVE-2026-77874
Severity: 7.1
Threat: 9
Security: High
Status: Open
```

There are also separate license-policy findings for LGPL licenses. Those are **not the same thing as the CVE**.

## Vulnerability

CVE-2026-77874 concerns SQL injection through a JSON-path argument in Hibernate ORM. The reported vulnerable code involves:

```text
JsonPathHelper.appendInlinedJsonPathIncludingPassingClause()
```

The affected database dialect context includes Oracle, DB2, or HANA.

## Recommended version

Hibernate's 6.6 release line currently lists:

```text
6.6.57.Final
```

Therefore change:

```xml
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>6.6.57.Final</version>
</dependency>
```

If other Hibernate ORM modules are explicitly versioned, align them to the same `6.6.57.Final` release where appropriate.

### Do not

- Upgrade to Hibernate 7.x.
- Upgrade Spring Boot.
- Randomly downgrade Hibernate.
- Mix unrelated Hibernate ORM versions.

## Verify

```bash
mvn dependency:tree -Dincludes=org.hibernate.orm
mvn dependency:tree -Dincludes=org.hibernate
mvn clean verify
```

Also inspect application code for JSON-path functionality and Criteria API usage.

---

# 4. Log4j API — Sonatype Advisory

## Current finding

```text
org.apache.logging.log4j:log4j-api:2.26.1
```

Sonatype advisory:

```text
sonatype-2026-006746
Severity: 9.2
Threat: 10
Security-Critical
Reachability: Reachable
Status: Open
```

## Important clarification

This is **not Log4Shell**.

The advisory concerns a Java-deserialization pattern involving Log4j objects, including concepts such as:

```text
FilteredObjectInputStream
LogEventProxy
ObjectInputStream
SerializedLayout
TcpSocketServer
UdpSocketServer
```

The risk depends on whether the application actually deserializes Java-serialized Log4j `LogEvent` objects from an untrusted source.

A Sonatype/Broadcom analysis indicates that Spring Boot applications are not automatically vulnerable simply because `log4j-api` is present.

## Do not blindly change Log4j

Do **not** downgrade or randomly upgrade Log4j merely to make the IQ finding disappear.

First inspect the application.

### Search for

```text
FilteredObjectInputStream
LogEventProxy
ObjectInputStream
TcpSocketServer
UdpSocketServer
SerializedLayout
```

Also inspect:

```text
log4j2.xml
log4j2-spring.xml
log4j.properties
```

Search for custom network logging receivers and Java serialization/deserialization.

## Decision

If the application:

- does not deserialize Java-serialized Log4j events,
- does not expose the relevant Log4j receiver,
- and does not accept untrusted serialized Log4j event data,

then document the evidence and request an approved Sonatype IQ waiver/exception if required.

### Copilot analysis prompt

```text
Analyze the repository for Sonatype advisory sonatype-2026-006746 affecting
org.apache.logging.log4j:log4j-api:2.26.1.

IMPORTANT:
- Do NOT change Spring Boot.
- Do NOT change the Log4j version as part of this analysis.
- Do NOT modify application code.
- This is an investigation only.

Search the complete repository for:

1. FilteredObjectInputStream
2. LogEventProxy
3. ObjectInputStream
4. TcpSocketServer
5. UdpSocketServer
6. SerializedLayout
7. Java serialization/deserialization involving Log4j LogEvent
8. log4j2.xml
9. log4j2-spring.xml
10. log4j.properties
11. Any custom Log4j network receiver
12. Any code that accepts serialized Log4j events from external/untrusted sources

Determine:

- Whether the application actually uses the vulnerable/deserialization pattern.
- Whether the relevant code path is reachable at runtime.
- Whether any external/untrusted input can reach it.
- Whether it is production code, test code, or unused configuration.
- Exact file names, class names, methods, and configuration entries that support the conclusion.
- Whether log4j-api is present only as a normal logging API dependency.
- Whether there is any evidence that the application exposes a Log4j socket receiver.

Produce a report with:

A. Finding
B. Evidence
C. Runtime reachability
D. External exposure
E. Risk assessment
F. Recommended remediation
G. Whether a Sonatype IQ waiver/exception appears technically justified

Do not invent evidence.
Do not change dependencies.
```

---

# 5. Flyway

## Current finding

```text
org.flywaydb:flyway-core:11.20.3
```

Sonatype advisory:

```text
sonatype-2026-002508
Severity: 6.9
Threat: 7
Security: Medium
Status: Open
```

## Important limitation

The exact fixed version for this proprietary Sonatype advisory has not been established from public information.

Therefore:

**Do not invent a fixed Flyway version.**

Do not blindly upgrade from:

```text
11.20.3
```

to a random 11.x, 12.x, or 13.x version simply to clear IQ.

## Required next step

Open the Sonatype IQ advisory details for:

```text
sonatype-2026-002508
```

Capture:

- vulnerability description,
- affected versions,
- fixed versions,
- vulnerable component range,
- remediation recommendation,
- exploitability/reachability information.

Then choose the fixed version that is demonstrably compatible with Spring Boot `3.5.16`.

## Verify dependency origin

```bash
mvn dependency:tree -Dincludes=org.flywaydb
mvn dependency:tree -Dincludes=org.flywaydb:flyway-core
```

Also check whether Flyway is used for:

- application startup migrations,
- database migration only,
- CLI/build-time activities,
- production runtime.

---

# 6. H2 Database

## Current finding

```text
com.h2database:h2:2.5.250
```

Sonatype advisory:

```text
sonatype-2018-0613
Severity: 6.0
Threat: 7
Security: Medium
Status: Open
Reachability: Reachable
```

There is also a license warning:

```text
LGPL-3.0
```

The license finding is separate from the security finding.

## Current version

H2 `2.5.250` is a current H2 2.5.x release.

Therefore:

**Do not downgrade H2.**

First determine why Sonatype is still associating `sonatype-2018-0613` with this version and what exact configuration/component is affected.

## Check whether H2 is test-only

Run:

```bash
mvn dependency:tree -Dincludes=com.h2database:h2
```

If H2 is only used for tests, prefer:

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

Do not make H2 a production runtime dependency if it is not required in production.

## Check

- H2 Console exposure
- H2 server mode
- Remote database connections
- Dynamic SQL/configuration
- Production runtime classpath
- Test-only usage

If the affected functionality is not used, document this and request an IQ waiver if necessary.

---

# 7. Jackson Databind

## Current finding

```text
com.fasterxml.jackson.core:jackson-databind:2.22.2
```

Findings:

```text
CVE-2026-91776
Severity: 6.9

CVE-2026-91777
Severity: 6.9
```

Threat:

```text
7 — Security Medium
```

## Important version status

Jackson upstream release information identifies these CVEs as fixes associated with:

```text
2.22.3
```

However, at the time of this analysis, `2.22.3` is not yet a released version.

Therefore:

**Do not put `2.22.3` into the POM before it is actually released.**

Do not invent or fabricate a version.

## Current recommendation

Temporarily remain on:

```text
2.22.2
```

while:

1. Monitoring for the official `2.22.3` release.
2. Reviewing the exact Sonatype vulnerability details.
3. Requesting an approved temporary IQ waiver/exception if the policy blocks the build.
4. Assessing whether the vulnerable functionality is actually reachable.

## Find the dependency path

```bash
mvn dependency:tree -Dincludes=com.fasterxml.jackson
mvn dependency:tree -Dincludes=com.fasterxml.jackson.core:jackson-databind
```

Because Jackson is transitive, identify which dependency is introducing:

```text
jackson-databind:2.22.2
```

Do not independently override Jackson without checking the complete Jackson module set.

---

# 8. Netty

## Current version

The application has been updated to:

```text
4.1.138.Final
```

The IQ report still shows findings for:

```text
io.netty:netty-codec-http:4.1.138.Final
io.netty:netty-handler:4.1.138.Final
```

## Recommendation

**Keep Netty at 4.1.138.Final for now.**

Do not downgrade.

Do not blindly move to another Netty version without a newer confirmed security release.

## Ensure all Netty modules align

Recommended dependency management:

```xml
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

Verify:

```bash
mvn dependency:tree -Dincludes=io.netty
```

Make sure there is no remaining:

```text
4.1.137.Final
```

or another older Netty module.

If everything is already `4.1.138.Final`, investigate why Sonatype IQ still reports the finding. This may be:

- IQ vulnerability-data timing,
- component mapping,
- advisory applicability,
- another transitive module,
- or an advisory whose fixed version differs from the public security baseline.

---

# 9. HdrHistogram

Current component:

```text
org.hdrhistogram:HdrHistogram:2.2.2
```

Earlier IQ findings classified this as:

```text
Threat 3
Security Low
```

Do not randomly downgrade it.

First establish:

```bash
mvn dependency:tree -Dincludes=org.hdrhistogram:HdrHistogram
```

Determine whether it is:

- direct,
- transitive,
- actually used,
- test-only,
- production runtime.

If the advisory has no confirmed patched release, use documented mitigation/waiver rather than an arbitrary downgrade.

---

# 10. LatencyUtils

Current component:

```text
org.latencyutils:LatencyUtils:2.0.3
```

Earlier IQ findings classified this as:

```text
Threat 3
Security Low
```

Verify:

```bash
mvn dependency:tree -Dincludes=org.latencyutils:LatencyUtils
```

If it is transitive and there is no confirmed fixed release, do not randomly downgrade.

Assess:

- actual use,
- runtime exposure,
- transitive dependency path,
- available vendor/upstream remediation,
- Sonatype waiver/exception.

---

# 11. Organization-Specific CTO Warning Policy Findings

The report also contains Threat 5 findings from the organization-specific:

```text
UBS CTO Warning Policy
```

Examples include:

```text
ch.qos.logback:logback-classic:1.6.3
ch.qos.logback:logback-core:1.6.3
jakarta.annotation:jakarta.annotation-api:2.1.1
jakarta.transaction:jakarta.transaction-api:2.0.1
javax.annotation:javax.annotation-api:1.3.2
net.java.dev.jna:jna:5.13.0
net.java.dev.jna:jna-platform:5.17.0
org.apache.tomcat.embed:tomcat-embed-core:10.1.59
org.hibernate.common:hibernate-commons-annotations:7.0.3
```

These should **not automatically be treated as CVEs**.

The next step is to inspect the exact policy condition.

For each finding determine:

1. Is this a security vulnerability?
2. Is this a license warning?
3. Is it an approved/unsupported version policy?
4. Is there a required organization-standard version?
5. Is the finding informational only?

Only then decide whether a dependency change is necessary.

---

# 12. License Findings

Several components have:

```text
Threat 7
License-Not-Assigned
```

Examples include:

```text
com.github.mwiede:jsch:0.2.18
com.sun.istack:istack-commons-runtime:4.1.2
jakarta.activation:jakarta.activation-api:2.1.4
jakarta.persistence:jakarta.persistence-api:3.1.0
jakarta.xml.bind:jakarta.xml.bind-api:4.0.5
org.eclipse.angus:angus-activation:2.0.3
org.eclipse.angus:jakarta.mail:2.0.5
org.glassfish.jaxb:jaxb-core:4.0.9
org.glassfish.jaxb:jaxb-runtime:4.0.9
org.glassfish.jaxb:txw2:4.0.9
org.reactivestreams:reactive-streams:1.0.4
```

These are primarily **license-policy problems**, not vulnerability remediation problems.

Do not upgrade dependencies merely because their license has not been mapped.

Instead:

1. Identify the actual license.
2. Review the organization's approved-license policy.
3. Map/assign the license in Sonatype IQ if permitted.
4. Request legal/license approval if required.
5. Only replace the component if the license itself is unacceptable.

---

# 13. Component-Unknown Findings

The latest report shows:

```text
137 components identified
79% identified
```

Examples of Component-Unknown findings include:

```text
bce-modernization-0.0.8-SNAPSHOT.jar
micrometer-commons-1.15.13.jar
micrometer-core-1.15.13.jar
micrometer-jakarta9-1.15.13.jar
micrometer-observation-1.15.13.jar
reactor-core-3.7.20.jar
reactor-netty-core-1.2.19.jar
reactor-netty-http-1.2.19.jar
Spring Framework 6.2.20 modules
Spring Batch 5.2.7
Spring Integration 6.5.11
spring-data-commons:3.5.14
spring-data-jpa:3.5.14
```

## Important

**Component-Unknown does not mean vulnerable.**

It means Sonatype has not successfully mapped/identified the component to the expected component metadata.

The internal application:

```text
bce-modernization-0.0.8-SNAPSHOT.jar
```

should normally be mapped as an internal application component or handled through the organization's approved policy.

## PURL example

A standard Maven Package URL can look like:

```text
pkg:maven/org.springframework/spring-core@6.2.20
```

Improve:

- Maven coordinates,
- SBOM generation,
- component identification,
- repository metadata,
- IQ component mapping.

---

# 14. Current Dependency Baseline

The current working baseline should be approximately:

```text
Spring Boot                  3.5.16
Netty                         4.1.138.Final
Tomcat                        10.1.59
Log4j                         2.26.1
Logback                       1.6.3
Jackson                       2.22.2
Micrometer                    1.15.13
Reactor Core                  3.7.20
Reactor Netty                 1.2.19
Spring Framework              6.2.20
Spring Batch                  5.2.7
Spring Integration            6.5.11
Spring Data JPA               3.5.14
Flyway                        11.20.3
H2                            2.5.250
PostgreSQL JDBC               42.7.13
Apache MINA SSHD              2.19.0
Hibernate ORM                 6.6.57.Final  <-- upgrade required
HdrHistogram                  2.2.2
LatencyUtils                  2.0.3
```

The list above is a remediation baseline, not a statement that every version is currently vulnerability-free according to Sonatype IQ.

---

# 15. Spring Boot 3.5.16 Compatibility Principle

The application must remain:

```text
Spring Boot 3.5.16
```

Therefore every proposed dependency change must be evaluated against:

- Spring Framework compatibility
- Jakarta API compatibility
- Hibernate ORM compatibility
- Jackson module alignment
- Reactor compatibility
- Micrometer compatibility
- Spring Data compatibility
- Spring Batch compatibility
- Spring Integration compatibility
- Java runtime compatibility
- Maven dependency convergence

Do not solve an IQ issue by upgrading the entire Spring Boot stack unless the project constraint is explicitly changed.

---

# 16. Maven Verification Commands

Run these commands after making changes.

## Full dependency tree

```bash
mvn dependency:tree > dependency-tree.txt
```

## Hibernate

```bash
mvn dependency:tree -Dincludes=org.hibernate.orm
mvn dependency:tree -Dincludes=org.hibernate
```

## Log4j

```bash
mvn dependency:tree -Dincludes=org.apache.logging.log4j
```

## Jackson

```bash
mvn dependency:tree -Dincludes=com.fasterxml.jackson
mvn dependency:tree -Dincludes=com.fasterxml.jackson.core:jackson-databind
```

## Flyway

```bash
mvn dependency:tree -Dincludes=org.flywaydb
mvn dependency:tree -Dincludes=org.flywaydb:flyway-core
```

## H2

```bash
mvn dependency:tree -Dincludes=com.h2database:h2
```

## Netty

```bash
mvn dependency:tree -Dincludes=io.netty
```

## HdrHistogram

```bash
mvn dependency:tree -Dincludes=org.hdrhistogram:HdrHistogram
```

## LatencyUtils

```bash
mvn dependency:tree -Dincludes=org.latencyutils:LatencyUtils
```

## Build and tests

```bash
mvn clean verify
```

---

# 17. Recommended Remediation Sequence

## Phase 1 — Known fixed version

Upgrade:

```text
Hibernate 6.6.53.Final
        ↓
Hibernate 6.6.57.Final
```

Run:

```bash
mvn clean verify
```

---

## Phase 2 — Log4j analysis

Keep:

```text
Log4j 2.26.1
```

Investigate:

```text
sonatype-2026-006746
```

Determine whether the vulnerable deserialization path exists.

If not applicable:

```text
Document evidence
        ↓
Request IQ waiver/exception
```

---

## Phase 3 — Jackson

Keep:

```text
Jackson 2.22.2
```

temporarily.

Monitor for the official release containing:

```text
CVE-2026-91776
CVE-2026-91777
```

Do not put an unreleased version in the POM.

When the fixed release becomes available:

```text
Upgrade
   ↓
Run dependency convergence
   ↓
Run tests
   ↓
Re-scan IQ
```

---

## Phase 4 — Flyway

Get exact Sonatype advisory details:

```text
sonatype-2026-002508
```

Then determine the exact fixed version.

---

## Phase 5 — H2

Get exact Sonatype advisory details:

```text
sonatype-2018-0613
```

Determine:

- affected functionality,
- fixed version,
- production/test scope,
- exploitability.

If H2 is test-only, ensure test scope.

---

## Phase 6 — Netty

Keep:

```text
4.1.138.Final
```

Verify all modules.

If IQ still flags the same version, investigate Sonatype advisory mapping rather than continuously upgrading.

---

## Phase 7 — Policy and license cleanup

Separate:

```text
Security
License
CTO Policy
Component Unknown
```

Do not mix these categories.

---

# 18. Copilot Prompt — Complete Dependency Remediation Analysis

Use the following prompt with Copilot/agent mode:

```text
You are reviewing the bce-modernization Maven Spring Boot application.

NON-NEGOTIABLE CONSTRAINT:
Spring Boot MUST remain exactly 3.5.16.
Do NOT upgrade Spring Boot.
Do NOT upgrade to Spring Boot 4.x.
Do NOT change Spring Boot to another release.

Goal:
Analyze and remediate Sonatype IQ findings while making the smallest safe dependency changes.

Current important findings:

1. Hibernate:
org.hibernate.orm:hibernate-core:6.6.53.Final
CVE-2026-77874
Severity 7.1

Required target:
6.6.57.Final

2. Log4j:
org.apache.logging.log4j:log4j-api:2.26.1
Sonatype advisory:
sonatype-2026-006746
Severity 9.2

Do NOT change Log4j version initially.
Analyze whether the vulnerable Java deserialization path actually exists.

Search for:
FilteredObjectInputStream
LogEventProxy
ObjectInputStream
TcpSocketServer
UdpSocketServer
SerializedLayout
Java serialization of Log4j LogEvent
log4j2.xml
log4j2-spring.xml
log4j.properties

Determine actual reachability and external exposure.

3. Flyway:
org.flywaydb:flyway-core:11.20.3
Sonatype:
sonatype-2026-002508
Severity 6.9

Do not guess a fixed version.
First identify the advisory, affected versions and fixed versions.

4. H2:
com.h2database:h2:2.5.250
Sonatype:
sonatype-2018-0613
Severity 6.0

Do not downgrade.
Determine whether H2 is test-only and whether the affected functionality is actually used.

5. Jackson:
com.fasterxml.jackson.core:jackson-databind:2.22.2

Findings:
CVE-2026-91776
CVE-2026-91777

Do not use unreleased 2.22.3.
Determine the official fixed released version before making any change.

6. Netty:
4.1.138.Final

Do not downgrade.
Verify every Netty module resolves to 4.1.138.Final and identify why Sonatype still reports a finding.

7. HdrHistogram:
2.2.2

8. LatencyUtils:
2.0.3

Analyze exact advisories and do not randomly downgrade.

For every dependency:

A. Identify direct vs transitive.
B. Show dependency path.
C. Identify current resolved version.
D. Identify Sonatype finding.
E. Determine whether a fixed version exists.
F. Verify Spring Boot 3.5.16 compatibility.
G. Make the smallest safe change.
H. Do not invent versions.
I. Do not upgrade unrelated dependencies.
J. Do not change application behavior unnecessarily.

Before editing pom.xml, produce a remediation plan.

After approved changes:

1. Update pom.xml.
2. Run dependency convergence checks.
3. Run:
   mvn clean verify
4. Generate:
   mvn dependency:tree > dependency-tree.txt
5. Verify no old vulnerable versions remain.
6. Produce a final before/after dependency table.
7. List findings that require Sonatype waiver/exception.
8. List remaining Component-Unknown and license findings separately.

Do not claim a vulnerability is fixed unless the resolved dependency version and advisory evidence support that conclusion.
```

---

# 19. Sonatype IQ Waiver/Exception Candidates

Potential candidates for an approved waiver, subject to organizational review:

### Log4j

```text
sonatype-2026-006746
```

Only if repository analysis confirms the vulnerable deserialization path is absent or not externally reachable.

### Jackson

```text
CVE-2026-91776
CVE-2026-91777
```

Temporary waiver may be necessary until an official fixed release is available.

### Flyway

```text
sonatype-2026-002508
```

Only after determining whether a compatible fixed release exists.

### H2

```text
sonatype-2018-0613
```

Potentially waiver/mitigation if the affected functionality is not used or H2 is strictly test-only, subject to IQ policy.

### Low severity dependencies

HdrHistogram and LatencyUtils may require waivers if no patched upstream release exists and the risk is accepted.

---

# 20. Important Rules

## Rule 1

Never downgrade a dependency merely to satisfy IQ.

## Rule 2

Never use an unreleased version in production.

Example:

```text
Jackson 2.22.3
```

should not be declared until officially released.

## Rule 3

Do not change Spring Boot from:

```text
3.5.16
```

## Rule 4

Do not treat:

```text
License-Not-Assigned
```

as a CVE.

## Rule 5

Do not treat:

```text
Component-Unknown
```

as proof of a vulnerability.

## Rule 6

A Sonatype "Reachable" label should not automatically be interpreted as proof that an exploitable application code path is reachable. Confirm the actual runtime path.

## Rule 7

Keep direct and transitive dependency analysis separate.

## Rule 8

After every dependency change:

```text
dependency tree
        ↓
compile
        ↓
unit tests
        ↓
integration tests
        ↓
application startup
        ↓
Sonatype IQ scan
```

---

# 21. Final Target State

The desired final state is:

```text
Spring Boot 3.5.16
        |
        +-- Hibernate ORM 6.6.57.Final
        |
        +-- Netty 4.1.138.Final
        |
        +-- Log4j 2.26.1
        |      |
        |      +-- advisory investigated
        |      +-- waiver if not applicable
        |
        +-- Jackson 2.22.2
        |      |
        |      +-- CVEs monitored
        |      +-- upgrade when official fixed release exists
        |
        +-- Flyway 11.20.3
        |      |
        |      +-- Sonatype advisory details required
        |
        +-- H2 2.5.250
        |      |
        |      +-- applicability/test scope assessed
        |
        +-- HdrHistogram 2.2.2
        |
        +-- LatencyUtils 2.0.3
        |
        +-- Remaining dependencies
               |
               +-- License policy handled separately
               +-- Component-Unknown mapping handled separately
               +-- CTO policy findings assessed separately
```

---

# 22. Verification Checklist

Before submitting the next Sonatype IQ scan:

- [ ] Spring Boot remains exactly `3.5.16`
- [ ] Hibernate upgraded to `6.6.57.Final`
- [ ] No Hibernate 7.x introduced
- [ ] No old Hibernate 6.6.53.Final remains
- [ ] Log4j `2.26.1` deserialization path investigated
- [ ] Sonatype `sonatype-2026-006746` applicability documented
- [ ] Flyway `sonatype-2026-002508` exact advisory details obtained
- [ ] H2 `sonatype-2018-0613` exact applicability investigated
- [ ] H2 scope verified
- [ ] Jackson dependency path identified
- [ ] Jackson CVE-2026-91776 investigated
- [ ] Jackson CVE-2026-91777 investigated
- [ ] No unreleased Jackson version added
- [ ] All Netty modules verified at `4.1.138.Final`
- [ ] No Netty `4.1.137.Final` remains
- [ ] HdrHistogram dependency path checked
- [ ] LatencyUtils dependency path checked
- [ ] License findings separated from security findings
- [ ] Component-Unknown findings separated from security findings
- [ ] CTO policy findings reviewed separately
- [ ] `mvn clean verify` passes
- [ ] Dependency tree reviewed
- [ ] Sonatype IQ re-scan completed
- [ ] Remaining violations have an explicit remediation or waiver plan

---

# 23. Bottom Line

The safest remediation strategy is **not** to upgrade every dependency that Sonatype IQ flags.

The immediate confirmed code-level dependency upgrade is:

```text
Hibernate ORM
6.6.53.Final
        ↓
6.6.57.Final
```

The other important findings require different handling:

```text
Log4j 2.26.1
    → investigate Sonatype advisory applicability

Jackson 2.22.2
    → CVE-2026-91776 / CVE-2026-91777
    → wait for an official released fixed version

Flyway 11.20.3
    → obtain exact Sonatype advisory/fixed-version details

H2 2.5.250
    → investigate advisory applicability and scope

Netty 4.1.138.Final
    → keep; verify dependency convergence and IQ mapping

HdrHistogram / LatencyUtils
    → investigate exact advisory and available remediation

License findings
    → handle through license policy

Component-Unknown
    → improve Sonatype component identification

CTO Warning Policy
    → review organization-specific policy conditions
```

This approach preserves the hard requirement of **Spring Boot 3.5.16**, minimizes unnecessary dependency changes, and separates genuine security remediation from policy, license, and component-identification work.
