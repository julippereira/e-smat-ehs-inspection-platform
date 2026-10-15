# Cascading Filters and Questionnaire Routing

## Overview

E-SMAT uses cascading filters to guide the user through the operational location hierarchy and ensure that each inspection is associated with a valid configured location.

The location selected during the inspection identification stage also provides the context used to load the applicable EMAT, SMAT, or ESMAT questionnaire.

The filtering structure is based on SharePoint reference data rather than fixed option lists embedded individually in each application screen.

> This document describes the functional filtering logic used by E-SMAT. Production Power Fx formulas, real site structures, proprietary mappings, and internal SharePoint identifiers are intentionally excluded from the public repository.

---

## Location Hierarchy

The inspection location is identified through the following hierarchy:

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

Each field depends on the selection made in the preceding field.

For example:

1. The selected **Site** defines the available segments;
2. The selected **Segment** defines the available business units or areas;
3. The selected **Business Unit / Area** defines the available lines;
4. The selected **Line** defines the available workstations.

The workstation is the final and most detailed level visible in the location selection structure. Depending on the configured operational context, this level may not be mandatory for every inspection.

---

## Purpose of the Cascading Logic

The cascading mechanism was implemented to:

- Prevent invalid combinations of operational locations;
- Reduce the number of options displayed in each field;
- Guide the user through the correct location structure;
- Standardize location identification across sites;
- Associate inspections with consistent reference data;
- Provide the context required for questionnaire selection;
- Reduce manual interpretation of site-specific rules;
- Improve the quality of the records stored in SharePoint.

Without cascading filters, users could select unrelated values, such as a line belonging to a different site or an area not associated with the selected segment.

---

## Reference Data Source

The location hierarchy is maintained in a SharePoint reference list.

Each reference record represents a valid relationship between the levels used by the application, such as:

```text
Site → Segment → Business Unit / Area → Line → Workstation
```

A simplified public representation could contain the following fields:

```text
location_id
site
segment
business_unit_area
line
workstation
active_status
```

The public repository should use synthetic values such as `SITE_A`, `AREA_01`, and `LINE_01` rather than real organizational names.

---

## Initial State

When the inspection identification screen is opened, the application prepares the location controls for a new inspection.

At this stage:

- The Site field is available for selection;
- Dependent fields do not yet have a valid upstream context;
- Segment options depend on the selected site;
- Business Unit / Area options depend on the selected segment;
- Line options depend on the selected business unit or area;
- Workstation options depend on the selected line.

The application only presents values that belong to the currently selected path.

```text
No Site Selected
      ↓
No Valid Downstream Location Context
      ↓
Select Site to Start the Cascade
```

---

## Step 1: Site Selection

The Site field is the first level of the cascading hierarchy.

When a user selects a site, E-SMAT uses that value to retrieve the segments associated with the selected site.

```text
Selected Site
      ↓
Filter Location Reference Data
      ↓
Return Valid Segments
```

If the user changes the site after selecting downstream values, those values must be cleared or reevaluated because the previously selected segment, area, line, or workstation may not belong to the new site.

---

## Step 2: Segment Selection

After a site is selected, the Segment field displays only the segments configured for that site.

When the user selects a segment, E-SMAT applies both the site and segment selections to determine the valid business units or areas.

```text
Selected Site
      +
Selected Segment
      ↓
Return Valid Business Units / Areas
```

A segment from another site is not presented as a valid option.

---

## Step 3: Business Unit / Area Selection

The Business Unit / Area field depends on the selected site and segment.

The application displays only the business units or areas associated with the active location path.

```text
Selected Site
      +
Selected Segment
      +
Selected Business Unit / Area
      ↓
Return Valid Lines
```

If the segment changes, the existing business unit or area selection and all subsequent selections must be reset or revalidated.

---

## Step 4: Line Selection

The Line field displays only the lines associated with the selected site, segment, and business unit or area.

After a line is selected, E-SMAT retrieves the workstations configured for that line.

```text
Selected Location Context
      +
Selected Line
      ↓
Return Valid Workstations
```

Changing the business unit or area invalidates the previously selected line and workstation unless the same relationship remains valid in the reference data.

---

## Step 5: Workstation Selection

The Workstation field is the final visible level of the location hierarchy.

Only workstations associated with the selected line and preceding location context are displayed.

The workstation field is not visually marked as mandatory in the provided application screen. Therefore, the public documentation does not state that a workstation is required in every configuration.

```text
Site
  + Segment
  + Business Unit / Area
  + Line
  + Workstation, when applicable
      ↓
Complete Inspection Location Context
```

---

## Downstream Reset Behavior

A change to an upstream field can make later selections invalid.

The expected dependency behavior is:

```text
Change Site
  → Reset Segment
  → Reset Business Unit / Area
  → Reset Line
  → Reset Workstation

Change Segment
  → Reset Business Unit / Area
  → Reset Line
  → Reset Workstation

Change Business Unit / Area
  → Reset Line
  → Reset Workstation

Change Line
  → Reset Workstation
```

This behavior prevents a record from retaining a downstream value that no longer belongs to the selected hierarchy.

The production implementation may reset controls directly or validate retained values against the updated filter context. The exact Power Fx formulas are not included in the portfolio version.

---

## Clear Action

The identification screen provides a **Clear** action.

This action resets the inspection identification inputs and returns the cascading controls to their initial state.

Conceptually, the action performs the following steps:

```text
Select Clear
      ↓
Reset Identification Inputs
      ↓
Reset Site Selection
      ↓
Clear All Dependent Location Selections
      ↓
Restore Initial Screen State
```

The Clear action allows the user to restart the identification process without returning to the home screen.

---

## Validation Before Navigation

The location context must be valid before E-SMAT loads a questionnaire.

The visible required fields on the inspection identification screen include:

- Inspection date;
- Employee identifier;
- Position or job role;
- Site;
- Segment;
- Business unit or area;
- Line.

The Workstation field is visible but is not marked with an asterisk in the provided screen.

When the user selects **Next**, the application checks whether the required identification and location fields have been completed.

```text
Select Next
      ↓
Validate Required Identification Fields
      ↓
Validate Location Path
      │
      ├── Invalid or Incomplete
      │      ↓
      │  Remain on Identification Screen
      │
      └── Valid
             ↓
        Continue to Questionnaire Routing
```

Production validation messages and formulas are not included in the public repository.

---

## Relationship with Questionnaire Routing

The cascading filters identify where the inspection is being conducted. The selected location context is then used by E-SMAT to determine which questionnaire configuration applies.

Location filtering and questionnaire filtering are related but perform different functions:

### Location filtering

Controls which operational values the user can select.

```text
Site → Segment → Business Unit / Area → Line → Workstation
```

### Questionnaire routing

Uses the valid operational context and selected workflow to determine which inspection requirements should be displayed.

```text
Selected Workflow
      +
Valid Location Context
      +
Configured Methodology and Version
      ↓
Applicable Questionnaire
```

---

## EMAT Questionnaire Filtering

The EMAT workflow has a dedicated entry point on the home screen.

After the user completes the location hierarchy and the required identification fields, E-SMAT loads the environmental requirements associated with the selected context.

The filtering process may consider:

- EMAT as the selected workflow;
- Site;
- Selected operational location;
- Questionnaire version;
- Environmental category;
- Active question status;
- Display order;
- Interface language.

```text
Create EMAT
      ↓
Select Valid Location
      ↓
Validate Identification
      ↓
Filter Active EMAT Requirements
      ↓
Load Environmental Questionnaire
```

Because EMAT is selected directly from the home screen, the application does not need to decide between SMAT and ESMAT in this branch.

---

## SMAT and ESMAT Routing

SMAT and ESMAT share the same home-screen entry point.

The user selects **Create SMAT / ESMAT**, completes the inspection identification fields, and selects the operational location.

The application then evaluates the configured relationship between the selected context and the applicable methodology.

```text
Create SMAT / ESMAT
      ↓
Select Valid Location
      ↓
Validate Identification
      ↓
Evaluate Configured Methodology
      │
      ├── SMAT
      │      ↓
      │  Load Active SMAT Requirements
      │
      └── ESMAT
             ↓
         Load Active ESMAT Requirements
```

The user does not manually choose between SMAT and ESMAT. The methodology is resolved by the application according to the configured business rules.

Real site-to-methodology mappings are not included in the public documentation.

---

## Question-Level Filtering

After the applicable methodology is determined, E-SMAT filters the SharePoint requirements list to retrieve the questions associated with the current inspection context.

The question metadata may include:

- Site;
- Site code;
- Methodology;
- Questionnaire version;
- Safety or environmental theme;
- Category;
- Question identifier;
- Question number;
- Question text;
- Language-related content;
- Active or inactive status.

The filtering logic ensures that the questionnaire displays only the requirements that match the selected context.

```text
Valid Location Context
      +
Applicable Methodology
      +
Active Questionnaire Version
      +
Active Question Status
      ↓
Filtered Requirement Set
      ↓
Ordered Dynamic Questionnaire
```

---

## Category and Display Order

Filtered questions are organized using reference metadata.

Question categories support logical grouping of related safety or environmental requirements. Display-order metadata controls the sequence in which questions are presented.

This allows the questionnaire structure to be maintained through reference data rather than by manually repositioning every question in the application.

The public repository should document the approach without exposing proprietary categories or complete internal requirement sets.

---

## Active and Inactive Questions

The requirements reference data includes an active-status concept.

Only questions marked as applicable to the active questionnaire configuration should be displayed to the user.

This supports controlled maintenance when a requirement is:

- Introduced;
- Updated;
- Replaced;
- Temporarily deactivated;
- Retained for historical reference.

The use of status metadata helps avoid deleting reference records that may still be relevant to historical inspections.

---

## Questionnaire Version Context

Questionnaire version metadata allows E-SMAT to distinguish between different requirement configurations over time.

A version-aware structure supports:

- Controlled questionnaire updates;
- Separation of current and historical requirement sets;
- Consistent interpretation of inspections created under different versions;
- Traceability in downstream processing and reporting.

The public repository does not disclose production version identifiers or internal change history.

---

## Language and Filtering Responsibilities

Language selection and questionnaire selection are separate parts of the application logic.

### Localization logic

Determines how interface content is displayed:

```text
Device or Browser Language
      ↓
Portuguese or Spanish Interface
```

### Location and methodology logic

Determines which questionnaire content applies:

```text
Selected Workflow
      +
Operational Location
      +
Configured Methodology
      ↓
EMAT, SMAT, or ESMAT Requirements
```

Changing the interface language does not change the selected site, methodology, or questionnaire context.

Detailed localization behavior is documented in [`localization.md`](localization.md).

---

## Simplified Logical Model

The following pseudocode illustrates the filtering concept without reproducing production formulas:

```text
availableSegments = locations filtered by selectedSite

availableAreas = locations filtered by:
    selectedSite
    selectedSegment

availableLines = locations filtered by:
    selectedSite
    selectedSegment
    selectedArea

availableWorkstations = locations filtered by:
    selectedSite
    selectedSegment
    selectedArea
    selectedLine

applicableMethodology = configuration resolved from:
    selectedWorkflow
    selectedSite
    selectedLocationContext

applicableQuestions = requirements filtered by:
    selectedSite
    applicableMethodology
    activeQuestionnaireVersion
    activeStatus
```

This example documents the dependency structure only. It is not a copy of the production Power Fx implementation.

---

## Data Quality Benefits

The cascading and routing logic improves data quality by:

- Restricting selections to configured location relationships;
- Reducing inconsistent site and area combinations;
- Standardizing operational location values;
- Avoiding manual methodology selection in the shared SMAT / ESMAT flow;
- Loading only questions that apply to the inspection context;
- Supporting version and active-status controls;
- Improving consistency between application records and analytical datasets.

---

## Maintainability Benefits

The reference-driven model improves application maintainability because location structures and questionnaire metadata can be managed outside the visual screen layout.

This approach supports:

- Centralized reference-data maintenance;
- Reuse of the same application across multiple sites;
- Reduced duplication of screens and controls;
- Easier activation or deactivation of questions;
- Controlled questionnaire versioning;
- Consistent routing rules;
- Future extension of the location structure or requirement set.

Changes must still follow the production governance and validation process before being released to users.

---

## Error Prevention Considerations

A reliable cascading implementation should prevent or handle the following situations:

- A downstream field retaining a value after an upstream selection changes;
- A user continuing with an incomplete required location path;
- Duplicate values appearing unnecessarily in a dropdown;
- Inactive reference values being presented as selectable options;
- A valid location returning no applicable questionnaire;
- Multiple active questionnaire configurations applying to the same context;
- A language change altering the inspection context;
- A temporary record restoring values that no longer match the current reference configuration.

The public portfolio documents these design considerations without exposing internal error-handling formulas.

---

## Relationship with Other Documentation

- [`app-overview.md`](app-overview.md) provides the application-level context;
- [`navigation-logic.md`](navigation-logic.md) documents screen transitions and workflow branching;
- [`state-persistence.md`](state-persistence.md) documents the preservation and restoration of inspections in progress;
- [`localization.md`](localization.md) documents automatic language detection and manual language switching;
- [`screenshots/`](screenshots/) contains anonymized or reconstructed visual examples.

---

## Portfolio Scope and Data Privacy

The public version of this document does not include:

- Real site, segment, area, line, or workstation names;
- Production SharePoint list names or URLs;
- Internal column names or identifiers;
- Site-to-methodology mappings;
- Complete questionnaire requirements;
- Production questionnaire versions;
- Production Power Fx formulas;
- Corporate credentials or connection details;
- Confidential business rules.

Synthetic reference data should preserve the same dependency structure while replacing all organizational values and proprietary requirements.
