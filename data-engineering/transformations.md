# Data Transformation

## Overview

The E-SMAT transformation process reads source-aligned Bronze Delta tables, applies explicit data typing, normalizes multilingual business values, creates analytical keys, enriches question-level responses, and writes standardized Silver Delta tables.

This stage separates business transformation from source ingestion. The ingestion notebook preserves the SharePoint structures and source representations, while the transformation notebook converts those records into consistent datasets suitable for quality validation and analytical publishing.

The implementation is available in:

[`notebooks/03_data_transformation.ipynb`](notebooks/03_data_transformation.ipynb)

Reusable Spark and schema-validation functions are maintained in:

[`notebooks/01_shared_functions.ipynb`](notebooks/01_shared_functions.ipynb)

> Production catalog names, schema names, table names, infrastructure paths, and confidential business values are represented by placeholders or generalized examples in the public repository.

---

## Transformation Scope

The notebook processes three Bronze datasets and produces three Silver datasets:

| Bronze source | Silver output | Grain |
|---|---|---|
| `inspection_header_bronze` | `inspection_header_silver` | One row per inspection |
| `inspection_responses_bronze` | `inspection_responses_silver` | One row per inspection and question |
| `inspection_requirements_bronze` | `inspection_requirements_silver` | One row per site, methodology, questionnaire version, and question |

The transformation stage does not create the final safe/unsafe classification table. That analytical fan-out is created only after the Silver quality gate is approved.

---

## High-Level Flow

```text
Bronze Delta tables
        ↓
Validate table configuration
        ↓
Read and validate Bronze schemas
        ↓
Convert source values to explicit types
        ↓
Normalize multilingual categorical values
        ↓
Create stable business keys
        ↓
Transform inspection headers
        ↓
Transform questionnaire requirements
        ↓
Transform question-level responses
        ↓
Enrich responses with header and requirement context
        ↓
Validate Silver structures
        ↓
Write Silver Delta tables
```

---

## Notebook Responsibilities

The transformation notebook performs the following operations:

1. Loads reusable functions from the shared notebook;
2. Defines Bronze source tables and Silver target tables;
3. Validates the table configuration;
4. Confirms that all required Bronze tables exist;
5. Verifies that source DataFrames do not contain duplicate columns;
6. Defines E-SMAT-specific normalization functions;
7. Transforms the questionnaire requirements;
8. Transforms the inspection headers;
9. Transforms and enriches the question-level responses;
10. Creates stable analytical keys;
11. Verifies required Silver columns;
12. Reviews record and column counts;
13. Persists the Silver Delta tables;
14. Produces a transformation and load summary.

---

## Layer Separation

The transformation notebook follows a clear Bronze-to-Silver responsibility boundary.

### Bronze layer

The Bronze layer contains:

- Standardized English column names;
- Source-aligned values represented mainly as strings;
- SharePoint values cleaned of technical formatting artifacts;
- Ingestion metadata;
- No final business typing;
- No analytical enrichment.

### Silver layer

The Silver layer contains:

- Explicit Spark data types;
- Standardized multilingual values;
- Stable analytical keys;
- Derived inspection attributes;
- Enriched response records;
- Deduplicated business grains;
- Processing metadata;
- Structures ready for data-quality validation.

```text
Bronze
  Source-aligned and traceable
        ↓
Silver
  Typed, normalized, enriched, and standardized
```

---

## Table Configuration

The notebook uses a configuration dictionary to define the relationship between Bronze sources and Silver targets.

Conceptually:

```python
TABLE_CONFIG = {
    "inspection_header": {
        "source_table": (
            "<catalog>.<bronze-schema>.inspection_header_bronze"
        ),
        "target_table": (
            "<catalog>.<silver-schema>.inspection_header_silver"
        ),
    }
}
```

The configuration keeps infrastructure references separate from the transformation logic and allows the same processing pattern to be applied consistently across datasets.

---

## Configuration Validation

Before reading any data, the notebook validates the table configuration.

The checks include:

- All required datasets are configured;
- Every source uses a fully qualified three-part table name;
- Every target uses a fully qualified three-part table name;
- Bronze source tables are unique;
- Silver target tables are unique;
- No required dataset mapping is missing.

Configuration errors stop the notebook before reading or transforming any table.

---

## Bronze Table Loading

Each configured Bronze table is checked with the Spark catalog before it is read.

Conceptually:

```python
if not spark.catalog.tableExists(source_table):
    raise RuntimeError(...)

bronze_dataframe = spark.table(source_table)
```

After loading, the notebook confirms that the DataFrame does not contain duplicate column names.

The resulting DataFrames are stored in a dictionary:

```text
bronze_dataframes
├── inspection_header
├── inspection_responses
└── inspection_requirements
```

This makes the datasets available through stable logical names rather than environment-specific table identifiers.

---

## Business-Specific Transformation Helpers

Generic utilities remain in `01_shared_functions.ipynb`. Rules specific to the E-SMAT business model remain in the transformation notebook.

The notebook defines reusable expressions for:

```text
normalize_boolean
normalize_theme
normalize_shift
normalize_activity_type
normalize_contract_type
parse_source_date
keep_numeric_identifier
```

Keeping these rules together makes the normalization logic visible, testable, and easier to maintain.

---

## Boolean Normalization

SharePoint values may arrive as localized text, numeric indicators, or textual Boolean values.

The transformation accepts representations such as:

```text
true
false
Sim
Não
Sí
No
1
0
Yes
```

The normalized Silver type is Spark Boolean:

```text
true
false
null
```

Unrecognized values are converted to `null` rather than silently interpreted.

The normalization is applied to fields such as:

- Active requirement status;
- Unsafe-condition indicator;
- Safety-tag indicator;
- Near-miss indicator.

---

## Theme Normalization

The source questionnaire may identify the question theme using abbreviations or localized labels.

Examples accepted by the transformation include:

```text
env
environment
environmental
meio ambiente
```

These values are normalized to:

```text
environment
```

Safety equivalents such as:

```text
saf
safety
segurança
seguranca
```

are normalized to:

```text
safety
```

This standardization is essential because `theme` is later used in:

- Data-quality controls;
- Safe and unsafe classifications;
- Environmental and safety filtering;
- Power BI measures and visuals.

---

## Shift Normalization

The application supports Portuguese and Spanish interface values. Equivalent shift values are standardized before analytical use.

```text
ADM / Administrativo       → ADM
Primeiro / Primer turno    → 1
Segundo / Segundo turno    → 2
Terceiro / Tercer turno    → 3
```

Values outside the known mapping remain available for later quality review instead of being discarded.

---

## Activity-Type Normalization

Equivalent Portuguese and Spanish activity labels are consolidated.

```text
Rotineira
Rutinaria
Rutinária
      ↓
Rotineira
```

```text
Não-rotineira
Nao-rotineira
Não rotineira
No rutinaria
      ↓
Não-rotineira
```

This prevents equivalent interface values from becoming separate analytical categories.

---

## Contract-Type Normalization

Contract values are also normalized across supported languages.

```text
Empregado interno
Empleado interno
Internal employee
      ↓
Empregado interno
```

```text
Terceiro
Contratista
Contractor
      ↓
Terceiro
```

The public implementation uses neutral terms and does not expose company-specific contract labels.

---

## Date Conversion

Bronze date fields remain strings to protect the ingestion process from inconsistent source formats. The transformation stage converts supported representations into Spark `date` values.

The parser attempts:

```text
DD/MM/YYYY
YYYY-MM-DD
ISO date or datetime beginning with YYYY-MM-DD
```

The normalized column is:

```text
inspection_date
```

The header transformation also derives:

```text
inspection_year
```

The data-quality notebook later verifies that the date is valid and that the derived year matches the date.

---

## Identifier Treatment

Employee and observed-person identifiers are retained only when the source value is strictly numeric.

The rule applies to:

```text
observer_id
observed_person_id_1
observed_person_id_2
observed_person_id_3
```

Non-numeric representations are converted to `null` in Silver.

This rule keeps the analytical structure consistent while preventing accidental propagation of free-text names or malformed identifiers.

The public sample files use synthetic identifiers for demonstration. No production employee identifiers are included in the repository.

---

# Requirements Transformation

## Purpose

The requirements dataset becomes the reference structure used to explain and enrich each questionnaire response.

The transformation validates the availability of fields such as:

```text
site
methodology
questionnaire_version
theme
category_id
category_name_pt_br
category_name_es_ar
question_id
question_column_id
question_number
requirement_text
active
```

---

## Requirement Typing and Standardization

The notebook applies:

- Whitespace trimming;
- Uppercase methodology values;
- Standardized theme values;
- Integer question numbers;
- Boolean active status;
- Stable question and category identifiers;
- Standardized Portuguese and Spanish category fields.

The methodology values remain:

```text
EMAT
SMAT
ESMAT
```

---

## Requirement Key

A stable requirement key is created from:

```text
site
methodology
questionnaire_version
question_id
```

Conceptually:

```text
SITE_A_SMAT_SMAT_SITE_A_V01_QUESTION_0001
```

The key identifies a question within a specific questionnaire context rather than assuming that `question_id` is globally unique across all sites and versions.

```python
requirement_key = concat_ws(
    "_",
    site,
    methodology,
    questionnaire_version,
    question_id,
)
```

The Silver requirements dataset is deduplicated by `requirement_key`.

---

## Requirements Silver Output

The resulting table is:

```text
inspection_requirements_silver
```

Its grain is:

```text
One row per site,
methodology,
questionnaire version,
and question
```

Both active and historical requirements remain available. The `active` Boolean field identifies the current configuration.

---

# Inspection Header Transformation

## Purpose

The inspection header represents the context shared by all question-level responses belonging to the same inspection.

The notebook validates fields associated with:

- Inspection identifier;
- Site and operational hierarchy;
- Inspection date;
- Shift;
- Activity and contract type;
- Observer and observed people;
- Unsafe condition;
- Safety Tag;
- Near miss;
- Audit methodology;
- Creator metadata.

---

## Inspection Date and Year

The source inspection date is converted to `inspection_date`, and the corresponding year is derived as `inspection_year`.

```text
inspection_datetime
        ↓
inspection_date
        ↓
inspection_year
```

The original source field may remain available for traceability unless explicitly removed in a later publishing layer.

---

## Audit-Type Derivation

The notebook first uses the normalized source value when present.

When the source value is empty, the methodology is derived from the beginning of `inspection_id`:

```text
EMAT-...  → EMAT
SMAT-...  → SMAT
ESMAT-... → ESMAT
```

Only the expected methodology prefixes are extracted.

The accepted values are subsequently validated in `04_data_quality.ipynb`.

---

## Observed-People Consolidation

The application supports up to three observed-person identifier fields.

The transformation derives:

```text
observed_people_count
observed_people_ids
```

### `observed_people_count`

Counts the populated observed-person identifiers.

### `observed_people_ids`

Combines the valid identifiers into a comma-separated analytical field.

Conceptually:

```text
observed_person_id_1 = 100001
observed_person_id_2 = 100002
observed_person_id_3 = null

observed_people_count = 2
observed_people_ids   = 100001, 100002
```

The original individual fields remain available when detailed analysis is required.

---

## Header Key

The notebook creates a stable header key from:

```text
inspection_id
inspection_date
```

Conceptually:

```text
SMAT-20260118103000-USER002_2026-01-18
```

Although `inspection_id` remains the principal business identifier, `header_key` provides an additional analytical key containing the inspection date context.

---

## Header Deduplication

The Silver header dataset is deduplicated by:

```text
inspection_id
```

The intended grain is:

```text
One row per inspection
```

Detailed duplicate diagnostics are handled by the data-quality stage.

---

## Header Silver Output

The resulting table is:

```text
inspection_header_silver
```

It contains:

- Typed dates;
- Derived year;
- Standardized categorical values;
- Boolean complementary answers;
- Consolidated observed-person information;
- Audit methodology;
- Stable header key;
- Processing timestamp.

---

# Inspection Response Transformation

## Purpose

The response dataset contains one row for each question submitted during an inspection.

The transformation validates fields such as:

```text
inspection_id
site
question_id
safe_act
unsafe_act
description
created_by
```

---

## Response Typing

The response dataset receives:

- Trimmed identifiers;
- Integer `safe_act` values;
- Integer `unsafe_act` values;
- Cleaned descriptions;
- Header-derived inspection dates;
- Header-derived audit methodology;
- Requirement metadata.

Missing observation quantities are initially converted to zero:

```text
null safe_act   → 0
null unsafe_act → 0
```

The accepted range is validated later by the data-quality notebook.

---

## Header Enrichment

The response dataset is joined to the transformed header using:

```text
inspection_id
```

The header provides:

```text
inspection_date
audit_type
```

This makes the header the authoritative source for the inspection date and methodology.

```text
inspection_header_silver
        │
        │ inspection_id
        ▼
inspection_responses_silver
```

---

## Requirement Enrichment

Active requirements are used to enrich responses.

The contextual join uses:

```text
site
question_id
audit_type = methodology
```

The enrichment adds:

```text
questionnaire_version
theme
category_id
category_name_pt_br
category_name_es_ar
question_number
requirement_text
```

This allows the response table to support analytical use without requiring every report interaction to reconstruct the full questionnaire logic.

---

## Why the Join Uses Context

A question identifier should not be assumed to uniquely identify a question across every site, methodology, and questionnaire configuration.

Using the operational and methodological context reduces the possibility of attaching the wrong requirement metadata to a response.

```text
Response
  site + question_id + audit_type
        ↓
Active requirement
  site + question_id + methodology
```

The data-quality stage later verifies that every response has a corresponding active requirement and complete enrichment.

---

## Response Key

A stable response key is created from:

```text
inspection_id
question_id
```

Conceptually:

```text
SMAT-20260118103000-USER002_QUESTION_0001
```

The intended grain is:

```text
One row per inspection and question
```

The response dataset is deduplicated by `response_key`.

---

## Response Silver Output

The resulting table is:

```text
inspection_responses_silver
```

It includes:

- Typed safe and unsafe quantities;
- Authoritative inspection date;
- Audit methodology;
- Questionnaire version;
- Safety or environmental theme;
- Category metadata;
- Question order and text;
- Stable response key;
- Processing timestamp.

---

## Processing Timestamp

Each Silver dataset receives a refreshed:

```text
processing_timestamp
```

Before adding the new timestamp, the notebook removes any existing column with the same name. This ensures that Silver records reflect the transformation execution rather than only the earlier ingestion execution.

---

## Structural Validation

Before writing the Silver tables, the notebook validates required analytical columns.

### Header requirements

Examples include:

```text
inspection_id
inspection_date
inspection_year
audit_type
header_key
processing_timestamp
```

### Response requirements

Examples include:

```text
inspection_id
question_id
inspection_date
safe_act
unsafe_act
theme
response_key
processing_timestamp
```

### Requirement requirements

Examples include:

```text
requirement_key
question_id
theme
active
processing_timestamp
```

The notebook also confirms that the transformed DataFrames do not contain duplicate column names.

---

## Transformation Summary

The notebook creates an execution summary containing:

```text
dataset
records
columns
target_table
status
```

The summary makes it possible to review:

- Which datasets were transformed;
- The number of records produced;
- The number of columns generated;
- The configured Silver destination;
- The transformation status.

The record counts are reused during the Silver load summary rather than recalculated unnecessarily.

---

## Silver Write Strategy

The portfolio implementation uses a full-refresh strategy:

```python
WRITE_MODE = "overwrite"
OVERWRITE_SCHEMA = "true"
```

The write pattern is:

```python
(
    dataframe.write.format("delta")
    .mode(WRITE_MODE)
    .option("overwriteSchema", OVERWRITE_SCHEMA)
    .saveAsTable(target_table)
)
```

This keeps the public implementation clear and reproducible.

A production implementation may later adopt Delta `MERGE`, incremental windows, change tracking, or other orchestration strategies depending on operational requirements.

---

## Silver Tables

The notebook publishes:

```text
inspection_header_silver
inspection_responses_silver
inspection_requirements_silver
```

The relationship between the tables is:

```text
inspection_header_silver
        │
        │ inspection_id
        ▼
inspection_responses_silver
        │
        │ site + audit_type + question_id
        ▼
inspection_requirements_silver
```

The response table contains denormalized questionnaire metadata for analytical convenience, while the requirements table remains available as the governed questionnaire reference.

---

## Data-Quality Boundary

The transformation notebook performs structural checks required to build valid Silver datasets, but it does not replace the dedicated quality stage.

Structural checks include:

- Required source columns;
- Valid table configuration;
- Existing Bronze tables;
- Duplicate DataFrame columns;
- Required Silver columns;
- Target-table structure.

The following controls are intentionally deferred to `04_data_quality.ipynb`:

- Mandatory business keys;
- Duplicate business keys;
- Referential integrity;
- Accepted values;
- Observation quantity ranges;
- Conditional descriptions;
- Safety-tag consistency;
- Near-miss consistency;
- Requirement-enrichment completeness;
- Blocking quality gate.

This separation keeps transformation logic focused while preserving an explicit and auditable quality stage.

---

## Relationship with Data Quality

After the Silver tables are written, the next notebook validates them:

[`notebooks/04_data_quality.ipynb`](notebooks/04_data_quality.ipynb)

```text
03_data_transformation.ipynb
        ↓
Silver Delta tables
        ↓
04_data_quality.ipynb
        ↓
Quality summary and failures
        ↓
Quality gate
```

The analytical publishing notebook runs only after the Silver quality gate is approved.

---

## Relationship with the Gold Layer

The final notebook reads the quality-approved Silver tables:

[`notebooks/05_delta_load.ipynb`](notebooks/05_delta_load.ipynb)

The Gold stage:

- Confirms the latest quality result;
- Publishes inspection headers;
- Publishes enriched responses;
- Publishes questionnaire requirements;
- Creates the safe/unsafe classification table;
- Validates analytical keys and counts;
- Writes governed Delta tables for Power BI.

```text
Bronze
   ↓
Transformation
   ↓
Silver
   ↓
Data quality
   ↓
Gold publishing
   ↓
Power BI
```

---

## Relationship with the Synthetic Data

The sample datasets in [`../data/sample/`](../data/sample/) illustrate the structures transformed by the notebook.

```text
sample-inspection-header.csv
      ↓ analogous to
inspection_header_silver

sample-inspection-responses.csv
      ↓ analogous to
inspection_responses_silver

sample-requirements.csv
      ↓ analogous to
inspection_requirements_silver
```

The public CSV files are not production exports. They contain synthetic or anonymized values structured to support portfolio review and local examples.

---

## Error Handling

The transformation notebook stops with controlled errors when:

- Required table configuration is missing;
- A table name is not fully qualified;
- Bronze source or Silver target names are duplicated;
- A Bronze table does not exist;
- A Bronze DataFrame contains duplicate columns;
- A required source column is missing;
- A required Silver column is missing;
- A transformed DataFrame contains duplicate columns;
- A Silver target cannot be written.

Unexpected source values are generally preserved or converted to `null` for later quality analysis rather than being silently mapped to an incorrect category.

---

## Security and Privacy

The public transformation documentation and notebook do not include:

- Production catalog names;
- Production schema names;
- Real table names;
- Real SharePoint identifiers;
- Credentials or secrets;
- Corporate URLs;
- Real employee identifiers;
- Real operational hierarchies;
- Confidential requirement texts;
- Real inspection history.

The transformation logic demonstrates the technical approach without exposing the production environment.

---

## Portfolio Scope

This documentation demonstrates how E-SMAT data moves from a source-aligned Bronze layer to standardized Silver datasets.

The transformation stage highlights:

- Separation of ingestion and business logic;
- Explicit data typing;
- Portuguese and Spanish normalization;
- Context-aware questionnaire enrichment;
- Stable business keys;
- Header-detail modeling;
- Reusable PySpark expressions;
- Delta Lake persistence;
- Clear boundaries between transformation, quality, and analytical publishing.

The public implementation is environment-agnostic and requires private infrastructure configuration before it can be executed against a production workspace.
