# Copilot Agent Prompt — Validate Spring Boot Application Properties Across Environments

## Objective

Analyze the entire repository and validate all Spring Boot `application*.properties` / environment-specific property files.

The goal is to ensure that:

1. All expected configuration properties are present across the required environment property files.
2. No property is accidentally missing from one environment.
3. No unexpected/extra property exists in only one environment without being explicitly justified.
4. Recipient/email-related properties follow the environment-specific rules defined below.
5. The **local-Postgre, default, dev, and QA** environments use the same recipient details.
6. **UAT and PROD** may contain their own real recipient details and must NOT be forced to match the lower environments.
7. Values that are intentionally environment-specific are identified and not incorrectly reported as synchronization issues.
8. The validation is read-only unless I explicitly ask you to fix the files.

---

# Input Parameter

I will provide the expected recipient email value as a parameter:

```text
EXPECTED_LOWER_ENV_RECIPIENT=<EMAIL_ADDRESS>
```

Example:

```text
EXPECTED_LOWER_ENV_RECIPIENT=test@example.com
```

Use this value as the expected recipient email for:

- local-Postgre
- default
- dev
- QA

Do NOT hard-code this value anywhere.

---

# Step 1 — Discover All Application Property Files

Scan the entire repository recursively and identify all relevant Spring Boot property files.

Look for files such as:

```text
application.properties
application-*.properties
application_*.properties
```

Also identify property files located under common directories such as:

```text
src/main/resources/
src/test/resources/
config/
resources/
```

Do not assume that only one directory contains the configuration.

First produce a list similar to:

```text
Discovered property files:

1. application.properties
2. application-local-postgre.properties
3. application-dev.properties
4. application-qa.properties
5. application-uat.properties
6. application-prod.properties
```

If additional property files are found, include them in the analysis and explain whether they are environment-specific, test-only, example/template files, or another type.

---

# Step 2 — Identify the Environment Mapping

Determine which property file corresponds to each environment.

Expected environments:

```text
default
local-Postgre
dev
QA
UAT
prod
```

Do not rely only on filename casing.

For example:

```text
application.properties                 -> default
application-local-postgre.properties  -> local-Postgre
application-dev.properties             -> dev
application-qa.properties              -> QA
application-uat.properties             -> UAT
application-prod.properties            -> prod
```

If the repository uses a different naming convention, detect it and show the mapping before performing validation.

If an expected environment file is missing, report it as:

```text
MISSING ENVIRONMENT FILE
```

Do not create the file.

---

# Step 3 — Parse All Properties Correctly

Read every property file carefully.

Handle:

- comments
- blank lines
- duplicate keys
- multiline values
- escaped characters
- URLs
- passwords/secrets
- placeholders such as `${VARIABLE_NAME}`
- Spring property placeholders
- comma-separated values
- semicolon-separated values
- arrays/lists
- indexed properties
- quoted values
- encrypted/encoded values if present

Treat the property key as the configuration identifier.

For example:

```properties
spring.datasource.url=...
spring.datasource.username=...
spring.datasource.password=...
mail.recipient.to=...
mail.recipient.cc=...
```

The key is:

```text
mail.recipient.to
```

---

# Step 4 — Build a Complete Property Inventory

Create a union of ALL property keys found across all environment files.

For example:

```text
Total unique properties found: 147
```

Then compare every property against every environment.

The comparison must be based on the property KEY, not merely on the line number or order.

---

# Step 5 — Detect Missing Properties

For every property key, determine whether it exists in each environment.

Example:

```text
Property:
spring.batch.job.enabled

default       : PRESENT
local-Postgre : PRESENT
dev           : PRESENT
QA            : MISSING
UAT           : PRESENT
prod          : PRESENT
```

Report this as:

```text
MISSING PROPERTY

Property: spring.batch.job.enabled
Missing from: QA
Present in: default, local-Postgre, dev, UAT, prod
```

Perform this validation for every property.

---

# Step 6 — Detect Extra Properties

Also detect properties that exist only in one or a subset of environments.

Example:

```text
Property:
some.new.configuration

default       : PRESENT
local-Postgre : PRESENT
dev           : MISSING
QA            : MISSING
UAT           : MISSING
prod          : MISSING
```

Report:

```text
ENVIRONMENT-SPECIFIC / POSSIBLY UNEXPECTED PROPERTY

Property: some.new.configuration
Present only in: default, local-Postgre
```

Do NOT automatically declare this a defect.

Classify it as:

```text
POSSIBLY ENVIRONMENT-SPECIFIC
```

and explain where it exists.

---

# Step 7 — Compare Property Values

For properties that exist in multiple environments, compare their values.

However, do NOT assume that every property value must be identical.

Some values are naturally environment-specific, for example:

```text
spring.datasource.url
spring.datasource.username
spring.datasource.password
server.port
external.api.url
azure.storage.account
database.schema
```

Therefore classify differences into:

### Category A — Same value expected

Properties that should normally be synchronized.

### Category B — Environment-specific value expected

Properties such as:

- database URLs
- API URLs
- hostnames
- ports
- credentials
- secrets
- environment identifiers
- Azure resources
- UAT/PROD endpoints

Do not report these simply because their values differ.

### Category C — Suspicious difference

If a property appears to be a common functional configuration but has different values across environments, report it for review.

Example:

```text
Property: batch.email.enabled

default = true
local-Postgre = true
dev = true
QA = false
UAT = true
prod = true
```

Report:

```text
SUSPICIOUS VALUE DIFFERENCE
```

Do not automatically modify it.

---

# Step 8 — Recipient Email Validation

This is a critical validation.

The following environments must use the exact recipient details specified by the parameter:

```text
default
local-Postgre
dev
QA
```

Expected value:

```text
EXPECTED_LOWER_ENV_RECIPIENT
```

For example:

```text
EXPECTED_LOWER_ENV_RECIPIENT=test@example.com
```

Compare all recipient-related properties against this expected value.

---

# Step 9 — Automatically Detect Recipient Properties

Do not assume the recipient property names.

Search for properties whose keys or values indicate email/recipient functionality.

Look for terms such as:

```text
email
emails
mail
recipient
recipients
to
cc
bcc
reply
notification
notify
distribution
distribution-list
distributionList
```

Examples:

```properties
mail.recipient.to=...
mail.recipient.cc=...
notification.email.to=...
notification.email.cc=...
email.recipients=...
```

Identify all likely recipient-related properties.

Show the detected properties before reporting the result.

---

# Step 10 — Lower Environment Recipient Rule

For:

```text
default
local-Postgre
dev
QA
```

the recipient details must match:

```text
EXPECTED_LOWER_ENV_RECIPIENT
```

Example expected:

```text
test@example.com
```

Then validate:

```text
default       -> test@example.com
local-Postgre -> test@example.com
dev           -> test@example.com
QA            -> test@example.com
```

If any differs, report:

```text
RECIPIENT MISMATCH
```

Example:

```text
Property: mail.recipient.to

Expected:
test@example.com

default:
test@example.com       PASS

local-Postgre:
test@example.com       PASS

dev:
dev-team@example.com   FAIL

QA:
test@example.com       PASS
```

Clearly highlight the incorrect environment.

---

# Step 11 — UAT and PROD Recipient Rule

Do NOT compare UAT and PROD recipient values against:

```text
EXPECTED_LOWER_ENV_RECIPIENT
```

UAT and PROD are expected to contain their real recipient details.

Therefore:

```text
UAT  -> validate that recipient property exists and is populated
PROD -> validate that recipient property exists and is populated
```

Do not replace or modify UAT/PROD recipients.

Also do not print sensitive production email addresses unnecessarily in the final report.

Where appropriate, mask them:

```text
p***@company.com
```

or:

```text
<PROD RECIPIENT CONFIGURED>
```

---

# Step 12 — Verify Lower Environment Recipient Consistency

Perform an explicit comparison:

```text
default       <-> local-Postgre
default       <-> dev
default       <-> QA
local-Postgre <-> dev
local-Postgre <-> QA
dev           <-> QA
```

All four must have identical recipient details.

Report any mismatch.

Example:

```text
Recipient Consistency:

default        : PASS
local-Postgre  : PASS
dev            : FAIL
QA             : PASS

Issue:
dev contains a different recipient configuration.
```

---

# Step 13 — Detect Recipient Lists

Recipient configuration may contain multiple email addresses.

Examples:

```properties
mail.to=user1@example.com,user2@example.com
```

or:

```properties
mail.to=user1@example.com;user2@example.com
```

or:

```properties
mail.to=user1@example.com
mail.cc=user2@example.com
mail.bcc=user3@example.com
```

Normalize the values before comparison.

Normalization should include:

1. Trim whitespace.
2. Handle comma/semicolon separators consistently.
3. Compare case-insensitively for email addresses.
4. Preserve the distinction between TO, CC, and BCC.
5. Detect missing recipients.
6. Detect additional recipients.

For example:

```text
user1@example.com,user2@example.com
```

and:

```text
user2@example.com, user1@example.com
```

should be treated as equivalent if the property represents an unordered recipient list.

But:

```text
TO = user1@example.com
CC = user2@example.com
```

must not be treated as identical to:

```text
TO = user2@example.com
CC = user1@example.com
```

---

# Step 14 — Detect Duplicate Properties

Check every file for duplicate property keys.

Example:

```properties
mail.recipient.to=test@example.com

...

mail.recipient.to=another@example.com
```

Report:

```text
DUPLICATE PROPERTY

File:
application-dev.properties

Property:
mail.recipient.to

Occurrences:
Line X
Line Y

Values:
...
```

Do not modify the file.

---

# Step 15 — Detect Empty Properties

Report properties such as:

```properties
mail.recipient.to=
```

or:

```properties
some.configuration=
```

Classify them appropriately:

```text
EMPTY VALUE
```

For recipient properties specifically, treat an empty value as an error/critical configuration issue.

---

# Step 16 — Detect Placeholder Differences

Compare placeholders carefully.

Example:

```properties
some.url=${API_URL}
```

versus:

```properties
some.url=${DEV_API_URL}
```

Report:

```text
PLACEHOLDER DIFFERENCE
```

but recognize that the difference may be intentional for an environment-specific configuration.

Do not automatically mark it as incorrect.

---

# Step 17 — Generate a Property Matrix

Create a comprehensive matrix like:

| Property | Default | Local-Postgre | Dev | QA | UAT | Prod | Status |
|---|---|---|---|---|---|---|---|
| property.a | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | PASS |
| property.b | ✓ | ✓ | MISSING | ✓ | ✓ | ✓ | FAIL |
| property.c | ✓ | ✓ | ✓ | ✓ | ✓ | DIFFERENT | ENV-SPECIFIC |
| mail.recipient.to | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | PASS |

For sensitive values, mask them.

---

# Step 18 — Generate a Missing Property Report

Create a separate section:

```text
## Missing Properties

| Property | Default | Local-Postgre | Dev | QA | UAT | Prod |
|---|---|---|---|---|---|---|
| xxx | ✓ | ✓ | MISSING | ✓ | ✓ | ✓ |
```

---

# Step 19 — Generate a Recipient Validation Report

Create:

```text
## Recipient Validation

Expected Lower Environment Recipient:
<EXPECTED_LOWER_ENV_RECIPIENT>

| Environment | Recipient Configuration | Result |
|---|---|---|
| Default | ... | PASS/FAIL |
| Local-Postgre | ... | PASS/FAIL |
| Dev | ... | PASS/FAIL |
| QA | ... | PASS/FAIL |
| UAT | Configured | PASS/REVIEW |
| Prod | Configured | PASS/REVIEW |
```

Mask actual UAT/PROD addresses.

---

# Step 20 — Generate a Value Difference Report

Report potentially significant value differences:

```text
## Value Differences

Property:
some.configuration

Default:
value1

Local-Postgre:
value1

Dev:
value1

QA:
value2

UAT:
value3

Prod:
value4

Classification:
ENVIRONMENT-SPECIFIC / REVIEW REQUIRED
```

Do not classify something as a defect merely because environments have different values.

---

# Step 21 — Generate Final Summary

At the end, provide a concise summary:

```text
========================================
APPLICATION PROPERTY VALIDATION SUMMARY
========================================

Property files discovered : X
Environments validated     : 6

Total unique properties   : X

Missing properties        : X
Extra properties          : X
Duplicate properties      : X
Empty properties          : X
Value differences         : X

Recipient validation:
Default       : PASS/FAIL
Local-Postgre : PASS/FAIL
Dev           : PASS/FAIL
QA            : PASS/FAIL
UAT           : CONFIGURED/REVIEW
Prod          : CONFIGURED/REVIEW

Overall configuration validation:
PASS / ISSUES FOUND
========================================
```

---

# Step 22 — Important Safety Rules

DO NOT:

- modify any application property file
- delete properties
- add missing properties
- change recipient emails
- change UAT recipients
- change PROD recipients
- change database URLs
- change passwords
- change secrets
- change environment-specific URLs
- reorder properties
- perform automatic cleanup

This task is **READ-ONLY VALIDATION ONLY**.

If changes appear necessary, report them as recommendations.

---

# Step 23 — Important Spring Boot Considerations

Also check whether:

```text
application.properties
```

acts as the common/default configuration and environment-specific files override it.

Consider Spring Boot profile behavior when analyzing configuration.

For example:

```text
application.properties
application-dev.properties
```

Do not automatically report a property as missing from `application-dev.properties` if it is intentionally inherited from `application.properties`.

However, provide TWO separate results:

### Physical File Comparison

Whether the property physically exists in each file.

### Effective Configuration Comparison

Whether the property would be available to that environment through Spring Boot property inheritance/profile resolution.

This distinction is important.

Example:

```text
spring.application.name
```

may exist only in:

```text
application.properties
```

and therefore still be available to DEV.

Report:

```text
Physical:
Only in default

Effective:
Available in DEV through inheritance
```

Do not incorrectly flag this as an effective configuration defect.

---

# Step 24 — Do Not Make Assumptions

If the repository contains custom configuration loading, for example:

```java
@PropertySource
Environment
PropertySources
ConfigData
@ConfigurationProperties
YamlPropertySourceLoader
```

or custom property files, inspect the relevant Java/Kotlin configuration code before determining whether a property is actually used.

If unsure, classify the finding as:

```text
REVIEW REQUIRED
```

instead of declaring it a defect.

---

# Step 25 — Final Deliverable

After completing the scan, provide:

1. Property files discovered
2. Environment-to-file mapping
3. Total property count
4. Missing properties
5. Extra properties
6. Duplicate properties
7. Empty properties
8. Value differences
9. Recipient properties discovered
10. Recipient validation
11. Lower-environment recipient consistency
12. UAT recipient validation
13. PROD recipient validation
14. Physical property comparison
15. Effective Spring Boot configuration comparison
16. Final summary
17. Recommended changes, WITHOUT applying them

Use clear severity levels:

```text
CRITICAL
ERROR
WARNING
REVIEW
PASS
```

Do not assign an overall score or ranking.

---

# Execution

Before starting, ask me for:

```text
EXPECTED_LOWER_ENV_RECIPIENT
```

If I have already supplied it, use the supplied value.

Then scan the repository and perform the complete read-only validation.

Do not modify any files unless I explicitly provide a second instruction asking you to apply fixes.
