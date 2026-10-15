# Data

## About This Folder

This folder documents the data structures used by the E-SMAT solution and provides synthetic sample files for portfolio demonstration.

The production application uses SharePoint Lists for two distinct purposes:

1. **Reference data**, used to configure the application, filter operational locations, load inspection requirements, organize question categories, and display bilingual interface content;
2. **Inspection data**, created when the user submits an EMAT, SMAT, or ESMAT inspection.

The public repository does not include production SharePoint exports or real inspection records. Sample datasets preserve the structure and relationships required to understand the solution while using anonymized or synthetic content.

---

## Folder Structure

```text
data/
├── README.md
├── data-dictionary.md
└── sample/
    ├── sample-locations.csv
    ├── sample-requirements.csv
    ├── sample-categories.csv
    ├── sample-translations.csv
    ├── sample-inspection-header.csv
    └── sample-inspection-responses.csv
```

---

## Data Domains

The sample data is organized into two domains:

### Reference Data

Reference datasets control how the application behaves.

```text
sample-locations.csv
sample-requirements.csv
sample-categories.csv
sample-translations.csv
```

These files represent the SharePoint-based configuration layer used by Power Apps.

### Inspection Data

Inspection datasets represent the records created during final submission.

```text
sample-inspection-header.csv
sample-inspection-responses.csv
```

These files reproduce the header-detail structure used to associate one inspection with multiple question-level responses.

---

## Reference Data Source

The reference-data design is based on four structured datasets maintained in SharePoint Lists.

In the source workbook used to prepare the portfolio structure, each worksheet represents one reference list:

```text
lista_areas
lista_questoes
lista_categorias
lista_traducoes
```

For the public repository, these structures are represented using descriptive English file names and non-confidential sample values.

| Source structure | Portfolio file | Purpose |
|---|---|---|
| `lista_areas` | `sample-locations.csv` | Operational location hierarchy used by the cascading filters |
| `lista_questoes` | `sample-requirements.csv` | Site- and methodology-specific inspection requirements |
| `lista_categorias` | `sample-categories.csv` | Standardized question-category reference |
| `lista_traducoes` | `sample-translations.csv` | Portuguese and Spanish interface content |

---

## Reference Datasets

### `sample/sample-locations.csv`

Represents the operational hierarchy used by the Power Apps cascading filters.

The hierarchy follows this sequence:

```text
Site
  ↓
Segment
  ↓
Business Unit / Area
  ↓
Line
  ↓
Workstation
```

The sample file is expected to contain fields equivalent to:

```text
site
segment
business_unit_area
line
workstation
```

Each row represents a valid location path. Power Apps filters this structure progressively so that users can select only combinations configured in the reference data.

The public file must use generalized values such as `SITE_A`, `SEGMENT_01`, `AREA_01`, `LINE_01`, and `WORKSTATION_01`.

### `sample/sample-requirements.csv`

Represents the inspection requirements loaded dynamically by E-SMAT.

The source structure contains metadata associated with:

- Site;
- Site abbreviation;
- Inspection methodology;
- Questionnaire version;
- Safety or environmental theme;
- Category identifier;
- Category name;
- Question identifier;
- Question-column identifier;
- Question number;
- Question text;
- Spanish category content;
- Active status.

The sample dataset demonstrates how E-SMAT identifies and orders the requirements applicable to a specific inspection context.

Only synthetic inspection questions should be included in the public repository. The production requirement texts are not published.

### `sample/sample-categories.csv`

Represents the standardized category reference used to group inspection requirements.

The source structure contains:

```text
category_name
category_id
```

Category identifiers create a stable relationship between the category reference and the requirements dataset.

The sample file should preserve this relationship while using generalized or synthetic category content where necessary.

### `sample/sample-translations.csv`

Represents the translation reference used by the bilingual interface.

The source structure contains:

```text
translation_key
pt_br
es_ar
```

Each row associates an application element with its Brazilian Portuguese and Argentine Spanish values.

The translation dataset supports content such as:

- Screen titles;
- Field labels;
- Buttons;
- Navigation actions;
- Response options;
- Validation messages;
- Submission confirmation messages.

The active interface language is determined automatically from the device or browser configuration and can be changed manually in the application.

---

## Inspection Datasets

### `sample/sample-inspection-header.csv`

Represents the inspection-level record created during final submission.

The header stores contextual information shared by all responses belonging to the same inspection, such as:

- Unique inspection identifier;
- Inspection date;
- Site;
- Segment;
- Business unit or area;
- Line;
- Workstation;
- Shift;
- Activity type;
- Contract type;
- Position or job role;
- Anonymized observer and observed-person identifiers;
- Unsafe-condition indicator and description;
- Near-miss indicator and description;
- Safety-tag indicator and reference.

The inspection identifier acts as the primary business key connecting the header to the question-level response records.

### `sample/sample-inspection-responses.csv`

Represents the question-level records created from the dynamic questionnaire gallery.

Each row corresponds to one question associated with one submitted inspection.

The response structure includes fields equivalent to:

- Inspection identifier;
- Inspection date;
- Site;
- Question identifier;
- Observation description;
- Safe-observation value;
- Unsafe-observation value.

A single inspection header can therefore be associated with multiple response rows.

```text
One Inspection Header
      ↓
Multiple Question-Level Responses
```

---

## Dataset Relationships

The main relationships between the public sample datasets are:

```text
sample-locations.csv
      ↓ provides valid operational context
sample-inspection-header.csv

sample-categories.csv
      ↓ category_id
sample-requirements.csv

sample-requirements.csv
      ↓ question_id
sample-inspection-responses.csv

sample-inspection-header.csv
      ↓ inspection_id
sample-inspection-responses.csv

sample-translations.csv
      ↓ translation_key and active language
Power Apps interface content
```

These relationships reproduce the logical structure of the solution without exposing production SharePoint configuration or real operational data.

---

## Header-Detail Storage Model

E-SMAT stores finalized inspections using a header-detail pattern.

### Header

Contains the information that applies to the inspection as a whole.

### Detail

Contains one record for each question displayed and submitted through the questionnaire.

```text
Inspection Header
      │
      ├── Question Response 01
      ├── Question Response 02
      ├── Question Response 03
      └── Question Response N
```

The same inspection identifier is stored in both structures, allowing downstream processes to reconstruct the complete inspection.

---

## Bilingual Value Normalization

The user interface supports Portuguese and Spanish, but selected values are normalized before being stored.

This prevents equivalent values from being treated as separate categories during data ingestion and reporting.

Examples include:

```text
Primero              → Primeiro
Tercero              → Terceiro
Rutina               → Rotineira
No rutinaria         → Não-rotineira
Capitán de Seguridad → Capitão de Segurança
Sí                   → true
Sim                  → true
```

The sample inspection files should contain the standardized storage values rather than duplicate categories created only by interface language.

---

## Public Sample Data Principles

The files in `sample/` follow these principles:

- No production inspection records;
- No employee names or e-mail addresses;
- No real employee identifiers;
- No production site, area, line, or workstation mappings;
- No complete proprietary inspection requirements;
- No SharePoint URLs or internal list identifiers;
- No credentials or connection details;
- No confidential organizational references.

Synthetic values should preserve the technical relationships, cardinality, and expected data types of the solution.

---

## File Format

The public sample datasets use CSV format because it is:

- Easy to inspect in GitHub;
- Compatible with Python, Power BI, Excel, and Databricks;
- Suitable for demonstrating ingestion and transformation logic;
- Independent from production SharePoint connections.

Recommended conventions:

- UTF-8 encoding;
- Comma-separated values;
- Header row included;
- ISO date format where applicable: `YYYY-MM-DD`;
- Boolean values represented consistently as `true` and `false`;
- Stable identifiers stored as text;
- Empty optional values left blank rather than replaced with confidential defaults.

---

## Data Dictionary

Detailed field definitions, data types, keys, nullability, and relationships are documented in:

[`data-dictionary.md`](data-dictionary.md)

The data dictionary covers both the reference datasets and the inspection datasets.

---

## Relationship with Other Repository Layers

### Power Apps

The application consumes the reference datasets and creates the inspection records.

See [`../app/`](../app/).

### Data Engineering

The Databricks layer ingests the SharePoint data, applies transformations and data-quality rules, and prepares the analytics dataset.

See [`../data-engineering/`](../data-engineering/).

### Power BI

The reporting layer consumes the curated analytical data produced by the data-engineering process.

See [`../power-bi/`](../power-bi/).

---

## Portfolio Scope

This folder demonstrates the E-SMAT data model and the relationships required by the application, data-engineering, and reporting layers.

The sample files are designed for technical demonstration only. They do not reproduce the complete production volume, internal SharePoint configuration, real inspection history, or proprietary safety and environmental requirement set.
