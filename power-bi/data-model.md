# Power BI Data Model

## Overview

The E-SMAT Power BI model organizes inspection records, question-level responses, questionnaire metadata, safe and unsafe act classifications, calendar attributes, and employee registration references.

The public report is a clone of the production analytical model connected to synthetic CSV files. Table and column names were preserved when practical so that existing relationships, measures, visuals, filters, and drill-through pages could continue to operate without extensive rework.

The model contains:

- One inspection header table;
- One question-level response fact table;
- One questionnaire dimension;
- One safe/unsafe classification table;
- One calendar dimension;
- One auxiliary employee registration dimension;
- One dedicated measure table.

> All public records, employee registrations, employee names, locations, questionnaire content, and operational identifiers are synthetic.

---

## Model Diagram

The following screenshot shows the semantic model implemented in Power BI, including the tables, relationship cardinalities, and active or inactive filter paths used by the public report.

![Power BI data model](screenshots/09-data-model.png)

> The screenshot is captured directly from Power BI Desktop after the mock data sources and relationships are configured. The textual diagram below provides an accessible and repository-friendly representation of the same model.

### Textual representation

```text
                                      d_questionario
                                      [Id_Questao]
                                            1
                                            │
                                            │ *
                                            ▼
 d_calendario                    f_formulario                    d_classificacao
 [id_data]                       [ChaveUnica]                    [ChaveUnica]
      1                                *   1                           *
      │                                ▲   │                           ▲
      │ *                              │   └───────────────────────────┘
      ▼                                │
 d_cabecalho                           │
 [Titulo] 1 ───────────────────────────┘
      *
      ▲
      │
      │ 1
 d_lista_matriculas
 [Matricula]

 d_medidas
 └── Dedicated measure table without a data relationship
```

The primary active filter path is:

```text
d_calendario
      ↓
d_cabecalho
      ↓
f_formulario
      ↓
d_classificacao
```

This path allows time filters to reach the classification table without requiring a second active relationship between `d_calendario` and `d_classificacao`.

---

## Tables

| Table | Role | Grain | Public source |
|---|---|---|---|
| `d_cabecalho` | Inspection header | One row per inspection | `mock-data/d_cabecalho.csv` |
| `f_formulario` | Response fact | One row per inspection and question | `mock-data/f_formulario.csv` |
| `d_questionario` | Questionnaire dimension | One row per question context | `mock-data/d_questionario.csv` |
| `d_classificacao` | Classification fact | Two rows per response | `mock-data/d_classificacao.csv` |
| `d_calendario` | Date dimension | One row per calendar day | `mock-data/d_calendario.csv` |
| `d_lista_matriculas` | Employee registration dimension | One row per registration | `mock-data/d_lista_matriculas.csv` |
| `d_medidas` | Measure container | No analytical row grain | Created inside Power BI |

---

# Core Tables

## `d_cabecalho`

### Purpose

`d_cabecalho` contains the context shared by all question-level answers for one inspection. The table stores the inspection date, methodology, operational location, observer, observed people, complementary safety fields, and source metadata.

### Grain

```text
One row per Titulo
```

### Key columns

| Column | Purpose | Recommended type |
|---|---|---|
| `Titulo` | Inspection business key and relationship key | Text |
| `ChaveUnica` | Equivalent inspection key retained for compatibility | Text |
| `Data` | Inspection date | Date |
| `Tipo` | Inspection methodology: EMAT, SMAT, or ESMAT | Text |

In the mock dataset:

```text
Titulo = ChaveUnica
```

The existing semantic model uses `Tipo`, rather than `TipoAuditoria`, to preserve compatibility with the cloned report.

### Operational hierarchy

```text
Planta
Segmento
BUarea
Linha
Posto
```

These columns support the report filters and the Registros, Atos Observados, Pessoas, Cargo, Consultar, and drill-through pages.

### Employee columns

```text
MatriculaObservador
MatriculaObservado
MatriculaObservado2
MatriculaObservado3
Cargo
Author
```

Registrations are stored as text, even when they contain only numbers. This avoids mathematical aggregation, formatting with thousands separators, and loss of possible leading zeros.

### Complementary safety columns

```text
OptCondicaoInsegura
DscCondicaoInsegura
TagAberta
NumeroTag
OptQuaseAcidente
DscQuaseAcidente
```

The paired indicator and description fields are used on the Consultar and DT_Registro pages.

### Derived column: `NumeroObservados`

`NumeroObservados` counts valid employee registrations in:

```text
MatriculaObservado
MatriculaObservado2
MatriculaObservado3
```

The observer is not included in this count.

The column is created in Power Query and is used by the `Atos Observados por Matrícula` measure.

Recommended type:

```text
Whole number
```

---

## `f_formulario`

### Purpose

`f_formulario` is the central question-level fact table. It stores the safe and unsafe quantities recorded for each question within an inspection.

### Grain

```text
One row per Titulo and ID_Questao
```

### Key columns

| Column | Purpose | Recommended type |
|---|---|---|
| `ChaveUnica` | Unique response key | Text |
| `Titulo` | Foreign key to `d_cabecalho` | Text |
| `ID_Questao` | Foreign key to `d_questionario` | Text |
| `Data` | Inspection date copied to the response | Date |

`ChaveUnica` is constructed conceptually as:

```text
Titulo + ID_Questao
```

### Measures and attributes

```text
SA_SafeAct
UA_UnsafeAct
Tema
Descricao
Planta
Tipo
Author
DataAtualizacao
```

`SA_SafeAct` and `UA_UnsafeAct` must be whole numbers. `Tema` contains `Segurança` or `Meio Ambiente`.

### Model role

The table connects the inspection header, questionnaire metadata, and classification fact:

```text
d_cabecalho
      ↓
f_formulario
      ↓
d_classificacao
```

---

## `d_questionario`

### Purpose

`d_questionario` provides descriptive questionnaire metadata for the response fact. It supplies question text, category, theme, methodology, version, and display sequence.

### Grain

```text
One row per questionnaire question context
```

Question identifiers in the public mock are unique across questionnaire contexts, allowing the existing model to use a direct one-to-many relationship on `Id_Questao`.

### Key columns

| Column | Purpose | Recommended type |
|---|---|---|
| `Titulo` | Unique questionnaire record key | Text |
| `Id_Questao` | Relationship key to `f_formulario[ID_Questao]` | Text |
| `Num_Questao` | Display sequence inside the questionnaire | Whole number |

### Descriptive columns

```text
Planta
Tipo
Versao
Tema
Id_Categoria
Nome_categoria
Nome_categoria_es
Id_Coluna
Dsc_Questao
Ativo
DataAtualizacao
```

`Ativo` remains text in the public clone because the mock uses `Sim` and the existing report schema is preserved.

### Hierarchical use

The DT_Registro page uses questionnaire fields in a hierarchy similar to:

```text
Nome_categoria
      ↓
Dsc_Questao
```

This allows the selected inspection to display question-level safe acts, unsafe acts, and descriptions grouped by category.

---

## `d_classificacao`

### Purpose

`d_classificacao` converts the two quantity columns from `f_formulario` into classification rows suitable for chart legends and category comparisons.

### Grain

```text
Two rows per f_formulario[ChaveUnica]
```

Each response produces:

```text
One safe row
One unsafe row
```

### Key columns

| Column | Purpose | Recommended type |
|---|---|---|
| `ChaveClassificacao` | Unique classification row key | Text |
| `ChaveUnica` | Foreign key to `f_formulario` | Text |
| `Classificacao` | `safe` or `unsafe` | Text |
| `Ato` | Safe or unsafe act quantity | Whole number |

### Context columns

```text
Data
ID_Questao
Tema
Titulo
Planta
Tipo
DataAtualizacao
```

These duplicated context fields support visuals and troubleshooting but do not replace the main relationship path.

### Classification transformation

```text
f_formulario row
├── SA_SafeAct   → classification = safe
└── UA_UnsafeAct → classification = unsafe
```

The public mock contains exactly two classification records for every response.

---

# Dimensions

## `d_calendario`

### Purpose

`d_calendario` provides day, week, month, quarter, semester, and year attributes for time analysis.

### Grain

```text
One row per date
```

The public mock covers:

```text
01/01/2026 to 31/12/2026
```

### Columns and types

| Column | Description | Recommended type |
|---|---|---|
| `ano` | Calendar year | Whole number |
| `ano_mes` | Year-month key, such as `2026-04` | Text |
| `ano_semana` | ISO year-week key | Text |
| `dia` | Day of month | Whole number |
| `dia_semana_num` | Monday = 1 through Sunday = 7 | Whole number |
| `id_data` | Calendar date key | Date |
| `mes` | Month number | Whole number |
| `nome_dia_semana` | Day name in Portuguese | Text |
| `nome_mes` | Full month name | Text |
| `nome_mes_abrev` | Abbreviated month name | Text |
| `periodo` | Month abbreviation and year, such as `abr-2026` | Text |
| `semana_ano` | ISO week number | Whole number |
| `semestre` | Semester number | Whole number |
| `semestre_desc` | Semester label | Text |
| `trimestre` | Quarter number | Whole number |
| `trimestre_desc` | Quarter label | Text |

### Sort-by-column settings

Configure:

```text
nome_mes          → mes
nome_mes_abrev    → mes
nome_dia_semana   → dia_semana_num
semestre_desc     → semestre
trimestre_desc    → trimestre
```

Because the current mock contains only 2026, `periodo` can be ordered by `mes`. For a multi-year model, use a numeric year-month order column or `ano_mes`.

### Date table configuration

In Power BI, mark `d_calendario` as the model date table and select:

```text
d_calendario[id_data]
```

---

## `d_lista_matriculas`

### Purpose

`d_lista_matriculas` consolidates registrations referenced by the observer and observed-person columns. The table supplies employee display names and stable ordering for the Pessoas, Cargo, and DT_Matricula pages.

### Grain

```text
One row per Matricula
```

### Columns and types

| Column | Description | Recommended type |
|---|---|---|
| `Matricula` | Employee registration key | Text |
| `NomeFuncionario` | Synthetic or authorized employee display name | Text |
| `Planta` | Latest associated site context | Text |
| `Turno` | Latest associated shift context | Text |
| `OrdemMatricula` | Stable numeric display order | Whole number |
| `FuncoesInspecao` | Roles found across the header columns | Text |
| `QuantidadeInspecoes` | Distinct related inspections | Whole number |
| `DataUltimaInspecao` | Latest related inspection date | Date |
| `StatusEnriquecimento` | Synthetic or enrichment status | Text |

Configure:

```text
Matricula       → Sort by OrdemMatricula
NomeFuncionario → Sort by OrdemMatricula
```

### Future enrichment

In the production workflow, `NomeFuncionario` can be populated from an authorized employee reference source. The public table uses clearly synthetic names.

### Current relationship strategy

The cloned model can maintain the active relationship to the employee role used by the report, commonly:

```text
d_lista_matriculas[Matricula]
        1 ─── *
d_cabecalho[MatriculaObservador]
```

Relationships to the observed-person columns may be inactive when needed:

```text
d_cabecalho[MatriculaObservado]
d_cabecalho[MatriculaObservado2]
d_cabecalho[MatriculaObservado3]
```

For a future normalized model, `inspection_employee_bridge` from `06_employee_dimension.ipynb` can replace the role-playing relationships.

---

## `d_medidas`

### Purpose

`d_medidas` is a dedicated measure container. The table keeps DAX calculations organized and separate from data columns.

The table has no analytical relationship to the remaining model.

Measures are documented in:

[`measures.md`](measures.md)

---

# Relationships

## Recommended Active Relationships

| From | To | Cardinality | Filter direction | Status |
|---|---|---|---|---|
| `d_calendario[id_data]` | `d_cabecalho[Data]` | One-to-many | Single | Active |
| `d_cabecalho[Titulo]` | `f_formulario[Titulo]` | One-to-many | Single | Active |
| `d_questionario[Id_Questao]` | `f_formulario[ID_Questao]` | One-to-many | Single | Active |
| `f_formulario[ChaveUnica]` | `d_classificacao[ChaveUnica]` | One-to-many | Single | Active |
| `d_lista_matriculas[Matricula]` | Employee role used by the report | One-to-many | Single | Active |

Use single-direction filtering from the one side toward the many side whenever possible.

---

## Inactive Relationships

The model can retain additional inactive relationships for specific calculations or drill-through behavior.

### Calendar to response and classification dates

```text
d_calendario[id_data] ┄┄┄ f_formulario[Data]
d_calendario[id_data] ┄┄┄ d_classificacao[Data]
```

These relationships should remain inactive because the active filter path already reaches both tables through `d_cabecalho`.

Activating the direct calendar-to-classification relationship creates two filter paths:

```text
d_calendario
→ d_cabecalho
→ f_formulario
→ d_classificacao
```

and:

```text
d_calendario
→ d_classificacao
```

Power BI blocks the second active relationship because the filter path would be ambiguous.

When a calculation must use the direct classification date, the inactive relationship can be activated inside a measure with `USERELATIONSHIP`.

### Employee role relationships

Additional employee relationships may remain inactive:

```text
d_lista_matriculas[Matricula] ┄┄┄ d_cabecalho[MatriculaObservado]
d_lista_matriculas[Matricula] ┄┄┄ d_cabecalho[MatriculaObservado2]
d_lista_matriculas[Matricula] ┄┄┄ d_cabecalho[MatriculaObservado3]
```

The appropriate relationship can be activated in a measure or replaced later with a normalized bridge.

---

## Relationship Rules

Related columns must use compatible data types:

```text
d_calendario[id_data]         Date
d_cabecalho[Data]             Date

d_cabecalho[Titulo]           Text
f_formulario[Titulo]          Text

d_questionario[Id_Questao]    Text
f_formulario[ID_Questao]      Text

f_formulario[ChaveUnica]       Text
d_classificacao[ChaveUnica]   Text

d_lista_matriculas[Matricula] Text
d_cabecalho employee columns  Text
```

A numeric-looking registration should not be loaded as a number.

---

# Filter Propagation

## Main path

A filter applied to `d_calendario` should propagate as follows:

```text
Calendar date
      ↓
Inspection headers
      ↓
Question responses
      ↓
Safe and unsafe classification rows
```

This supports monthly and weekly charts without activating redundant date relationships.

## Questionnaire filtering

A category, question, or questionnaire attribute from `d_questionario` filters `f_formulario` directly.

Whether that filter should continue from `f_formulario` to `d_classificacao` depends on the active direction of the response-classification relationship. The recommended path is from `f_formulario` to `d_classificacao`.

## Employee filtering

The active employee relationship determines the role filtered automatically by `d_lista_matriculas`.

The DT_Matricula page must use the relationship or filter logic that corresponds to the observed-person field used by that page. The clone can preserve the existing behavior rather than redesigning the employee model.

---

# Drill-Through Dependencies

## `DT_Registro`

Recommended drill-through field:

```text
d_cabecalho[Titulo]
```

The page depends on:

```text
d_cabecalho
f_formulario
d_questionario
```

The selected inspection filters its header context and all related question responses.

## `DT_Matricula`

Recommended drill-through field:

```text
d_lista_matriculas[Matricula]
```

The page depends on:

```text
d_lista_matriculas
d_cabecalho
f_formulario
d_classificacao
```

The existing page logic should be preserved because the clone already implements the appropriate employee-role context.

---

# Model Validation

The mock tables were validated before import.

## Row counts

| Table | Rows |
|---|---:|
| `d_cabecalho` | 72 |
| `f_formulario` | 952 |
| `d_questionario` | 119 |
| `d_classificacao` | 1,904 |
| `d_calendario` | 365 |
| `d_lista_matriculas` | 93 |

## Validated conditions

- All primary keys are unique and populated;
- Every response has a valid inspection header;
- Every response has a valid questionnaire question;
- Every response has exactly one safe and one unsafe classification row;
- Classification quantities reconcile with the response quantities;
- Every used date exists in the calendar;
- Every employee registration in the four header columns exists in the registration dimension;
- No orphan relationship keys were found.

The detailed report is available in:

[`key-validation-report.md`](key-validation-report.md)

---

# Model Compatibility Decisions

The public model intentionally preserves several production-compatible names and structures:

- `d_cabecalho[Tipo]` is retained instead of `TipoAuditoria`;
- Existing table names are preserved;
- Existing measure names are preserved;
- `Titulo` remains the header-to-response relationship key;
- `ChaveUnica` remains the response-to-classification relationship key;
- Employee registrations remain in the four header columns;
- Hidden drill-through pages retain their current filter logic;
- Redundant date relationships remain inactive;
- Synthetic segment cardinality is not used to redesign visual layouts.

These decisions minimize changes to the cloned PBIX while keeping the public data anonymous and internally consistent.

---

# Public Data Sources

```text
mock-data/
├── d_cabecalho.csv
├── f_formulario.csv
├── d_questionario.csv
├── d_classificacao.csv
├── d_calendario.csv
└── d_lista_matriculas.csv
```

The files reproduce the semantic structure expected by the report. The production report uses governed enterprise sources rather than these CSV files.

---

# Portfolio Scope

The public semantic model demonstrates:

- Header-detail modeling;
- A question-level inspection fact;
- Questionnaire metadata enrichment;
- Safe and unsafe classification fan-out;
- Date intelligence attributes;
- Employee registration consolidation;
- Active and inactive relationship management;
- Deterministic filter propagation;
- Drill-through support;
- DAX measure separation;
- Synthetic data validation;
- Compatibility-focused report cloning.
