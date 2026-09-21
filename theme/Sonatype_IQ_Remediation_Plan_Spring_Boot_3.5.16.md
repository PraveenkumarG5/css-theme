# Sonatype IQ Remediation Plan — Spring Boot 3.5.16

**Application:** `bce-modernization`  
**Latest IQ Build:** `bce-modernization_aa38512-bce_396176`  
**Build report:** 2026-09-21 18:44:18 UTC+0530  
**Hard constraint:** Spring Boot **must remain exactly 3.5.16**

---

## 1. Current Sonatype IQ Status

Latest report:

- **69 violations**
- **58 components affected**
- **137 components identified**
- **79% identified**
- **2 Critical**
- **64 High**
- **3 Medium**
- **0 Legacy violations**
- **Application Risk Score: 448**

The two most important visible security findings are:

1. `org.apache.logging.log4j:log4j-api:2.26.1`
   - Threat: 10
   - Security-Critical
   - CVSS: 9.2
   - Sonatype advisory: `sonatype-2026-006746`
   - Reachability: Reachable
   - Current dependency path:
     `log4j-to-slf4j:2.26.1 -> log4j-api:2.26.1`

2. `org.hibernate.orm:hibernate-core:6.6.57.Final`
   - Threat: 9
   - Security-High
   - CVE: `CVE-2026-77874`
   - CVSS: 7.1

---

# 2. Non-Negotiable Constraint

Do **not**:

- Upgrade Spring Boot beyond `3.5.16`
- Upgrade to Spring Boot 4.x
- Change the Spring Boot parent/version
- Randomly downgrade dependencies to older versions
- Claim a vulnerability is fixed without a fresh Sonatype IQ scan
- Change unrelated dependencies while fixing one component

Spring Boot must remain:

```text
3.5.16
```

---

# 3. Recommended Remediation Order

Use this order:

1. **Hibernate 6.6.57.Final → 6.6.58.Final**
2. Run tests and dependency-tree verification
3. Run a fresh Sonatype IQ scan
4. Re-check Netty
5. Investigate the second Critical finding if still present
6. Resolve exact Flyway advisory/fixed-version information
7. Resolve exact H2 advisory-ID discrepancy
8. Handle Jackson through waiver/exception until an official fixed release exists
9. Perform reachability analysis for Log4j
10. Handle HdrHistogram and LatencyUtils as waiver candidates if no safe fixed version exists
11. Handle Component-Unknown and license findings separately
12. Run the final clean Sonatype IQ scan

---

# 4. Hibernate — Immediate Fix

## Current

```text
org.hibernate.orm:hibernate-core:6.6.57.Final
CVE-2026-77874
CVSS: 7.1
Threat: Security-High
```

The current Hibernate ORM 6.6 line has:

```text
6.6.58.Final
```

Use `6.6.58.Final` as the next remediation candidate while keeping Spring Boot 3.5.16.

## Important

The previous Copilot report marked Hibernate as "FIXED" after moving it to `6.6.57.Final`.

That status is **not final** because the current Sonatype IQ scan still flags:

```text
hibernate-core:6.6.57.Final
CVE-2026-77874
```

Therefore:

> Hibernate is NOT considered fixed until the fresh IQ scan confirms that the CVE is no longer reported.

## Copilot Prompt — Hibernate

```text
We need to continue the Sonatype IQ remediation.

IMPORTANT HARD CONSTRAINT:
Spring Boot MUST remain exactly 3.5.16.
Do NOT upgrade Spring Boot.
Do NOT change Spring Boot parent/version.
Do NOT upgrade to Spring Boot 4.x.

The latest Sonatype IQ scan still reports:

org.hibernate.orm:hibernate-core:6.6.57.Final
CVE-2026-77874
Severity: 7.1
Threat: Security-High
Status: Open

The current Hibernate 6.6 series now has:

org.hibernate.orm:hibernate-core:6.6.58.Final

TASK:

1. Inspect the current pom.xml and determine how Hibernate 6.6.57.Final is currently overridden.

2. Change Hibernate ORM from:
   6.6.57.Final
   to:
   6.6.58.Final

3. Prefer using the existing Hibernate version property if the project currently has one, for example:
   <hibernate.version>6.6.58.Final</hibernate.version>

4. Do NOT introduce unnecessary individual version overrides if existing dependency management can align all Hibernate ORM modules.

5. Run:
   mvn dependency:tree -Dincludes=org.hibernate.orm

6. Ensure there is no mixture of:
   6.6.53.Final
   6.6.57.Final
   6.6.58.Final
   across Hibernate ORM modules.

7. Align all org.hibernate.orm modules consistently to 6.6.58.Final where appropriate.

8. Do NOT upgrade Hibernate to 7.x.

9. Do NOT change Spring Boot 3.5.16.

10. Search the application for:
    json_query
    json_exists
    json_value
    JSON path
    JsonPathHelper
    CriteriaBuilder
    Criteria API
    Hibernate JSON functions

11. Determine whether Oracle, DB2, or HANA dialects are used.

12. If the application does not use the affected JSON functionality, document this as additional mitigation evidence.

13. Do not rely on reachability alone to ignore the CVE. The dependency must still be moved to 6.6.58.Final if compatible.

14. Run:
    mvn clean verify

15. Run:
    mvn dependency:tree -Dincludes=org.hibernate.orm

16. Verify Spring Boot:
    mvn dependency:tree -Dincludes=org.springframework.boot:spring-boot

17. Confirm Spring Boot is exactly 3.5.16.

18. Update:
    docs/nexus/sonatype_iq_remediation_status_2026-09-21.md

    Record:
    Hibernate:
    6.6.57.Final -> 6.6.58.Final

    Reason:
    Remediation attempt for CVE-2026-77874.

19. DO NOT claim the CVE is fixed yet.
    Mark it fixed only after a fresh Sonatype IQ scan confirms that
    CVE-2026-77874 is no longer reported against Hibernate 6.6.58.Final.

20. Do not modify Log4j, Jackson, Flyway or H2 in this step.

21. Show the git diff before making unrelated changes.

At the end provide:

A. Exact pom.xml change
B. Hibernate dependency tree
C. Spring Boot version
D. mvn clean verify result
E. JSON-path/dialect usage analysis
F. Remaining Sonatype findings
```

---

# 5. Log4j — Do Not Blindly Upgrade

## Current

```text
org.apache.logging.log4j:log4j-api:2.26.1
```

Sonatype:

```text
Threat: 10
Severity: 9.2
Advisory: sonatype-2026-006746
Reachable: Yes
Status: Open
```

Dependency path:

```text
log4j-to-slf4j:2.26.1
        |
        +--> log4j-api:2.26.1
```

Occurrence:

```text
/target/bce-modernization-0.0.8-SNAPSHOT.jar/BOOT-INF/lib/log4j-api-2.26.1.jar
```

## Important finding

Apache Log4j `2.26.1` is currently the latest 2.26.x release.

Do **not** invent a version such as `2.26.2`.

The Sonatype advisory concerns Log4j Java deserialization behavior involving `FilteredObjectInputStream` / `LogEventProxy`.

This is different from Log4Shell.

The key question is whether the application actually exposes the vulnerable deserialization path.

## Recommended action

Perform a reachability/configuration analysis.

Check for:

```text
FilteredObjectInputStream
LogEventProxy
ObjectInputStream
LogEvent deserialization
serialized Log4j events
Socket receiver
JMS receiver
network-based Log4j event receivers
```

Also inspect application configuration for Log4j receivers or deserialization-related configuration.

If the application only uses:

```text
log4j-to-slf4j
```

as a logging bridge and does not expose the affected deserialization receiver path, document the evidence and request a Sonatype IQ waiver/exception.

Do not remove `log4j-to-slf4j` just to make the scanner disappear without checking whether any library depends on the Log4j API.

---

# 6. Jackson — Do Not Use an Unreleased Version

Current:

```text
com.fasterxml.jackson.core:jackson-databind:2.22.2
```

Findings:

```text
CVE-2026-91776
CVE-2026-91777
Severity: 6.9
Threat: Security-Medium
Reachable: Yes
Status: Open
```

The relevant Jackson release information indicates that:

```text
2.22.3
```

contains the fixes, but at the time of the current analysis it was **not yet released**.

Likewise, the corresponding 2.21.7 fix release was not yet released.

Therefore:

### Do NOT

```xml
<jackson.version>2.22.3</jackson.version>
```

Do not use an unreleased version.

Do not downgrade to 2.21.6 and claim the current CVEs are fixed.

## Recommended action

Use a temporary Sonatype IQ waiver/exception until an officially released compatible Jackson version containing the fixes becomes available.

Then update Jackson and run a fresh scan.

---

# 7. Flyway — Investigate Advisory Before Changing

Current:

```text
org.flywaydb:flyway-core:11.20.3
```

Sonatype:

```text
sonatype-2026-002508
Severity: 6.9
Threat: Security-Medium
Status: Open
```

Do not remove Flyway or change its version blindly.

Flyway participates in database migration/startup behavior in the project.

## Copilot Prompt

```text
Investigate the Sonatype IQ advisory:

sonatype-2026-002508

Current dependency:

org.flywaydb:flyway-core:11.20.3

Do NOT change the dependency yet.

Determine:

1. Exact advisory description.
2. Exact affected versions.
3. Exact fixed version, if one exists.
4. Whether 11.20.3 is affected.
5. Whether the issue is security-related or policy-related.
6. Whether a fixed version is compatible with Spring Boot 3.5.16.
7. Whether the project actually executes Flyway at runtime.
8. Inspect application.properties/application.yml and profiles.
9. Inspect Flyway configuration and startup behavior.
10. Determine whether changing/removing Flyway would affect production startup or migration behavior.

Do not make any dependency change until the evidence is documented.

At the end provide:

A. Advisory details
B. Affected versions
C. Fixed version
D. Runtime usage
E. Compatibility assessment
F. Recommended remediation or waiver
```

---

# 8. H2 — Resolve Advisory ID First

Current:

```text
com.h2database:h2:2.5.250
```

The project uses H2 as a datasource in configuration.

There is an advisory-ID discrepancy in previous analysis:

```text
sonatype-2018-0613
```

versus:

```text
sonatype-2018-0863
```

Do not guess which is correct.

## Copilot Prompt

```text
Investigate the current Sonatype IQ finding for:

com.h2database:h2:2.5.250

IMPORTANT:
There is an advisory ID discrepancy in previous analysis:
sonatype-2018-0613
vs
sonatype-2018-0863

Use the CURRENT Sonatype IQ report as the authoritative source.

Determine:

1. Exact current Sonatype advisory ID.
2. Advisory description.
3. Affected H2 versions.
4. Fixed H2 version, if available.
5. Whether H2 2.5.250 is actually affected.
6. Whether the finding relates to H2 Console, server mode, runtime database functionality, or another component.
7. Whether H2 is required at runtime or only for tests.
8. Inspect all application profiles/configuration.
9. Determine whether changing H2 scope to test would break runtime behavior.
10. Do not remove, downgrade, or change scope until this analysis is complete.

At the end provide:

A. Exact advisory ID
B. Exact affected versions
C. Fixed version
D. Runtime/test usage
E. Recommended remediation
F. Waiver recommendation if no safe fix exists
```

---

# 9. Netty — Already Aligned, Verify

The project was previously aligned to:

```text
io.netty:*:4.1.138.Final
```

Copilot's remediation report says Netty modules are fixed/aligned.

Verify:

```bash
mvn dependency:tree -Dincludes=io.netty
```

Expected result:

```text
No unintended older Netty versions
```

Also check the fresh IQ report.

Do not automatically upgrade beyond `4.1.138.Final`.

If Sonatype still reports Netty:

1. Identify the exact artifact.
2. Identify the exact advisory.
3. Check why 4.1.138.Final is still affected.
4. Only then choose another version.

---

# 10. HdrHistogram

Current:

```text
org.hdrhistogram:HdrHistogram:2.2.2
```

Reported CVEs include:

```text
CVE-2026-14683
CVE-2026-14686
```

It is a transitive Micrometer dependency.

Current analysis found no confirmed safe downgrade/fixed version.

Recommended approach:

- Do not randomly downgrade.
- Verify dependency tree.
- Confirm no direct application usage.
- If no safe fixed release exists, use a documented Sonatype waiver/exception based on reachability and exposure analysis.

Dependency command:

```bash
mvn dependency:tree -Dincludes=org.hdrhistogram:HdrHistogram
```

---

# 11. LatencyUtils

Current:

```text
org.latencyutils:LatencyUtils:2.0.3
```

Reported finding:

```text
CVE-2026-82596
```

It is transitive through Micrometer.

Recommended approach:

- Do not randomly downgrade.
- Verify dependency tree.
- Confirm no direct application usage.
- If no safe fixed release exists, use a documented waiver/exception.

Command:

```bash
mvn dependency:tree -Dincludes=org.latencyutils:LatencyUtils
```

---

# 12. Component-Unknown Findings

The current IQ report has many:

```text
Component-Unknown
```

These include internal application artifacts and various Spring/Micrometer/Reactor/Spring Batch/Spring Integration/Spring Data components.

Component-Unknown is **not automatically a vulnerability**.

The report has:

```text
137 components identified
79% identified
```

Recommended actions:

1. Improve component identification/PURL/SBOM mapping.
2. Identify internal application artifacts.
3. Map internal artifacts correctly.
4. Use waiver/approval only where appropriate.
5. Do not change versions merely to eliminate Component-Unknown.

Internal artifact:

```text
bce-modernization-0.0.8-SNAPSHOT.jar
```

should be treated as an internal component and mapped/waived according to the organization's Nexus IQ policy.

---

# 13. License Findings

Several findings are license-policy findings rather than security vulnerabilities.

Examples include Hibernate:

```text
LGPL-2.1
LGPL-3.0
EDL-1.0
```

Other components also have license-unknown or license-policy findings.

Do not change dependency versions merely to avoid a license policy finding.

Handle these independently through:

- Legal/license review
- Approved-license mapping
- Organization policy
- Component waiver
- Component replacement where required

---

# 14. CTO Warning Policy Findings

The report also contains policy findings such as:

- Logback
- Jakarta Annotation
- Jakarta Transaction API
- `javax.annotation`
- JNA
- Tomcat
- Hibernate Commons Annotations

These are not necessarily CVEs.

For each:

1. Open the policy detail.
2. Identify whether it is a security issue, license issue, or organization policy.
3. Record the exact reason.
4. Determine whether remediation is required.
5. Avoid unrelated version changes.

---

# 15. Commands to Run After Hibernate Change

## Hibernate

```bash
mvn dependency:tree -Dincludes=org.hibernate.orm
```

## Spring Boot

```bash
mvn dependency:tree -Dincludes=org.springframework.boot:spring-boot
```

Expected:

```text
3.5.16
```

## Netty

```bash
mvn dependency:tree -Dincludes=io.netty
```

## Jackson

```bash
mvn dependency:tree -Dincludes=com.fasterxml.jackson
```

## Log4j

```bash
mvn dependency:tree -Dincludes=org.apache.logging.log4j
```

## Flyway

```bash
mvn dependency:tree -Dincludes=org.flywaydb
```

## H2

```bash
mvn dependency:tree -Dincludes=com.h2database:h2
```

## HdrHistogram

```bash
mvn dependency:tree -Dincludes=org.hdrhistogram:HdrHistogram
```

## LatencyUtils

```bash
mvn dependency:tree -Dincludes=org.latencyutils:LatencyUtils
```

## Build validation

```bash
mvn clean verify
```

---

# 16. Mandatory Verification Rule

Never mark a finding as:

```text
FIXED
```

just because:

- the POM was changed,
- Maven build passed,
- a newer version was selected,
- Copilot says it is fixed.

The final authority for the Sonatype IQ remediation status is the **fresh Sonatype IQ scan**.

Correct lifecycle:

```text
Change dependency
       |
       v
Maven dependency tree
       |
       v
mvn clean verify
       |
       v
Application/runtime tests
       |
       v
Fresh Sonatype IQ scan
       |
       +---- Finding gone ----> FIXED
       |
       +---- Finding remains -> Investigate / remediate / waiver
```

---

# 17. Final Copilot Validation Prompt

After all approved dependency changes are made:

```text
Perform a final Sonatype IQ remediation validation for the bce-modernization project.

HARD CONSTRAINT:
Spring Boot MUST remain exactly 3.5.16.

Do not upgrade Spring Boot.

Run:

1. mvn clean verify

2. mvn dependency:tree -Dincludes=org.hibernate.orm

3. mvn dependency:tree -Dincludes=io.netty

4. mvn dependency:tree -Dincludes=com.fasterxml.jackson

5. mvn dependency:tree -Dincludes=org.apache.logging.log4j

6. mvn dependency:tree -Dincludes=org.flywaydb

7. mvn dependency:tree -Dincludes=com.h2database:h2

8. mvn dependency:tree -Dincludes=org.hdrhistogram:HdrHistogram

9. mvn dependency:tree -Dincludes=org.latencyutils:LatencyUtils

10. Verify Spring Boot:
    mvn dependency:tree -Dincludes=org.springframework.boot:spring-boot

Confirm exactly:
3.5.16

11. Search for:
    Hibernate JSON functions
    Log4j deserialization
    Flyway runtime usage
    H2 runtime usage
    HdrHistogram direct usage
    LatencyUtils direct usage

12. Review docs/nexus/sonatype_iq_remediation_status_2026-09-21.md.

13. Do not mark anything FIXED unless the fresh Sonatype IQ scan confirms it.

14. Do not modify unrelated dependencies.

Produce a final table with:

Component
Current version
Finding/advisory
Severity
Reachability
Action
Evidence
Fresh IQ status

Separate the findings into:

A. Confirmed fixed
B. Pending fresh IQ verification
C. Waiver/exception candidates
D. Requires user/security/legal approval
E. Component identification/license policy findings
F. Remaining unresolved security vulnerabilities
```

---

# 18. Target End State

The desired final state is **not necessarily zero IQ policy rows immediately**.

The objective is:

### Security vulnerabilities

- Fixed versions where officially available.
- No unsupported/unreleased dependency versions.
- Reachability documented.
- Remaining unavoidable findings formally waived/approved.
- Fresh IQ scan performed.

### Spring Boot

```text
3.5.16
```

must remain unchanged.

### Hibernate

Target remediation candidate:

```text
6.6.58.Final
```

### Netty

Current aligned target:

```text
4.1.138.Final
```

### Jackson

Current:

```text
2.22.2
```

Temporary waiver/exception until an official fixed release for the reported CVEs is available.

### Log4j

Current:

```text
2.26.1
```

Perform reachability/configuration analysis and seek a waiver if the affected deserialization path is not exposed.

### Flyway

Investigate exact Sonatype advisory before changing.

### H2

Resolve the exact current advisory ID and affected/fixed versions before changing.

### HdrHistogram / LatencyUtils

Treat as transitive dependency waiver candidates unless a confirmed safe fixed release is identified.

### Component-Unknown

Resolve mapping/SBOM/PURL identification separately.

### License / CTO Policy

Handle separately from CVE remediation.

---

# 19. Practical Execution Checklist

## Step 1

Run the Hibernate Copilot prompt.

## Step 2

Review the diff.

Expected primary change:

```text
Hibernate 6.6.57.Final
        ->
Hibernate 6.6.58.Final
```

## Step 3

Run:

```bash
mvn clean verify
```

## Step 4

Verify:

```bash
mvn dependency:tree -Dincludes=org.hibernate.orm
```

## Step 5

Verify:

```bash
mvn dependency:tree -Dincludes=org.springframework.boot:spring-boot
```

Confirm:

```text
3.5.16
```

## Step 6

Run a fresh Sonatype IQ scan.

## Step 7

Check whether:

```text
CVE-2026-77874
```

is gone.

## Step 8

If Hibernate is clean, move to the next unresolved security finding.

## Step 9

Do not make multiple speculative dependency changes at once.

## Step 10

Repeat the IQ scan after each meaningful remediation group.

---

# 20. Key Rules to Remember

1. **Never upgrade Spring Boot beyond 3.5.16.**
2. **Do not invent dependency versions.**
3. **Do not use unreleased versions.**
4. **Do not downgrade a dependency just because the scanner reports it.**
5. **Do not remove runtime dependencies without checking application behavior.**
6. **Do not treat Component-Unknown as a CVE.**
7. **Do not treat license findings as security vulnerabilities.**
8. **Do not mark a finding fixed until Sonatype IQ confirms it.**
9. **Use waivers for genuine no-fix/no-safe-upgrade situations with documented evidence.**
10. **Keep each remediation change isolated so failures can be diagnosed.**

---

## Current Priority Summary

| Priority | Component | Current Version | Action |
|---|---|---:|---|
| 1 | Hibernate | 6.6.57.Final | Move to 6.6.58.Final and rescan |
| 2 | Critical findings | Multiple | Identify both Threat-10 findings |
| 3 | Netty | 4.1.138.Final | Verify IQ is clean |
| 4 | Log4j | 2.26.1 | Reachability/deserialization analysis; likely waiver path if not exposed |
| 5 | Jackson | 2.22.2 | Wait for official fixed release; temporary waiver |
| 6 | Flyway | 11.20.3 | Determine exact advisory/fixed version |
| 7 | H2 | 2.5.250 | Determine exact advisory/fixed version |
| 8 | HdrHistogram | 2.2.2 | Transitive dependency; waiver candidate |
| 9 | LatencyUtils | 2.0.3 | Transitive dependency; waiver candidate |
| 10 | Component-Unknown | Various | SBOM/PURL/component mapping |
| 11 | License findings | Various | Separate license/policy review |

---

## Bottom Line

The immediate technical change should be:

```text
Hibernate 6.6.57.Final
        ↓
Hibernate 6.6.58.Final
```

while keeping:

```text
Spring Boot 3.5.16
```

Then run:

```bash
mvn clean verify
mvn dependency:tree -Dincludes=org.hibernate.orm
```

and perform a **fresh Sonatype IQ scan**.

Only after that scan should Hibernate be classified as fixed.
