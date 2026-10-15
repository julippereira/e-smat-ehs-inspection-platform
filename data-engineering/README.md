# E-SMAT Data Engineering Pipeline

## Overview

This folder contains the data-engineering pipeline that moves E-SMAT inspection data from SharePoint to governed Delta tables for analytical consumption.

The pipeline is organized as independent Databricks notebooks with clear responsibilities across ingestion, transformation, data quality, and analytical publishing. Shared utilities are centralized to reduce duplication, while detailed technical decisions are documented in separate Markdown files.

> The public version uses placeholders, anonymized structures, and synthetic sample data. It does not include production credentials, internal identifiers, confidential inspection records, or environment-specific infrastructure names.

---

## Pipeline Architecture

```text
SharePoint Lists
      ↓
Microsoft Graph API
      ↓
02_sharepoint_ingestion.ipynb
      ↓
Bronze Delta tables
      ↓
03_data_transformation.ipynb
      ↓
Silver Delta tables
      ↓
04_data_quality.ipynb
      ↓
Quality gate
      ↓
05_delta_load.ipynb
      ↓
Gold Delta tables
      ↓
Power BI
```

The pipeline follows a layered architecture:

- **Bronze:** source-aligned records with standardized column names and ingestion metadata;
- **Silver:** typed, normalized, enriched, and deduplicated business datasets;
- **Quality:** persisted validation results and blocking controls;
- **Gold:** quality-approved analytical tables prepared for reporting.

---

## Folder Structure

```text
data-engineering/
├── README.md
├── ingestion.md
├── transformations.md
├── data-quality.md
│
├── config/
│   └── graph_api_config.example.json
│
└── notebooks/
    ├── 01_shared_functions.ipynb
    ├── 02_sharepoint_ingestion.ipynb
    ├── 03_data_transformation.ipynb
    ├── 04_data_quality.ipynb
    └── 05_delta_load.ipynb
```

---

## Notebook Sequence

### 1. Shared Functions

[`notebooks/01_shared_functions.ipynb`](notebooks/01_shared_functions.ipynb)

Provides reusable utilities for:

- Microsoft Graph requests and pagination;
- SharePoint value extraction;
- HTML and date cleaning;
- Source-record schema validation;
- Spark DataFrame creation;
- Column standardization;
- Required-column checks;
- Processing timestamps.

This notebook defines functions only. It does not authenticate, extract data, or write tables.

### 2. SharePoint Ingestion

[`notebooks/02_sharepoint_ingestion.ipynb`](notebooks/02_sharepoint_ingestion.ipynb)

Authenticates with Microsoft Graph API, discovers the configured SharePoint Lists, extracts paginated records, validates source schemas, and writes the Bronze tables.

Outputs:

```text
inspection_header_bronze
inspection_responses_bronze
inspection_requirements_bronze
```

### 3. Data Transformation

[`notebooks/03_data_transformation.ipynb`](notebooks/03_data_transformation.ipynb)

Reads the Bronze tables and applies data typing, multilingual normalization, business-key creation, header-detail standardization, and questionnaire enrichment.

Outputs:

```text
inspection_header_silver
inspection_responses_silver
inspection_requirements_silver
```

### 4. Data Quality

[`notebooks/04_data_quality.ipynb`](notebooks/04_data_quality.ipynb)

Runs 23 checks covering mandatory keys, uniqueness, accepted values, quantity ranges, referential integrity, requirement enrichment, dates, and conditional fields.

Outputs:

```text
data_quality_summary
data_quality_failures
```

Failed checks with severity `error` block the analytical publishing stage. Warnings remain available for review without automatically stopping the pipeline.

### 5. Delta Load and Analytical Publishing

[`notebooks/05_delta_load.ipynb`](notebooks/05_delta_load.ipynb)

Confirms the latest Silver quality gate, prepares reporting-ready datasets, creates the safe/unsafe classification fact table, publishes the Gold tables, and verifies persisted record counts.

Outputs:

```text
inspection_header
inspection_responses
inspection_requirements
inspection_classification
```

---

## Analytical Model

```text
inspection_header
        │
        │ inspection_id
        ▼
inspection_responses
        │
        ├── question_id ──► inspection_requirements
        │
        └── response_key ─► inspection_classification
```

### Table grains

| Table | Grain |
|---|---|
| `inspection_header` | One row per inspection |
| `inspection_responses` | One row per inspection and question |
| `inspection_requirements` | One row per questionnaire context and question |
| `inspection_classification` | Two rows per response: one `safe` and one `unsafe` |

---

## Key Design Decisions

### Configuration-driven ingestion

SharePoint lists, expected source fields, aliases, rename mappings, and Bronze targets are defined through configuration dictionaries rather than repeated extraction blocks.

### Validation before DataFrame creation

Extracted records are checked against the configured schema before Spark creates the DataFrame. The standardized schema is validated again after column renaming.

### Persisted layer boundaries

Each processing stage writes Delta tables before the next notebook begins. This avoids dependencies on notebook memory and allows stages to be rerun or investigated independently.

### Context-aware questionnaire enrichment

Responses are enriched using operational and methodological context rather than relying only on a question identifier.

### Explicit quality gate

Gold publishing depends on the latest persisted quality result. Blocking errors prevent analytical tables from being refreshed with invalid Silver data.

### Reporting-oriented classification table

Safe and unsafe quantities are expanded into separate classification rows to simplify Power BI measures, legends, filters, and comparisons.

---

## Configuration

The repository includes a safe template:

[`config/graph_api_config.example.json`](config/graph_api_config.example.json)

```json
{
  "graph_api": {
    "tenant_id": "<tenant-id>",
    "client_id": "<client-id>",
    "client_secret": "<client-secret>",
    "site_id": "<sharepoint-site-id>"
  }
}
```

The real configuration must remain outside source control and should be stored in an approved protected location, such as a Unity Catalog Volume or secret-management service.

---

## Execution Order

Run the notebooks in this order:

```text
01_shared_functions.ipynb
02_sharepoint_ingestion.ipynb
03_data_transformation.ipynb
04_data_quality.ipynb
05_delta_load.ipynb
```

The executable notebooks load the shared functions with:

```python
%run ./01_shared_functions
```

The notebooks should therefore remain in the same Databricks folder unless the `%run` path is updated.

---

## Environment Setup

Before running the public notebooks in a Databricks workspace:

1. Import all notebooks into the same workspace folder;
2. Create a private Graph API configuration file;
3. Store the private configuration outside the Git repository;
4. Replace catalog and schema placeholders;
5. Replace the public SharePoint list placeholders;
6. Configure private SharePoint internal-field aliases when required;
7. Confirm the application registration has the required Graph permissions;
8. Confirm the cluster or serverless environment can access the target catalogs and schemas;
9. Run the notebooks in sequence.

This portfolio version is not expected to run unchanged against a real environment because production infrastructure details were intentionally removed.

---

## Write Strategy

The public implementation uses full-refresh writes:

```python
WRITE_MODE = "overwrite"
OVERWRITE_SCHEMA = "true"
```

This keeps the pipeline easy to review and reproduce. Depending on production requirements, the same architecture can be extended with:

- Delta `MERGE` operations;
- Incremental extraction windows;
- Change tracking;
- Batch identifiers;
- Historical quality results;
- Workflow orchestration;
- Notifications and retry policies.

---

## Technical Documentation

Detailed implementation documentation is separated by pipeline responsibility:

- [`ingestion.md`](ingestion.md): authentication, Microsoft Graph extraction, pagination, schema validation, and Bronze persistence;
- [`transformations.md`](transformations.md): data typing, normalization, enrichment, analytical keys, and Silver persistence;
- [`data-quality.md`](data-quality.md): quality rules, failure structures, monitoring tables, and the blocking quality gate.

This README provides the pipeline map and execution guidance without repeating the detailed rules documented in those files.

---

## Relationship with Public Sample Data

The synthetic datasets in [`../data/sample/`](../data/sample/) reproduce the logical structures used by the application and pipeline.

They support:

- Repository review without production connections;
- Schema inspection;
- Relationship analysis;
- Local demonstrations;
- Power BI prototyping;
- Portfolio documentation.

The production notebooks retrieve data from SharePoint. They do not use the sample CSV files as their operational source.

---

## Security and Confidentiality

The public data-engineering layer excludes:

- Credentials and access tokens;
- Real tenant, client, site, list, catalog, and schema identifiers;
- Corporate URLs;
- Production table names;
- Real employee identifiers;
- Real operational hierarchies;
- Real inspection history;
- Proprietary questionnaire content;
- Production quality-failure records.

---

## Portfolio Scope

This folder demonstrates an end-to-end data-engineering workflow using Microsoft Graph API, SharePoint, PySpark, Databricks, Delta Lake, and data-quality controls.

The implementation emphasizes:

- Separation of responsibilities;
- Reusable functions;
- Configuration-driven processing;
- Layered Bronze, Silver, and Gold architecture;
- Schema validation;
- Multilingual normalization;
- Header-detail modeling;
- Context-aware enrichment;
- Persisted quality evidence;
- Blocking quality gates;
- Reporting-ready analytical outputs;
- Public-safe documentation and anonymization.

