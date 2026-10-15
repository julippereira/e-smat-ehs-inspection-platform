# Data Quality

## Overview

The E-SMAT data-quality process validates the Silver datasets before any information is published to the analytical Gold layer.

The quality stage evaluates structural integrity, mandatory business keys, uniqueness, accepted values, header-detail relationships, questionnaire enrichment, conditional fields, and execution readiness. Results are consolidated into monitoring tables, and blocking failures stop the downstream publishing process.

The implementation is available in:

[`notebooks/04_data_quality.ipynb`](notebooks/04_data_quality.ipynb)

Reusable Spark and validation functions are maintained in:

[`notebooks/01_shared_functions.ipynb`](notebooks/01_shared_functions.ipynb)

> Production catalog names, schema names, table names, operational identifiers, and real failure records are not included in the public repository.

---

## Quality Scope

The notebook validates three Silver datasets:

| Silver dataset | Grain | Main quality focus |
|---|---|---|
| `inspection_header_silver` | One row per inspection | Inspection keys, dates, audit type, and complementary answers |
| `inspection_responses_silver` | One row per inspection and question | Response keys, quantities, header relationship, and requirement enrichment |
| `inspection_requirements_silver` | One row per questionnaire context and question | Requirement keys, themes, question numbers, and active-question uniqueness |

The notebook produces two quality-monitoring tables:

```text
data_quality_summary
data_quality_failures
```

---

## High-Level Flow

```text
Silver Delta tables
        ↓
Validate expected schemas
        ↓
Run business and integrity checks
        ↓
Count failed records
        ↓
Create quality summary
        ↓
Create detailed failure records
        ↓
Persist quality-monitoring tables
        ↓
Are blocking errors present?
        │
        ├── Yes → Stop downstream publishing
        └── No  → Approve Gold publishing
```

---

## Notebook Responsibilities

The data-quality notebook performs the following operations:

1. Loads reusable functions from the shared notebook;
2. Configures Silver sources and quality-monitoring targets;
3. Validates fully qualified table names;
4. Confirms that all required Silver tables exist;
5. Validates the expected Silver schemas;
6. Executes mandatory-key checks;
7. Detects duplicate business keys;
8. Validates accepted categorical values;
9. Validates safe and unsafe observation quantities;
10. Evaluates header-detail referential integrity;
11. Confirms questionnaire-requirement enrichment;
12. Validates conditional complementary fields;
13. Checks dates and questionnaire numbering;
14. Runs non-blocking monitoring controls;
15. Consolidates summary and failure records;
16. Persists the quality tables;
17. Applies a blocking quality gate.

---

## Quality Severity Levels

Each check is assigned one of three severity levels.

### Error

An `error` indicates that the dataset is not safe for analytical publishing.

Examples include:

- Null business keys;
- Duplicate inspection identifiers;
- Orphan response rows;
- Invalid audit types;
- Missing questionnaire enrichment;
- Inconsistent safety-tag fields.

Failed error checks block the Gold load.

### Warning

A `warning` identifies a condition that should be reviewed but does not automatically prevent publishing.

Examples include:

- A submitted header with no question-level responses;
- An unsafe observation without a description.

Warnings remain available in the monitoring tables.

### Info

An `info` check can be used for contextual monitoring that does not represent a failure condition. The framework supports this severity even though the current public notebook focuses on blocking controls and warnings.

---

## Configuration

The notebook separates Silver sources from quality-monitoring targets.

Conceptually:

```python
SILVER_TABLES = {
    "inspection_header": (
        "<catalog>.<silver-schema>.inspection_header_silver"
    ),
    "inspection_responses": (
        "<catalog>.<silver-schema>.inspection_responses_silver"
    ),
    "inspection_requirements": (
        "<catalog>.<silver-schema>.inspection_requirements_silver"
    ),
}

QUALITY_TABLES = {
    "summary": (
        "<catalog>.<quality-schema>.data_quality_summary"
    ),
    "failures": (
        "<catalog>.<quality-schema>.data_quality_failures"
    ),
}
```

The public notebook uses placeholders so that no production catalog or schema names are exposed.

---

## Configuration and Source Validation

Before running any quality rule, the notebook verifies:

- All required Silver datasets are configured;
- All source and target names are fully qualified;
- Each required Silver table exists;
- The loaded DataFrames do not contain duplicate column names;
- All columns required by the quality rules are present.

The notebook stops immediately when a required source table or required schema element is unavailable.

---

## Quality-Check Framework

The notebook uses a shared registration function for every rule:

```python
register_quality_check(
    check_id,
    dataset_name,
    check_name,
    severity,
    failure_df,
    key_columns,
    description,
)
```

Each rule provides a DataFrame containing only failed records. The framework then:

1. Validates that the failure DataFrame contains the requested key columns;
2. Counts the failed records;
3. Assigns `passed` or `failed` status;
4. Adds one result to the quality summary;
5. Standardizes detailed failure rows;
6. Generates a readable record key;
7. Prints a concise execution result.

This prevents every quality rule from implementing its own summary and persistence logic.

---

## Quality Summary Structure

The summary table contains one row per executed check.

```text
check_id
dataset
check_name
severity
status
failure_count
description
processing_timestamp
pipeline_layer
```

Example conceptual row:

```text
DQ012
inspection_responses
response_has_header
error
passed
0
Every response must reference an existing inspection header.
```

The summary provides a compact operational view of the complete quality execution.

---

## Failure Detail Structure

Failed records from different datasets are standardized into one generic structure:

```text
check_id
dataset
check_name
severity
record_key
failure_reason
processing_timestamp
pipeline_layer
```

The `record_key` is assembled from the most relevant identifying fields for each rule.

Conceptually:

```text
inspection_id=SMAT-20260118103000-USER002 |
response_key=SMAT-20260118103000-USER002_QUESTION_0001
```

This structure allows failures from headers, responses, and requirements to be stored in one monitoring table.

---

# Quality Checks

## DQ001: Mandatory Header Keys

Validates that every inspection header contains:

```text
inspection_id
site
inspection_date
```

Severity:

```text
error
```

A header without these fields cannot reliably participate in the analytical model.

---

## DQ002: Mandatory Response Keys

Validates that every response contains:

```text
inspection_id
question_id
response_key
```

Severity:

```text
error
```

These fields define the response grain and its relationship with the inspection and questionnaire.

---

## DQ003: Mandatory Requirement Keys

Validates that every requirement contains:

```text
requirement_key
question_id
site
```

Severity:

```text
error
```

These fields are required to identify the question within its operational questionnaire context.

---

## DQ004: Unique Inspection Identifier

Checks whether `inspection_id` appears more than once in the header dataset.

Expected grain:

```text
One row per inspection
```

Severity:

```text
error
```

---

## DQ005: Unique Response Key

Checks whether `response_key` appears more than once in the response dataset.

Expected grain:

```text
One row per inspection and question
```

Severity:

```text
error
```

---

## DQ006: Unique Requirement Key

Checks whether `requirement_key` appears more than once in the requirements dataset.

Expected grain:

```text
One row per site,
methodology,
questionnaire version,
and question
```

Severity:

```text
error
```

---

## DQ007: Accepted Audit Type

Validates that every header uses one of the supported inspection methodologies:

```text
EMAT
SMAT
ESMAT
```

Null or unexpected values fail the check.

Severity:

```text
error
```

---

## DQ008: Accepted Requirement Theme

Validates that requirement themes contain only:

```text
environment
safety
```

Severity:

```text
error
```

This ensures that questionnaire metadata can be consistently filtered and aggregated.

---

## DQ009: Accepted Response Theme

Validates that enriched response themes contain only:

```text
environment
safety
```

A null theme also fails the check because it indicates incomplete requirement enrichment.

Severity:

```text
error
```

---

## DQ010: Valid Observation Quantity Range

Validates that both quantity fields contain values between zero and three:

```text
safe_act
unsafe_act
```

Accepted range:

```text
0 to 3
```

Null, negative, or greater-than-three values fail the check.

Severity:

```text
error
```

---

## DQ011: At Least One Observation

Identifies responses where both quantities equal zero:

```text
safe_act = 0
unsafe_act = 0
```

Severity:

```text
warning
```

This situation may be valid in exceptional contexts, but it deserves review because the response contains no classified observation.

---

## DQ012: Response Has Header

Uses a left anti join to identify question-level responses with no corresponding inspection header.

Relationship:

```text
inspection_header_silver
        │
        │ inspection_id
        ▼
inspection_responses_silver
```

Severity:

```text
error
```

Every response must belong to an existing inspection.

---

## DQ013: Response and Header Context Consistency

Compares the response and header context for the same `inspection_id`.

The following fields must agree:

```text
site
inspection_date
audit_type
```

Severity:

```text
error
```

This prevents a response from being analytically associated with a different operational or methodological context than its header.

---

## DQ014: Response Has Active Requirement

Validates that every response references an active requirement with the same:

```text
site
audit_type = methodology
question_id
```

Severity:

```text
error
```

The check uses the active requirements dataset rather than relying only on the metadata already copied into the response.

---

## DQ015: Complete Requirement Enrichment

Validates that each response contains:

```text
questionnaire_version
theme
category_id
question_number
```

Severity:

```text
error
```

Missing enrichment indicates that the Bronze-to-Silver contextual join did not resolve a valid question configuration.

---

## DQ016: Unsafe-Condition Description Consistency

Validates two directions of the condition:

```text
unsafe_condition = true
      ↓
unsafe_condition_description must be populated
```

```text
unsafe_condition = false
      ↓
unsafe_condition_description must be empty
```

Severity:

```text
error
```

---

## DQ017: Safety-Tag Consistency

Validates that:

```text
safety_tag_opened = true
      ↓
safety_tag_number must be populated
```

```text
safety_tag_opened = false
      ↓
safety_tag_number must be empty
```

It also validates that an opened Tag requires an unsafe condition:

```text
safety_tag_opened = true
      ↓
unsafe_condition = true
```

Severity:

```text
error
```

---

## DQ018: Near-Miss Description Consistency

Validates two directions of the condition:

```text
near_miss = true
      ↓
near_miss_description must be populated
```

```text
near_miss = false
      ↓
near_miss_description must be empty
```

Severity:

```text
error
```

---

## DQ019: Valid Inspection Date

Validates that:

- `inspection_date` is not null;
- `inspection_year` matches the year derived from `inspection_date`.

Severity:

```text
error
```

This confirms that the date conversion performed in the Silver transformation was successful and internally consistent.

---

## DQ020: Valid Question Number

Validates that `question_number` is:

- Not null;
- Greater than zero.

Severity:

```text
error
```

---

## DQ021: Unique Active Requirement Context

Validates that an active question is unique within:

```text
site
methodology
questionnaire_version
question_id
```

Severity:

```text
error
```

This prevents multiple active requirement records from representing the same question in the same questionnaire context.

---

## DQ022: Header Has Responses

Identifies submitted inspection headers with no question-level responses.

Severity:

```text
warning
```

The condition is monitored because a header may have been written while the detail records were not completed or persisted.

---

## DQ023: Unsafe Observation Has Description

Identifies responses where:

```text
unsafe_act > 0
```

but `description` is empty.

Severity:

```text
warning
```

The notebook does not block publishing for this condition, but the result remains visible for operational review.

---

## Summary of Controls

The public notebook contains 23 checks:

```text
DQ001 to DQ023
```

They cover:

- Mandatory fields;
- Unique keys;
- Allowed values;
- Quantity ranges;
- Header-detail integrity;
- Requirement integrity;
- Context consistency;
- Conditional descriptions;
- Date integrity;
- Questionnaire uniqueness;
- Operational monitoring.

---

## Referential Integrity Model

The quality rules validate the central analytical relationships.

```text
inspection_header_silver
        │
        │ inspection_id
        ▼
inspection_responses_silver
```

```text
inspection_requirements_silver
        │
        │ site + methodology + question_id
        ▼
inspection_responses_silver
```

The first relationship ensures every response has a valid inspection. The second ensures every response has a valid questionnaire requirement in the same business context.

---

## Conditional-Field Model

The inspection header contains paired indicators and descriptions.

```text
unsafe_condition
      ↓
unsafe_condition_description
```

```text
safety_tag_opened
      ↓
safety_tag_number
```

```text
near_miss
      ↓
near_miss_description
```

The quality stage validates both the required population and the required emptiness of these dependent fields.

This two-direction validation prevents contradictory combinations such as:

- `false` with a populated description;
- `true` with an empty description;
- Tag number without an opened Tag;
- Opened Tag without an unsafe condition.

---

## Quality Result Consolidation

After all checks are executed, the notebook creates:

```text
quality_summary_df
quality_failures_df
quality_overview_df
```

### `quality_summary_df`

Contains one row per check.

### `quality_failures_df`

Contains standardized detailed failures from every failed rule.

### `quality_overview_df`

Aggregates the results by:

```text
severity
status
```

The overview reports:

```text
check_count
failure_count
```

The three views are displayed in the notebook for immediate inspection.

---

## Empty Failure Handling

When all checks pass, the notebook still creates an empty failure DataFrame using an explicit schema.

This guarantees that:

```text
data_quality_failures
```

can be persisted consistently even when there are no failed records.

The explicit schema contains:

```text
check_id
dataset
check_name
severity
record_key
failure_reason
```

Processing metadata is added afterward.

---

## Quality Monitoring Tables

The notebook persists:

```text
data_quality_summary
data_quality_failures
```

Both tables receive:

```text
processing_timestamp
pipeline_layer
```

The current layer is identified as:

```text
silver
```

These tables support:

- Execution monitoring;
- Failure investigation;
- Trend analysis;
- Operational reporting;
- Pipeline gating;
- Future alerting or orchestration.

---

## Write Strategy

The public notebook uses:

```python
WRITE_MODE = "overwrite"
OVERWRITE_SCHEMA = "true"
```

This means the quality tables represent the latest execution.

A production implementation may instead retain historical executions using append mode and an explicit execution identifier. That enhancement would support long-term quality trends without changing the check framework.

---

## Quality Gate

The notebook applies a final blocking rule after the monitoring tables are persisted.

```python
FAIL_ON_ERROR = True
```

Blocking failures are defined as checks where:

```text
severity = error
status   = failed
```

The quality gate follows this flow:

```text
Complete all checks
        ↓
Persist summary and failures
        ↓
Find failed error checks
        ↓
Are blocking failures present?
        │
        ├── Yes → Raise RuntimeError
        └── No  → Approve Gold publishing
```

The error message identifies the failed check IDs, for example:

```text
Blocking checks: [DQ012, DQ015]
```

Warnings do not stop the pipeline.

---

## Why Results Are Persisted Before the Exception

The notebook writes the quality summary and detailed failures before raising the blocking exception.

This ordering ensures that the failure evidence remains available for diagnosis even when the pipeline is intentionally stopped.

```text
Detect failures
      ↓
Persist evidence
      ↓
Stop pipeline
```

If the exception occurred before persistence, the failed execution could be visible only in notebook logs and would be harder to monitor systematically.

---

## Relationship with Transformation

The quality stage reads the outputs of:

[`notebooks/03_data_transformation.ipynb`](notebooks/03_data_transformation.ipynb)

```text
03_data_transformation.ipynb
        ↓
inspection_header_silver
inspection_responses_silver
inspection_requirements_silver
        ↓
04_data_quality.ipynb
```

The transformation notebook performs structural preparation. The quality notebook evaluates whether the resulting business data is reliable enough for publishing.

---

## Relationship with Gold Publishing

The final pipeline notebook is:

[`notebooks/05_delta_load.ipynb`](notebooks/05_delta_load.ipynb)

Before publishing Gold tables, that notebook reads the latest execution from:

```text
data_quality_summary
```

Gold publishing is blocked when the latest quality execution contains failed error checks.

```text
Silver tables
      ↓
04_data_quality.ipynb
      ↓
Quality gate passed?
      │
      ├── No  → Stop
      └── Yes → 05_delta_load.ipynb
```

This provides an explicit dependency between validated Silver data and the analytical layer.

---

## Relationship with Synthetic Data

The public sample datasets support local inspection of the same logical quality rules:

- [`../data/sample/sample-inspection-header.csv`](../data/sample/sample-inspection-header.csv)
- [`../data/sample/sample-inspection-responses.csv`](../data/sample/sample-inspection-responses.csv)
- [`../data/sample/sample-requirements.csv`](../data/sample/sample-requirements.csv)

The sample data was designed to preserve:

- Header-detail relationships;
- Requirement references;
- Safe and unsafe quantities;
- Conditional complementary fields;
- Safety and environmental themes;
- Synthetic identifiers and dates.

The files contain no production inspection records or identifiable users.

---

## Error Handling

The notebook stops with controlled errors when:

- Required quality configuration is missing;
- A source or target name is invalid;
- A Silver table does not exist;
- A Silver DataFrame contains duplicate columns;
- A required quality column is missing;
- A check uses an invalid severity;
- The failure DataFrame lacks its configured key columns;
- A quality target cannot be written;
- One or more blocking checks fail.

The monitoring tables are written before the blocking quality exception is raised.

---

## Security and Privacy

The public quality documentation and notebook do not include:

- Production catalog names;
- Production schema names;
- Real table identifiers;
- Corporate URLs;
- Credentials;
- Real employee identifiers;
- Real inspection records;
- Confidential operational locations;
- Proprietary requirement text;
- Actual production failure records.

Detailed failure keys in the public implementation are based on generalized or synthetic identifiers only.

---

## Potential Production Enhancements

The quality framework can be extended with:

- Batch or execution identifiers;
- Historical append-mode monitoring;
- Quality thresholds by severity;
- Automated notifications;
- Orchestration-level task dependencies;
- Quarantine tables;
- Rule ownership metadata;
- Expected-volume checks;
- Freshness and latency checks;
- Statistical anomaly detection;
- Site-specific tolerances;
- Trend dashboards for recurring failures.

These enhancements are intentionally outside the current portfolio scope.

---

## Portfolio Scope

This documentation demonstrates how E-SMAT applies a dedicated quality stage between Silver transformation and Gold publishing.

The implementation highlights:

- Reusable rule registration;
- Standardized summary and failure structures;
- Mandatory-key validation;
- Duplicate detection;
- Referential integrity;
- Accepted-value controls;
- Conditional-field validation;
- Questionnaire-enrichment checks;
- Warning and error severities;
- Persisted failure evidence;
- A blocking quality gate;
- Clear separation between transformation and analytical publishing.

The public implementation is environment-agnostic and requires private infrastructure configuration before it can be executed against a production workspace.
