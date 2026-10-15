# Application Screenshots

## Overview

This folder contains screenshots of the E-SMAT Power Apps application prepared for the public portfolio.

The screenshots document the main user journeys for the EMAT, SMAT, and ESMAT inspection workflows, including workflow selection, inspection identification, dynamic questionnaires, complementary questions, and submission confirmation.

All fields, operational references, identifiers, and visible records were anonymized before the images were added to this repository.

---

## Folder Structure

```text
screenshots/
├── README.md
├── shared/
│   └── 01-home-screen.png
│
├── emat/
│   ├── 01-inspection-identification.png
│   ├── 02-environmental-questionnaire.png
│   ├── 03-complementary-questions.png
│   └── 04-submission-confirmation.png
│
└── smat-esmat/
    ├── 01-inspection-identification.png
    ├── 02-dynamic-questionnaire.png
    ├── 03-complementary-questions.png
    └── 04-submission-confirmation.png
```

The shared home screen is stored separately because it is the common entry point for both workflow branches.

---

## Shared Screen

### `shared/01-home-screen.png`

The home screen is the shared entry point for E-SMAT.

The screen provides access to:

- **Create EMAT**;
- **Create SMAT / ESMAT**;
- Brazilian Portuguese interface control;
- Argentine Spanish interface control.

During initialization, E-SMAT detects the language configured on the user's device or browser and automatically loads the interface in Portuguese or Spanish. The language controls allow the user to override the detected language manually.

The home screen also introduces the visual identity of each workflow:

- Green for EMAT;
- Blue for SMAT / ESMAT.

---

## EMAT Workflow

The `emat/` folder contains the screens associated with the environmental inspection workflow.

### `emat/01-inspection-identification.png`

The inspection identification screen collects the information required to establish the EMAT context.

The screen includes fields for:

- Inspection date;
- Observer identifier;
- Position or job role;
- Site;
- Segment;
- Business unit or area;
- Line;
- Workstation.

The operational location follows a cascading structure:

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

The screen also provides:

- **Clear**, to reset the identification fields;
- **Next**, to validate the required information and open the questionnaire;
- **Home**, to return to the application entry screen;
- Language controls, to switch between Portuguese and Spanish.

### `emat/02-environmental-questionnaire.png`

The environmental questionnaire displays the EMAT requirements associated with the selected inspection context.

The screen contains:

- Environmental category;
- Question number;
- Environmental requirement;
- Safe-observation quantity selector;
- Unsafe-observation quantity selector;
- Description field;
- Back and Next navigation controls.

Green identifies safe observations, while red identifies unsafe observations.

The questions are loaded dynamically according to the selected site, operational location, methodology, questionnaire version, active status, and display order.

### `emat/03-complementary-questions.png`

The complementary-questions screen collects additional information before final submission.

The visible questions address:

- Whether an unsafe condition was identified;
- Description of the unsafe condition, when applicable;
- Whether a safety tag was opened;
- Whether a near miss was identified.

The screen provides:

- Yes and No response options;
- A conditional description field;
- **Back**, to return to the questionnaire;
- **Submit**, to initiate final validation and record creation.

### `emat/04-submission-confirmation.png`

The confirmation screen is displayed after the EMAT submission is processed successfully.

The screen contains:

- A visual success indicator;
- An EMAT registration message;
- An action to return to the home screen;
- Portuguese and Spanish language controls.

The application does not provide the inspection user with a post-submission editing option.

---

## SMAT / ESMAT Workflow

The `smat-esmat/` folder contains the screens associated with the shared SMAT and ESMAT entry flow.

### `smat-esmat/01-inspection-identification.png`

The inspection identification screen collects the information required to determine the applicable SMAT or ESMAT questionnaire.

The screen includes fields for:

- Inspection date;
- Observer identifier;
- Position or job role;
- Site;
- Segment;
- Business unit or area;
- Line;
- Workstation.

The operational location uses the same cascading hierarchy as the EMAT workflow.

After validation, E-SMAT evaluates the selected operational context and determines whether the applicable methodology is SMAT or ESMAT. The user does not manually select between the two methodologies.

### `smat-esmat/02-dynamic-questionnaire.png`

The dynamic questionnaire displays the requirements associated with the applicable SMAT or ESMAT configuration.

The screen contains:

- Question category;
- Question number;
- Inspection instruction or requirement;
- Safe-observation input;
- Unsafe-observation input;
- Description field;
- Back and Next navigation controls.

Green identifies safe observations, red identifies unsafe observations, and blue remains the primary visual color of the workflow.

### `smat-esmat/03-complementary-questions.png`

The final questionnaire stage records complementary information, including:

- Identification of an unsafe condition;
- Description of the condition, when applicable;
- Safety-tag information;
- Near-miss information.

The screen provides Back and Submit actions.

Information entered during the active inspection session remains available when the user navigates between the questionnaire and complementary-question screens.

### `smat-esmat/04-submission-confirmation.png`

The confirmation screen is displayed after the inspection header and question-level records are processed.

The screen contains:

- A visual success indicator;
- A message reflecting the submitted methodology;
- An action to return to the home screen;
- Portuguese and Spanish language controls.

The confirmation completes the current inspection flow. Submitted records are not made available to the inspection user for later editing within the application.

---

## Screenshot Naming Convention

The image names follow the order in which the user encounters each screen.

```text
01-home-screen.png
01-inspection-identification.png
02-environmental-questionnaire.png
02-dynamic-questionnaire.png
03-complementary-questions.png
04-submission-confirmation.png
```

The naming convention uses:

- Lowercase letters;
- Sequential numbering;
- Hyphens between words;
- PNG format.

This approach keeps the repository organized and makes relative Markdown references predictable.

---

## Image Presentation

The screenshots were prepared to preserve the functional meaning of the application while protecting internal information.

The published images demonstrate:

- Application layout;
- Bilingual interface controls;
- Separate workflow identities;
- Inspection identification;
- Cascading location selection;
- Dynamic questionnaire loading;
- Safe and unsafe observation inputs;
- Complementary questions;
- Submission confirmation.

The images do not reproduce the complete production environment or the complete proprietary requirement set.

---

## Portfolio Scope

The screenshot set is intended to demonstrate the E-SMAT interface and application flow without exposing confidential corporate information.

The anonymized images preserve the technical and functional context required to understand the solution while excluding identifiable users, real operational structures, production connections, and complete proprietary inspection requirements.
