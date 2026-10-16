# E-SMAT Power BI Dashboard

## Overview

This folder contains the public Power BI layer of E-SMAT, an inspection solution that consolidates EMAT, SMAT, and ESMAT records into an interactive analytical report.

The public report is a clone of the operational dashboard connected to synthetic data. The semantic model, calculations, navigation, visible pages, and hidden drill-through pages were preserved whenever possible, while production records, employee information, organizational identifiers, and infrastructure references were replaced with public-safe mock content.

The dashboard supports analysis of:

- Inspection volume by methodology and operational context;
- Safe and unsafe acts;
- Safety and environmental themes;
- Plants, segments, areas, lines, and workstations;
- Observer and observed-person participation;
- Job roles and synthetic employees;
- Inspection-level and employee-level drill through;
- Monthly, weekly, and daily trends.

> All data displayed in the public report is synthetic. The repository does not contain production inspection records, real employee information, corporate credentials, or confidential organizational structures.

---

## Dashboard Preview

![E-SMAT overview dashboard](./screenshots/01-overview.png)

The public dashboard contains six visible pages and two hidden drill-through pages:

```text
Visão Geral
Registros
Atos Observados
Pessoas
Cargo
Consultar
DT_Registro
DT_Matricula
```

Detailed page descriptions are available in [`dashboard-pages.md`](dashboard-pages.md).

---

## Power BI Architecture

```text
Synthetic CSV sources
        ↓
Power Query transformations
        ↓
Semantic model
        ↓
DAX measures
        ↓
Visible analytical pages
        ↓
Hidden drill-through pages
```

The principal active filter path is:

```text
d_calendario
      ↓
d_cabecalho
      ↓
f_formulario
      ↓
d_classificacao
```

Questionnaire and employee dimensions provide additional analytical context:

```text
d_questionario
      ↓
f_formulario
```

```text
d_lista_matriculas
      ↓
d_cabecalho
```

![E-SMAT Power BI data model](./screenshots/09-data-model.png)

The complete semantic-model documentation is available in [`data-model.md`](data-model.md).

---

## Folder Structure

```text
power-bi/
├── README.md
├── dashboard-pages.md
├── data-model.md
├── measures.md
├── key-validation-report.md
│
├── mock-data/
│   ├── d_cabecalho.csv
│   ├── f_formulario.csv
│   ├── d_questionario.csv
│   ├── d_classificacao.csv
│   ├── d_calendario.csv
│   └── d_lista_matriculas.csv
│
├── screenshots/
│   ├── README.md
│   ├── 01-overview.png
│   ├── 02-records.png
│   ├── 03-observed-acts.png
│   ├── 04-people.png
│   ├── 05-job-roles.png
│   ├── 06-record-search.png
│   ├── 07-drill-through-record.png
│   ├── 08-drill-through-employee.png
│   └── 09-data-model.png
│
└── theme/
    └── esmat-theme.json
```

---

## Report Pages

| Page | Visibility | Analytical purpose |
|---|---|---|
| **Visão Geral** (Overview) | Visible | Executive KPIs, themes, site comparisons, and monthly trends |
| **Registros** (Records) | Visible | Inspection volume by operational hierarchy and date |
| **Atos Observados** (Observed Acts) | Visible | Safe and unsafe acts by plant, segment, area, line, and workstation |
| **Pessoas** (People) | Visible | Observer and observed-person analyses |
| **Cargo** (Job Role) | Visible | Indicators by job role and synthetic employee |
| **Consultar** (Record Search) | Visible | Searchable inspection-level detail and navigation |
| **DT_Registro** (Drill-Through Record) | Hidden | Complete context and question detail for one inspection |
| **DT_Matricula** (Drill-Through Employee) | Hidden | Inspection history and observed acts for one selected registration |

The screenshots and detailed visual descriptions are documented in [`dashboard-pages.md`](dashboard-pages.md).

---

## Semantic Model

The public model contains seven logical tables:

| Table | Role | Grain |
|---|---|---|
| `d_cabecalho` | Inspection header | One row per inspection |
| `f_formulario` | Question-level fact | One row per inspection and question |
| `d_questionario` | Questionnaire dimension | One row per question context |
| `d_classificacao` | Safe/unsafe classification fact | Two rows per response |
| `d_calendario` | Date dimension | One row per calendar day |
| `d_lista_matriculas` | Employee registration dimension | One row per registration |
| `d_medidas` | DAX measure container | No analytical grain |

### Principal relationships

```text
d_calendario[id_data]
        1 ─── *
d_cabecalho[Data]
```

```text
d_cabecalho[Titulo]
        1 ─── *
f_formulario[Titulo]
```

```text
d_questionario[Id_Questao]
        1 ─── *
f_formulario[ID_Questao]
```

```text
f_formulario[ChaveUnica]
        1 ─── *
d_classificacao[ChaveUnica]
```

The direct date relationships to downstream tables remain inactive when they would create ambiguous filter paths. The cloned report preserves the relationship structure already used by the implemented visuals and calculations.

See [`data-model.md`](data-model.md) for table definitions, data types, active and inactive relationships, filter propagation, employee-role handling, and drill-through dependencies.

---

## Measures

The report uses a dedicated `d_medidas` table to organize DAX calculations.

The principal measures include:

```text
EMAT
ESMAT
SMAT
AtosSeguros
AtosInseguros
Atos Seguros por Registro
Atos Inseguros por Registro
Atos Observados por Matrícula
Registros por Matrícula
SA ENV
SA SAF
UA ENV
UA SAF
Ultima_Atualizacao
```

The public mock returns the following unfiltered totals:

```text
EMAT                           24
ESMAT                          16
SMAT                           32
Total inspection records       72
Safe acts                   1,481
Unsafe acts                   341
Total acts                  1,822
Unsafe acts per record        4.74
```

The complete DAX definitions, dependencies, formatting, and reconciliation rules are documented in [`measures.md`](measures.md).

---

## Mock Data

The `mock-data/` folder reproduces the schema expected by the cloned PBIX.

| File | Rows | Purpose |
|---|---:|---|
| `d_cabecalho.csv` | 72 | Synthetic inspection headers |
| `f_formulario.csv` | 952 | Synthetic question-level responses |
| `d_questionario.csv` | 119 | Synthetic questionnaire metadata |
| `d_classificacao.csv` | 1,904 | Safe and unsafe classification rows |
| `d_calendario.csv` | 365 | Complete 2026 calendar |
| `d_lista_matriculas.csv` | 93 | Synthetic employee registrations and names |

The mock tables preserve:

- Expected table and column names;
- Relationship keys;
- Record grains;
- Methodology totals;
- Safe and unsafe act totals;
- Calendar coverage;
- Employee registration references;
- Conditional inspection fields;
- Questionnaire categories and questions.

The production report uses governed enterprise sources. The CSV files exist only to make the public portfolio version reviewable without production access.

---

## Key Validation

The mock model was checked through 31 key and relationship controls.

```text
Overall status      PASS
Checks executed       31
Passed                31
Failed                 0
Blocking failures      0
```

The validation confirms that:

- Primary keys are unique and populated;
- Every response references a valid inspection;
- Every response references a valid question;
- Every response has one `safe` and one `unsafe` classification row;
- Classification quantities reconcile with the response fact;
- All used dates exist in the calendar dimension;
- All referenced registrations exist in `d_lista_matriculas`;
- No orphan relationship keys were found.

See [`key-validation-report.md`](key-validation-report.md) for the complete result.

---

## Calendar and Time Analysis

`d_calendario` covers all days from January 1, 2026 through December 31, 2026.

The dimension supports:

```text
Year
Month
Period
ISO week
Day
Day of week
Quarter
Semester
```

Examples:

```text
ano_mes        2026-04
ano_semana     2026-14
periodo        abr-2026
semestre_desc  1º Semestre
trimestre_desc 2º Trimestre
```

The table is used for monthly, weekly, and daily analysis throughout the report.

---

## Employee Registration Analysis

`d_lista_matriculas` consolidates the registrations used in:

```text
MatriculaObservador
MatriculaObservado
MatriculaObservado2
MatriculaObservado3
```

The public dimension contains synthetic employee names and stable sorting fields.

The model also creates `NumeroObservados` in Power Query by counting valid registrations across the three observed-person columns. This field supports normalized measures on the Pessoas and DT_Matricula pages.

A separate auxiliary Databricks notebook, `06_employee_dimension.ipynb`, documents a future normalized approach using an employee dimension and inspection-employee bridge.

---

## Theme

The report visual identity is stored in:

```text
theme/esmat-theme.json
```

The theme centralizes the public dashboard color palette and visual defaults so the PBIX can preserve a consistent appearance across cards, charts, matrices, filters, and drill-through pages.

---

## Screenshots

The `screenshots/` folder contains the six visible report pages, two hidden drill-through pages, and the semantic-model view.

```text
01-overview.png
02-records.png
03-observed-acts.png
04-people.png
05-job-roles.png
06-record-search.png
07-drill-through-record.png
08-drill-through-employee.png
09-data-model.png
```

The image inventory is documented in [`screenshots/README.md`](screenshots/README.md).

---

## Compatibility Decisions

The public report prioritizes compatibility with the cloned PBIX rather than redesigning the semantic model or visuals.

The following structures were intentionally preserved:

- Existing table names;
- Existing measure names;
- `d_cabecalho[Tipo]` as the methodology field;
- `Titulo` as the header-to-response key;
- `ChaveUnica` as the response-to-classification key;
- Existing drill-through pages and filter behavior;
- Existing active and inactive relationships;
- Existing page layout and visual structure;
- Existing employee-role logic;
- Existing filter dimensions.

The mock contains more synthetic segment values than the original report layout was designed to display. Some labels are therefore truncated, but the underlying values and calculations remain valid. The layout was intentionally not redesigned for this portfolio adaptation.

---

## Security and Confidentiality

The public Power BI layer does not include:

- Real employee names or registrations;
- Production inspection records;
- Internal plant, area, line, workstation, or segment names;
- Corporate email addresses;
- Production URLs or workspace identifiers;
- Catalog or schema names;
- Credentials, access tokens, or configuration files;
- Confidential questionnaire text;
- Production quality results.

All visible values come from synthetic datasets created specifically for the public portfolio.

---

## Documentation Map

```text
README.md
├── dashboard-pages.md
├── data-model.md
├── measures.md
├── key-validation-report.md
└── screenshots/README.md
```

- [`dashboard-pages.md`](dashboard-pages.md): visible and hidden report pages;
- [`data-model.md`](data-model.md): semantic model, relationships, data types, and filter paths;
- [`measures.md`](measures.md): DAX definitions and expected results;
- [`key-validation-report.md`](key-validation-report.md): mock-data integrity checks;
- [`screenshots/README.md`](screenshots/README.md): image inventory and documentation references.

---

## Portfolio Scope

This folder demonstrates the reporting layer of an end-to-end E-SMAT solution that connects Power Apps, SharePoint, Microsoft Graph API, Databricks, Delta Lake, data-quality controls, and Power BI.

The public Power BI implementation highlights:

- Semantic-model design;
- Header-detail relationships;
- Questionnaire enrichment;
- Safe and unsafe classification analysis;
- DAX measures;
- Time intelligence attributes;
- Employee registration analysis;
- Active and inactive relationship management;
- Drill-through navigation;
- Synthetic-data validation;
- Theme-based visual consistency;
- Public-safe portfolio documentation.
