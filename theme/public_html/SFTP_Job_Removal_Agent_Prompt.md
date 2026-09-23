# Prompt — Safely Remove Specified SFTP Jobs and Their Associated Code

You are working on a **Spring Boot + Spring Batch** application that was created as part of modernizing a Mainframe application.

The application contains multiple batch jobs, including some **SFTP/FTP-related jobs**.

I will provide you with a list of **specific SFTP job names that must be completely removed from the application**.

## SFTP Jobs to Remove

Use ONLY the following list as the source of truth for which jobs must be removed:

```text
[PASTE THE LIST OF SFTP JOB NAMES HERE]
```

---

# Primary Objective

For **ONLY the SFTP jobs listed above**, safely remove:

- The job definitions
- Their Spring Batch configuration
- Their job/step/tasklet implementations
- Their SFTP/FTP-specific processing code
- Their associated services
- Their associated processors
- Their associated readers/writers
- Their associated DTOs/models/entities, if they are used exclusively by the removed jobs
- Their associated utilities/helpers, if they are used exclusively by the removed jobs
- Their associated configuration/properties
- Their associated XML/JSON/YAML/configuration entries
- Their associated scheduler/trigger configuration
- Their associated listeners
- Their associated validators
- Their associated mappers
- Their associated repository/DAO code, if used exclusively by the removed jobs
- Their associated generated code/discussions/artifacts, if they are part of the implementation and are no longer required
- Their associated tests and test data
- Their associated resources and sample files
- Their associated logging/metrics configuration
- Their associated dependency/configuration entries, **only if they are no longer required anywhere else**
- Any dead code that becomes unused **only as a direct consequence of removing these specified jobs**

The final codebase should contain **none of the specified SFTP jobs or their job-specific implementation**.

---

# VERY IMPORTANT: Scope Restriction

This is a **targeted deletion task**, NOT a general cleanup or refactoring task.

### DO NOT:

- Remove any job that is not explicitly listed above.
- Modify unrelated batch jobs.
- Modify business logic belonging to other jobs.
- Refactor unrelated code.
- Rename unrelated classes/methods/packages.
- Upgrade dependencies.
- Change Spring Boot/Spring Batch versions.
- Change application architecture.
- Change database schemas unless absolutely required and explicitly justified.
- Change shared utilities merely because they are related to SFTP.
- Delete shared classes that are still used by other jobs.
- Remove common configuration that is required by other jobs.
- Remove common FTP/SFTP infrastructure if another job still depends on it.
- Make opportunistic code-quality improvements.
- Change behavior of remaining jobs.

**Preserving all non-SFTP functionality is the highest priority.**

---

# Step 1 — Understand the Application Before Making Changes

Before modifying any files:

1. Inspect the project structure.
2. Identify all Spring Batch jobs.
3. Identify the exact implementation and configuration of every job in the supplied SFTP list.
4. Trace each specified job end-to-end:
   - Job definition
   - Job instance/configuration
   - Steps
   - Tasklets/chunks
   - Readers
   - Processors
   - Writers
   - Services
   - Utilities
   - Repositories/DAOs
   - DTOs/models
   - Mappers
   - Listeners
   - Schedulers/triggers
   - Configuration
   - Properties
   - Resources
   - Tests
5. Search the entire repository for references to the specified jobs and their associated classes.
6. Determine which code is:
   - Used exclusively by the SFTP jobs
   - Shared with other jobs
   - Potentially shared indirectly

Do **not** delete anything until this dependency analysis is complete.

---

# Step 2 — Build a Dependency/Impact Map

For every SFTP job in the supplied list, create an internal dependency map.

For example:

```text
SFTP Job
  ├── Job Configuration
  ├── Step Configuration
  ├── Tasklet
  ├── Service
  ├── SFTP Client
  ├── DTO
  ├── Repository
  ├── Mapper
  ├── Properties
  ├── Scheduler
  ├── Tests
  └── Resources
```

For every candidate file/class/configuration, determine whether it is:

### A. SFTP-job-specific

Used only by the specified SFTP job(s).

→ Safe candidate for removal.

### B. Shared

Used by one or more jobs that are NOT in the supplied SFTP list.

→ **DO NOT DELETE.**

### C. Unclear

Usage cannot be confidently determined.

→ **DO NOT DELETE automatically. Investigate further.**

---

# Step 3 — Protect Other Jobs

Before deleting a shared class, method, configuration, property, dependency, utility, or resource, search for all usages.

For example:

```text
Search all references to:
- class
- interface
- method
- bean
- property
- configuration key
- job name
- step name
- service
- utility
```

If another non-SFTP job depends on it, preserve it.

If a shared class contains SFTP-specific functionality but is also used by another job:

- Do NOT delete the class.
- Do NOT modify it unnecessarily.
- Do NOT change the behavior of the remaining job.

Only remove the SFTP-specific portion if you can prove that the change cannot affect the remaining functionality.

When in doubt, **preserve the existing code**.

---

# Step 4 — Remove the Specified Jobs

After dependency analysis, remove the specified SFTP jobs and their exclusively-owned code.

Pay particular attention to:

## Spring Batch

Check for:

- `@Bean` job definitions
- `JobBuilder`
- `StepBuilder`
- `Tasklet`
- Chunk-oriented steps
- Job parameters
- Job listeners
- Step listeners
- Job launchers
- Job registries
- Job explorers
- Schedulers
- `@Scheduled`
- Conditional job configuration
- Job names stored as strings
- Step names stored as strings
- Job-related constants

## Configuration

Search:

```text
application.properties
application.yml
application-*.properties
application-*.yml
bootstrap configuration
Spring @Configuration classes
XML configuration
environment variables
configuration constants
```

Remove configuration only when it belongs exclusively to the specified SFTP jobs.

## SFTP/FTP

Search for:

```text
SFTP
sftp
FTP
ftp
SessionFactory
DefaultSftpSessionFactory
RemoteFileTemplate
SftpRemoteFileTemplate
SftpInboundFileSynchronizer
SftpInboundFileSynchronizingMessageSource
FTPClient
Apache Mina SSHD
JSch
Spring Integration SFTP
```

Do NOT remove common SFTP/FTP libraries or infrastructure if they are still required by another job.

---

# Step 5 — Dependencies

Review `pom.xml` / `build.gradle`.

Only remove an SFTP/FTP dependency if:

1. It was used exclusively by the specified jobs, AND
2. No remaining application code requires it.

Do NOT remove a dependency merely because it sounds SFTP-related.

After removing a dependency, verify that the application still compiles.

---

# Step 6 — Generated Code / Artifacts / Discussions

Identify generated artifacts, generated source files, generated configurations, documentation, metadata, or other implementation artifacts that exist specifically for the removed SFTP jobs.

Remove them only when they are clearly associated exclusively with the specified jobs.

Do not remove shared generated code or artifacts used by other jobs.

---

# Step 7 — Tests

Remove or update tests only when they are specifically associated with the removed SFTP jobs.

Do NOT modify tests for unrelated jobs.

After the changes, verify that:

- Remaining job tests still compile.
- Remaining job tests still execute.
- No unrelated test behavior has changed.

---

# Step 8 — Search for Orphaned References

After removal, perform a repository-wide search for:

1. Every removed job name.
2. Every removed job's step name.
3. Removed class names.
4. Removed service names.
5. Removed configuration keys.
6. Removed properties.
7. Removed bean names.
8. Removed scheduler references.
9. Removed SFTP-specific constants.
10. Removed resources.

There should be no remaining references to the removed jobs unless the reference is intentionally retained for historical/documentation purposes.

---

# Step 9 — Compile and Validate

Run the appropriate project validation commands.

For Maven, use the command appropriate to the project, for example:

```bash
mvn clean test
```

or:

```bash
mvn clean verify
```

For Gradle:

```bash
./gradlew clean test
```

or:

```bash
./gradlew clean build
```

Use the build system actually used by the project.

Fix compilation/test failures caused by the removal.

Do NOT introduce unrelated fixes just to make the build pass.

---

# Step 10 — Final Safety Review

Before considering the task complete, perform a final impact analysis.

## Removed

- [ ] Every specified SFTP job is removed.
- [ ] Its job configuration is removed.
- [ ] Its exclusively-used implementation is removed.
- [ ] Its exclusively-used services/utilities/models are removed.
- [ ] Its exclusively-used configuration is removed.
- [ ] Its exclusively-used tests/resources are removed.
- [ ] Its exclusively-used dependencies are removed where applicable.
- [ ] No orphaned references remain.

## Preserved

- [ ] Every non-SFTP job remains present.
- [ ] Non-SFTP job configurations remain unchanged.
- [ ] Shared services remain intact.
- [ ] Shared utilities remain intact.
- [ ] Shared configuration remains intact.
- [ ] Shared dependencies remain intact.
- [ ] Database-related functionality for remaining jobs is unchanged.
- [ ] Scheduling for remaining jobs is unchanged.
- [ ] No unrelated refactoring was performed.

---

# Critical Rule

**If there is any uncertainty about whether a class, method, configuration, dependency, resource, or utility is shared, DO NOT DELETE IT.**

Preserve it and report it as a potentially shared component.

The goal is:

> **Maximum removal of the explicitly specified SFTP jobs, with minimum possible impact to the rest of the application.**

This should be treated as a **surgical removal**, not a general cleanup.

---

# Final Response Required

After completing the changes, provide a concise but detailed summary containing:

## 1. Removed Jobs

List every SFTP job that was removed.

## 2. Removed Files/Components

List the major files, classes, configurations, resources, tests, and dependencies removed.

## 3. Preserved Shared Components

List important components that were intentionally retained because they are shared with other jobs.

## 4. Dependency/Impact Analysis

Explain how you verified that the removed code was not required by other jobs.

## 5. Validation

Provide the exact build/test commands executed and their results.

Example:

```text
mvn clean verify
BUILD SUCCESS
```

## 6. Remaining Concerns

Clearly identify anything that could not be safely removed or anything that requires manual review.

## 7. Change Scope

Confirm explicitly:

```text
Only the SFTP jobs supplied in the input list were targeted.
No unrelated jobs were intentionally modified.
Shared code was preserved where required.
```

# Final Principle

**Do not optimize. Do not refactor. Do not modernize. Do not change unrelated code.**

**Analyze → Trace Dependencies → Identify Exclusive Code → Remove → Search for References → Build/Test → Review Impact.**

The priority order is:

1. Protect existing non-SFTP jobs.
2. Remove only the specified SFTP jobs.
3. Remove code exclusively belonging to those jobs.
4. Validate the complete application.
5. Report anything uncertain rather than making assumptions.
