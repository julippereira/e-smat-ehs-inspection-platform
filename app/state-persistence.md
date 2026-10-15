# In-Session State Persistence

## Overview

E-SMAT preserves the information entered by the user while the user navigates between the application screens during the same inspection session.

This behavior is implemented through Power Apps variables and control state. It allows the user to move forward and backward through the inspection flow without losing the identification data and questionnaire responses already entered.

This functionality must not be interpreted as a user-accessible draft or editing feature:

- The inspection is not made available in a personal draft list;
- The user cannot reopen the inspection later to edit it;
- The user cannot retrieve a previously submitted inspection for modification;
- Navigation persistence applies only while the current application session and inspection flow remain active;
- SharePoint receives the completed inspection when the user performs the final submission.

> The production application uses internal Power Apps control names, variables, SharePoint lists, and formulas. This portfolio document explains the functional logic without publishing the full production implementation.

---

## Functional Purpose

The in-session persistence mechanism was designed to:

- Preserve identification data while the user navigates between screens;
- Avoid requesting the same information again on each stage;
- Keep the selected operational context available to the questionnaire;
- Maintain questionnaire responses when the user returns to a previous screen;
- Support final consolidation of the inspection record;
- Prevent incomplete inspections from being written as finalized records before submission.

---

## What Is Preserved During Navigation

After the identification screen is validated, E-SMAT stores the inspection context in application variables.

The preserved context includes information such as:

- Unique inspection identifier;
- Inspection date;
- Site;
- Segment;
- Business unit or area;
- Line;
- Workstation;
- Shift;
- Activity type;
- Contract type;
- Observer identifier;
- Observed-person identifiers;
- Position or job role.

Questionnaire responses remain available while the user navigates between the questionnaire and complementary-question screens during the active session.

---

## Scope of Persistence

The persistence behavior applies to navigation within the current inspection flow.

```text
Inspection Identification
      ↓
Validate Required Fields
      ↓
Store Inspection Context in Variables
      ↓
Open Dynamic Questionnaire
      ↓
Enter Inspection Responses
      ↓
Navigate Forward or Backward
      ↓
Keep Current Session Data Available
      ↓
Submit Final Inspection
```

The application does not provide a user-facing workflow such as:

```text
Save Draft
      ↓
Close Application
      ↓
Open Draft List Later
      ↓
Edit Saved Inspection
```

That workflow is outside the implemented scope.

---

## Next Button Logic

The **Next** button on the inspection identification screen performs three main operations:

1. Initializes the navigation-validation flag;
2. Validates the required identification fields;
3. Stores the inspection context in variables and opens the questionnaire when validation succeeds.

### 1. Initialize Validation State

The application initially assumes that navigation is allowed:

```powerfx
Set(VarPodeAvancar; true)
```

This flag is changed only when a required field is missing.

### 2. Validate Required Fields

Before advancing, the application checks the mandatory controls on the identification screen.

The validation covers fields associated with:

- Site;
- Segment;
- Business unit or area;
- Line;
- Shift;
- Activity type;
- Contract type;
- Position or job role;
- Observer identifier;
- Primary observed-person identifier;
- Inspection date.

If any required field is blank, the application:

- Displays an error notification;
- Sets the validation flag to `false`;
- Keeps the user on the identification screen;
- Does not generate the inspection identifier;
- Does not navigate to the questionnaire.

A simplified representation is:

```powerfx
Set(VarPodeAvancar; true);

If(
    required fields are blank;
    Notify(
        "Please complete all required fields before proceeding.";
        NotificationType.Error
    );
    Set(VarPodeAvancar; false)
)
```

The production message is displayed according to the application's localization behavior.

### 3. Create the Inspection Context

When validation succeeds, the application generates a unique inspection identifier.

The identifier combines:

- Inspection methodology prefix;
- Current date and time;
- User-related identifier derived from the authenticated account.

Conceptually:

```text
Methodology Prefix
      +
Timestamp
      +
Authenticated User Component
      ↓
Unique Inspection Identifier
```

The resulting identifier links the inspection header to its individual questionnaire responses during final submission.

### 4. Store Values in Variables

After generating the identifier, the application stores the validated identification values in variables.

A simplified portfolio representation is:

```powerfx
If(
    VarPodeAvancar;

    Set(VarInspectionID; generated inspection identifier);
    Set(VarSite; selected site);
    Set(VarSegment; selected segment);
    Set(VarArea; selected business unit or area);
    Set(VarLine; selected line);
    Set(VarWorkstation; selected or manually entered workstation);
    Set(VarShift; selected shift);
    Set(VarActivityType; selected activity type);
    Set(VarContractType; selected contract type);
    Set(VarObserverID; observer identifier);
    Set(VarObservedPersonID; observed-person identifier);
    Set(VarRole; selected role);
    Set(VarInspectionDate; selected date);

    Navigate(QuestionnaireScreen; ScreenTransition.Fade)
)
```

These variables preserve the inspection context when the user moves to the questionnaire and subsequent screens.

---

## Workstation Input Handling

The workstation field supports either:

- A value selected from the available options; or
- A manually entered search value when no predefined option is selected.

Conceptually:

```powerfx
Set(
    VarWorkstation;
    If(
        selected workstation is blank;
        typed workstation value;
        selected workstation value
    )
)
```

This behavior allows the inspection context to retain a workstation value even when the user uses the editable search input.

Because the workstation field is not visually marked as mandatory in the provided screen, this document does not classify it as universally required.

---

## Navigation Between Screens

Once the context variables are set, the user can continue through the questionnaire flow.

Using **Back** returns the user to the previous stage without creating a finalized SharePoint record. The information already held by the current session remains available to the application.

```text
Identification Screen
      ↓ Next
Questionnaire Screen
      ↓ Next
Complementary Questions
      ↑ Back
Questionnaire Screen with Current Responses
```

The application therefore separates:

- **In-session state**, used while the inspection is being completed;
- **Persistent storage**, written during final submission.

---

## Final Submission Logic

The **Submit** button writes the completed inspection to SharePoint.

The submission process has two main stages:

1. Create or update the inspection header;
2. Save the response associated with each questionnaire item.

The application proceeds to the confirmation screen only after the header has been saved successfully.

---

## Header Record

The header record contains the inspection-level information shared by all questionnaire responses.

The submitted header includes information such as:

- Unique inspection identifier;
- Inspection date;
- Site;
- Segment;
- Business unit or area;
- Line;
- Workstation;
- Shift;
- Activity type;
- Contract type;
- Observer identifier;
- Observed-person identifiers;
- Position or job role;
- Unsafe-condition indicator and description;
- Near-miss indicator and description;
- Safety-tag indicator and number.

The application uses the unique inspection identifier to locate the corresponding header record during the `Patch` operation.

Conceptually:

```powerfx
Set(
    VarSavedHeader;
    Patch(
        InspectionHeaderList;
        LookUp(InspectionHeaderList; RecordID = VarInspectionID);
        {
            RecordID: VarInspectionID;
            InspectionDate: VarInspectionDate;
            Site: VarSite;
            Segment: VarSegment;
            Area: VarArea;
            Line: VarLine;
            Workstation: VarWorkstation
        }
    )
)
```

---

## Language Normalization Before Storage

The interface supports Portuguese and Spanish, but selected values are normalized before being stored in the inspection header.

This prevents equivalent concepts from being saved as separate analytical values only because the application was displayed in a different language.

Examples of normalized concepts include:

- Shift;
- Activity type;
- Contract type;
- Position or job role;
- Yes and No responses.

Conceptually:

```text
Localized Display Value
      ↓
Normalization Rule
      ↓
Standard Storage Value
```

Examples include:

```text
Primero          → Primeiro
Tercero          → Terceiro
Rutina           → Rotineira
No rutinaria     → Não-rotineira
Capitán de Seguridad → Capitão de Segurança
Sí               → true
Sim              → true
```

The goal is to maintain one standardized value for downstream ingestion, transformation, and reporting.

---

## Conditional Complementary Data

The complementary answers are saved according to their selected values.

For binary fields, both Portuguese and Spanish affirmative responses are interpreted as `true`.

Descriptions or identifiers are saved only when the corresponding answer is affirmative.

Conceptually:

```powerfx
UnsafeCondition = selected value is "Sim" or "Sí";

UnsafeConditionDescription =
    If(
        UnsafeCondition;
        entered description;
        ""
    )
```

The same pattern is applied to near-miss and safety-tag information.

This avoids storing a description or tag number when the corresponding event was not identified.

---

## Questionnaire Response Records

After the header is saved successfully, the application processes all items displayed in the questionnaire gallery.

A separate SharePoint record is created for each question response.

Each response record contains information such as:

- Unique inspection identifier;
- Inspection date;
- Site;
- Question identifier;
- Description;
- Safe observation value;
- Unsafe observation value.

A simplified representation is:

```powerfx
ForAll(
    QuestionnaireGallery.AllItems;
    Patch(
        InspectionResponseList;
        Defaults(InspectionResponseList);
        {
            RecordID: VarInspectionID;
            InspectionDate: VarInspectionDate;
            Site: VarSite;
            QuestionID: Text(QuestionID);
            Description: DescriptionInput.Text;
            SafeObservation: SafeInput.Selected.Value;
            UnsafeObservation: UnsafeInput.Selected.Value
        }
    )
)
```

The shared inspection identifier creates the relationship between the header and the question-level records.

---

## Submission Sequence

The complete final-submission sequence is:

```text
Select Submit
      ↓
Refresh Header Data Source
      ↓
Create or Update Inspection Header
      ↓
Check Whether Header Was Saved
      │
      ├── Header Not Confirmed
      │      ↓
      │  Do Not Continue to Response Loop
      │
      └── Header Confirmed
             ↓
        Process Questionnaire Gallery
             ↓
        Create One Response Record per Question
             ↓
        Navigate to Success Screen
```

This sequence ensures that question-level records are not intentionally processed before the application confirms the header result.

---

## Confirmation Screen

After the header and questionnaire responses are processed, the application navigates to the methodology-specific success screen.

The confirmation screen indicates that the inspection was registered and provides an action to return to the home screen.

The confirmation screen does not provide an option to reopen or edit the submitted inspection.

```text
Final Submission
      ↓
Successful Processing
      ↓
Confirmation Screen
      ↓
Return to Home Screen
```

---

## No Post-Submission Editing

Once the inspection is submitted:

- The record is not presented to the user as an editable draft;
- The user does not receive an application screen for reopening the record;
- The success screen completes the current inspection flow;
- A new inspection must be started from the home screen for a new record.

This design keeps the creation workflow separate from any administrative correction, governance, or data-maintenance process that may exist outside the app.

The public documentation does not claim that records are technically immutable in SharePoint. It states only that E-SMAT does not provide the inspection user with a post-submission editing workflow.

---

## In-Session Persistence vs. Persistent Storage

The solution uses two distinct mechanisms:

### In-session persistence

Used while the inspection is being completed.

```text
Power Apps variables and control state
      ↓
Preserve data during screen navigation
```

### Persistent storage

Used when the inspection is submitted.

```text
SharePoint header record
      +
SharePoint question-level records
      ↓
Finalized inspection dataset
```

This distinction is important because the application does not implement a user-accessible draft-recovery workflow.

---

## Session Limitations

Because the navigation behavior relies on the active application session, information that has not been submitted may no longer be available if the session is interrupted.

Potential interruption scenarios include:

- Closing the application;
- Refreshing or closing the browser;
- Losing the active Power Apps session;
- Signing out;
- Opening a new inspection that resets the current state;
- Triggering another application action that clears the variables or controls.

The application should therefore not be described as providing durable draft recovery unless a separate persistent draft mechanism is implemented and verified.

---

## Data Integrity Considerations

The implementation supports data integrity through:

- Validation before leaving the identification screen;
- A unique identifier shared by header and response records;
- Header-first submission sequencing;
- Question-level records created only after header confirmation;
- Standardization of bilingual values before storage;
- Conditional storage of complementary descriptions;
- Navigation to the success screen only after the submission sequence runs.

Additional production error handling, transaction reconciliation, and retry behavior are outside the scope of this public document.

---

## Privacy Considerations

The production inspection identifier contains a user-related component derived from the authenticated e-mail account.

The public portfolio must not expose:

- Real identifiers generated in production;
- E-mail addresses;
- Employee numbers;
- Names of observed people;
- SharePoint list names or URLs;
- Internal control names;
- Real inspection responses.

Public examples should replace production identifiers with synthetic values such as:

```text
SMAT-20260101120000-USER001
```

---

## Simplified State Model

```text
New Inspection
      ↓
Identification in Progress
      ↓
Identification Validated
      ↓
Context Stored in Session Variables
      ↓
Questionnaire in Progress
      ↓
Complementary Questions in Progress
      ↓
Final Submission
      ↓
Header and Responses Stored in SharePoint
      ↓
Submission Confirmed
```

There is no user-facing `Draft Available for Later Editing` state in the implemented workflow.

---

## Relationship with Other Documentation

- [`app-overview.md`](app-overview.md) provides the application-level context;
- [`navigation-logic.md`](navigation-logic.md) documents screen transitions and validation-based navigation;
- [`cascade-filtering.md`](cascade-filtering.md) documents the operational location dependencies;
- [`localization.md`](localization.md) documents automatic language detection and bilingual value handling;
- [`screenshots/`](screenshots/) contains anonymized or reconstructed examples of the application screens.

---

## Portfolio Scope

This document intentionally omits:

- Complete production Power Fx formulas;
- Internal Power Apps control names;
- Production variable names where disclosure is unnecessary;
- Real SharePoint list and column names;
- Real employee or user identifiers;
- Production e-mail addresses;
- Proprietary validation rules;
- Complete inspection requirements;
- Corporate credentials and connection details.

The public version focuses on the design pattern: validation, state preservation during navigation, bilingual normalization, header-detail submission, and completion without user-facing post-submission editing.
