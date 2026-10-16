# Business Context

## Overview

E-SMAT is an end-to-end solution for standardizing safety and environmental inspections, consolidating field observations, and transforming operational records into information that can support monitoring and decision-making.

The solution connects a multilingual Power Apps application, SharePoint Lists, a Databricks data pipeline, quality controls, and a Power BI dashboard. The public repository documents the complete solution architecture using synthetic data and generalized operational structures.

> The public version describes the solution logic without exposing production records, internal locations, real employee information, credentials, or confidential business rules.

---

## Operational Context

Safety and environmental inspections involve more than capturing a single form. Each inspection must preserve a consistent operational context, route the user to the appropriate questionnaire, record safe and unsafe observations, and retain complementary information when an unsafe condition, Safety Tag, or near miss is reported.

Before E-SMAT, processes of this type can become fragmented across different forms, lists, spreadsheets, languages, and reporting routines. This fragmentation makes it harder to:

- Compare inspection results across sites;
- Maintain consistent questionnaires;
- Track observations by methodology;
- Relate inspection headers to question-level answers;
- Monitor safe and unsafe acts;
- Review environmental and safety themes together;
- Consolidate information for operational teams and leadership;
- Preserve traceability from data capture to reporting.

E-SMAT addresses this need with a standardized digital workflow and a governed analytical pipeline.

---

## Business Problem

The solution was designed around five connected challenges.

### Standardized data capture

Inspection records require common fields for site, area, line, workstation, shift, activity, contract type, observer, observed people, and inspection methodology.

The application standardizes these inputs while preserving the flexibility required by different sites and methodologies.

### Dynamic questionnaire routing

Not every inspection uses the same questionnaire. The correct set of questions depends on the selected operational and methodological context.

E-SMAT applies cascading filters to route the user to the appropriate questionnaire instead of presenting one fixed form for every inspection.

### Multilingual operation

The application supports Portuguese and Spanish interfaces. Labels, instructions, validation messages, and questionnaire structures must remain consistent across languages without duplicating the complete application.

### Reliable analytical data

Operational-list data can contain inconsistent types, localized values, optional fields, duplicated records, and incomplete relationships. The engineering pipeline standardizes and validates this information before it reaches the analytical layer.

### Decision-oriented reporting

The final report must support both executive monitoring and detailed investigation. Users need to move from high-level indicators to inspection-level and employee-level drill through without losing the filter context.

---

## Solution Objectives

E-SMAT aims to:

- Digitize safety and environmental inspections;
- Standardize EMAT, SMAT, and ESMAT workflows;
- Provide Portuguese and Spanish interfaces;
- Route questionnaires according to inspection context;
- Preserve temporary inspection progress across application screens;
- Store inspection headers, responses, and questionnaire metadata separately;
- Validate data before analytical publishing;
- Monitor safe and unsafe acts;
- Compare safety and environmental themes;
- Support operational and leadership reporting;
- Maintain traceability across the complete data lifecycle.

---

## End-to-End Business Flow

```text
Inspection preparation
        ↓
Operational context selection
        ↓
Questionnaire routing
        ↓
Safe and unsafe act recording
        ↓
Complementary safety information
        ↓
Validation and submission
        ↓
SharePoint persistence
        ↓
Databricks ingestion and transformation
        ↓
Data-quality gate
        ↓
Gold analytical tables
        ↓
Power BI monitoring and drill through
```

---

## Main Users

The solution supports different analytical and operational needs.

### Inspection users

Users performing inspections need a clear application flow, contextual questionnaires, field validation, temporary save, and confirmation that the inspection was submitted.

### EHS teams

EHS teams need standardized records, traceability, quality controls, detailed inspection review, and consolidated safe, unsafe, safety, and environmental indicators.

### Operational leadership

Leadership needs summarized indicators, trends, site comparisons, and visibility into inspection activity without reviewing individual source-list records.

### Data and analytics teams

Technical users need reusable ingestion logic, stable schemas, documented relationships, data-quality evidence, analytical tables, and public-safe artifacts for maintenance and portfolio review.

---

## Business Information Captured

E-SMAT separates inspection information into logical groups.

### Inspection context

```text
Inspection methodology
Site
Segment
Business unit or area
Line
Workstation
Shift
Activity type
Contract type
Observer
Observed people
Job role
Inspection date
```

### Question-level observations

```text
Question identifier
Questionnaire context
Safety or environmental theme
Safe-act quantity
Unsafe-act quantity
Description
```

### Complementary safety information

```text
Unsafe condition
Unsafe-condition description
Safety Tag opened
Safety Tag number
Near miss
Near-miss description
```

### Analytical metadata

```text
Processing timestamps
Source system
Source list
Stable business keys
Questionnaire version
Category
Classification
```

---

## Business Outputs

The solution produces:

- Standardized inspection records;
- Question-level observation history;
- Governed questionnaire metadata;
- Safe and unsafe classification records;
- Data-quality summaries and failure details;
- Employee-registration analytical support;
- Power BI indicators and trends;
- Inspection-level drill through;
- Observed-person drill through;
- Public-safe documentation and synthetic datasets.

---

## Analytical Questions Supported

The dashboard helps answer questions such as:

- How many EMAT, SMAT, and ESMAT inspections were recorded?
- How many safe and unsafe acts were observed?
- What is the average number of unsafe acts per inspection?
- Which plants, areas, lines, segments, or workstations concentrate more observations?
- How do safety and environmental themes compare?
- How do inspection and act volumes change over time?
- Which observers recorded inspections?
- How many people were observed in each inspection context?
- What details were recorded for one selected inspection?
- What inspection history is associated with one selected synthetic employee registration?

---

## Business Value

E-SMAT contributes value by:

- Replacing fragmented capture with a standardized process;
- Improving consistency among sites and methodologies;
- Reducing manual consolidation effort;
- Preserving inspection detail and analytical traceability;
- Making quality failures visible before report publishing;
- Supporting operational follow-up and broader trend analysis;
- Combining safety and environmental monitoring in one solution;
- Providing a scalable architecture based on Microsoft technologies.

---

## Public Portfolio Scope

The repository reproduces the solution structure without publishing production content.

The public version includes:

- Application documentation;
- Data-engineering notebooks;
- Synthetic CSV datasets;
- Power BI mock data and screenshots;
- DAX measures;
- Architecture diagrams;
- Data-quality controls;
- Technical and privacy documentation.

The repository does not represent an externally distributable production application. It is a technical portfolio artifact that demonstrates the solution design, data lifecycle, and analytical implementation.

---

## Related Documentation

```text
../architecture/README.md
../power-apps/README.md
../data-engineering/README.md
../power-bi/README.md
inspection-methodologies.md
technical-decisions.md
data-privacy.md
limitations.md
```
