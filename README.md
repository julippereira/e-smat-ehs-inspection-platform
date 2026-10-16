# E-SMAT app & EHS Inspection Platform

## Overview

E-SMAT is a digital solution for managing Safety and Environmental inspections across industrial operations. 

The project was created to replace fragmented inspection processes with a standardized workflow that connects shopfloor data collection, operational records, data engineering, quality validation, and business intelligence. 

Through a multilingual Power Apps application, inspection users can select the industrial context, access the appropriate questionnaire, record safe and unsafe observations, complete complementary fields, and submit the inspection through a guided process. 

Inspection data is stored in SharePoint, processed through an Azure Databricks pipeline, validated before analytical publishing, and made available in Power BI for monitoring inspection activity, operational conditions, Safety and Environmental themes, and observed acts. 

The application supports different inspection record types used by industrial sites in Latin America. EMAT, SMAT, and ESMAT are treated as operational categories within the solution, while the broader architecture remains consistent across sites and methodologies.

> **Corporate ownership and portfolio scope**
>
> The original solution, its business context, application, data structures, analytical assets, and related materials belong to the company where the project was developed. This repository presents only a **simplified, anonymized, and mock-data-based portfolio version** created to demonstrate technical skills and solution architecture.
>
> No production application package, real inspection record, employee information, credential, internal URL, proprietary identifier, confidential questionnaire, or production connection is included.

---

## Application Preview

The Power Apps layer is the main focus of this portfolio project. It was designed to turn a multi-step inspection process into a guided and consistent user experience.

The application supports:

- EMAT, SMAT, and ESMAT inspection flows;
- Portuguese and Spanish localization;
- Automatic language selection;
- Cascading filters for operational context;
- Dynamic questionnaire routing;
- Safety and environmental questions;
- Safe and unsafe act registration;
- Conditional fields for unsafe conditions, Safety Tags, and near misses;
- Up to three observed-person registrations per inspection;
- Temporary session storage;
- Validation before submission;
- SharePoint integration;
- Standardized header and response records;
- Data prepared for downstream analytics.

Application documentation is available in [`app/`](app/).

---

## Why the Application Matters

The application is more than a digital form. The main challenge was to translate inspection rules, operational hierarchies, multilingual content, and different methodologies into a usable workflow that reduces manual interpretation.

The app controls the progression from context selection to questionnaire completion:

```text
Language and user context
        ↓
Site and operational hierarchy
        ↓
Inspection methodology
        ↓
Questionnaire version and routing
        ↓
Safe and unsafe act registration
        ↓
Complementary safety information
        ↓
Validation and submission
```

This design keeps the user experience simple while handling the business logic behind the scenes.

### Key application capabilities

#### Cascading filtering

Operational selections are progressively filtered so users see only valid combinations of site, segment, business unit or area, line, and workstation.

This context is also used to select the appropriate questionnaire configuration.

#### Dynamic questionnaire routing

The application determines the questionnaire according to the selected site, methodology, language, and active configuration rather than embedding one fixed questionnaire in the interface.

#### Multilingual interface

Labels, navigation elements, instructions, categories, and questionnaire content support Portuguese and Spanish. The localization approach is documented separately so interface text remains independent from the screen logic.

#### Temporary save

Inspection progress can be stored temporarily during the active session. This reduces the risk of losing work when the inspection contains several questions or requires interruption.

#### Conditional safety fields

Additional fields appear only when relevant, including:

```text
Unsafe condition description
Safety Tag number
Near-miss description
```

The same dependencies are validated later in the data-quality pipeline.

#### Structured submission

The submission process separates inspection-level context from question-level responses:

```text
Inspection header
        1
        │
        *
Inspection responses
```

This structure supports traceability, questionnaire enrichment, quality checks, and analytical reporting.

---

## Solution Architecture

```text
Power Apps
    ↓
SharePoint Lists
    ↓
Microsoft Graph API
    ↓
Databricks / PySpark
    ↓
Bronze Delta tables
    ↓
Silver transformations
    ↓
Data-quality gate
    ↓
Gold analytical tables
    ↓
Power BI
```

### Application layer

The Power Apps application manages user interaction, navigation, business-rule execution, multilingual content, temporary values, validation, and submission.

### Data-source layer

SharePoint Lists store inspection headers, question-level responses, questionnaire requirements, locations, categories, and translation references.

### Data-engineering layer

Databricks notebooks authenticate through Microsoft Graph API, ingest SharePoint records, standardize multilingual values, create analytical keys, validate data quality, and publish Delta tables.

### Reporting layer

Power BI uses the curated analytical model to present inspection records, safe and unsafe acts, themes, operational hierarchies, employees, job roles, trends, and drill-through details.

---

## Repository Structure

```text
E-SMAT/
├── README.md
├── .gitignore
│
├── app/
│   ├── README.md
│   ├── app-overview.md
│   ├── navigation-logic.md
│   ├── cascade-filtering.md
│   ├── localization.md
│   └── temporary-save.md
│
├── data/
│   ├── README.md
│   └── sample/
│       ├── sample-locations.csv
│       ├── sample-categories.csv
│       ├── sample-translations.csv
│       ├── sample-requirements.csv
│       ├── sample-inspection-header.csv
│       └── sample-inspection-responses.csv
│
├── data-engineering/
│   ├── README.md
│   ├── ingestion.md
│   ├── transformations.md
│   ├── data-quality.md
│   ├── config/
│   │   └── graph_api_config.example.json
│   └── notebooks/
│       ├── 01_shared_functions.ipynb
│       ├── 02_sharepoint_ingestion.ipynb
│       ├── 03_data_transformation.ipynb
│       ├── 04_data_quality.ipynb
│       ├── 05_delta_load.ipynb
│       └── 06_employee_dimension.ipynb
│
└── power-bi/
    ├── README.md
    ├── e-smat-dashboard.pbix
    ├── dashboard-pages.md
    ├── data-model.md
    ├── measures.md
    ├── key-validation-report.md
    ├── mock-data/
    ├── screenshots/
    └── theme/
        └── esmat-theme.json
```

---

## Application Documentation

The [`app/`](app/) folder explains the main functional and technical decisions of the application.

| Document | Scope |
|---|---|
| [`app/README.md`](app/README.md) | Application entry point and documentation map |
| [`app/app-overview.md`](app/app-overview.md) | Screens, inspection flow, components, and responsibilities |
| [`app/navigation-logic.md`](app/navigation-logic.md) | Screen transitions, validation, and state handling |
| [`app/cascade-filtering.md`](app/cascade-filtering.md) | Operational hierarchy and questionnaire routing |
| [`app/localization.md`](app/localization.md) | Portuguese and Spanish localization strategy |
| [`app/temporary-save.md`](app/temporary-save.md) | Session persistence and temporary inspection state |

The application documentation focuses on architecture and logic. The production app package and internal connection information are not published.

---

## Synthetic Data

The [`data/`](data/) folder contains synthetic CSV files that reproduce the logical structures used by the application.

The public datasets support review of:

- Locations and operational hierarchies;
- Questionnaire categories;
- Translations;
- Questionnaire requirements;
- Inspection headers;
- Question-level responses.

The synthetic values preserve relationships and expected grains without reproducing production records.

See [`data/README.md`](data/README.md) for the dataset documentation.

---

## Data Engineering

The [`data-engineering/`](data-engineering/) folder documents the pipeline that moves inspection data from SharePoint to curated Delta tables.

```text
01_shared_functions.ipynb
        ↓
02_sharepoint_ingestion.ipynb
        ↓
Bronze
        ↓
03_data_transformation.ipynb
        ↓
Silver
        ↓
04_data_quality.ipynb
        ↓
Quality gate
        ↓
05_delta_load.ipynb
        ↓
Gold
```

`06_employee_dimension.ipynb` is an auxiliary workflow that consolidates observer and observed-person registrations and prepares future employee enrichment from an authorized source.

The public pipeline demonstrates:

- Microsoft Graph API authentication and pagination;
- Configuration-driven SharePoint ingestion;
- PySpark transformations;
- Portuguese and Spanish normalization;
- Stable analytical keys;
- Header-detail enrichment;
- Delta Lake persistence;
- Data-quality monitoring;
- Blocking quality controls;
- Analytical publishing.

See [`data-engineering/README.md`](data-engineering/README.md) for the complete pipeline map.

---

## Power BI

The [`power-bi/`](power-bi/) folder contains a downloadable PBIX connected to synthetic data, the semantic-model documentation, DAX measures, report screenshots, visual theme, and validated mock tables.

The report includes:

```text
Visão Geral (Overview)
Registros (Records)
Atos Observados (Observed Acts)
Pessoas (People)
Cargo (Job Roles)
Consultar (Search Records)
DT_Registro (Drill-Through Record)
DT_Matricula (Drill-Through Employee)
```

The public mock reproduces the expected table structures and returns:

```text
72 inspection records
1,481 safe acts
341 unsafe acts
1,822 total observed acts
```

The reporting layer is documented in [`power-bi/README.md`](power-bi/README.md).

---

## Technical Highlights

### Power Platform

- Power Apps canvas application;
- SharePoint Lists;
- Cascading filters;
- Dynamic forms and questionnaire routing;
- Conditional visibility and validation;
- Session collections and temporary state;
- Multilingual interface content;
- Structured submission logic.

### Data Engineering

- Microsoft Graph API;
- OAuth 2.0 client-credentials pattern;
- Python and PySpark;
- Databricks notebooks;
- Bronze, Silver, and Gold layers;
- Delta Lake;
- Schema validation;
- Referential-integrity checks;
- Data-quality gates.

### Analytics

- Power Query;
- Semantic modeling;
- DAX measures;
- Date dimension;
- Active and inactive relationships;
- Drill-through navigation;
- Theme-based formatting;
- Mock-data validation.

---

## Public Version and Confidentiality

This repository is not the production source code repository and does not represent an official company publication.

The public version intentionally excludes:

- The production Power Apps package;
- Production SharePoint exports;
- Real employee names, registrations, or email addresses;
- Real inspection records;
- Internal plant, segment, area, line, or workstation names;
- Production tenant, client, site, catalog, schema, or table identifiers;
- Client secrets, access tokens, and credentials;
- Corporate URLs and workspace references;
- Proprietary questionnaire text;
- Internal operational documentation.

The published notebooks, CSV files, screenshots, and PBIX use placeholders, anonymized structures, or fully synthetic values.

This repository is intended only to demonstrate technical competencies involved in designing the solution, including application architecture, business-rule implementation, data integration, data engineering, quality validation, semantic modeling, and analytical reporting.

---

## Skills Demonstrated

```text
Power Apps
Power Fx
SharePoint Online
Microsoft Graph API
Python
PySpark
Databricks
Delta Lake
Data quality
Power Query
Power BI
DAX
Semantic modeling
Portuguese and Spanish localization
Technical documentation
Synthetic data design
```

---

## Documentation Map

```text
README.md
├── app/README.md
├── data/README.md
├── data-engineering/README.md
└── power-bi/README.md
```

Start with the application documentation to understand the user workflow and business logic:

[`app/README.md`](app/README.md)

Then review the supporting layers:

- [`data/README.md`](data/README.md)
- [`data-engineering/README.md`](data-engineering/README.md)
- [`power-bi/README.md`](power-bi/README.md)

---

## Portfolio Scope

E-SMAT demonstrates an end-to-end solution, but the portfolio narrative intentionally gives greater weight to the application layer.

The strongest focus is the translation of a real inspection process into a guided digital product that combines:

- User-centered navigation;
- Business rules;
- Operational context;
- Multilingual content;
- Dynamic questionnaires;
- Data integrity at the point of entry;
- Integration with downstream data and analytics.

The pipeline and dashboard show how the application data can be governed and analyzed, while the application itself represents the main product and solution-design contribution documented in this repository.
