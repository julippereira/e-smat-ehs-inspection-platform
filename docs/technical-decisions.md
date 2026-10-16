# Technical Decisions

## Overview

This document records the principal design decisions behind the public E-SMAT implementation. The decisions prioritize clear responsibility boundaries, compatibility with the implemented solution, data traceability, public-safe documentation, and practical maintenance.

---

## 1. Power Apps for Field Data Capture

### Decision

Use a Power Apps canvas application for multilingual field inspections.

### Rationale

Power Apps supports rapid construction of guided forms, conditional controls, cascading filters, validation, temporary collections, SharePoint integration, and context-aware navigation.

### Implication

Application logic is distributed across screens, formulas, collections, and connected lists. The repository therefore documents the behavior rather than publishing an executable production application package.

---

## 2. SharePoint Lists as the Operational Source

### Decision

Store inspection headers, question-level responses, questionnaire metadata, locations, categories, and translations in separate SharePoint Lists.

### Rationale

Separate lists preserve logical grains and allow the application to write inspection context independently from its response details.

### Implication

The engineering pipeline must reconstruct relationships and apply explicit schema validation.

---

## 3. Header-Detail Data Model

### Decision

Separate one inspection header from its question-level responses.

```text
inspection_header
        1
        │
        *
inspection_responses
```

### Rationale

Operational context should not be duplicated for every question in the source layer. The model also supports independent validation of inspection and response grains.

---

## 4. Questionnaire as Governed Metadata

### Decision

Maintain questionnaire definitions separately from recorded responses.

### Rationale

Questions vary by site, methodology, version, theme, and category. A separate reference table allows contextual enrichment and historical questionnaire management.

### Implication

Response enrichment uses operational and methodological context rather than assuming that a question identifier is always globally unique.

---

## 5. Microsoft Graph API for Ingestion

### Decision

Use Microsoft Graph API to extract SharePoint List records into Databricks.

### Rationale

The API provides a consistent application-based access pattern and supports list discovery, field retrieval, and pagination.

### Implication

Credentials remain outside the repository, pagination is mandatory, and SharePoint internal-field aliases are configuration-driven.

---

## 6. Configuration Outside the Code

### Decision

Store Graph authentication values and environment-specific identifiers in a protected private configuration.

### Rationale

Credentials and infrastructure identifiers must not be embedded in notebooks or public documentation.

### Public artifact

```text
data-engineering/config/graph_api_config.example.json
```

contains placeholders only.

---

## 7. Shared Functions Notebook

### Decision

Centralize reusable ingestion and validation utilities in:

```text
01_shared_functions.ipynb
```

### Rationale

Pagination, field extraction, HTML cleaning, date normalization, schema validation, DataFrame creation, and required-column checks should not be reimplemented in every notebook.

### Implication

Executable notebooks remain in the same workspace folder and load the shared notebook with `%run`.

---

## 8. Persisted Pipeline Boundaries

### Decision

Persist data between ingestion, transformation, quality, and analytical publishing.

```text
Bronze → Silver → Quality → Gold
```

### Rationale

Persisted boundaries improve traceability, independent reruns, troubleshooting, and responsibility separation. Later notebooks do not depend on DataFrames remaining in memory from earlier notebooks.

---

## 9. Bronze Values Initially Stored as Strings

### Decision

Create source-aligned Bronze DataFrames with nullable string fields.

### Rationale

SharePoint values can arrive in inconsistent representations. String ingestion reduces extraction failures, while explicit typing is applied in the Silver transformation.

### Tradeoff

The Silver layer must perform rigorous conversion and quality validation.

---

## 10. Explicit Silver Normalization

### Decision

Apply explicit Spark types and normalize multilingual values in Silver.

### Examples

```text
Sim / Sí / Yes / 1 → true
Não / No / 0       → false
```

```text
Segurança / safety → safety
Meio Ambiente      → environment
```

### Rationale

Analytical values need stable types and categories regardless of application language.

---

## 11. Stable Business Keys

### Decision

Create explicit keys for headers, responses, requirements, and classifications.

```text
header_key
response_key
requirement_key
classification_key
```

### Rationale

Stable keys allow uniqueness checks, relationships, diagnostics, and deterministic Gold publishing.

---

## 12. Dedicated Data-Quality Stage

### Decision

Run data-quality controls after Silver transformation and before Gold publishing.

### Rationale

Transformation and validation serve different purposes. The quality stage provides an auditable summary and detailed failure evidence.

### Quality gate

Failed checks with severity `error` block Gold publishing. Warnings remain visible without automatically stopping the pipeline.

---

## 13. Persist Quality Evidence Before Failure

### Decision

Write `data_quality_summary` and `data_quality_failures` before raising the blocking exception.

### Rationale

Failure evidence must remain available for investigation even when the pipeline is intentionally stopped.

---

## 14. Gold Tables Designed for Reporting

### Decision

Publish stable Gold tables for Power BI rather than connecting the report directly to raw operational lists.

### Rationale

Power BI should consume typed, normalized, quality-approved structures with stable keys and predictable grains.

---

## 15. Safe and Unsafe Classification Fan-Out

### Decision

Transform one response containing safe and unsafe quantity columns into two classification rows.

```text
response
├── safe
└── unsafe
```

### Rationale

The structure simplifies chart legends, filters, comparisons, and DAX calculations.

### Tradeoff

The classification table contains exactly twice the number of response rows, including zero-quantity classification rows.

---

## 16. Auxiliary Employee Dimension Notebook

### Decision

Keep employee-registration consolidation in a separate auxiliary notebook.

```text
06_employee_dimension.ipynb
```

### Rationale

Employee enrichment is not part of the core inspection model and may depend on a separately authorized employee source.

### Future model

```text
employee_dimension
        1
        │
        *
inspection_employee_bridge
        *
        │
        1
inspection_header
```

---

## 17. Preserve PBIX Compatibility

### Decision

Retain existing Power BI table, column, and measure names where practical.

### Examples

```text
d_cabecalho[Tipo]
Titulo
ChaveUnica
```

### Rationale

The public report is a clone of an implemented dashboard. Preserving names avoids unnecessary rework of measures, relationships, filters, and visuals.

---

## 18. Main Power BI Filter Path

### Decision

Use the active path:

```text
d_calendario
      ↓
d_cabecalho
      ↓
f_formulario
      ↓
d_classificacao
```

### Rationale

A direct active relationship between `d_calendario` and `d_classificacao` would create an ambiguous filter path.

### Implication

Redundant date relationships remain inactive and can be used with `USERELATIONSHIP` when required.

---

## 19. Existing Employee Role Relationships Retained

### Decision

Keep the cloned model's current employee-registration relationship logic rather than redesigning the PBIX around the new bridge.

### Rationale

The existing report pages and drill-through behavior already depend on the current structure. The normalized bridge is documented as a future enhancement.

---

## 20. Synthetic Data for Public Demonstration

### Decision

Create internally consistent synthetic CSV datasets for the public portfolio.

### Rationale

The report and pipeline need realistic grains, keys, relationships, quantities, and conditional fields without exposing production data.

### Validation

The Power BI mock was checked through 31 key and relationship controls with no failures.

---

## 21. Separate Documentation by Layer

### Decision

Maintain focused documentation in:

```text
architecture/
power-apps/
data-engineering/
power-bi/
docs/
```

### Rationale

Each file has one principal responsibility. The root README can guide navigation without repeating every implementation detail.

---

## 22. Images as Evidence, Text as Explanation

### Decision

Use real application and Power BI screenshots where implementation fidelity matters, and architecture diagrams where conceptual explanation is more useful.

### Rationale

Screenshots demonstrate the implemented result. Markdown documentation explains design, relationships, calculations, and limitations in a searchable format.

---

## Related Documentation

```text
business-context.md
inspection-methodologies.md
data-privacy.md
limitations.md
../architecture/README.md
../data-engineering/README.md
../power-bi/data-model.md
../power-bi/measures.md
```
