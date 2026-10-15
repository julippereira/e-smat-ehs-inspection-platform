# Power BI Screenshots

This folder contains screenshots of the public E-SMAT Power BI report built with synthetic data.

The images document the visible report pages and the hidden drill-through pages without exposing production records, employee information, internal locations, or confidential identifiers.

## Recommended Files

```text
screenshots/
├── README.md
├── 01-overview.png
├── 02-records.png
├── 03-observed-acts.png
├── 04-people.png
├── 05-job-roles.png
├── 06-record-search.png
├── 07-drill-through-record.png
├── 08-drill-through-employee.png
└── 09-data-model.png
```

## Screenshot Index

| File | Report page | Visibility | Purpose |
|---|---|---|---|
| `01-overview.png` | Visão Geral | Visible | Executive KPIs, act classifications, themes, plants, and monthly trends |
| `02-records.png` | Registros | Visible | Inspection records by operational hierarchy and time |
| `03-observed-acts.png` | Atos Observados | Visible | Safe and unsafe acts by plant, segment, area, line, and workstation |
| `04-people.png` | Pessoas | Visible | Analyses by observer and observed-person registrations |
| `05-job-roles.png` | Cargo | Visible | Indicators by job role and synthetic employee |
| `06-record-search.png` | Consultar | Visible | Detailed inspection search and record-level filters |
| `07-drill-through-record.png` | DT_Registro | Hidden | Inspection context and question-level details |
| `08-drill-through-employee.png` | DT_Matricula | Hidden | Observed-person history and related indicators |
| `09-data-model.png` | Model view | Internal documentation | Semantic model tables, relationships, and cardinalities |

## Capture Guidelines

Before capturing each image:

1. Confirm that the report is connected only to the synthetic mock data;
2. Clear temporary selections that are not relevant to the page example;
3. Use a consistent report-page zoom level;
4. Keep the complete page visible whenever possible;
5. Avoid showing Power BI Desktop account information or local file paths;
6. Confirm that employee names and registrations are synthetic;
7. Confirm that no production workspace, URL, catalog, or schema name is visible;
8. Capture drill-through pages using a synthetic selected record or registration;
9. Keep hidden pages hidden in the final published report;
10. Export images as PNG for consistent rendering on GitHub.

## File Naming

Use lowercase file names with hyphens:

```text
01-overview.png
02-records.png
03-observed-acts.png
```

The numeric prefix preserves the report navigation order when the files are displayed alphabetically.

## README Usage

The main Power BI README can show selected screenshots using relative paths:

```markdown
![Overview dashboard](screenshots/01-overview.png)
```

```markdown
![Power BI data model](screenshots/09-data-model.png)
```

The complete screenshot set does not need to be embedded in the main README. The main README should prioritize the overview page, the model diagram, and links to the detailed documentation.

## Privacy

All screenshots in this folder must use synthetic data only. Screenshots must not expose:

- Real employee names or registrations;
- Production inspection records;
- Internal plant or operational location names;
- Corporate email addresses;
- Production URLs, workspaces, catalogs, or schemas;
- Confidential questionnaire content;
- Credentials, tokens, or local file paths.
