# Power BI Measures

## Overview

This document describes the DAX measures used in the E-SMAT Power BI report.

The measures are stored in the dedicated `d_medidas` table to keep calculations separate from the model tables. The report uses `d_cabecalho`, `f_formulario`, and `d_classificacao` as the main calculation sources.

> The mock model preserves the column name `d_cabecalho[Tipo]` to remain compatible with the cloned production report and avoid rebuilding existing measures, relationships, and visuals.

---

## Measure Groups

The measures are organized into five logical groups:

```text
Inspection records
Safe and unsafe acts
Averages and rates
Safety and environmental themes
Data refresh information
```

---

# Inspection Records

## EMAT

Counts the distinct EMAT inspection records in the current filter context.

```dax
EMAT =
CALCULATE(
    DISTINCTCOUNT(d_cabecalho[Titulo]),
    d_cabecalho[Tipo] = "EMAT"
)
```

Expected unfiltered result in the public mock:

```text
24
```

---

## ESMAT

Counts the distinct ESMAT inspection records in the current filter context.

```dax
ESMAT =
CALCULATE(
    DISTINCTCOUNT(d_cabecalho[Titulo]),
    d_cabecalho[Tipo] = "ESMAT"
)
```

Expected unfiltered result in the public mock:

```text
16
```

---

## SMAT

Counts the distinct SMAT inspection records in the current filter context.

```dax
SMAT =
CALCULATE(
    DISTINCTCOUNT(d_cabecalho[Titulo]),
    d_cabecalho[Tipo] = "SMAT"
)
```

Expected unfiltered result in the public mock:

```text
32
```

---

## Registros por Matrícula

Calculates the average number of distinct inspection records per observer in the current filter context.

```dax
Registros por Matrícula =
VAR TotalRegistros =
    DISTINCTCOUNT(d_cabecalho[Titulo])

VAR TotalObservadores =
    DISTINCTCOUNT(d_cabecalho[MatriculaObservador])

RETURN
    DIVIDE(
        TotalRegistros,
        TotalObservadores,
        0
    )
```

Although the existing report name is preserved, the measure represents:

```text
Distinct inspection records
÷
Distinct observer registrations
```

When a visual is already grouped by a single observer registration, the denominator normally equals one and the result corresponds to that observer's number of records.

---

# Safe and Unsafe Acts

## AtosSeguros

Returns the total number of safe acts in the current filter context.

```dax
AtosSeguros =
SUM(f_formulario[SA_SafeAct])
```

Expected unfiltered result in the public mock:

```text
1,481
```

---

## AtosInseguros

Returns the total number of unsafe acts in the current filter context.

```dax
AtosInseguros =
SUM(f_formulario[UA_UnsafeAct])
```

Expected unfiltered result in the public mock:

```text
341
```

---

## Atos Seguros por Registro

Calculates the average number of safe acts per distinct inspection record.

```dax
Atos Seguros por Registro =
DIVIDE(
    [AtosSeguros],
    DISTINCTCOUNT(d_cabecalho[Titulo]),
    0
)
```

Calculation:

```text
Safe acts
÷
Distinct inspection records
```

The measure responds to the filters propagated to both `d_cabecalho` and `f_formulario`.

---

## Atos Inseguros por Registro

Calculates the average number of unsafe acts per distinct inspection record.

```dax
Atos Inseguros por Registro =
DIVIDE(
    [AtosInseguros],
    DISTINCTCOUNT(d_cabecalho[Titulo]),
    0
)
```

Calculation:

```text
Unsafe acts
÷
Distinct inspection records
```

Expected unfiltered result in the public mock:

```text
341 ÷ 72 = 4.74
```

The report displays the result according to the measure's configured decimal formatting.

---

# Observed People

## Atos Observados por Matrícula

Calculates the average number of observed acts per person observed in the inspection records included in the current filter context.

```dax
Atos Observados por Matrícula =
VAR TotalAtos =
    SUM(d_classificacao[Ato])

VAR TotalPessoasObservadas =
    SUM(d_cabecalho[NumeroObservados])

RETURN
    DIVIDE(
        TotalAtos,
        TotalPessoasObservadas,
        0
    )
```

Calculation:

```text
Safe and unsafe acts
÷
Number of valid observed-person registrations
```

The measure depends on the custom `NumeroObservados` column created in `d_cabecalho`.

That column counts valid values across:

```text
MatriculaObservado
MatriculaObservado2
MatriculaObservado3
```

The observer registration is not included in this denominator.

### Required Power Query column

```powerquery
#"Número de observados adicionado" =
    Table.AddColumn(
        #"Etapa Anterior",
        "NumeroObservados",
        each
            List.Count(
                List.Select(
                    {
                        [MatriculaObservado],
                        [MatriculaObservado2],
                        [MatriculaObservado3]
                    },
                    each
                        let
                            valor =
                                try Text.Trim(Text.From(_))
                                otherwise ""
                        in
                            valor <> ""
                )
            ),
        Int64.Type
    )
```

Replace `#"Etapa Anterior"` with the actual previous step name in the query.

---

# Theme Measures

The report separates acts associated with environmental and safety questions.

The measures use `SUM`, rather than `COUNTA`, because the analytical objective is to total act quantities. `COUNTA` would count populated response rows, including rows whose act quantity equals zero.

## SA ENV

Returns safe acts associated with environmental questions.

```dax
SA ENV =
CALCULATE(
    [AtosSeguros],
    f_formulario[Tema] = "Meio Ambiente"
)
```

---

## SA SAF

Returns safe acts associated with safety questions.

```dax
SA SAF =
CALCULATE(
    [AtosSeguros],
    f_formulario[Tema] = "Segurança"
)
```

---

## UA ENV

Returns unsafe acts associated with environmental questions.

```dax
UA ENV =
CALCULATE(
    [AtosInseguros],
    f_formulario[Tema] = "Meio Ambiente"
)
```

---

## UA SAF

Returns unsafe acts associated with safety questions.

```dax
UA SAF =
CALCULATE(
    [AtosInseguros],
    f_formulario[Tema] = "Segurança"
)
```

---

# Data Refresh Information

## Ultima_Atualizacao

Displays the latest update timestamp available in `d_cabecalho`.

```dax
Ultima_Atualizacao =
VAR DataAtualizacao =
    MAX(d_cabecalho[DataAtualizacao])

RETURN
    "Dados atualizados em: "
        & FORMAT(
            DataAtualizacao,
            "dd/MM/yyyy HH:mm"
        )
```

Expected result in the public mock:

```text
Dados atualizados em: 15/10/2026 15:00
```

The `DataAtualizacao` column must use the `Date/Time` data type.

---

## Measure Dependencies

```text
d_cabecalho
├── Titulo
├── Tipo
├── MatriculaObservador
├── NumeroObservados
└── DataAtualizacao

f_formulario
├── SA_SafeAct
├── UA_UnsafeAct
└── Tema

d_classificacao
└── Ato
```

The measures also depend on filters reaching these tables through the active relationship path:

```text
d_calendario
      ↓
d_cabecalho
      ↓
f_formulario
      ↓
d_classificacao
```

The direct relationship between `d_calendario` and `d_classificacao` can remain inactive because the calendar filter already reaches the classification table through the main relationship path.

---

## Recommended Formatting

Configure the measures in Power BI as follows:

```text
EMAT                              Whole number
ESMAT                             Whole number
SMAT                              Whole number
AtosSeguros                       Whole number
AtosInseguros                     Whole number
Atos Seguros por Registro         Decimal number, 2 decimal places
Atos Inseguros por Registro       Decimal number, 2 decimal places
Atos Observados por Matrícula     Decimal number, 2 decimal places
Registros por Matrícula           Decimal number, 2 decimal places
SA ENV                            Whole number
SA SAF                            Whole number
UA ENV                            Whole number
UA SAF                            Whole number
Ultima_Atualizacao                Text
```

Thousands separators can be enabled for act totals. Average and ratio measures should display two decimal places unless a visual requires a more concise format.

---

## Validation Totals

Without report filters, the public mock should return:

```text
EMAT                           24
ESMAT                          16
SMAT                           32
Total inspection records       72
AtosSeguros                 1,481
AtosInseguros                 341
Total acts                  1,822
Atos Inseguros por Registro   4.74
```

The following reconciliation should also hold:

```text
[SA ENV] + [SA SAF] = [AtosSeguros]
```

```text
[UA ENV] + [UA SAF] = [AtosInseguros]
```

These identities are useful for confirming that the theme labels and filter propagation remain consistent after replacing the report sources.

---

## Naming Compatibility

The measure names were preserved because the cloned report already references them in cards, charts, matrices, tooltips, and drill-through pages.

The public mock also preserves:

```text
d_cabecalho[Tipo]
```

instead of renaming the field to `TipoAuditoria` inside the semantic model. This avoids unnecessary changes to existing DAX and visual configurations.

---

## Report Usage

The measures support the visible and hidden report pages:

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

The principal usage pattern is:

- Record measures for inspection KPIs and distributions;
- Act measures for safe/unsafe comparisons;
- Theme measures for safety and environmental breakdowns;
- Per-record measures for normalized comparisons;
- Per-person measures for the People and registration drill-through analyses;
- Refresh text for the report update indicator.

---

## Portfolio Notes

The measures operate on synthetic data and preserve the calculation patterns of the cloned report. The public repository does not expose production values, employee names, internal organizational identifiers, or proprietary inspection records.
