# E-SMAT Application

## About This Folder

This folder documents the Power Apps component of the E-SMAT platform.

E-SMAT is a bilingual application developed to standardize the execution and registration of safety and environmental inspections across multiple operational sites in Brazil and Argentina.

The application supports three inspection methodologies:

- **SMAT — Safety Management Audit Training**
- **EMAT — Environment Management Audit Training**
- **ESMAT — Integrated Environment and Safety Management Audit Training**

Each operational site uses a specific inspection methodology and questionnaire according to its local requirements.

To support these differences without creating separate applications, E-SMAT uses SharePoint reference lists, business rules, and cascading filters to dynamically direct users to the appropriate questionnaire.

## Application Objectives

The Power Apps solution was designed to:

- Standardize safety and environmental inspection records;
- Support different inspection methodologies in a single application;
- Accommodate site-specific forms and requirements;
- Provide interfaces in Brazilian Portuguese and Argentine Spanish;
- Guide users through a structured location identification process;
- Automatically select the appropriate inspection questionnaire;
- Allow inspections to be temporarily saved and resumed;
- Validate required information before final submission;
- Centralize inspection records in SharePoint Lists.

## Core Application Features

### Bilingual Interface

The application supports:

- Brazilian Portuguese (`pt-BR`);
- Argentine Spanish (`es-AR`).

Interface elements and operational terminology are retrieved according to the applicable language configuration.

### Multi-Site Configuration

E-SMAT supports six operational sites located in Brazil and Argentina.

Each site has its own configuration based on:

- Local inspection methodology;
- Operational structure;
- Location hierarchy;
- Applicable questionnaire;
- Safety and environmental requirements;
- Language requirements.

In this public portfolio repository, real site names and organizational structures are anonymized.

### Cascading Location Filters

The inspection starts with the identification of the inspected location.

The available location options are progressively filtered according to the user's previous selections. This ensures that only valid combinations from the operational hierarchy can be selected.

The location selection is also used to determine which inspection questionnaire should be displayed.

### Dynamic Questionnaire Routing

After the location is identified, the application evaluates the selected site and applicable business rules.

Based on this configuration, the user is automatically directed to the appropriate questionnaire:

- SMAT;
- EMAT;
- ESMAT.

This process prevents incompatible forms from being selected and ensures that each inspection follows the requirements established for its operational context.

### SharePoint-Based Configuration

The application uses SharePoint Lists as its reference and operational storage layer.

These lists provide the application with:

- Location hierarchies;
- Inspection requirements;
- Question categories;
- Questionnaire configurations;
- Language translations;
- Temporary inspection records;
- Completed inspection records.

This structure separates the application interface from its reference content and supports the maintenance of requirements without duplicating the application.

### Temporary Save

Users can save an inspection before final submission.

This functionality allows inspections to be resumed later while preserving the information already registered.

Temporary records are stored in SharePoint and remain distinguishable from completed inspections.

## High-Level Application Flow

```text
Open E-SMAT
      ↓
Load Reference Data
      ↓
Determine Application Language
      ↓
Identify the Inspected Location
      ↓
Apply Cascading Location Filters
      ↓
Evaluate Site and Business Rules
      ↓
Select the Applicable Methodology
      ↓
Load SMAT, EMAT, or ESMAT Questionnaire
      ↓
Complete the Inspection
      ↓
Temporary Save or Final Submission
      ↓
Store the Record in SharePoint
```

### Folder Contents

- `app-overview.md`: application purpose, scope, users, and main capabilities;
- `application-flow.md`: step-by-step application workflow;
- `cascading-filter-logic.md`: documentation of the location filtering mechanism;
- `questionnaire-routing.md`: rules used to select SMAT, EMAT, or ESMAT;
- `bilingual-interface.md`: Portuguese and Spanish localization strategy;
- `temporary-save.md`: temporary save and inspection recovery process;
- `screenshots/`: anonymized or reconstructed application screens.

### Privacy Notice

This folder contains an anonymized representation of a corporate application.

It does not include:

- The production Power Apps package;
- Real SharePoint connections;
- Internal URLs;
- User or employee information;
- Production credentials;
- Complete proprietary requirements;
- Real operational hierarchies;
- Real inspection records;
- Unredacted production screenshots.

The content is intended exclusively to demonstrate the application architecture, business logic, user experience, and technical capabilities applied during development.
