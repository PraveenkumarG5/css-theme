# BCE Monthly Jobs — Repository Extraction and Git Patch Generation

## Objective

I have an existing Spring Boot + Spring Batch application repository called `bce-modernization`.

The repository currently contains:

- Daily BCE jobs
- Weekly BCE jobs
- Monthly BCE jobs
- Common/shared batch framework code
- Controllers for job operations
- `DefaultJobParameter`
- Application configuration
- Environment-specific properties
- Readers/processors/writers
- Services
- Repositories
- Utilities
- Resources
- Tests

The client has requested that **all monthly BCE jobs be moved to a completely new Git repository**.

The monthly jobs will be identified from the explicit list provided below.

### Monthly Job List

Replace the example entries below with the actual monthly job list before executing this prompt:

```text
BCEM800P
BCEM801P
BCEM802P
BCEM803P
```

The `BCEM` prefix may be used for discovery, but the **explicit monthly job list is authoritative**.

Do not migrate a job merely because its name starts with `BCEM` if it is not present in the supplied list.

---

# CRITICAL SAFETY RULE

This migration must be performed in two distinct phases.

## Phase 1 — Analyse and create migration patch

Analyse the repository and prepare the monthly-job migration patch and documentation.

## Phase 2 — Cleanup

Only after the patch has been successfully imported into the new repository, built, tested, and reviewed should the monthly implementation be removed from the original repository.

**Do not combine migration and deletion into one uncontrolled operation.**

Do not delete or modify monthly code from the original repository during the initial analysis phase.

---

# 1. Create a Migration Branch

Before making migration changes:

1. Ensure the working tree is clean.
2. Identify the current base branch.
3. Create a dedicated migration branch.

Recommended branch:

```bash
git checkout <base-branch>
git pull
git checkout -b feature/extract-bce-monthly-jobs
```

Do not perform this work directly on `main`, `master`, `develop`, or another protected branch.

---

# 2. Analyse Every Monthly Job

For every job in the supplied monthly-job list, trace its complete dependency tree.

Example:

```text
BCEM800P
   |
   +-- Job configuration
   |
   +-- Job
   |
   +-- Step(s)
   |
   +-- Flow / Decision
   |
   +-- Tasklet / Reader
   |
   +-- Processor
   |
   +-- Writer
   |
   +-- Services
   |
   +-- Repositories
   |
   +-- DTOs / Models
   |
   +-- Mappers
   |
   +-- Utilities
   |
   +-- Constants
   |
   +-- Configuration
   |
   +-- Resources
   |
   +-- Properties
   |
   +-- Tests
   |
   +-- Controllers / APIs
```

Trace both direct and indirect dependencies.

Do not limit analysis to files containing `BCEM` in their name.

A monthly job may depend on generic services, utilities, repositories, configuration classes, database components, resources, or other shared framework code.

---

# 3. Classify Every Dependency

Every discovered source/configuration/resource file must be classified as exactly one of:

```text
MONTHLY_ONLY
SHARED
DAILY_OR_WEEKLY
UNKNOWN
```

## MONTHLY_ONLY

Used exclusively by the supplied monthly jobs.

Action:

- Migrate it to the new monthly repository.
- Include it in the migration patch.

## SHARED

Used by monthly jobs and also by Daily/Weekly jobs.

Action:

- Do NOT blindly move it.
- Do NOT delete it from the original repository.
- Determine the safest way for the new repository to use the functionality.
- If duplication is unavoidable, document why.
- Prefer a clean shared-library approach only if it is genuinely justified and practical.

## DAILY_OR_WEEKLY

Required by Daily or Weekly jobs.

Action:

- Keep it in the original repository.
- Do not migrate it merely because a monthly job also references it.

## UNKNOWN

If usage cannot be determined confidently:

- Do not guess.
- Do not move or delete it automatically.
- Add it to the manual-review section.

---

# 4. Controllers and REST Endpoints

The new monthly repository must contain the controller/runtime functionality required to operate monthly jobs.

Identify the existing implementation for:

```text
START
STOP
RESTART
STATUS
```

Determine whether the controller implementation is:

- Monthly-specific
- Shared with Daily/Weekly
- Generic framework code

If it is shared, do not blindly move the entire controller.

Ensure the new monthly repository provides the required monthly operations while preserving Daily/Weekly functionality in the original repository.

Preserve existing endpoint behaviour, request/response contracts, error handling, and security behaviour unless a change is explicitly required.

---

# 5. DefaultJobParameter

Find `DefaultJobParameter` and analyse:

1. Where it is defined.
2. Which jobs use it.
3. Which services/configurations use it.
4. Whether it is monthly-only.
5. Whether Daily/Weekly jobs also use it.
6. Which properties it depends on.

If monthly-only:

- Include it in the migration.

If shared:

- Keep the original implementation required by Daily/Weekly.
- Ensure the new monthly repository has everything required for monthly execution.
- Do not create unnecessary duplicate implementations.

Document the final decision.

---

# 6. Application Properties and Configuration

This is a mandatory part of the migration.

The patch must include **all configuration required by the monthly jobs**.

Inspect all relevant configuration files, including but not limited to:

```text
src/main/resources/application.properties
src/main/resources/application.yml
src/main/resources/application.yaml
src/main/resources/application-*.properties
src/main/resources/application-*.yml
src/main/resources/application-*.yaml
```

Pay particular attention to:

```text
application.properties
application-default.properties
application-local-postgres.properties
application-dev.properties
application-qa.properties
application-uat.properties
application-prod.properties
```

Also inspect:

```text
bootstrap.properties
bootstrap.yml
custom property files
custom YAML files
@ConfigurationProperties
@Value
Environment.getProperty()
PropertyResolver
Spring configuration classes
```

Search the complete repository for property usage.

---

# 7. Property Classification

For every property referenced by monthly jobs, classify it as:

```text
MONTHLY_ONLY
SHARED
DAILY_OR_WEEKLY
UNKNOWN
```

Do not blindly copy complete application property files.

For example:

```properties
bce.monthly.BCEM800P.input-path=...
bce.monthly.BCEM800P.output-path=...
bce.monthly.BCEM800P.archive-path=...
```

If these are used only by monthly jobs, they should be migrated.

If a property is shared, document the dependency and ensure the new repository has the configuration required to run.

If a property belongs only to Daily/Weekly functionality, keep it in the original repository.

---

# 8. Analyse Every Environment

Perform property analysis for every environment available in the repository, such as:

```text
local
default
local-postgres
dev
qa
uat
prod
```

Do not assume that a property present in one environment exists in another.

Create a property migration matrix.

Example:

| Property | Local | Default | Dev | QA | UAT | Prod | Classification |
|---|---|---|---|---|---|---|---|
| `bce.monthly.BCEM800P.input-path` | Yes | Yes | Yes | Yes | Yes | Yes | MONTHLY_ONLY |
| `bce.monthly.BCEM800P.output-path` | Yes | Yes | Yes | Yes | Yes | Yes | MONTHLY_ONLY |
| `spring.datasource.url` | Yes | Yes | Yes | Yes | Yes | Yes | SHARED |
| `bce.daily.some-property` | Yes | Yes | Yes | Yes | Yes | Yes | DAILY_OR_WEEKLY |

Identify and report:

- Missing properties
- Environment-specific properties
- Duplicate properties
- Conflicting property values
- Monthly properties missing in an environment
- Properties that appear unused
- Properties whose ownership is unclear

Do not modify property values merely to make environments consistent.

---

# 9. Secrets and Credentials

Never include real secrets in the patch.

Do not copy:

- Passwords
- API keys
- Access tokens
- Client secrets
- Private keys
- Database credentials
- Secret certificates
- Other sensitive credentials

Preserve the existing externalized configuration pattern.

For example:

```properties
external.service.password=${EXTERNAL_SERVICE_PASSWORD}
```

If a secret is currently hardcoded, flag it for manual remediation rather than exposing it in the migration patch.

---

# 10. Resources

Analyse:

```text
src/main/resources
src/test/resources
```

Identify all resources required by monthly jobs, including:

- SQL files
- JSON
- XML
- CSV
- Templates
- Schemas
- Mapping files
- Copybooks
- Control files
- Test input files
- Test output files
- Monthly-specific configuration files
- Other runtime resources

Classify every resource as:

```text
MONTHLY_ONLY
SHARED
DAILY_OR_WEEKLY
UNKNOWN
```

Migrate monthly-only resources.

Do not delete shared resources.

---

# 11. Tests

Identify all tests related to monthly jobs.

Include:

- Unit tests
- Integration tests
- Spring Batch tests
- Controller tests
- Repository tests
- End-to-end tests
- Test resources
- Test configuration

Migrate the tests required to validate the monthly repository.

Do not migrate tests that exclusively validate Daily/Weekly functionality.

---

# 12. Maven / Gradle Dependencies

Analyse:

```text
pom.xml
build.gradle
build.gradle.kts
```

Determine:

- Dependencies required by monthly jobs
- Dependencies required only by Daily/Weekly jobs
- Dependencies shared by both
- Plugin/configuration changes required for the new repository

Do not remove dependencies based on superficial analysis.

Only classify a dependency as removable after verifying all usages.

---

# 13. Spring Configuration

Inspect all relevant Spring configuration, including:

```text
@Configuration
@ConfigurationProperties
@Bean
@ComponentScan
@EnableBatchProcessing
@EnableScheduling
```

and related configuration.

Classify configuration as:

```text
MONTHLY_ONLY
SHARED
DAILY_OR_WEEKLY
UNKNOWN
```

Pay particular attention to:

- Job configuration
- Batch infrastructure
- Job launcher
- Job repository
- Job explorer
- Transaction management
- DataSource
- Scheduling
- REST configuration
- Security configuration
- External service configuration
- Component scanning
- Job registration

The new repository must be independently buildable and runnable.

---

# 14. Create Migration Manifest

Create:

```text
MONTHLY_MIGRATION_MANIFEST.md
```

Include:

## Monthly Jobs

Complete list of migrated monthly jobs.

## Java Files

All Java/Kotlin source files to migrate.

## Controllers

Monthly controller/runtime functionality.

## DefaultJobParameter

Migration decision and dependencies.

## Configuration

All configuration classes and files.

## Properties

All monthly-related properties by environment.

## Resources

All migrated resources.

## Tests

All migrated tests.

## Dependencies

Required Maven/Gradle dependencies.

## Shared Code

Every shared dependency and how it will be handled.

## Files Not to Move

Explicitly list Daily/Weekly/shared files that must remain in the original repository.

---

# 15. Create Dependency Graph

Create:

```text
MONTHLY_DEPENDENCY_GRAPH.md
```

Show relationships such as:

```text
BCEM800P
 |
 +--> MonthlyJobConfiguration
 |       |
 |       +--> MonthlyService
 |              |
 |              +--> MonthlyRepository
 |
 +--> MonthlyReader
 |
 +--> MonthlyProcessor
 |
 +--> MonthlyWriter
 |
 +--> DefaultJobParameter
 |
 +--> MonthlyController
 |
 +--> application-dev.properties
 |
 +--> monthly SQL/resource files
```

Clearly identify shared dependencies.

---

# 16. Create Property Migration Report

Create:

```text
MONTHLY_PROPERTY_MIGRATION.md
```

Include:

1. All monthly-related properties.
2. Environment in which each property exists.
3. Property classification.
4. Property usage location.
5. Whether it is migrated.
6. Whether it is shared.
7. Missing properties.
8. Conflicting values.
9. Manual-review items.
10. Secret/external configuration considerations.

The report must make it possible for a reviewer to understand exactly which properties moved to the new repository.

---

# 17. Create Shared Code Analysis

Create:

```text
MONTHLY_SHARED_CODE_ANALYSIS.md
```

For every shared class/configuration/resource, document:

```text
File
Used by Monthly
Used by Daily
Used by Weekly
Classification
Migration decision
Reason
```

Do not make assumptions.

---

# 18. Perform Migration on a Dedicated Branch

Once analysis is complete, make the migration changes on:

```text
feature/extract-bce-monthly-jobs
```

Prefer logical Git commits.

Recommended commits:

```text
1. Analyse BCE monthly job dependencies
2. Extract BCEM monthly job implementation
3. Extract monthly configuration and properties
4. Extract monthly resources
5. Extract monthly controllers and runtime support
6. Extract/update monthly tests
7. Add migration documentation
```

Do not include destructive cleanup of the original repository in these migration commits unless explicitly instructed.

---

# 19. Generate Git Patch

The preferred approach is to preserve logical Git commits and generate patches using Git.

After migration changes are complete:

```bash
git format-patch <base-commit>..HEAD
```

Store the generated patches under:

```text
migration-patch/
```

The patch must preserve:

- Java/Kotlin source
- Package structure
- Directory structure
- Controllers
- `DefaultJobParameter` where required
- Configuration
- Environment-specific properties
- Resources
- Tests
- Maven/Gradle changes
- Required supporting code

Do not include secrets.

---

# 20. Create Patch README

Create:

```text
migration-patch/README.md
```

Document:

1. Source repository.
2. Source branch.
3. Migration branch.
4. Base commit.
5. Monthly jobs included.
6. Number of files migrated.
7. Number of properties migrated.
8. Number of resources migrated.
9. Shared dependencies.
10. Manual-review items.
11. Target repository expectations.
12. Exact patch import commands.
13. Conflict-resolution instructions.
14. Validation commands.
15. Rollback instructions.

Example import flow:

```bash
git clone <new-repository-url>
cd bce-modernization-monthly

git checkout -b feature/import-bce-monthly-jobs

git am /path/to/migration-patch/*.patch
```

---

# 21. Validate the New Repository

The new monthly repository must be independently buildable.

Run:

```bash
mvn clean test
mvn clean verify
```

or the equivalent Gradle commands.

Verify:

- All monthly jobs are discovered.
- Spring Batch configuration starts correctly.
- Required properties are available.
- Required resources are available.
- Controllers are available.
- `START` works.
- `STOP` works.
- `RESTART` works.
- `STATUS` works.
- Monthly tests pass.

Where practical, validate every supplied monthly job.

---

# 22. Regression Validation for Original Repository

Before removing monthly code, verify that the original repository still supports:

- Daily jobs
- Weekly jobs
- Shared framework functionality
- Shared configuration
- Shared properties
- Shared resources
- Existing controllers
- Existing tests

Search for:

- Broken imports
- Missing beans
- Missing properties
- Missing resources
- Broken component scanning
- Missing dependencies
- References to monthly classes
- Broken tests

---

# 23. Cleanup Patch for Original Repository

Only after the new monthly repository has successfully built and passed validation should a separate cleanup commit/patch be created.

The cleanup must remove only:

- Monthly job implementations
- Monthly-only configuration
- Monthly-only properties
- Monthly-only resources
- Monthly-only tests
- Monthly-only controllers
- Monthly-only services
- Monthly-only repositories
- Monthly-only dependencies

Do NOT remove:

- Daily code
- Weekly code
- Shared framework code
- Shared utilities
- Shared configuration
- Shared properties
- Shared resources

After cleanup:

```bash
mvn clean test
mvn clean verify
```

must pass in the original repository.

---

# 24. Final Validation Report

Create:

```text
MONTHLY_MIGRATION_VALIDATION.md
```

Include:

```text
Monthly jobs migrated:
<list>

Monthly Java/Kotlin files migrated:
<count>

Monthly properties migrated:
<count>

Monthly resources migrated:
<count>

Monthly tests migrated:
<count>

Monthly controllers:
<list>

DefaultJobParameter:
<migration decision>

Shared files retained:
<list>

Daily/Weekly files intentionally retained:
<list>

Potential manual-review items:
<list>

New repository build:
PASS / FAIL

New repository tests:
PASS / FAIL

Monthly endpoint validation:
PASS / FAIL

Original repository regression build:
PASS / FAIL

Original repository regression tests:
PASS / FAIL
```

---

# 25. Final Deliverables

The migration work must produce:

```text
MONTHLY_MIGRATION_MANIFEST.md
MONTHLY_DEPENDENCY_GRAPH.md
MONTHLY_PROPERTY_MIGRATION.md
MONTHLY_SHARED_CODE_ANALYSIS.md
MONTHLY_MIGRATION_VALIDATION.md

migration-patch/
├── README.md
├── *.patch
└── optional checksum/metadata files if useful
```

The patch must be suitable for applying to a **brand-new Git repository**.

---

# 26. Final Rules

These rules are mandatory:

1. The supplied monthly job list is authoritative.
2. Do not migrate jobs outside the supplied list without explicitly flagging them.
3. Do not blindly move shared code.
4. Do not blindly copy complete property files.
5. Include all monthly-job-related properties required by every supported environment.
6. Include monthly controllers for START, STOP, RESTART, and STATUS.
7. Include `DefaultJobParameter` if required by monthly functionality.
8. Include monthly resources.
9. Include monthly tests.
10. Do not expose secrets.
11. Do not delete Daily/Weekly code.
12. Do not delete shared code.
13. Do not modify property values without justification.
14. Do not perform destructive cleanup during the initial migration phase.
15. The new repository must build independently.
16. The original repository must continue to build after monthly cleanup.
17. Preserve Git history through logical commits wherever practical.
18. Generate a Git-format patch that can be applied to a brand-new repository.
19. Clearly document every shared dependency.
20. Clearly report anything that requires manual review.
21. Never guess when dependency ownership is unclear.
22. Before destructive cleanup, stop and report unresolved `UNKNOWN` dependencies or validation failures.

## Execution Order

Follow this exact order:

```text
1. Read monthly job list
        ↓
2. Create migration branch
        ↓
3. Analyse jobs
        ↓
4. Trace dependencies
        ↓
5. Analyse controllers
        ↓
6. Analyse DefaultJobParameter
        ↓
7. Analyse all properties/configuration
        ↓
8. Analyse resources
        ↓
9. Analyse tests
        ↓
10. Analyse dependencies
        ↓
11. Classify shared code
        ↓
12. Generate migration reports
        ↓
13. Extract monthly implementation
        ↓
14. Create logical Git commits
        ↓
15. Generate git format-patch
        ↓
16. Validate migration patch
        ↓
17. Import patch into new repository
        ↓
18. Build/test new repository
        ↓
19. Validate monthly endpoints/jobs
        ↓
20. Only then create cleanup commit for old repository
        ↓
21. Build/test old repository
        ↓
22. Produce final validation report
```

**Start with analysis and reporting. Do not delete anything from the original repository until the new repository has been successfully validated.**
