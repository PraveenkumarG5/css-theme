# GitHub Copilot Agent Prompt — Job Property Consistency + Mandatory/Optional Property Contract Validation

## Objective

I have an existing Spring Boot + Spring Batch application that was created as part of a mainframe-to-Java modernization.

The application contains approximately 34+ Spring Batch jobs.

There is already an exposed endpoint named `properties`.

The endpoint accepts a job name as a parameter and returns the list of properties required to start that particular job.

Conceptually:

```text
GET /properties?jobName=<JOB_NAME>
```

The exact endpoint URL, request parameter name, response structure, controller implementation, and property representation MUST be discovered from the existing codebase. Do not assume the example above exactly matches the application.

The implementation must perform two related validations:

1. **Cross-profile property consistency** across:
   - dev
   - qa
   - uat
   - prod

2. **Mandatory vs optional property classification** for every job, with a separate Markdown (`.md`) contract document generated for each job.

---

# PRIMARY GOAL

Create a reusable automated test/utility framework that validates the properties returned by the existing `properties` endpoint for EVERY job across these Spring profiles:

```text
dev
qa
uat
prod
```

For every job, the framework must invoke the properties endpoint using each profile and compare the returned property set.

The fundamental validation is:

```text
For each Job:

    DEV properties
        == QA properties
        == UAT properties
        == PROD properties
```

The comparison must be based on the property names/set, NOT on ordering.

For example:

```text
DEV:
A
B
C

QA:
C
A
B

UAT:
B
C
A

PROD:
A
B
C
```

This must PASS because all four profiles contain the same property set.

However:

```text
DEV:
A
B
C

QA:
A
B
C
D
```

must FAIL.

The failure must clearly identify:

- Job name
- Profile
- Missing properties
- Unexpected/additional properties
- Reference profile
- Complete property sets where useful

---

# IMPORTANT ADDITIONAL REQUIREMENT — MANDATORY VS OPTIONAL PROPERTIES

For every job, create a separate Markdown file that documents:

1. Job name
2. Mandatory properties
3. Optional properties
4. Properties returned by the endpoint
5. Properties found in each environment/profile
6. Any profile differences
7. Validation status

The mandatory/optional classification MUST follow this rule:

### Mandatory properties

The following categories are ALWAYS mandatory:

- Input fields
- Output fields

The implementation must inspect the existing job configuration, endpoint response, DTOs, job parameters, readers, writers, processors, services, configuration classes, and related code to identify which returned properties represent input and output fields.

Do NOT simply assume a property is an input or output based only on its name.

Use the existing application's semantics and implementation to determine this.

If a property clearly represents an input field or output field, classify it as:

```text
MANDATORY
```

### Optional properties

All remaining properties returned by the `properties` endpoint become:

```text
OPTIONAL
```

unless the existing code clearly proves that another property is also mandatory to start or execute the job.

If the codebase provides explicit evidence that a non-input/non-output property is required for successful job execution, flag it for review rather than silently overriding the requested rule.

The final report should clearly distinguish:

```text
Mandatory because:
    Input
    Output
    Other explicitly required dependency, if proven by code

Optional because:
    Returned by properties endpoint but not identified as mandatory
```

Do not invent mandatory properties.

---

# PER-JOB MARKDOWN CONTRACT

Create one Markdown file per job.

Recommended location:

```text
docs/job-property-contracts/
```

Example:

```text
docs/job-property-contracts/
    CUSTOMER_DAILY_JOB.md
    ACCOUNT_DAILY_JOB.md
    PAYMENT_MONTHLY_JOB.md
    CLAIM_WEEKLY_JOB.md
```

Use the actual discovered job names.

Do not invent job names.

Each file should follow a consistent structure similar to:

```markdown
# CUSTOMER_DAILY_JOB — Property Contract

## Overview

| Item | Value |
|---|---|
| Job Name | CUSTOMER_DAILY_JOB |
| Profiles Validated | DEV, QA, UAT, PROD |
| Property Endpoint | `/properties?jobName=CUSTOMER_DAILY_JOB` |
| Overall Status | PASS |

## Mandatory Properties

The following properties are mandatory because they represent
the job's input/output fields or are explicitly required by the
existing implementation.

| Property | Classification | Reason |
|---|---|---|
| customer.input.path | Mandatory | Input |
| customer.output.path | Mandatory | Output |

## Optional Properties

| Property | Classification | Reason |
|---|---|---|
| customer.processing.date | Optional | Not classified as input/output |
| customer.archive.path | Optional | Not classified as input/output |

## Properties Returned by Endpoint

```text
customer.input.path
customer.output.path
customer.processing.date
customer.archive.path
```

## Profile Comparison

### DEV

```text
customer.input.path
customer.output.path
customer.processing.date
customer.archive.path
```

### QA

```text
customer.input.path
customer.output.path
customer.processing.date
customer.archive.path
```

### UAT

```text
customer.input.path
customer.output.path
customer.processing.date
customer.archive.path
```

### PROD

```text
customer.input.path
customer.output.path
customer.processing.date
customer.archive.path
```

## Cross-Profile Validation

| Profile | Status | Missing | Additional |
|---|---|---|---|
| DEV | PASS | - | - |
| QA | PASS | - | - |
| UAT | PASS | - | - |
| PROD | PASS | - | - |

## Validation Notes

Document any assumptions, ambiguities, or properties requiring developer confirmation.
```

The exact format may be improved if the repository has an existing documentation convention.

---

# IMPORTANT — DO NOT HARDCODE THE MANDATORY/OPTIONAL LIST

Do NOT create a manually guessed list like:

```text
CUSTOMER_DAILY_JOB:
    mandatory:
        A
        B
    optional:
        C
        D
```

Instead:

1. Discover the job.
2. Invoke/analyze the existing `properties` endpoint.
3. Inspect the implementation that determines input/output properties.
4. Identify input fields.
5. Identify output fields.
6. Classify those as mandatory.
7. Classify remaining endpoint properties as optional unless explicit code evidence proves otherwise.
8. Generate the Markdown contract.

The generated `.md` files become a human-readable, reviewable contract.

---

# FIRST ANALYZE THE EXISTING CODEBASE

Before modifying anything, inspect the repository thoroughly.

Identify:

1. The existing `properties` endpoint.
2. Its controller.
3. Its service implementation.
4. Its request parameter.
5. Its response DTO/model.
6. How job names are obtained.
7. How the application determines the properties required for a job.
8. How input properties are identified.
9. How output properties are identified.
10. Existing Spring profiles.
11. Existing:
    - `application.properties`
    - `application-dev.properties`
    - `application-qa.properties`
    - `application-uat.properties`
    - `application-prod.properties`
12. Existing test framework.
13. Existing JUnit version.
14. Existing Maven test configuration.
15. Existing integration-test conventions.
16. Existing Testcontainers or WireMock usage, if any.
17. How jobs are registered/configured.
18. Whether the properties endpoint requires authentication.
19. Whether the endpoint can be called internally without starting the full application.
20. Whether job metadata/property definitions already exist somewhere in the code.

Do NOT duplicate existing functionality.

Reuse existing DTOs, services, utilities, configuration and test infrastructure wherever practical.

Do not change production behavior unless absolutely necessary.

---

# PART 1 — DISCOVER ALL JOBS

Create a reliable mechanism to obtain the complete list of Spring Batch jobs.

Do NOT assume job names follow a naming convention such as:

```text
D_*
W_*
M_*
```

The job names may have completely different naming conventions.

Inspect the existing application and determine the safest source of truth.

Possible sources include:

- Spring Batch Job beans
- Job registry
- Existing job configuration
- Existing job metadata
- Existing endpoint
- Existing configuration
- Existing tests

Prefer an existing authoritative source rather than maintaining a second manually duplicated job list.

If there is no suitable existing source, create a test-resource configuration file containing the job names.

For example:

```text
src/test/resources/job-property-validation/jobs.yaml
```

with:

```yaml
jobs:
  - CUSTOMER_DAILY_JOB
  - ACCOUNT_DAILY_JOB
  - POLICY_MONTHLY_JOB
```

Document why the chosen source was selected.

---

# PART 2 — SUPPORT ALL FOUR PROFILES

The validation must cover:

```text
dev
qa
uat
prod
```

The framework must make it obvious which profile is being tested.

Do not hard-code profile-specific property values in Java.

The test must use the application's existing configuration.

Do not copy values from one environment into another.

The objective is to validate the actual profile-specific configuration.

---

# PART 3 — PROPERTY ENDPOINT VALIDATION

For every discovered job:

```text
JOB 1
    -> dev
    -> qa
    -> uat
    -> prod

JOB 2
    -> dev
    -> qa
    -> uat
    -> prod

...

JOB N
    -> dev
    -> qa
    -> uat
    -> prod
```

Call the existing `properties` endpoint for each combination.

Conceptually:

```text
/properties?jobName=JOB1
```

with:

```text
spring.profiles.active=dev
```

then:

```text
spring.profiles.active=qa
```

then:

```text
spring.profiles.active=uat
```

then:

```text
spring.profiles.active=prod
```

The implementation MUST use the existing endpoint implementation rather than recreating its business logic in the test.

If calling the HTTP endpoint directly is practical, use the existing endpoint.

If running four application instances is impractical, determine whether the underlying service can safely be invoked within profile-specific Spring test contexts.

Prefer a maintainable solution over unnecessarily complex infrastructure.

---

# PART 4 — COMPARE PROPERTY SETS

For each job, treat DEV as the baseline/reference profile unless the existing project has a more appropriate canonical profile.

Example:

```text
Job: CUSTOMER_DAILY_JOB

DEV:
customer.input.path
customer.output.path
customer.processing.date

QA:
customer.input.path
customer.output.path
customer.processing.date

UAT:
customer.input.path
customer.output.path
customer.processing.date

PROD:
customer.input.path
customer.output.path
customer.processing.date
```

Result:

```text
PASS
```

---

# PART 5 — DETECT MISSING PROPERTIES

Example:

```text
DEV:
A
B
C

QA:
A
B

UAT:
A
B
C

PROD:
A
B
C
```

The test must fail with something similar to:

```text
Property contract mismatch

Job: CUSTOMER_DAILY_JOB
Reference profile: DEV
Profile: QA

Missing properties:
  C

Additional properties:
  none
```

---

# PART 6 — DETECT ADDITIONAL PROPERTIES

Example:

```text
DEV:
A
B
C

QA:
A
B
C
D
```

Failure:

```text
Job: CUSTOMER_DAILY_JOB
Profile: QA

Missing properties:
  none

Additional properties:
  D
```

---

# PART 7 — DETECT BOTH DIFFERENCES

Example:

```text
DEV:
A
B
C

QA:
A
B
D
E
```

Failure:

```text
Job: CUSTOMER_DAILY_JOB
Profile: QA

Missing compared with DEV:
  C

Additional compared with DEV:
  D
  E
```

---

# PART 8 — ORDER MUST NOT MATTER

The following must PASS:

```text
DEV:
A
B
C

QA:
C
B
A
```

Convert the endpoint response into a normalized set before comparison.

Also handle duplicates appropriately.

If the endpoint unexpectedly returns duplicate property names, report that separately because it may indicate a defect.

Example:

```text
Duplicate properties returned:

customer.input.path
customer.input.path
```

---

# PART 9 — NORMALIZATION

Create a reusable normalization utility.

It should:

- Ignore property ordering.
- Trim accidental whitespace.
- Ignore empty property names where appropriate.
- Detect duplicates.
- Preserve the original property name for reporting.
- Avoid changing case unless the application's property semantics explicitly require case-insensitive comparison.

Do not blindly convert property names to lowercase.

Spring property names may have case/format semantics that should not be altered without evidence from the existing application.

---

# PART 10 — MANDATORY/OPTIONAL CLASSIFICATION

Implement a reusable classifier.

For each job:

```text
Endpoint property list
        |
        v
Identify input fields
        |
        v
Identify output fields
        |
        v
Mark input + output as MANDATORY
        |
        v
Inspect remaining properties
        |
        v
Mark remaining properties OPTIONAL
```

### Mandatory classification

Input and output fields are mandatory.

Use actual source-code evidence to identify them.

Look for relevant constructs such as:

- Job parameter definitions
- Input DTOs
- Input readers
- Output writers
- File/input configuration
- Output configuration
- Reader/Writer bean definitions
- Existing property metadata
- Existing endpoint implementation
- Job configuration
- `@ConfigurationProperties`
- `@Value`
- `Environment#getProperty`
- Existing property descriptors
- Existing validation annotations

Do not infer merely from names such as `input`, `output`, `source`, or `target` if the code gives stronger evidence.

### Remaining properties

Properties returned by the endpoint that are not input/output properties should be classified as OPTIONAL by default.

However, if the code clearly demonstrates that another property is required to successfully start or execute the job, record that as:

```text
MANDATORY
Reason: Explicitly required by job implementation
```

and flag it in the generated Markdown as an exception to the basic input/output rule.

Do not silently make assumptions.

---

# PART 11 — PROPERTY CONTRACT MARKDOWN FILES

Generate one `.md` file per job.

Location:

```text
docs/job-property-contracts/
```

Example:

```text
docs/job-property-contracts/
    CUSTOMER_DAILY_JOB.md
    ACCOUNT_DAILY_JOB.md
    PAYMENT_MONTHLY_JOB.md
```

Every generated document must contain:

1. Job name
2. Endpoint
3. Mandatory property list
4. Optional property list
5. Reason for mandatory classification
6. Complete endpoint property list
7. DEV property list
8. QA property list
9. UAT property list
10. PROD property list
11. Cross-profile comparison
12. Missing properties
13. Additional properties
14. Duplicate properties, if any
15. Overall validation status
16. Assumptions/ambiguities
17. Developer-confirmation items, if any

Use tables where they improve readability.

---

---

# PART 13 — CENTRAL TEST RESULT

Create a reusable test class rather than creating 34+ almost-identical test classes.

Preferred conceptual structure:

```text
JobPropertyConsistencyTest
        |
        +-- discovers jobs
        |
        +-- loads profiles
        |
        +-- invokes properties endpoint
        |
        +-- normalizes response
        |
        +-- identifies input/output properties
        |
        +-- classifies mandatory/optional
        |
        +-- compares property sets
        |
        +-- generates job Markdown contracts
        |
        +-- produces detailed failures
```

Use JUnit 5 parameterized tests where appropriate.

Avoid generating:

```text
CustomerJobPropertyTest
AccountJobPropertyTest
PaymentJobPropertyTest
...
```

unless the existing architecture genuinely requires separate tests.

---

# PART 14 — TEST EXECUTION

The test should be runnable using the normal Maven test lifecycle.

For example:

```text
mvn test
```

or, if the project separates integration tests:

```text
mvn verify
```

Determine the correct approach from the existing project.

Do not arbitrarily change the Maven lifecycle.

---

# PART 15 — ENVIRONMENT CONFIGURATION

Do not hard-code URLs such as:

```text
https://dev.example.com
https://qa.example.com
```

unless those already exist in the project.

If the application needs external environment URLs, make them configurable.

For example:

```properties
job.property.validation.dev.url=...
job.property.validation.qa.url=...
job.property.validation.uat.url=...
job.property.validation.prod.url=...
```

But first inspect whether the existing project already has suitable configuration.

Do not introduce duplicate configuration unnecessarily.

---

# PART 16 — SECURITY / AUTHENTICATION

Inspect whether the existing `/properties` endpoint requires:

- JWT
- OAuth
- Basic authentication
- API key
- internal authentication
- no authentication

Do not disable application security globally just to make the tests pass.

If authentication is required, use the existing test security mechanism or provide a clean configurable test mechanism.

Never hard-code real production credentials.

Never commit secrets.

---

# PART 17 — FAILURE REPORT

The test failure must be developer-friendly.

For every failure, report:

```text
===========================================================
JOB PROPERTY VALIDATION FAILED
===========================================================

Job:
    CUSTOMER_DAILY_JOB

Reference Profile:
    DEV

Compared Profile:
    QA

Missing Properties:
    customer.processing.date

Additional Properties:
    customer.archive.path

DEV Properties:
    [customer.input.path,
     customer.output.path,
     customer.processing.date]

QA Properties:
    [customer.input.path,
     customer.output.path,
     customer.archive.path]

Mandatory Properties:
    customer.input.path
    customer.output.path

Optional Properties:
    customer.processing.date
    customer.archive.path

===========================================================
```

For a completely successful run, print a concise summary:

```text
===========================================================
JOB PROPERTY VALIDATION SUMMARY
===========================================================

Profiles:
    DEV
    QA
    UAT
    PROD

Total Jobs:
    34

Passed:
    34

Failed:
    0

===========================================================
ALL JOB PROPERTY CONTRACTS ARE CONSISTENT
===========================================================
```

---

# PART 18 — MACHINE-READABLE REPORT

If practical within the existing build setup, also produce a machine-readable report.

Preferred:

```text
target/job-property-validation-report.json
```

Example:

```json
{
  "summary": {
    "totalJobs": 34,
    "passed": 32,
    "failed": 2
  },
  "failures": [
    {
      "jobName": "CUSTOMER_DAILY_JOB",
      "referenceProfile": "dev",
      "profile": "qa",
      "missingProperties": [
        "customer.processing.date"
      ],
      "additionalProperties": [
        "customer.archive.path"
      ],
      "mandatoryProperties": [
        "customer.input.path",
        "customer.output.path"
      ],
      "optionalProperties": [
        "customer.processing.date",
        "customer.archive.path"
      ]
    }
  ]
}
```

Do this only if it fits naturally with the existing project.

Do not add unnecessary third-party dependencies.

---

# PART 19 — VALIDATE THAT THE PROPERTIES ENDPOINT RETURNS ALL CONFIGURED JOB PROPERTIES

Add an additional, independent validation:

> For each job and each environment/profile, verify that the existing `properties` endpoint returns ALL properties that are configured/required for that job in the corresponding Spring application configuration.

This is different from simply comparing DEV vs QA vs UAT vs PROD.

The framework must validate both directions:

```text
Application configuration
        |
        |  properties configured for this job/profile
        v
Expected job property set
        |
        | compare
        v
/properties?jobName=<JOB_NAME>
        |
        v
Actual property set returned by endpoint
```

The validation must fail if the endpoint omits a property that is actually configured/required for that job.

## Important Scope Rule

Do NOT compare the endpoint response against every property in the entire `application-{profile}.properties` file.

The application properties file may contain global/shared properties unrelated to a particular job.

Instead, determine which configured properties belong to the specific job using the application's existing structure and semantics.

Inspect:

- Job configuration classes
- Job-specific prefixes
- `@ConfigurationProperties`
- `@Value`
- `Environment#getProperty`
- Job parameters
- Readers
- Writers
- Processors
- Services
- Tasklets
- Step configuration
- Existing property metadata
- Existing endpoint/service logic
- Any job-to-property mapping already present

Use the strongest available source of truth from the existing application.

Do not invent a job-property relationship based only on naming patterns if the code provides a more reliable mapping.

## Required Comparison

For every combination:

```text
Job × Profile
```

compare:

```text
Configured/required properties for the job
                VS
Properties returned by /properties
```

For example:

```text
CUSTOMER_DAILY_JOB + DEV

Configured job properties:
    customer.input.path
    customer.output.path
    customer.processing.date
    customer.archive.path

Endpoint returns:
    customer.input.path
    customer.output.path
    customer.processing.date

Result:
    FAIL

Missing from endpoint:
    customer.archive.path
```

The test must throw/fail with a clear error.

## Detect Missing Endpoint Properties

Example:

```text
===========================================================
JOB PROPERTY ENDPOINT VALIDATION FAILED
===========================================================

Job:
    CUSTOMER_DAILY_JOB

Profile:
    DEV

Configured/Required Properties:
    customer.input.path
    customer.output.path
    customer.processing.date
    customer.archive.path

Properties Returned by Endpoint:
    customer.input.path
    customer.output.path
    customer.processing.date

Missing From Endpoint:
    customer.archive.path

ERROR:
The /properties endpoint does not return all properties
configured/required for CUSTOMER_DAILY_JOB in DEV.
===========================================================
```

## Detect Unexpected Endpoint Properties

Also compare the opposite direction.

If the endpoint returns:

```text
A
B
C
D
```

but the configured/required job property set is:

```text
A
B
C
```

report:

```text
Unexpected properties returned by endpoint:
    D
```

Do NOT necessarily fail this condition automatically if the application's endpoint intentionally returns derived/default/shared properties.

First determine the semantics from the existing implementation.

If the endpoint is intended to return exactly the job's configured/required property set, fail on unexpected properties as well.

If it intentionally returns additional properties, document that behavior and distinguish:

```text
Missing required/configured properties = ERROR
Additional endpoint properties = WARNING or INFO
```

Do not silently ignore unexpected properties.

## Empty Configuration

If a job has no identifiable configured properties for a profile:

- Do not automatically treat this as a pass.
- Determine whether the job genuinely requires no properties.
- If the endpoint returns properties but the configuration analysis cannot identify their source, report the situation as an ambiguity requiring developer confirmation.
- Do not invent a property mapping.

## Profile-Specific Configuration

Perform this validation separately for:

```text
DEV
QA
UAT
PROD
```

Example:

```text
CUSTOMER_DAILY_JOB

DEV:
    Configured: A B C
    Endpoint:   A B C
    PASS

QA:
    Configured: A B C
    Endpoint:   A B
    FAIL
    Missing: C

UAT:
    Configured: A B C
    Endpoint:   A B C
    PASS

PROD:
    Configured: A B C D
    Endpoint:   A B C D
    PASS
```

This validation must use the actual profile-specific configuration.

Do not copy DEV configuration into QA/UAT/PROD for testing.

## Relationship With Cross-Profile Validation

Keep these validations separate:

### Validation 1 — Cross-profile consistency

```text
DEV endpoint property set
        ==
QA endpoint property set
        ==
UAT endpoint property set
        ==
PROD endpoint property set
```

### Validation 2 — Configuration-to-endpoint completeness

```text
Configured job properties for DEV
        ==
/properties endpoint result for DEV

Configured job properties for QA
        ==
/properties endpoint result for QA

Configured job properties for UAT
        ==
/properties endpoint result for UAT

Configured job properties for PROD
        ==
/properties endpoint result for PROD
```

A job may pass Validation 1 but fail Validation 2.

Example:

```text
DEV endpoint:
A B

QA endpoint:
A B

UAT endpoint:
A B

PROD endpoint:
A B
```

Cross-profile consistency:

```text
PASS
```

But if the actual PROD configuration contains:

```text
A B C
```

and `C` is a required property for that job, then:

```text
Configuration-to-endpoint completeness:
FAIL
```

This is an important defect and MUST be reported.

## Include This In The Single Markdown Summary

The existing single file:

```text
docs/job-property-validation-summary.md
```

must include the new validation.

For each job, include a table such as:

```markdown
## CUSTOMER_DAILY_JOB

### Property Classification

| Property | Classification | Reason |
|---|---|---|
| customer.input.path | Mandatory | Input |
| customer.output.path | Mandatory | Output |
| customer.processing.date | Optional | Not classified as input/output |

### Profile Validation

| Profile | Cross-Profile | Configured vs Endpoint | Missing From Endpoint | Unexpected From Endpoint |
|---|---|---|---|---|
| DEV | PASS | PASS | - | - |
| QA | PASS | FAIL | customer.processing.date | - |
| UAT | PASS | PASS | - | - |
| PROD | PASS | PASS | - | - |

### Endpoint Completeness Details

#### DEV

**Configured/Required:**

```text
customer.input.path
customer.output.path
customer.processing.date
```

**Returned by Endpoint:**

```text
customer.input.path
customer.output.path
customer.processing.date
```

**Result:** PASS

#### QA

**Configured/Required:**

```text
customer.input.path
customer.output.path
customer.processing.date
```

**Returned by Endpoint:**

```text
customer.input.path
customer.output.path
```

**Result:** FAIL

**Missing from endpoint:**

```text
customer.processing.date
```
```

The summary file should make it immediately obvious which environment has an incomplete endpoint response.

---

# PART 20 — START-ENDPOINT VALIDATION USING PROPERTIES ENDPOINT RESPONSE

Add a third runtime validation for every job and every profile.

There is an existing endpoint exposed by the application to start/trigger a Spring Batch job.

The exact endpoint URL, HTTP method, request structure, request body, headers, authentication, response structure, and required parameters MUST be discovered from the existing codebase.

Do NOT assume the endpoint details.

The validation flow must be:

```text
1. Identify Job
        |
        v
2. Select Profile
        |
        v
3. Call /properties endpoint
        |
        v
4. Read the properties returned by /properties
        |
        v
5. Build the request required by the existing job-start endpoint
        |
        v
6. Call the job-start endpoint
        |
        v
7. Validate the response
        |
        v
8. Confirm the job-start request does NOT fail because of
   missing/invalid/unrecognized properties
```

## Critical Requirement

The job-start validation MUST use the properties returned by the existing `properties` endpoint.

Do not create a separate manually maintained property list for the start request.

Conceptually:

```text
/properties?jobName=CUSTOMER_DAILY_JOB
                |
                v
       Returned properties
                |
                v
       Build start request
                |
                v
/start-job
                |
                v
       Validate response
```

This verifies that the output of the `properties` endpoint is actually sufficient for starting the job.

---

## START-ENDPOINT DISCOVERY

Before implementing the test, inspect the existing application to identify:

1. Job-start endpoint controller.
2. HTTP method.
3. Endpoint path.
4. Request DTO.
5. Request body.
6. Query parameters.
7. Path variables.
8. Required headers.
9. Authentication.
10. Job name parameter.
11. How properties are supplied to the endpoint.
12. Expected successful response.
13. Known failure responses.
14. Error DTO/model.
15. HTTP status codes.
16. Whether the endpoint starts the job synchronously or asynchronously.
17. Whether it returns an execution ID/job execution ID.
18. Whether duplicate job execution parameters are required.
19. Whether a unique run identifier is required.
20. Whether the endpoint has any existing test utilities.

Reuse the existing DTOs, services, controllers, request builders, and test utilities where possible.

Do not duplicate the start-job business logic in the test.

---

# PART 21 — START JOB WITH ENDPOINT-RETURNED PROPERTIES

For every:

```text
Job × Profile
```

perform:

```text
GET /properties?jobName=<JOB_NAME>
        |
        v
Read returned property list
        |
        v
Create start-job request
        |
        v
POST/GET <existing-start-endpoint>
        |
        v
Validate response
```

The exact HTTP method and request format must be discovered from the application.

For example, if the existing endpoint expects:

```json
{
  "jobName": "CUSTOMER_DAILY_JOB",
  "properties": {
    "customer.input.path": "/...",
    "customer.output.path": "/...",
    "customer.processing.date": "..."
  }
}
```

then construct the request using the actual properties returned by `/properties`.

Do not invent request fields.

---

# PART 22 — SUCCESS CRITERIA FOR START-ENDPOINT VALIDATION

The job-start validation must verify that the start endpoint does NOT return an error caused by the properties returned by the properties endpoint.

A successful response could be represented by:

```text
HTTP 200
HTTP 201
HTTP 202
```

or another status explicitly documented by the existing application.

Determine the valid success response from the existing implementation.

Do not assume HTTP 200 is the only successful response.

If the endpoint returns a job execution ID, capture it.

Example:

```text
Job:
    CUSTOMER_DAILY_JOB

Profile:
    DEV

Properties endpoint:
    PASS

Start endpoint:
    PASS

Execution ID:
    123456
```

---

# PART 23 — ERROR RESPONSE VALIDATION

The test MUST fail if the start endpoint returns an application/business validation error caused by the supplied properties.

Examples include:

```text
Missing required property
Invalid property
Unknown property
Property validation failed
Unable to bind property
Could not resolve property
Invalid job parameter
Job could not be started
Job configuration error
```

The exact error patterns must be derived from the existing application's response/error model.

Do not rely only on string matching if structured error information is available.

---

# PART 24 — PROPERTY-RELATED START FAILURE

Example:

```text
Properties endpoint returns:

customer.input.path
customer.output.path
customer.processing.date
```

The test then starts:

```text
CUSTOMER_DAILY_JOB
```

If the start endpoint returns:

```text
HTTP 400

Missing required property:
customer.processing.date
```

the validation MUST FAIL.

The failure should clearly state:

```text
===========================================================
JOB START VALIDATION FAILED
===========================================================

Job:
    CUSTOMER_DAILY_JOB

Profile:
    DEV

Properties Returned by /properties:
    customer.input.path
    customer.output.path
    customer.processing.date

Start Endpoint:
    <actual discovered endpoint>

Response:
    HTTP 400

Error:
    Missing required property: customer.processing.date

Conclusion:
    The properties endpoint did not provide a property set
    sufficient to start the job.
===========================================================
```

---

# PART 25 — NO ERROR EXPECTED

The expected result for a valid job/profile combination is:

```text
Properties endpoint:
    PASS

Configuration-to-endpoint completeness:
    PASS

Start endpoint:
    PASS
```

The test should report:

```text
CUSTOMER_DAILY_JOB | DEV | PASS
```

only when the job-start endpoint accepts the property set without a property/configuration-related error.

---

# PART 26 — AVOID UNCONTROLLED JOB EXECUTION

IMPORTANT:

Before implementing the start-endpoint test, inspect whether calling the endpoint causes the actual business job to execute against real DEV/QA/UAT/PROD systems.

Do NOT blindly trigger real production jobs.

The test must determine whether the existing application provides:

- test mode
- dry-run mode
- validation-only mode
- mock execution
- test profile
- isolated test database
- Testcontainers
- stubbed downstream systems
- safe execution parameters
- dedicated test endpoint
- existing integration-test infrastructure

For DEV/QA/UAT, determine the safest supported execution approach.

For PROD, DO NOT execute a real production job merely to validate configuration unless the existing project explicitly provides a safe non-destructive mechanism.

If no safe mechanism exists for PROD, the test must not trigger a real production job.

Instead, report:

```text
PROD start-endpoint execution:
NOT EXECUTED

Reason:
No safe non-destructive execution mechanism was identified.
```

Do not weaken this safety requirement just to make the test pass.

---

# PART 27 — IDEMPOTENCY AND DUPLICATE JOB EXECUTION

Inspect whether Spring Batch requires unique job parameters for each execution.

For example:

```text
run.id
timestamp
business.date
execution.id
```

If a unique execution parameter is required, use a deterministic/test-safe value or the application's existing test mechanism.

Do not accidentally cause:

```text
JobInstanceAlreadyCompleteException
JobExecutionAlreadyRunningException
```

and classify those as property validation failures unless they are genuinely caused by the property set.

The test must distinguish:

```text
Property validation failure
```

from:

```text
Job execution lifecycle failure
```

Report lifecycle issues separately.

---

# PART 28 — START-ENDPOINT RESPONSE VALIDATION

Validate at minimum:

1. HTTP status.
2. Structured error response, if available.
3. Error code, if available.
4. Error message, if available.
5. Job name, if returned.
6. Execution ID, if returned.
7. Job execution status, if the endpoint provides it.
8. Whether the request was accepted.

If the endpoint is asynchronous and returns:

```text
202 Accepted
```

do not automatically assume the job itself completed successfully.

Determine whether the existing application provides an execution-status endpoint.

If a safe test environment is available and the application exposes execution status, optionally validate that the job reaches the expected initial state without a property-related failure.

Do not wait indefinitely for job completion.

---

# PART 29 — KEEP START VALIDATION SEPARATE FROM JOB BUSINESS VALIDATION

The objective of this test is:

```text
Can the property list returned by /properties
be accepted by the job-start endpoint?
```

It is NOT intended to verify:

- business output
- data correctness
- downstream system results
- complete batch processing
- reconciliation
- performance
- business rules

A job may legitimately fail later because of:

```text
Database unavailable
External service unavailable
Input file unavailable
Downstream system unavailable
Business data issue
```

Those failures should not automatically be classified as a property-contract failure.

The test must identify whether the failure is related to the properties/start request.

---

# PART 30 — SINGLE MARKDOWN SUMMARY UPDATE

Update the existing single Markdown file:

```text
docs/job-property-validation-summary.md
```

For every job, include the start-endpoint validation.

Example:

```markdown
## CUSTOMER_DAILY_JOB

### Property Classification

| Property | Classification | Reason |
|---|---|---|
| customer.input.path | Mandatory | Input |
| customer.output.path | Mandatory | Output |
| customer.processing.date | Optional | Not classified as input/output |

### Validation Summary

| Profile | Cross-Profile | Config vs Endpoint | Start Endpoint | Overall |
|---|---|---|---|---|
| DEV | PASS | PASS | PASS | PASS |
| QA | PASS | PASS | PASS | PASS |
| UAT | PASS | PASS | FAIL | FAIL |
| PROD | PASS | PASS | NOT EXECUTED | REVIEW |

### Start Endpoint Details

#### DEV

**Properties supplied from `/properties`:**

```text
customer.input.path
customer.output.path
customer.processing.date
```

**Start endpoint:** PASS

**HTTP status:** 202

**Execution ID:** 123456

#### QA

**Properties supplied from `/properties`:**

```text
customer.input.path
customer.output.path
customer.processing.date
```

**Start endpoint:** PASS

**HTTP status:** 202

**Execution ID:** 123457

#### UAT

**Properties supplied from `/properties`:**

```text
customer.input.path
customer.output.path
customer.processing.date
```

**Start endpoint:** FAIL

**HTTP status:** 400

**Error:**

```text
Missing required property: customer.processing.date
```

#### PROD

**Start endpoint:** NOT EXECUTED

**Reason:**

```text
No safe non-destructive execution mechanism identified.
```
```

---

# PART 31 — OVERALL VALIDATION MODEL

The complete validation framework should now perform these independent checks:

```text
                         JOB
                          |
             ┌────────────┼────────────┐
             |            |            |
             v            v            v
       Properties      Profile       Source
        Endpoint      Config        Analysis
             |            |            |
             └──────┬─────┴────────────┘
                    |
                    v
        ┌───────────────────────────┐
        │ Validation 1              │
        │ Cross-profile consistency │
        └────────────┬──────────────┘
                     |
                     v
        ┌───────────────────────────┐
        │ Validation 2              │
        │ Config → Endpoint         │
        │ completeness              │
        └────────────┬──────────────┘
                     |
                     v
        ┌───────────────────────────┐
        │ Validation 3              │
        │ Mandatory/Optional        │
        │ classification            │
        └────────────┬──────────────┘
                     |
                     v
        ┌───────────────────────────┐
        │ Validation 4              │
        │ Start endpoint accepts    │
        │ endpoint-returned props  │
        └────────────┬──────────────┘
                     |
                     v
              Final Job Result
```

The overall result should only be `PASS` when all applicable validations pass.

For example:

```text
Cross-profile:
PASS

Configuration → Endpoint:
PASS

Mandatory/Optional classification:
PASS

Start endpoint:
PASS

Overall:
PASS
```

If the PROD start validation is intentionally not executed because no safe execution mechanism exists, show:

```text
PROD:
REVIEW / NOT EXECUTED
```

rather than falsely reporting PASS.

---

# PART 32 — FINAL TEST REPORT COUNTS

The final execution summary must include:

```text
Total jobs discovered:
Total profiles:
Total job/profile combinations:

Cross-profile validation:
    Passed:
    Failed:

Configuration-to-endpoint validation:
    Passed:
    Failed:

Mandatory/optional classification:
    Completed:
    Ambiguous:

Start-endpoint validation:
    Passed:
    Failed:
    Not Executed:

Overall:
    Passed:
    Failed:
    Review:
```

Also report the number of jobs for which the start endpoint could not safely be executed.

# PART 20 — PROPERTY VALUE VALIDATION

The primary requirement is to compare the PROPERTY NAMES returned by the endpoint.

Do NOT require the actual values to be identical between:

```text
DEV
QA
UAT
PROD
```

For example, this is completely valid:

```text
DEV:
customer.input.path=/dev/customer

QA:
customer.input.path=/qa/customer

UAT:
customer.input.path=/uat/customer

PROD:
customer.input.path=/prod/customer
```

The property name is identical, therefore the contract passes.

The test is about:

```text
PROPERTY NAME CONSISTENCY
```

not:

```text
PROPERTY VALUE CONSISTENCY
```

# PART 19 — PROPERTY VALUE VALIDATION

The primary requirement is to compare the PROPERTY NAMES returned by the endpoint.

Do NOT require the actual values to be identical between:

```text
DEV
QA
UAT
PROD
```

For example, this is completely valid:

```text
DEV:
customer.input.path=/dev/customer

QA:
customer.input.path=/qa/customer

UAT:
customer.input.path=/uat/customer

PROD:
customer.input.path=/prod/customer
```

The property name is identical, therefore the contract passes.

The test is about:

```text
PROPERTY NAME CONSISTENCY
```

not:

```text
PROPERTY VALUE CONSISTENCY
```

---

# PART 20 — EMPTY OR INVALID ENDPOINT RESPONSES

Handle these cases explicitly.

### Case 1 — Job does not exist

Fail with:

```text
Job not found:
CUSTOMER_DAILY_JOB
```

### Case 2 — HTTP error

Report:

```text
Job:
CUSTOMER_DAILY_JOB

Profile:
UAT

Endpoint response:
HTTP 500

Property validation could not be completed.
```

### Case 3 — Empty property list

Do not automatically classify this as PASS.

Determine whether an empty property list is valid for that job.

If the existing application provides no indication that it is valid, report it as a validation issue requiring attention.

### Case 4 — Duplicate properties

Report duplicates separately.

---

# PART 21 — DO NOT CHANGE PRODUCTION CODE UNNECESSARILY

The task is primarily a TESTING and VALIDATION framework.

Do not:

- change batch job logic
- change job configuration
- change production property files
- rename jobs
- change endpoint behavior
- remove properties
- add fake properties
- change profile behavior

unless a minimal change is required to make the test framework possible.

If production code must be changed, clearly identify the reason and minimize the change.

---

# PART 22 — OPTIONAL STATIC PROPERTY VALIDATION

After implementing the endpoint consistency validation, investigate whether it is practical to add a second validation layer that checks the actual property files.

For example:

```text
application-dev.properties
application-qa.properties
application-uat.properties
application-prod.properties
```

For each job, determine whether the properties returned by `/properties` actually exist in the corresponding profile configuration.

Example:

```text
Endpoint says CUSTOMER_DAILY_JOB requires:

customer.input.path
customer.output.path
customer.processing.date
```

Then verify that the corresponding profile contains those properties.

This should be a separate validation from the cross-profile property-set comparison.

Do not mix the two concepts.

The final framework should conceptually support:

```text
Validation 1:
Does DEV == QA == UAT == PROD?

Validation 2:
Does each profile actually define the properties required by the job?

Validation 3:
Are the mandatory/optional classifications documented correctly?
```

Only implement Validation 2 if it can be done reliably using the existing Spring configuration structure.

---

# PART 23 — FUTURE CENTRAL PROPERTY CONTRACT

Design the code so that the generated per-job Markdown contracts can later become or feed a machine-readable contract.

For example:

```text
docs/job-property-contracts/
    CUSTOMER_DAILY_JOB.md
    ACCOUNT_MONTHLY_JOB.md
    ...
```

Potential future machine-readable structure:

```yaml
jobs:

  CUSTOMER_DAILY_JOB:

    mandatory:
      - customer.input.path
      - customer.output.path

    optional:
      - customer.processing.date
      - customer.archive.path
```

Do not manually create this YAML with guessed values.

If you generate it, derive it from the actual endpoint and source-code analysis.

---

# PART 24 — CODE QUALITY

Follow the existing project's conventions.

Use:

- JUnit 5
- AssertJ if already available
- Spring Boot test facilities
- Existing HTTP test utilities
- Existing DTOs
- Existing configuration mechanisms

Avoid introducing a new testing framework unless required.

Use clear names such as:

```text
JobPropertyConsistencyTest
JobPropertyValidationService
JobPropertyClassifier
JobPropertyContractGenerator
JobPropertyResponse
PropertySetComparator
JobDiscoveryService
PropertyValidationReport
```

Adjust names if equivalent project conventions already exist.

---

# PART 25 — TEST ISOLATION

Make sure one failed job does not prevent validation of all other jobs if the test architecture allows it.

Ideally the final report should identify ALL failing jobs in one execution rather than stopping at the first failure.

For example:

```text
34 jobs checked

31 PASS
3 FAIL

Failures:

CUSTOMER_DAILY_JOB
    QA missing: customer.processing.date

PAYMENT_MONTHLY_JOB
    UAT additional: payment.archive.path

CLAIM_WEEKLY_JOB
    PROD missing: claim.input.path
```

Do not simply stop at:

```text
AssertionError: CUSTOMER_DAILY_JOB failed
```

if the framework can reasonably collect all failures.

---

# PART 26 — DOCUMENTATION

Create or update:

```text
docs/job-property-validation.md
```

or use the project's existing documentation location.

Document:

1. What the test does.
2. How jobs are discovered.
3. How profiles are tested.
4. How the `properties` endpoint is called.
5. How properties are normalized.
6. How property sets are compared.
7. How input/output fields are identified.
8. How mandatory properties are determined.
9. How optional properties are determined.
10. How per-job Markdown contracts are generated.
11. How failures are reported.
12. How to add a new job.
13. How to add a new profile.
14. How to run the tests locally.
15. How the tests should be integrated into GitLab CI.
16. How authentication is handled.
17. What constitutes PASS/FAIL.
18. Difference between property-name validation and property-value validation.
19. Future mandatory/optional contract support.

---

# PART 27 — COPILOT AGENT SAFETY RULES

Before making changes:

1. Inspect the repository.
2. Identify existing implementation.
3. Identify existing tests.
4. Identify all relevant configuration.
5. Identify how input and output properties are represented.
6. Produce a short implementation plan.
7. Then implement.

Do not blindly create files.

Do not duplicate existing classes.

Do not change production logic unnecessarily.

Do not invent job names.

Do not invent property names.

Do not invent endpoint behavior.

Do not invent profile configuration.

Do not invent mandatory/optional classifications without code evidence.

Do not hard-code secrets.

Do not modify actual DEV/QA/UAT/PROD configuration values.

---

# ADDITIONAL ACCEPTANCE CRITERIA — CONFIGURATION TO ENDPOINT COMPLETENESS

The implementation must additionally satisfy:

### AC-ENDPOINT-1

For every job and every profile, the test identifies the configured/required properties belonging to that job.

### AC-ENDPOINT-2

For every job/profile combination, the test compares the configured/required property set with the properties returned by the `properties` endpoint.

### AC-ENDPOINT-3

If a configured/required property is missing from the endpoint response, the test fails.

### AC-ENDPOINT-4

The failure clearly identifies:

- Job
- Profile
- Missing property/properties
- Configured/required property set
- Endpoint-returned property set

### AC-ENDPOINT-5

The validation does not incorrectly compare all global properties in an application properties file against a single job.

### AC-ENDPOINT-6

Profile-specific configuration is validated independently for DEV, QA, UAT, and PROD.

### AC-ENDPOINT-7

Unexpected endpoint properties are detected and reported.

### AC-ENDPOINT-8

If unexpected endpoint properties are known to be intentional based on the existing endpoint semantics, they are reported as INFO/WARNING rather than incorrectly treated as missing configuration.

### AC-ENDPOINT-9

A job that passes cross-profile endpoint comparison can still fail configuration-to-endpoint completeness, and both results are reported independently.

### AC-ENDPOINT-10

The single Markdown summary contains the configuration-to-endpoint validation results for every job/profile combination.

---

# FINAL ACCEPTANCE CRITERIA

The implementation is considered complete only when all of the following are satisfied:

### AC31

All existing jobs can be discovered reliably.

### AC22

The test validates:

```text
DEV
QA
UAT
PROD
```

### AC23

Every job is checked against every profile.

### AC24

Property ordering does not affect the result.

### AC25

Missing properties are detected.

### AC26

Additional properties are detected.

### AC27

Duplicate properties are detected.

### AC28

The test does NOT compare actual environment-specific property values.

### AC29

Input fields are identified and classified as mandatory.

### AC310

Output fields are identified and classified as mandatory.

### AC311

Remaining properties are classified as optional unless explicit source-code evidence proves another property is mandatory.

### AC312

A single Markdown summary file is generated containing a complete section for every job, including mandatory/optional properties and profile validation results.

### AC313

The test produces a clear failure report.

### AC314

The test continues validating other jobs where practical and reports all failures.

### AC315

No production behavior is changed unnecessarily.

### AC316

No secrets are committed.

### AC317

The solution uses the existing `properties` endpoint/business logic rather than duplicating its implementation.

### AC318

The solution is reusable for future jobs.

### AC319

The architecture supports future machine-readable mandatory/optional property contracts.

### AC220

The test can be executed from the normal Maven test/verification lifecycle.

### AC221

Documentation is provided.

---

# FINAL VERIFICATION

After implementation, perform a complete review.

Run the relevant Maven compilation/tests.

Verify that the generated Markdown files are actually produced and contain real data from the application rather than placeholders.

Provide a final summary containing:

```text
1. Files created
2. Files modified
3. Why each file was changed
4. How jobs are discovered
5. How each profile is tested
6. How the properties endpoint is invoked
7. How property sets are compared
8. How input properties are identified
9. How output properties are identified
10. Mandatory properties for each job
11. Optional properties for each job
12. Configuration-to-endpoint completeness validation
13. Start-endpoint validation using the properties endpoint response
14. Safe handling of PROD/non-destructive execution
15. Example generated single Markdown summary file
16. Example failure output
14. How to run the tests
15. Any assumptions
16. Any areas requiring developer confirmation
17. Any production-code changes made
```

Also explicitly state:

```text
Total jobs discovered:
Total profiles tested:
Total job/profile combinations tested:
Total configuration-to-endpoint validations performed:
Total start-endpoint validations performed:
Total start-endpoint validations passed:
Total start-endpoint validations failed:
Total start-endpoint validations not executed:
Total mandatory properties identified:
Total optional properties identified:
Total jobs with ambiguous classification:
Total jobs with endpoint completeness failures:
Total jobs with start-endpoint failures:
Total jobs passing:
Total jobs failing:
```

Do not claim the implementation is complete until the tests compile successfully and the new test suite passes for the available test environment.
