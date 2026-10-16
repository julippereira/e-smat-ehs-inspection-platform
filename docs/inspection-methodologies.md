# Inspection Methodologies

## Overview

E-SMAT supports three inspection methodologies: EMAT, SMAT, and ESMAT. The application preserves a common inspection structure while routing each record to the questionnaire configured for its methodology, site, and version.

This document explains how the methodologies are represented in the solution. It does not publish proprietary definitions, internal governance rules, or production questionnaire content.

---

## Shared Inspection Structure

All three methodologies use a common lifecycle:

```text
Select inspection context
        ↓
Identify methodology
        ↓
Resolve questionnaire
        ↓
Record question-level observations
        ↓
Complete complementary fields
        ↓
Validate and submit
        ↓
Publish for analysis
```

Shared fields include:

- Inspection identifier;
- Site and operational hierarchy;
- Date and shift;
- Activity and contract type;
- Observer and observed people;
- Safe and unsafe observation quantities;
- Question description;
- Unsafe condition fields;
- Safety Tag fields;
- Near-miss fields;
- Processing and analytical metadata.

---

## EMAT

EMAT is represented as one of the supported inspection methodologies in the application, engineering pipeline, and dashboard.

In the public solution, EMAT records are identified by:

```text
Tipo = EMAT
```

or by an inspection identifier beginning with:

```text
EMAT-
```

The correct questionnaire is resolved using the operational context and active questionnaire configuration.

The public mock contains 24 EMAT inspection records. This quantity exists only to support the public dashboard demonstration and does not represent production activity.

---

## SMAT

SMAT records use the same header-detail architecture while referencing SMAT-specific questionnaire configurations.

Public records are identified by:

```text
Tipo = SMAT
```

or by an identifier beginning with:

```text
SMAT-
```

The public mock contains 32 SMAT inspection records.

---

## ESMAT

ESMAT records follow the same standardized capture and analytical process while using the questionnaire context configured for ESMAT.

Public records are identified by:

```text
Tipo = ESMAT
```

or by an identifier beginning with:

```text
ESMAT-
```

The public mock contains 16 ESMAT inspection records.

---

## Methodology Comparison

| Aspect | EMAT | SMAT | ESMAT |
|---|---|---|---|
| Shared inspection header | Yes | Yes | Yes |
| Contextual questionnaire | Yes | Yes | Yes |
| Safe and unsafe quantities | Yes | Yes | Yes |
| Safety and environmental themes | According to configuration | According to configuration | According to configuration |
| Complementary fields | Yes | Yes | Yes |
| Data-quality validation | Yes | Yes | Yes |
| Power BI analysis | Yes | Yes | Yes |

The methodologies remain distinct analytical categories but share the same governed data architecture.

---

## Questionnaire Resolution

A question identifier should not be interpreted without its questionnaire context. The solution resolves questionnaire metadata using fields such as:

```text
Site
Methodology
Questionnaire version
Question identifier
```

In the Silver transformation, response enrichment considers:

```text
site
question_id
audit_type = methodology
```

This contextual approach reduces the risk of associating a response with the wrong questionnaire definition.

---

## Questionnaire Metadata

The questionnaire reference contains:

```text
Site
Methodology
Questionnaire version
Theme
Category identifier
Category name in Portuguese
Category name in Spanish
Question identifier
Source-column identifier
Question number
Requirement text
Active status
```

A stable requirement key combines:

```text
site
methodology
questionnaire_version
question_id
```

---

## Themes

Questions are normalized into two analytical themes:

```text
safety
```

```text
environment
```

In the public Power BI clone, the display labels are:

```text
Segurança
Meio Ambiente
```

Theme normalization allows equivalent localized or abbreviated source values to be analyzed consistently.

---

## Observation Quantities

Each response contains:

```text
safe_act
unsafe_act
```

The public quality rules accept values between zero and three. At least one classified observation is expected for normal analytical use, although a zero-and-zero condition is treated as a warning rather than an automatic blocking error.

The Gold classification table converts one response into two analytical rows:

```text
safe
unsafe
```

This structure simplifies Power BI comparisons and legends.

---

## Complementary Conditions

The inspection header supports:

### Unsafe condition

```text
unsafe_condition
unsafe_condition_description
```

### Safety Tag

```text
safety_tag_opened
safety_tag_number
```

### Near miss

```text
near_miss
near_miss_description
```

The quality layer validates the consistency between each indicator and dependent field.

---

## Multilingual Support

The Power Apps interface supports Portuguese and Spanish. The application localizes labels and instructions while preserving standardized analytical fields downstream.

Questionnaire metadata can retain localized category names, including:

```text
category_name_pt_br
category_name_es_ar
```

The engineering pipeline normalizes analytical values without removing the localized display fields required by the application or report.

---

## Analytical Representation

The Power BI model uses:

```text
d_cabecalho[Tipo]
```

as the methodology field to preserve compatibility with the cloned report.

The methodology measures are:

```text
EMAT
SMAT
ESMAT
```

They count distinct inspection identifiers in the current filter context.

---

## Public Documentation Boundary

This repository intentionally does not define:

- Formal corporate methodology ownership;
- Proprietary inspection standards;
- Internal procedural requirements;
- Production questionnaire wording;
- Real question-to-category mappings;
- Site-specific governance rules;
- Compliance interpretations.

The purpose is to document how multiple methodologies are implemented technically in one application and analytical architecture.

---

## Related Documentation

```text
business-context.md
technical-decisions.md
data-privacy.md
limitations.md
../power-apps/README.md
../data-engineering/transformations.md
../data-engineering/data-quality.md
../power-bi/measures.md
```
