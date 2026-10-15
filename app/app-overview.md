# E-SMAT Application Overview

## Overview

E-SMAT is a bilingual Microsoft Power Apps application developed to standardize the registration of safety and environmental inspections across six operational sites in Brazil and Argentina.

The application supports three inspection methodologies:

- **EMAT — Environment Management Audit Training**
- **SMAT — Safety Management Audit Training**
- **ESMAT — Integrated Environment and Safety Management Audit Training**

E-SMAT combines a shared application structure with site-specific configurations. Based on the selected inspection workflow, operational location, applicable methodology, active questionnaire version, and interface language, the application dynamically presents the appropriate inspection experience.

The application uses SharePoint Lists as its reference and operational storage layer. These lists provide the location hierarchy, inspection requirements, question categories, bilingual interface content, temporary records, and finalized inspection records used throughout the workflow.

> This repository presents an anonymized portfolio representation of the application. Production connections, internal identifiers, proprietary requirements, real inspection records, and confidential organizational information are not included.

---

## Business Context

The participating operational sites have different safety and environmental requirements, location structures, inspection methodologies, questionnaires, and language needs.

Creating a separate application for each site would duplicate components and business logic, increase maintenance effort, and make process standardization more difficult.

E-SMAT was therefore developed as a single configurable application capable of:

- Standardizing inspection registration across multiple sites;
- Supporting safety, environmental, and integrated inspections;
- Adapting questionnaires to each operational context;
- Supporting users in Brazil and Argentina;
- Guiding users through valid location hierarchies;
- Preserving inspections that are still in progress;
- Centralizing temporary and finalized records in SharePoint.

---

## Operational Scope

E-SMAT supports seven operational sites in Brazil and Argentina.

Each site uses a specific inspection form according to its operational characteristics and predefined business requirements. The application supports:

- Site-specific inspection questionnaires;
- Different operational location hierarchies;
- EMAT, SMAT, and ESMAT methodologies;
- Brazilian Portuguese and Argentine Spanish;
- Temporary inspection records;
- Final inspection submissions;
- Centralized SharePoint-based storage.

In the public repository, site names, internal location structures, and business-specific configurations are generalized or replaced with synthetic examples.

---

## Inspection Methodologies

### EMAT

**Environment Management Audit Training**

EMAT is the environmental inspection workflow within E-SMAT.

The workflow has a dedicated entry point on the home screen and dynamically loads the environmental requirements associated with the selected site and inspected location.

The EMAT interface uses green as its primary visual color, differentiating the environmental workflow from the safety and integrated inspection workflows.

For each displayed requirement, the user can register quantities associated with safe and unsafe observations and add a description when necessary.

### SMAT

**Safety Management Audit Training**

SMAT is focused on occupational safety inspections and the registration of safe and unsafe observations.

SMAT shares an initial entry point with ESMAT. The user does not manually select between the two methodologies on the home screen. The applicable questionnaire is determined by the application after the inspection context and operational location are identified.

### ESMAT

**Integrated Environment and Safety Management Audit Training**

ESMAT combines safety and environmental requirements in a single inspection workflow.

This methodology is used only by sites with the corresponding configuration. E-SMAT determines whether the applicable questionnaire is SMAT or ESMAT based on the selected site, location hierarchy, and configured business rules.

SMAT and ESMAT use blue as the primary visual color of the workflow.

---

## Application Entry Points

The home screen provides two inspection options:

- **Create EMAT**
- **Create SMAT / ESMAT**

The EMAT option starts a dedicated environmental inspection workflow.

The SMAT / ESMAT option starts a shared workflow. After the user identifies the inspected location, the application evaluates the corresponding configuration and loads the applicable SMAT or ESMAT questionnaire.

This approach reduces unnecessary user decisions and helps prevent the selection of a questionnaire that is incompatible with the inspection context.

---

## Main Features

### Automatic Language Detection

When E-SMAT starts, the application identifies the language configured on the user's device or browser and automatically loads the corresponding interface:

- Brazilian Portuguese (`pt-BR`);
- Argentine Spanish (`es-AR`).

Language controls remain available in the application header, allowing the user to manually switch between Portuguese and Spanish when necessary.

Detailed localization behavior is documented in [`localization.md`](localization.md).

### Inspection Identification

Before loading a questionnaire, the application collects the information required to identify the inspection and its operational context.

The visible identification fields include:

- Inspection date;
- Employee identifier;
- Position or job role;
- Site;
- Segment;
- Business unit or area;
- Line;
- Workstation.

Required fields are visually identified, and the application validates the necessary information before allowing the user to proceed.

### Cascading Location Filters

The inspected location is selected through a cascading hierarchy:

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

Each selection filters the options available in the next field. This mechanism helps prevent invalid combinations and ensures that the inspection is associated with a location configured in the SharePoint reference data.

The detailed filtering behavior is documented in [`cascade-filtering.md`](cascade-filtering.md).

### Dynamic Questionnaire Loading

After the inspection context is identified, E-SMAT evaluates the applicable configuration and dynamically loads the corresponding questionnaire.

Question selection may consider:

- Selected inspection workflow;
- Operational site;
- Inspected location;
- Applicable methodology;
- Questionnaire version;
- Safety or environmental theme;
- Question category;
- Active or inactive status;
- Display order;
- Selected interface language.

Inspection requirements are maintained in SharePoint reference lists rather than being individually hard-coded into the application screens.

### Safe and Unsafe Observation Registration

The questionnaire allows users to register safe and unsafe observations for each applicable requirement.

The interface uses visual differentiation:

- Green for safe observations;
- Red for unsafe observations.

The user can also provide descriptive information about the observed condition or practice.

In the EMAT workflow, numeric selectors allow quantities of safe and unsafe observations to be associated with an environmental requirement.

### Complementary Questions

After completing the main questionnaire, the application presents a final set of complementary questions.

This stage records information such as:

- Whether the operator identified an unsafe condition;
- A description of the unsafe condition, when applicable;
- Whether a safety tag was opened;
- Whether a near miss was identified.

These questions complement the main inspection requirements and capture additional information before final submission.

### Temporary Save

E-SMAT supports the temporary storage of inspections that have not yet been finalized.

This functionality allows the user to preserve an inspection in progress and resume it later without treating the incomplete record as a final submission.

The temporary-save process is documented separately in [`temporary-save.md`](temporary-save.md).

### Final Submission and Confirmation

Before final submission, the application validates the required information.

After the record is successfully submitted, E-SMAT displays a confirmation screen containing:

- A visual success indicator;
- A methodology-specific registration message;
- An action to return to the home screen.

The navigation flow is documented in [`navigation-logic.md`](navigation-logic.md).

---

## SharePoint-Based Configuration

E-SMAT uses structured SharePoint Lists as its reference and operational data layer.

The application consumes reference data related to:

- Operational location hierarchy;
- Inspection requirements;
- Question categories;
- Questionnaire methodology and version;
- Active and inactive questions;
- Portuguese and Spanish interface content.

SharePoint is also used to store temporary and finalized inspection records.

This configuration-driven approach separates application behavior from business content. It allows reference information to be maintained centrally without creating a separate Power Apps application for every operational site.

---

## High-Level Application Flow

```text
Open E-SMAT
      ↓
Detect Device or Browser Language
      ↓
Load Portuguese or Spanish Interface
      ↓
Allow Manual Language Switching
      ↓
Select Inspection Workflow
      │
      ├── Create EMAT
      │        ↓
      │   Identify Inspection and Location
      │        ↓
      │   Apply Cascading Location Filters
      │        ↓
      │   Load Environmental Requirements
      │        ↓
      │   Register Safe and Unsafe Observations
      │        ↓
      │   Complete Complementary Questions
      │        ↓
      │   Save Temporarily or Submit EMAT
      │
      └── Create SMAT / ESMAT
               ↓
          Identify Inspection and Location
               ↓
          Apply Cascading Location Filters
               ↓
          Determine SMAT or ESMAT
               ↓
          Load Applicable Requirements
               ↓
          Register Safe and Unsafe Observations
               ↓
          Complete Complementary Questions
               ↓
          Save Temporarily or Submit Inspection
      ↓
Display Submission Confirmation
      ↓
Return to Home Screen
```

---

## Design Approach

E-SMAT follows a configuration-driven and reusable design approach.

Instead of duplicating screens and logic for every site, the application uses reference data and business rules to control:

- Available location options;
- Interface language;
- Applicable inspection methodology;
- Questionnaire selection;
- Displayed requirements;
- Question order and status;
- Navigation behavior;
- Record status.

This approach supports consistency across sites while preserving location-specific requirements.

---

## Technologies

- Microsoft Power Apps;
- Microsoft SharePoint Online;
- SharePoint Lists;
- Power Fx.

The downstream data ingestion, transformation, and analytics architecture is documented outside the `app/` folder.

---

## Documentation Structure

The application documentation is organized as follows:

- [`app-overview.md`](app-overview.md): application purpose, scope, methodologies, and core capabilities;
- [`navigation-logic.md`](navigation-logic.md): screen flow, workflow branching, validation, and submission navigation;
- [`cascade-filtering.md`](cascade-filtering.md): operational hierarchy filters and questionnaire routing context;
- [`temporary-save.md`](temporary-save.md): storage and recovery of inspections in progress;
- [`localization.md`](localization.md): automatic language detection and bilingual interface behavior;
- [`screenshots/`](screenshots/): anonymized or reconstructed application screens.

---

## Portfolio Scope and Data Privacy

This folder documents the application architecture, user journey, business logic, localization strategy, and main functional capabilities.

The public repository does not include:

- Production Power Apps packages;
- Production Power Fx formulas;
- Corporate credentials or access tokens;
- SharePoint URLs or production connections;
- Employee names, e-mail addresses, or identifiers;
- Real inspection records;
- Complete proprietary inspection requirements;
- Internal operational hierarchies;
- Unredacted production screenshots;
- Confidential organizational references.

Screenshots and sample data published in the portfolio should be anonymized or reconstructed with synthetic content. The repository is intended exclusively to demonstrate the architecture, development approach, and technical capabilities applied to the E-SMAT solution.
