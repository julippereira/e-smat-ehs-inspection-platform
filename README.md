# E-SMAT | Multi-Site EHS Inspection and Analytics Platform

## Overview

E-SMAT is a bilingual, end-to-end inspection management and analytics platform developed to standardize safety and environmental risk inspections across six operational sites in Brazil and Argentina.

The solution supports three inspection methodologies:

- **EMAT — Environment Management Audit Training**
- **SMAT — Safety Management Audit Training**
- **ESMAT — Integrated Environment and Safety Management Audit Training**

Because each operational site follows specific inspection requirements, the application uses a cascade filtering mechanism to identify the inspected location and automatically direct the user to the correct inspection methodology and questionnaire.

The platform combines a bilingual Power Apps application, SharePoint-based operational storage, Python data processing in Azure Databricks, and an interactive Power BI dashboard refreshed hourly.

> This repository presents an anonymized portfolio version of the solution. Organizational identifiers, production connections, internal requirements, real inspection records, and sensitive information were removed, generalized, or replaced with synthetic examples.

---

## Business Challenge

Safety and environmental inspections were conducted across multiple operational sites with different processes, requirements, languages, and inspection methodologies.

Each site required a specific questionnaire based on its operational and regulatory context. Some locations used environmental inspections, others used safety inspections, while two operational sites required an integrated methodology combining both areas.

The solution needed to:

- Standardize safety and environmental inspection records;
- Support different inspection methodologies;
- Direct each user to the correct questionnaire;
- Accommodate site-specific requirements;
- Provide interfaces in Brazilian Portuguese and Argentine Spanish;
- Preserve temporary and incomplete inspection records;
- Centralize operational data;
- Transform raw inspection records into analytics-ready datasets;
- Provide frequently updated management indicators.

---

## Inspection Methodologies

### EMAT

**Environment Management Audit Training**

A structured inspection methodology focused on environmental conditions, controls, risks, and operational practices.

### SMAT

**Safety Management Audit Training**

A structured inspection methodology focused on occupational safety conditions, unsafe acts, controls, and risk prevention practices.

### ESMAT

**Environment and Safety Management Audit Training**

An integrated methodology combining environmental and occupational safety requirements in a single inspection workflow.

The ESMAT methodology is used only at the two operational sites located in the same industrial region, according to their specific business requirements.

---

## Multi-Site Configuration

E-SMAT was designed to serve six operational sites in Brazil and Argentina.

Each site uses a specific inspection methodology and questionnaire according to its local characteristics and predefined business requirements.

To protect organizational information, the public portfolio version represents the locations using anonymized identifiers:

- Site A;
- Site B;
- Site C;
- Site D;
- Site E;
- Site F.

The application dynamically determines the appropriate questionnaire based on the information entered during the location identification stage.

---

## Bilingual User Experience

The application is available in:

- **Brazilian Portuguese — pt-BR**
- **Argentine Spanish — es-AR**

The Spanish version was developed specifically to support the operational site in Argentina.

The localization process covers interface components such as:

- Navigation elements;
- Field labels;
- Instructions;
- Validation messages;
- Inspection requirements;
- Response options;
- Temporary save notifications;
- Submission confirmations.

This approach provides a consistent inspection experience while preserving the terminology required for each local context.

---

## Dynamic Questionnaire Routing

One of the core features of E-SMAT is its cascade filtering mechanism.

At the beginning of an inspection, the user completes a location identification screen. The available options are progressively filtered according to the previous selections.

A simplified version of the navigation logic is:

1. The user selects the operational site;
2. The application displays the corresponding location hierarchy;
3. The user identifies the area or location being inspected;
4. The filtering mechanism evaluates the selected site and location;
5. The application determines the applicable inspection methodology;
6. The user is automatically directed to the correct questionnaire;
7. Only the requirements applicable to that inspection context are displayed.

This mechanism prevents users from selecting an incompatible form and ensures that each inspection follows the requirements established for the selected location.

---

## Business Rules

The application applies predefined business rules to determine:

- Which language should be displayed;
- Which location options are available;
- Which inspection methodology applies to each site;
- Which questionnaire should be loaded;
- Which requirements must be displayed;
- Which fields are mandatory;
- Whether an inspection may be temporarily saved;
- Whether the record is ready for final submission.

By centralizing these rules within the application workflow and supporting data structure, E-SMAT reduces manual interpretation and improves consistency across sites.

---

## Temporary Save Functionality

E-SMAT allows users to save an inspection before completing the final submission.

Temporary records are stored in SharePoint, enabling users to:

- Preserve an inspection in progress;
- Resume data entry later;
- Avoid losing information during longer inspections;
- Complete the questionnaire according to operational availability;
- Distinguish draft records from completed inspections.

The final submission process validates the required information before registering the inspection as completed.

---

## Solution Architecture

The platform follows an end-to-end architecture:

```text
Predefined EHS Requirements
            ↓
Bilingual Power Apps Application
            ↓
Site and Location Identification
            ↓
Cascade Filtering and Business Rules
            ↓
Dynamic Questionnaire Selection
            ↓
EMAT | SMAT | ESMAT
            ↓
SharePoint Lists
Operational Records and Temporary Saves
            ↓
Azure Databricks
Python-Based Ingestion and Transformation
            ↓
Curated Analytics Dataset
            ↓
Power BI Dashboard
Hourly Scheduled Refresh

```
---

## Data Privacy and Anonymization

This repository is an anonymized portfolio representation of a corporate solution.

The public version does not include:

- Real inspection records;
- Employee or user information;
- Internal credentials;
- SharePoint URLs;
- Production connections;
- Databricks workspace or catalog identifiers;
- Proprietary inspection requirements;
- Internal organizational structures;
- Unredacted application screenshots;
- Production Power Apps packages;
- Production Power BI files.


All sample data is synthetic and was created exclusively to demonstrate the technical architecture, business logic, data pipeline, and analytical capabilities of the project.

---

## Disclaimer

This repository is intended exclusively to demonstrate the architecture, technologies, development approach, and analytical capabilities applied in the project.

It does not reproduce the complete production environment, proprietary inspection requirements, or internal organizational processes.
