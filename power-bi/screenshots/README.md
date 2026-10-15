# Power BI Screenshots

This folder contains screenshots of the public E-SMAT Power BI report built with synthetic data.

The images document the visible dashboard pages, hidden drill-through pages, and semantic model without exposing production records, real employee information, internal locations, credentials, or confidential identifiers.

---

## Folder Structure

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

---

## Screenshot Index

| File | Report page or view | Visibility | Purpose |
|---|---|---|---|
| `01-overview.png` | Visão Geral | Visible | Executive KPIs, act classifications, themes, plants, and monthly trends |
| `02-records.png` | Registros | Visible | Inspection records by operational hierarchy and time |
| `03-observed-acts.png` | Atos Observados | Visible | Safe and unsafe acts by plant, segment, area, line, and workstation |
| `04-people.png` | Pessoas | Visible | Analyses by observer and observed-person registrations |
| `05-job-roles.png` | Cargo | Visible | Indicators by job role and synthetic employee |
| `06-record-search.png` | Consultar | Visible | Detailed inspection search and record-level filters |
| `07-drill-through-record.png` | DT_Registro | Hidden | Inspection context, complementary information, and question-level details |
| `08-drill-through-employee.png` | DT_Matricula | Hidden | Observed-person history, related observers, categories, and act indicators |
| `09-data-model.png` | Model view | Documentation | Implemented semantic model, cardinalities, and active or inactive relationships |

---

## Documentation Usage

The main Power BI README references representative screenshots, including the overview page and semantic model.

```markdown
![E-SMAT overview dashboard](screenshots/01-overview.png)
```

```markdown
![E-SMAT Power BI data model](screenshots/09-data-model.png)
```

The model screenshot is also referenced directly in:

```text
power-bi/data-model.md
```

The visible and hidden report-page screenshots support the page-by-page documentation in:

```text
power-bi/dashboard-pages.md
```

---

## Report Pages

### Visible pages

```text
Visão Geral
Registros
Atos Observados
Pessoas
Cargo
Consultar
```

### Hidden drill-through pages

```text
DT_Registro
DT_Matricula
```

### Model documentation

```text
Power BI Model view
```

---

## Privacy

All screenshots use synthetic data. The public images do not contain:

- Real employee names or registrations;
- Production inspection records;
- Internal plant or operational location names;
- Corporate email addresses;
- Production URLs or workspace names;
- Catalog, schema, or internal table names;
- Confidential questionnaire content;
- Credentials or access tokens;
- Local file paths;
- Power BI account or tenant information.

---

## Related Documentation

```text
../README.md
../dashboard-pages.md
../data-model.md
../measures.md
../key-validation-report.md
```

- `README.md`: overall Power BI portfolio summary;
- `dashboard-pages.md`: page-by-page report documentation;
- `data-model.md`: semantic model and relationships;
- `measures.md`: DAX calculations;
- `key-validation-report.md`: mock-data key and relationship validation;
- `screenshots/README.md`: screenshot inventory and documentation references.
