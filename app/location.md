# Localization

## Overview

E-SMAT provides a bilingual user experience in:

- **Brazilian Portuguese (`pt-BR`)**
- **Argentine Spanish (`es-AR`)**

When the application starts, E-SMAT identifies the language configured on the user's device or browser and automatically loads the corresponding interface.

The user can also manually switch between Portuguese and Spanish through the language controls displayed in the application header.

The localization strategy allows the same Power Apps application to support operational sites in Brazil and Argentina without duplicating the application structure.

> This document describes the functional localization approach. Production Power Fx formulas, internal translation identifiers, proprietary terminology, and complete SharePoint records are not included in the public portfolio repository.

---

## Supported Languages

E-SMAT supports the following language configurations:

```text
pt-BR → Brazilian Portuguese
es-AR → Argentine Spanish
```

Brazilian Portuguese is used by the Brazilian operational sites, while Argentine Spanish supports the operational site in Argentina.

The application preserves the same inspection workflow and business rules in both languages. Localization changes how information is displayed, but does not independently change the selected site, methodology, questionnaire, or inspection record.

---

## Automatic Language Detection

During application initialization, E-SMAT evaluates the language configured on the user's device or browser.

The detected value is used to define the initial interface language before the user starts the inspection workflow.

```text
Launch E-SMAT
      ↓
Read Device or Browser Language
      ↓
Evaluate Supported Language
      │
      ├── Portuguese
      │      ↓
      │  Load pt-BR Interface
      │
      └── Spanish
             ↓
         Load es-AR Interface
      ↓
Display Localized Home Screen
```

Automatic detection removes the need for a separate language-selection screen and allows the user to enter the application directly through the localized home screen.

---

## Initial Language Resolution

The language identified during application initialization determines the initial values displayed by the interface.

Conceptually, the application follows this logic:

```text
Detected language starts with "pt"
      → Use Brazilian Portuguese

Detected language starts with "es"
      → Use Argentine Spanish
```

The exact production Power Fx expression and fallback handling are not reproduced in the public repository.

A public implementation should define a controlled fallback language for devices or browsers configured with an unsupported language. The production fallback behavior must be documented only if it can be disclosed accurately.

---

## Manual Language Switching

The application header displays two language controls:

- Brazilian flag for Portuguese;
- Argentine flag for Spanish.

These controls allow the user to override the automatically detected language at any point in the application.

```text
Current Interface Language
      ↓
Select Language Control
      ↓
Update Active Language
      ↓
Refresh Localized Interface Content
```

Manual language switching is useful when:

- The browser language does not match the user's preferred language;
- A shared device is used by people with different language preferences;
- The user needs to review the interface in another supported language;
- The automatically detected regional configuration does not correspond to the intended interface.

---

## Persistent Language Controls

The Portuguese and Spanish controls remain available in the header throughout the main application journey, including:

- Home screen;
- Inspection identification;
- Dynamic questionnaire;
- Complementary questions;
- Submission confirmation.

This allows the user to switch languages without returning to the home screen or restarting the inspection.

The language change should preserve the current inspection state and operational context while updating the localized interface content.

---

## Localized Interface Components

The active language is applied to interface components such as:

- Screen titles;
- Field labels;
- Input placeholders;
- Buttons;
- Navigation controls;
- Instructions;
- Response options;
- Required-field indicators and related messages;
- Validation messages;
- Complementary questions;
- Submission actions;
- Success confirmation messages.

Examples of localized interface concepts include:

```text
Portuguese              Spanish
Criar EMAT              Crear EMAT
Próximo                 Siguiente
Voltar                  Volver
Limpar                  Limpiar
Enviar                  Enviar
Sim                     Sí
Não                     No
```

The examples above illustrate the localization structure and should not be treated as a complete export of the production translation list.

---

## SharePoint Translation Reference

Localized interface content is maintained through a SharePoint reference list.

Each translation record associates an application element with its Portuguese and Spanish values.

A simplified public structure could contain:

```text
translation_key
pt_br
es_ar
active_status
```

A synthetic example could be represented as:

```text
home_create_emat | Criar EMAT | Crear EMAT | Active
button_next      | Próximo    | Siguiente  | Active
button_back      | Voltar     | Volver     | Active
button_clear     | Limpar     | Limpiar    | Active
button_submit    | Enviar     | Enviar     | Active
option_yes       | Sim        | Sí         | Active
option_no        | Não        | No         | Active
```

The public repository should use generalized translation keys and synthetic examples rather than production identifiers or proprietary terminology.

---

## Configuration-Driven Localization

E-SMAT uses a configuration-driven localization approach.

Instead of maintaining separate Portuguese and Spanish applications, the same controls retrieve or display content according to the active language.

Conceptually:

```text
Interface Component
      +
Translation Key
      +
Active Language
      ↓
Localized Display Value
```

This approach supports:

- Reuse of the same application screens;
- Centralized translation maintenance;
- Reduced duplication of controls;
- Consistent terminology across screens;
- Easier correction of translated content;
- Simpler support for Brazilian and Argentine users;
- Potential extension to additional languages in the future.

---

## Language State

The application maintains an active-language state during the user's session.

The initial value comes from the device or browser language. If the user selects a flag, the active-language value is updated manually.

```text
Application Start
      ↓
Set Language from Device or Browser
      ↓
User Continues in Detected Language
      │
      └── Optional Manual Override
                 ↓
          Update Active Language
```

The active language controls which translated values are displayed across the current application session.

Production variable names and formulas are intentionally omitted.

---

## Localization and Navigation

Changing the language updates the visible interface but should not redirect the user to another workflow stage.

For example, changing the language while completing a questionnaire should preserve:

- Current screen;
- Selected inspection workflow;
- Identification information;
- Selected location hierarchy;
- Applicable methodology;
- Loaded questionnaire;
- Responses already entered;
- Temporary or final record context.

The language switch changes presentation, not the inspection's operational identity.

Navigation behavior is documented in [`navigation-logic.md`](navigation-logic.md).

---

## Localization and Cascading Filters

Language selection and cascading location filtering have separate responsibilities.

### Localization

Determines how interface content is displayed:

```text
Active Language
      ↓
Portuguese or Spanish Labels
```

### Cascading filters

Determine which operational location values are available:

```text
Site
  → Segment
  → Business Unit / Area
  → Line
  → Workstation
```

Changing the interface language must not alter the selected location path.

The cascading-filter logic is documented in [`cascade-filtering.md`](cascade-filtering.md).

---

## Localization and Questionnaire Routing

The active language and applicable questionnaire are determined through different inputs.

```text
Device, Browser, or Manual Selection
      ↓
Interface Language
```

```text
Selected Workflow and Operational Location
      ↓
EMAT, SMAT, or ESMAT Questionnaire
```

The site and operational context determine which questionnaire applies. The language setting determines how supported text is displayed.

This separation prevents a language change from incorrectly changing the inspection methodology.

---

## Questionnaire Content

Questionnaire requirements are stored as reference data and loaded dynamically according to the inspection context.

Where bilingual question content is available, the application should display the version associated with the active language while preserving the same question identity and analytical meaning.

A simplified conceptual structure could include:

```text
question_id
methodology
question_version
question_pt_br
question_es_ar
active_status
```

The same stable question identifier should remain associated with both language versions so that changing the display language does not create a different business requirement or break downstream analysis.

The public repository should not include complete proprietary inspection questions.

---

## Terminology Consistency

Safety and environmental terminology requires consistent translation across the application.

Centralized reference data helps ensure that the same concept is rendered consistently in:

- Field labels;
- Questionnaire instructions;
- Safe and unsafe classifications;
- Complementary questions;
- Validation feedback;
- Submission confirmation.

Translation maintenance should consider the terminology used in each operational context rather than relying only on literal word replacement.

For the Argentine interface, the selected terms should follow the Spanish usage established for the Córdoba operation and the responsible business area.

---

## Validation Messages

Validation feedback should be displayed in the active language.

Examples include messages associated with:

- Missing required fields;
- Incomplete location selection;
- Invalid inspection context;
- Required questionnaire responses;
- Submission problems;
- Successful record creation.

The validation rule remains the same across languages. Only the message presented to the user changes.

Production message text and internal error handling are not included in the public version.

---

## Submission Confirmation

After a successful submission, E-SMAT displays a confirmation message in the active interface language.

The confirmation should reflect the applicable methodology while maintaining the selected language.

Conceptually:

```text
Successful Submission
      +
Applicable Methodology
      +
Active Language
      ↓
Localized Confirmation Message
```

Changing the language on the confirmation screen updates the visible interface content without creating or submitting another inspection record.

---

## Temporary Records and Language

A temporary inspection record must preserve the business identity of the inspection independently from the language used to display the application.

When a saved inspection is resumed, the application can display the record according to the current active language while preserving:

- Inspection identifier;
- Selected workflow;
- Operational location;
- Applicable methodology;
- Questionnaire version;
- Previously recorded responses;
- Draft status.

The language used during the original entry should not create a separate inspection or change the meaning of stored answers.

Temporary-record behavior is documented in [`temporary-save.md`](temporary-save.md).

---

## Simplified Localization Model

The following pseudocode illustrates the localization concept without reproducing production formulas:

```text
detectedLanguage = read device or browser language

if detectedLanguage is Portuguese:
    activeLanguage = pt-BR
else if detectedLanguage is Spanish:
    activeLanguage = es-AR
else:
    activeLanguage = configured fallback

if user selects Portuguese control:
    activeLanguage = pt-BR

if user selects Spanish control:
    activeLanguage = es-AR

localizedText = translation record selected by:
    translation key
    active language
    active status
```

This pseudocode documents the functional approach only.

---

## Quality Considerations

A reliable localization implementation should prevent or handle the following situations:

- Missing Portuguese or Spanish translations;
- Duplicate translation keys;
- Inactive translations being displayed;
- Unsupported browser-language values;
- Language switching resetting the current inspection;
- Different question identifiers being used for equivalent translations;
- Inconsistent terminology across screens;
- Validation messages appearing in a different language from the interface;
- Manual language changes altering questionnaire routing;
- Partial translation of the confirmation screen.

These considerations support a consistent bilingual user experience and reliable downstream data interpretation.

---

## Maintainability Benefits

The localization model provides the following benefits:

- One application supports users in two countries;
- Interface content is maintained centrally;
- Translation updates do not require duplicated screen structures;
- Terminology can be reviewed independently from navigation logic;
- Language selection remains separate from methodology selection;
- New interface elements can reuse the same translation pattern;
- Future language extensions can follow the established structure.

Any production translation update must follow the applicable governance, terminology review, testing, and release process.

---

## Relationship with Other Documentation

- [`app-overview.md`](app-overview.md) provides the application-level context;
- [`navigation-logic.md`](navigation-logic.md) documents screen transitions and language controls in the navigation flow;
- [`cascade-filtering.md`](cascade-filtering.md) documents location dependencies and questionnaire routing;
- [`temporary-save.md`](temporary-save.md) documents how inspections in progress are preserved and restored;
- [`screenshots/`](screenshots/) contains anonymized or reconstructed examples of the bilingual interface.

---

## Portfolio Scope and Data Privacy

The public version of this document does not include:

- Production Power Fx localization formulas;
- Internal variable or control names;
- Complete production translation records;
- Proprietary inspection requirements;
- Confidential operational terminology;
- Production SharePoint list names or URLs;
- Employee or user information;
- Corporate credentials or connection details.

Public examples should preserve the localization architecture while using generalized translation keys, synthetic question content, and non-confidential terminology.
