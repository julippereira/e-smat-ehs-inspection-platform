# E-SMAT Application

## About This Folder

This folder documents the Microsoft Power Apps component of the E-SMAT solution.

E-SMAT is a bilingual application developed to standardize safety and environmental inspections across multiple operational sites in Brazil and Argentina. The application supports EMAT, SMAT, and ESMAT inspection methodologies through configurable workflows, cascading location filters, dynamic questionnaires, and SharePoint-based reference data.

The application automatically loads the interface in Brazilian Portuguese or Argentine Spanish according to the language configured on the user's device or browser. Users can also switch the interface language manually during the inspection flow.

> This folder presents an anonymized portfolio representation of the application. Production connections, proprietary requirements, internal identifiers, and confidential records are not included.

---

## Application Workflows

E-SMAT provides two inspection entry points:

- **Create EMAT**, for environmental inspections;
- **Create SMAT / ESMAT**, for safety or integrated safety and environmental inspections.

The applicable questionnaire is loaded dynamically according to the selected workflow, operational location, methodology, active questionnaire version, and interface language.

---

## Documentation

- [`app-overview.md`](app-overview.md)  
  Application purpose, business context, methodologies, scope, and main capabilities.

- [`navigation-logic.md`](navigation-logic.md)  
  Screen sequence, workflow branching, navigation controls, validation, and submission flow.

- [`cascade-filtering.md`](cascade-filtering.md)  
  Cascading location hierarchy and its relationship with questionnaire routing.

- [`state-persistence.md`](state-persistence.md)  
  In-session data persistence, navigation state, final submission, and SharePoint storage logic.

- [`localization.md`](localization.md)  
  Automatic device or browser language detection, manual language switching, and translation management.

- [`screenshots/`](screenshots/)  
  Anonymized screenshots of the shared, EMAT, and SMAT / ESMAT application workflows.

---

## Folder Structure

```text
app/
├── README.md
├── app-overview.md
├── navigation-logic.md
├── cascade-filtering.md
├── state-persistence.md
├── localization.md
└── screenshots/
    ├── README.md
    ├── shared/
    ├── emat/
    └── smat-esmat/
```

---

## Key Application Capabilities

- Bilingual interface in Brazilian Portuguese and Argentine Spanish;
- Automatic device or browser language detection;
- Manual language switching;
- Separate EMAT and SMAT / ESMAT entry points;
- Cascading operational-location filters;
- Automatic SMAT or ESMAT routing;
- Dynamic questionnaire loading;
- Safe and unsafe observation registration;
- In-session preservation of entered information;
- SharePoint-based reference and inspection data;
- Final submission confirmation;
- No user-facing post-submission editing workflow.

---

## Technologies

- Microsoft Power Apps;
- Power Fx;
- Microsoft SharePoint Online;
- SharePoint Lists.

The downstream data ingestion, transformation, and analytics components are documented outside this folder.

---

## Suggested Reading Order

For a complete understanding of the application, follow this sequence:

1. [`app-overview.md`](app-overview.md)
2. [`navigation-logic.md`](navigation-logic.md)
3. [`cascade-filtering.md`](cascade-filtering.md)
4. [`localization.md`](localization.md)
5. [`state-persistence.md`](state-persistence.md)
6. [`screenshots/README.md`](screenshots/README.md)

---

## Portfolio Scope

This documentation focuses on the application architecture, user journey, configuration approach, localization, navigation state, and questionnaire-routing logic.

The public repository does not include production Power Apps packages, complete Power Fx formulas, real SharePoint connections, identifiable users, real operational hierarchies, confidential inspection records, or the complete proprietary requirement set.
