# Navigation Logic

## Overview

This document describes the screen navigation and workflow logic of the E-SMAT Power Apps application.

E-SMAT provides two inspection entry points:

- **Create EMAT**
- **Create SMAT / ESMAT**

Both workflows use a consistent navigation structure, including inspection identification, questionnaire completion, complementary questions, final submission, and confirmation. The main differences are the applicable methodology, questionnaire content, routing rules, and visual identity.

The application also provides persistent access to home and language controls throughout the inspection journey.

---

## Navigation Principles

The E-SMAT navigation was designed around the following principles:

- Guide the user through a defined inspection sequence;
- Prevent access to a questionnaire before required context is provided;
- Route the user to the correct methodology and questionnaire;
- Preserve a consistent experience across EMAT, SMAT, and ESMAT;
- Allow backward navigation without restarting the workflow;
- Validate required information before advancing or submitting;
- Provide clear confirmation after a successful submission;
- Keep home and language controls accessible throughout the application.

---

## Application Initialization

When E-SMAT starts, the application first identifies the language configured on the user's device or browser.

The initial interface is automatically loaded in:

- Brazilian Portuguese (`pt-BR`); or
- Argentine Spanish (`es-AR`).

The automatic language selection does not create a separate navigation step. The user enters the application directly through the home screen and can manually switch the interface language through the flags displayed in the header.

Detailed language behavior is documented in [`localization.md`](localization.md).

```text
Launch E-SMAT
      ↓
Detect Device or Browser Language
      ↓
Load the Localized Interface
      ↓
Display Home Screen
```

---

## Home Screen

The home screen is the shared entry point for all inspection workflows.

The screen provides two primary actions:

- **Create EMAT**
- **Create SMAT / ESMAT**

It also displays the manual language controls for Brazilian Portuguese and Argentine Spanish.

### Create EMAT

Selecting **Create EMAT** starts the environmental inspection workflow and opens the EMAT inspection identification screen.

### Create SMAT / ESMAT

Selecting **Create SMAT / ESMAT** starts the shared safety and integrated inspection workflow and opens the SMAT / ESMAT inspection identification screen.

The user does not manually choose between SMAT and ESMAT on the home screen. The applicable methodology is determined later from the selected operational context.

```text
Home Screen
      │
      ├── Create EMAT
      │        ↓
      │   EMAT Identification
      │
      └── Create SMAT / ESMAT
               ↓
          SMAT / ESMAT Identification
```

---

## Shared Header Navigation

The application header remains visually consistent throughout the inspection workflows.

The header includes:

- A home control;
- The current workflow title;
- A Brazilian Portuguese language control;
- An Argentine Spanish language control.

### Home Control

The home control returns the user to the application home screen.

Because returning home may interrupt the current inspection journey, the production behavior should preserve or discard entered information only according to the application's implemented record-state rules. This public documentation does not expose the corresponding production formulas.

### Language Controls

The language controls allow the user to switch the interface manually without returning to the home screen.

Changing the interface language affects localized application content. Questionnaire selection remains controlled by the inspection context and methodology configuration.

---

## Inspection Identification

After selecting an inspection workflow, the user is directed to the corresponding identification screen.

The visible identification fields include:

- Inspection date;
- Employee identifier;
- Position or job role;
- Site;
- Segment;
- Business unit or area;
- Line;
- Workstation.

Required fields are identified visually with an asterisk.

The operational location fields follow a cascading sequence:

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

The detailed filter dependencies are documented in [`cascade-filtering.md`](cascade-filtering.md).

### Clear Action

The **Clear** action resets the information entered or selected on the identification screen, allowing the user to restart the identification step without returning to the home screen.

### Next Action

The **Next** action performs the transition from inspection identification to the applicable questionnaire.

Before navigation, the application validates the required identification fields. If the required information is incomplete, the user remains on the current screen and receives the applicable validation feedback.

When the required information is valid, the next step depends on the selected workflow.

---

## EMAT Navigation

The EMAT workflow has a dedicated entry point and follows a direct methodology path.

After the identification data is validated, E-SMAT loads the environmental questionnaire associated with the selected site, location, active configuration, and language.

```text
Home Screen
      ↓
Create EMAT
      ↓
EMAT Inspection Identification
      ↓
Validate Required Information
      ↓
Load Applicable EMAT Questionnaire
      ↓
Complete Environmental Requirements
      ↓
Complete Complementary Questions
      ↓
Submit EMAT
      ↓
Display EMAT Confirmation
      ↓
Return to Home Screen
```

### EMAT Visual Identity

The EMAT workflow uses green as its primary visual color. The header and primary forward actions visually distinguish the environmental workflow from SMAT and ESMAT.

### Environmental Questionnaire

The environmental questionnaire displays the requirements applicable to the selected inspection context.

For each item, the screen may provide:

- Environmental category;
- Requirement or inspection question;
- Safe observation quantity;
- Unsafe observation quantity;
- Description field.

The visible interface includes numeric selection for safe and unsafe observations.

### Questionnaire Navigation

The EMAT questionnaire provides:

- **Back**, to return to the identification screen or preceding questionnaire step;
- **Next**, to continue to the following questionnaire step or complementary questions;
- **Home**, to return to the application home screen;
- Language controls, to switch the interface language.

---

## SMAT / ESMAT Navigation

SMAT and ESMAT share the same initial entry point and identification experience.

After the identification data is validated, E-SMAT evaluates the selected site and operational location to determine the applicable methodology.

```text
Home Screen
      ↓
Create SMAT / ESMAT
      ↓
Inspection Identification
      ↓
Validate Required Information
      ↓
Evaluate Site and Location Configuration
      │
      ├── SMAT Configuration
      │        ↓
      │   Load SMAT Questionnaire
      │
      └── ESMAT Configuration
               ↓
          Load ESMAT Questionnaire
```

The methodology decision occurs within the application logic. The user is not required to manually interpret whether SMAT or ESMAT applies.

### SMAT / ESMAT Visual Identity

The shared SMAT / ESMAT workflow uses blue as its primary visual color.

### Dynamic Questionnaire

After routing, the application displays the requirements applicable to the selected context.

Each questionnaire item may contain:

- Question category;
- Question number;
- Requirement or inspection instruction;
- Safe observation input;
- Unsafe observation input;
- Description field.

The application loads questions dynamically from SharePoint reference data rather than maintaining a separate hard-coded screen for each site.

### Questionnaire Navigation

The SMAT / ESMAT questionnaire provides:

- **Back**, to return to the identification screen or preceding questionnaire step;
- **Next**, to continue to the following questionnaire step or complementary questions;
- **Home**, to return to the application home screen;
- Language controls, to switch the interface language.

---

## Complementary Questions

After the main questionnaire is completed, both workflow branches proceed to a complementary questions screen.

The visible complementary questions capture:

- Whether the operator identified an unsafe condition;
- A description of the condition, when applicable;
- Whether a safety tag was opened;
- Whether a near miss was identified.

The screen uses standardized **Yes** and **No** options and provides a description field for additional context.

### Back Action

The **Back** action returns the user to the preceding questionnaire step, allowing the user to review or change previously entered inspection information before submission.

### Submit Action

The **Submit** action initiates final validation and submission.

Before the record is finalized, E-SMAT verifies the information required by the applicable workflow. If validation fails, the user remains in the inspection flow and receives the relevant feedback.

If validation succeeds, the application stores the finalized inspection record and navigates to the corresponding confirmation screen.

```text
Main Questionnaire
      ↓
Complementary Questions
      │
      ├── Back
      │      ↓
      │  Return to Questionnaire
      │
      └── Submit
             ↓
        Validate Record
             │
             ├── Invalid
             │      ↓
             │  Remain in Workflow
             │
             └── Valid
                    ↓
               Store Final Record
                    ↓
               Open Confirmation Screen
```

---

## Temporary Save Navigation

E-SMAT supports the temporary storage of inspections that have not yet been finalized.

A temporary save preserves the inspection in progress without sending it through the final submission and confirmation path. When the inspection is resumed, the application restores the applicable context and previously recorded information according to the implemented temporary-record logic.

Because the provided portfolio screens focus on the creation and final submission journey, the detailed screen entry point and production formulas for temporary recovery are not reproduced here.

The complete temporary-save behavior is documented in [`temporary-save.md`](temporary-save.md).

Conceptually, the navigation is represented as follows:

```text
Inspection in Progress
      ↓
Save Temporarily
      ↓
Store Draft Record in SharePoint
      ↓
Exit or Return to the Application
      ↓
Resume Saved Inspection
      ↓
Restore Inspection Context and Responses
      ↓
Continue the Workflow
      ↓
Submit When Complete
```

---

## Submission Confirmation

After a record is submitted successfully, E-SMAT displays a methodology-specific confirmation screen.

The confirmation screen contains:

- A visual success indicator;
- A successful registration message;
- An action to return to the home screen;
- The shared header navigation and language controls.

The displayed message reflects the submitted workflow, such as SMAT, ESMAT, or EMAT.

The confirmation screen marks the end of the current inspection journey.

```text
Successful Submission
      ↓
Display Confirmation
      ↓
Return to Home Screen
      ↓
Start a New Inspection
```

---

## Complete Navigation Map

```text
Launch E-SMAT
      ↓
Detect Device or Browser Language
      ↓
Home Screen
      │
      ├── Create EMAT
      │        ↓
      │   EMAT Identification
      │        ↓
      │   Validate Required Fields
      │        ↓
      │   Load Environmental Questionnaire
      │        ↓
      │   Complete Environmental Requirements
      │        ↓
      │   Complementary Questions
      │        ↓
      │   Temporary Save or Final Submission
      │        ↓
      │   EMAT Confirmation
      │
      └── Create SMAT / ESMAT
               ↓
          Inspection Identification
               ↓
          Validate Required Fields
               ↓
          Evaluate Site and Location Rules
               │
               ├── Load SMAT Questionnaire
               └── Load ESMAT Questionnaire
                         ↓
               Complete Applicable Requirements
                         ↓
               Complementary Questions
                         ↓
               Temporary Save or Final Submission
                         ↓
               SMAT or ESMAT Confirmation
      ↓
Return to Home Screen
```

---

## Navigation Controls Summary

### Home

Returns the user to the shared application home screen.

### Clear

Resets the information entered on the inspection identification screen.

### Back

Returns the user to the previous step without initiating final submission.

### Next

Validates the current step and advances to the next applicable stage.

### Submit

Performs final validation, stores the completed inspection, and opens the confirmation screen when submission succeeds.

### Language Controls

Switch the visible interface between Brazilian Portuguese and Argentine Spanish without changing the applicable inspection methodology.

---

## Validation and Navigation Behavior

Navigation is conditional when the current step requires valid information.

The application uses validation to prevent progression when required data is missing or when the current inspection context is incomplete.

Examples include:

- Preventing questionnaire access before required identification fields are completed;
- Preventing routing before the operational context is valid;
- Preventing final submission before required inspection information is provided;
- Keeping the user on the current screen when validation fails;
- Displaying confirmation only after a successful final submission.

Production validation formulas and internal rule mappings are intentionally excluded from the public portfolio repository.

---

## Portfolio Scope

This document describes the functional navigation visible in the E-SMAT application and the logical relationship between the main workflow stages.

The public repository does not include:

- Production Power Fx navigation formulas;
- Internal screen or control names;
- Site-to-methodology mappings;
- Proprietary validation rules;
- SharePoint connection details;
- Production record identifiers;
- Internal error messages;
- Confidential organizational references.

Screenshots used to illustrate this flow should be anonymized or reconstructed with synthetic content before publication.
