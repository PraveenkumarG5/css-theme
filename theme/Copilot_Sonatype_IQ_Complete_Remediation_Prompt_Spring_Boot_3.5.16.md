# Copilot Prompt — Complete Sonatype IQ Remediation with Spring Boot 3.5.16 Fixed

## Purpose

Use this prompt with **GitHub Copilot Agent mode** together with:

1. The complete exported **Sonatype IQ vulnerability/policy PDF**.
2. The existing Maven/Spring Boot source repository.

The objective is to read the entire Sonatype IQ report, map every finding to the actual repository/dependency tree/configuration, remediate all safely fixable issues, and keep **Spring Boot exactly at 3.5.16**.

---

```text
# Sonatype IQ Complete Security Remediation — Spring Boot 3.5.16 MUST REMAIN FIXED

You are acting as a senior Java/Spring Boot dependency-security remediation engineer.

I will provide:

1. The complete exported Sonatype IQ vulnerability/policy PDF.
2. The existing Maven Spring Boot source repository.

Your responsibility is to analyze the Sonatype IQ PDF, identify every applicable issue, trace each issue into the actual source/dependency tree/configuration, and remediate the issues as safely as possible.

============================================================
NON-NEGOTIABLE REQUIREMENT
============================================================

SPRING BOOT MUST REMAIN EXACTLY:

    Spring Boot 3.5.16

This is a hard constraint.

DO NOT:

- Upgrade Spring Boot.
- Downgrade Spring Boot.
- Change Spring Boot to another 3.5.x version.
- Upgrade to Spring Boot 4.x.
- Change the Spring Boot parent version.
- Replace the Spring Boot dependency-management strategy just to bypass IQ.
- Make any dependency change that requires changing Spring Boot unless you first stop and ask me.

If a Sonatype vulnerability cannot be fixed without changing Spring Boot 3.5.16, STOP and ask me for explicit instructions.

============================================================
PRIMARY OBJECTIVE
============================================================

Read the COMPLETE Sonatype IQ PDF first.

Do NOT immediately start changing pom.xml.

First build a complete understanding of:

- All vulnerabilities
- All affected components
- Current versions
- Severity
- CVE/Sonatype advisory IDs
- Direct vs transitive dependencies
- Reachability information
- License violations
- Organization-specific policy violations
- Component-Unknown findings
- Recommended/fixed versions, if provided
- Any additional information shown in the PDF

Then compare those findings with the actual repository.

Your goal is to remediate as many findings as technically and safely possible while keeping:

    Spring Boot = 3.5.16

============================================================
PHASE 1 — READ AND ANALYZE THE PDF
============================================================

Before modifying any files, read the ENTIRE Sonatype IQ PDF.

Do not rely only on the summary page.

Extract every finding into an internal remediation table containing:

- Component
- Group ID
- Artifact ID
- Current version
- Direct/transitive
- Vulnerability ID
- CVE ID if available
- Sonatype advisory ID if available
- Severity
- CVSS
- Threat level
- Reachability
- Policy name
- License information
- Affected version
- Fixed version
- Sonatype recommendation
- Status
- Any special conditions

If the PDF contains multiple pages of findings, process ALL pages.

Do not stop after the first page.

============================================================
PHASE 2 — INSPECT THE ACTUAL REPOSITORY
============================================================

After understanding the PDF, inspect the complete repository.

Inspect at minimum:

- pom.xml
- parent POM
- dependencyManagement
- Maven profiles
- settings relevant to dependency resolution
- application configuration
- application.yml
- application.yaml
- application.properties
- log4j configuration
- logback configuration
- Flyway configuration
- Hibernate configuration
- Jackson configuration
- H2 configuration
- Netty configuration
- security configuration
- Docker/container files
- CI/CD configuration where relevant
- test configuration
- generated dependency files if present
- relevant Java/Kotlin source code

Run:

    mvn dependency:tree

Also run focused dependency-tree commands for affected components.

For example:

    mvn dependency:tree -Dincludes=org.hibernate.orm
    mvn dependency:tree -Dincludes=org.apache.logging.log4j
    mvn dependency:tree -Dincludes=com.fasterxml.jackson
    mvn dependency:tree -Dincludes=org.flywaydb
    mvn dependency:tree -Dincludes=com.h2database:h2
    mvn dependency:tree -Dincludes=io.netty

Use the appropriate commands for every affected dependency discovered from the PDF.

============================================================
PHASE 3 — MAP EVERY IQ FINDING TO THE CODE
============================================================

For every Sonatype finding, determine:

1. Where the component comes from.
2. Whether it is direct or transitive.
3. Which dependency introduces it.
4. Whether a newer fixed version exists.
5. Whether that fixed version is compatible with Spring Boot 3.5.16.
6. Whether the vulnerable functionality is actually used.
7. Whether the vulnerable code path is reachable.
8. Whether it is production or test-only.
9. Whether configuration affects exploitability.
10. Whether a code/configuration change is required.
11. Whether a Maven dependency change is required.
12. Whether an IQ waiver/exception is more appropriate.

Do NOT assume that a Sonatype "Reachable" label means the vulnerable functionality is actually exploitable.

Verify the application code and runtime configuration.

============================================================
PHASE 4 — VERSION REMEDIATION RULES
============================================================

When a vulnerability has a confirmed fixed version:

1. Determine the newest appropriate FIXED version.
2. Verify compatibility with Spring Boot 3.5.16.
3. Check related modules.
4. Check dependency convergence.
5. Upgrade only what is necessary.
6. Do not upgrade unrelated dependencies.

Example:

If:

    Hibernate 6.6.53.Final

is affected and:

    Hibernate 6.6.57.Final

is confirmed fixed and compatible with Spring Boot 3.5.16,

then upgrade Hibernate to 6.6.57.Final.

But do not jump to Hibernate 7.x.

============================================================
DO NOT INVENT VERSIONS
============================================================

This is extremely important.

NEVER:

- Invent a fixed version.
- Assume a version exists.
- Use an unreleased version.
- Use a version just because it is newer.
- Downgrade randomly.
- Override a dependency without checking compatibility.

If Sonatype says a vulnerability exists but no confirmed fixed version is available:

DO NOT randomly change the dependency.

Instead:

1. Investigate applicability.
2. Investigate reachability.
3. Check official upstream information if available.
4. Check whether mitigation exists.
5. Determine whether the dependency is test-only.
6. Determine whether configuration can eliminate the exposure.
7. If there is no safe technical fix, report that an IQ waiver/exception may be required.

============================================================
TRANSITIVE DEPENDENCIES
============================================================

For transitive dependencies, prefer fixing the dependency that introduces the vulnerable component when practical.

For example:

    Application
        |
        +-- Dependency A
              |
              +-- Vulnerable Dependency B

Determine whether:

- Dependency A has a newer compatible version.
- Dependency B can safely be managed through dependencyManagement.
- Spring Boot 3.5.16's dependency management already controls the version.
- Overriding B independently could create incompatibility.

Do not blindly override transitive dependencies.

============================================================
JACKSON RULES
============================================================

Pay particular attention to Jackson vulnerabilities.

If multiple Jackson modules are present, verify that they remain aligned.

Check:

    jackson-core
    jackson-databind
    jackson-annotations
    jackson-dataformat-*
    jackson-module-*
    jackson-datatype-*

Do not upgrade only jackson-databind if that creates an inconsistent Jackson stack.

Determine the official released fixed version.

Do not use an unreleased version.

============================================================
HIBERNATE RULES
============================================================

Keep Hibernate compatible with Spring Boot 3.5.16.

Do not upgrade to Hibernate 7.x.

If Hibernate ORM modules are explicitly versioned, inspect:

    hibernate-core
    hibernate-envers
    hibernate-jcache
    hibernate-community-dialects
    other org.hibernate.orm modules

Make sure incompatible versions are not mixed.

============================================================
LOG4J RULES
============================================================

For Log4j findings, do not blindly change the version.

Inspect the repository for:

    FilteredObjectInputStream
    LogEventProxy
    ObjectInputStream
    TcpSocketServer
    UdpSocketServer
    SerializedLayout

Also inspect:

    log4j2.xml
    log4j2-spring.xml
    log4j.properties

Determine whether the application actually:

- deserializes Log4j LogEvent objects,
- accepts serialized Log4j events,
- exposes a network receiver,
- accepts untrusted input,
- uses the affected functionality.

If the vulnerability is not applicable to the actual application, document the evidence and recommend an IQ waiver rather than making an unnecessary dependency change.

============================================================
H2 RULES
============================================================

Determine whether H2 is:

- production runtime,
- development only,
- integration-test only,
- unit-test only.

Run:

    mvn dependency:tree -Dincludes=com.h2database:h2

If H2 is only used for testing, assess whether it can safely be declared with:

    <scope>test</scope>

Do not downgrade H2 merely because Sonatype reports an old advisory.

Investigate the actual advisory and affected functionality.

============================================================
FLYWAY RULES
============================================================

For Flyway vulnerabilities:

- Identify the exact Sonatype advisory.
- Determine affected versions.
- Determine fixed versions.
- Check compatibility with Spring Boot 3.5.16.
- Check whether Flyway is used at application startup.
- Check Flyway database migration compatibility.

Do not blindly upgrade across major versions.

============================================================
NETTY RULES
============================================================

Inspect the complete Netty dependency graph.

Verify all relevant modules.

For example:

    netty-common
    netty-buffer
    netty-transport
    netty-codec
    netty-codec-http
    netty-handler
    netty-resolver
    netty-handler-proxy
    reactor-netty modules

Prefer Netty BOM/dependency management when appropriate.

Do not leave mixed Netty versions unless there is a documented reason.

============================================================
LICENSE FINDINGS
============================================================

Separate license findings from security vulnerabilities.

For example:

    License-Not-Assigned
    LGPL
    GPL
    EDL
    other license-policy findings

Do not change a dependency version merely because its license has not been mapped.

Determine:

- actual license,
- organizational policy,
- whether the license is approved,
- whether Sonatype mapping is missing,
- whether legal approval is required.

If the issue is purely license-policy related, classify it separately.

============================================================
COMPONENT-UNKNOWN FINDINGS
============================================================

Component-Unknown does NOT automatically mean vulnerable.

For each Component-Unknown finding determine:

- Maven coordinates
- JAR name
- version
- package metadata
- PURL if available
- whether the component can be identified automatically
- whether Sonatype IQ component mapping is required

Do not replace a component simply because Sonatype cannot identify it.

============================================================
ORGANIZATION-SPECIFIC POLICIES
============================================================

Some findings may come from organization-specific policies such as:

    CTO Warning Policy
    License-Not-Assigned
    Component-Unknown
    Security policy

Do not assume every policy violation requires a code change.

Classify every finding as one of:

A. Security vulnerability
B. License policy
C. Organization policy
D. Component identification
E. Configuration/exploitability issue
F. False positive/non-applicable
G. Requires waiver/exception
H. Requires user input

============================================================
PHASE 5 — ASK QUESTIONS ONLY WHEN REQUIRED
============================================================

You are allowed and expected to ask me questions when required.

However, do NOT ask unnecessary questions.

First inspect:

- PDF
- pom.xml
- dependency tree
- source code
- configuration
- tests

Only ask me if the required information cannot be determined safely from the repository and Sonatype PDF.

Examples of questions you SHOULD ask:

1. A dependency has multiple possible fixed versions and compatibility cannot be determined.
2. A vulnerability depends on a business/runtime configuration that is not available in the repository.
3. A dependency may be production or test-only, but repository evidence is insufficient.
4. Sonatype recommends a version that conflicts with Spring Boot 3.5.16.
5. A code change could alter business behavior.
6. Removing a dependency could break an unknown external integration.
7. A license decision requires organizational/legal approval.
8. A waiver/exception is the only technically safe option.
9. A vulnerability requires knowledge of the production deployment topology.
10. You need me to choose between two technically valid remediation approaches.

When asking a question:

- Explain exactly why the information is required.
- Give the relevant finding/component.
- Explain the possible choices.
- Recommend the safest technical option if appropriate.
- Do not make assumptions.

Example:

"Sonatype reports CVE-XXXX against component X. The repository does not show whether feature Y is enabled in production. The remediation differs depending on this configuration.

Please confirm:
A. Y is enabled in production
B. Y is disabled in production
C. I should treat Y as enabled until confirmed."

============================================================
PHASE 6 — CREATE A REMEDIATION PLAN BEFORE EDITING
============================================================

Before changing files, produce a table like:

| # | Component | Current | Finding | Severity | Direct/Transitive | Fixed Version | Action | Reason |
|---|---|---|---|---|---|---|---|---|

For each finding classify:

    FIX
    CONFIGURATION CHANGE
    CODE CHANGE
    TEST-SCOPE CHANGE
    WAIVER REQUIRED
    LICENSE ACTION
    COMPONENT MAPPING
    NO ACTION / NOT APPLICABLE
    NEED USER INPUT

Do not modify files until this analysis is complete.

If all changes are straightforward and supported by repository evidence, proceed.

If any change is potentially breaking or ambiguous, ask me first.

============================================================
PHASE 7 — IMPLEMENT THE REMEDIATION
============================================================

After the remediation plan is established:

1. Modify pom.xml only where necessary.
2. Modify dependencyManagement only where necessary.
3. Modify application configuration only when required.
4. Modify source code only when required to eliminate an actual vulnerability.
5. Preserve existing application behavior.
6. Preserve existing APIs.
7. Preserve existing batch jobs.
8. Preserve existing database behavior.
9. Do not remove functionality merely to clear an IQ finding.
10. Do not change Spring Boot 3.5.16.

Before making destructive changes, explain them.

============================================================
PHASE 8 — VALIDATION
============================================================

After changes run:

    mvn clean verify

Then:

    mvn dependency:tree

Also run focused trees for every affected dependency.

Verify:

- no vulnerable version remains,
- no duplicate conflicting versions remain,
- no dependency convergence problem exists,
- application compiles,
- unit tests pass,
- integration tests pass where available,
- application context starts successfully,
- Flyway migrations still work,
- Hibernate starts correctly,
- Jackson serialization/deserialization tests pass,
- logging configuration still works,
- H2 tests still work,
- Netty/HTTP functionality still works.

============================================================
PHASE 9 — SECURITY REGRESSION CHECK
============================================================

For every security vulnerability fixed, provide:

    Vulnerability
    Before version
    After version
    Why the new version fixes it
    Compatibility verification
    Tests performed
    Remaining risk

Do not state "fixed" unless there is evidence supporting the conclusion.

============================================================
PHASE 10 — FINAL SONATYPE IQ READINESS REPORT
============================================================

At the end produce a complete report:

# Sonatype IQ Remediation Report

## 1. Summary

- Original violations
- Findings analyzed
- Findings fixed
- Findings requiring waiver
- Findings requiring user input
- License findings
- Component-Unknown findings
- Remaining security findings

## 2. Dependency Changes

| Component | Before | After | Reason |
|---|---|---|---|

## 3. Code Changes

| File | Change | Security reason |
|---|---|---|

## 4. Configuration Changes

| File | Change | Reason |
|---|---|---|

## 5. Findings Requiring Waiver

| Advisory | Component | Reason waiver may be required |
|---|---|---|

## 6. Findings Requiring User Input

List each question and why it is required.

## 7. License Findings

Separate security findings from license findings.

## 8. Component-Unknown Findings

List the components that require Sonatype mapping.

## 9. Validation

Include the exact commands executed and their results.

## 10. Remaining Risks

Clearly identify anything that could not be safely remediated.

============================================================
CRITICAL SAFETY RULES
============================================================

1. Spring Boot MUST remain 3.5.16.
2. Never invent a dependency version.
3. Never use an unreleased version.
4. Never downgrade merely to satisfy Sonatype.
5. Never upgrade unrelated dependencies unnecessarily.
6. Never claim a vulnerability is fixed without evidence.
7. Never remove functionality merely to make IQ green.
8. Never modify business logic unless required.
9. Never ignore a high/critical vulnerability without documenting why.
10. Never hide a finding through configuration changes unless the vulnerable functionality is genuinely disabled.
11. Never assume "Reachable" means exploitable without checking the actual code path.
12. Never treat license findings as CVEs.
13. Never treat Component-Unknown as proof of a vulnerability.
14. Keep all changes compatible with Spring Boot 3.5.16.
15. Ask me whenever repository/PDF evidence is insufficient for a safe decision.

============================================================
START HERE
============================================================

Step 1:
Read the entire Sonatype IQ PDF I provided.

Step 2:
Create the complete finding inventory.

Step 3:
Inspect the repository and Maven dependency tree.

Step 4:
Map every IQ finding to the repository.

Step 5:
Create the remediation plan.

Step 6:
If there are ambiguous/high-risk decisions, ask me the necessary questions.

Step 7:
Otherwise implement the safe fixes.

Step 8:
Run all validation tests.

Step 9:
Produce the final Sonatype IQ remediation report.

REMEMBER:

    Spring Boot 3.5.16 MUST NOT CHANGE.

The goal is to make the application as secure and policy-compliant as possible WITHOUT upgrading or downgrading Spring Boot 3.5.16.
```

---

## Recommended Usage

### Step 1 — Open Copilot Agent Mode

Open the `bce-modernization` repository in VS Code/your IDE and select **GitHub Copilot Agent mode**.

### Step 2 — Provide the Sonatype PDF

Attach the exported Sonatype IQ PDF to the Copilot conversation.

The PDF should be the complete report rather than only the summary page.

### Step 3 — Paste the prompt

Paste the complete prompt above.

### Step 4 — Let Copilot analyze first

The agent should initially produce:

```text
Sonatype IQ Finding Inventory
            ↓
Repository Dependency Analysis
            ↓
Finding → Dependency/Code Mapping
            ↓
Remediation Plan
            ↓
Questions, if required
            ↓
Implementation
            ↓
Testing
            ↓
Final IQ Readiness Report
```

### Step 5 — Review before destructive changes

For any change involving:

- dependency removal,
- major-version upgrade,
- application configuration,
- database behavior,
- security behavior,
- business logic,

Copilot should ask for confirmation if the repository does not provide enough evidence.

---

## Most Important Constraint

The final application **must still use:**

```text
Spring Boot 3.5.16
```

The agent must solve the Sonatype findings around that constraint rather than solving them by upgrading Spring Boot.
