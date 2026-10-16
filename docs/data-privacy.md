# Data Privacy

## Overview

The public E-SMAT repository was designed to demonstrate the solution architecture and technical implementation without publishing production data, real employee information, credentials, confidential questionnaire content, or internal infrastructure identifiers.

Privacy is addressed across the application, engineering, analytical, and documentation layers.

---

## Public Data Principle

The repository follows this rule:

```text
Demonstrate the structure and logic
without exposing production content
```

Public artifacts use:

- Synthetic records;
- Generalized site and operational names;
- Synthetic employee registrations;
- Synthetic employee names;
- Example email addresses;
- Placeholder credentials;
- Placeholder catalog and schema names;
- Generalized questionnaire descriptions;
- Public-safe screenshots.

---

## Excluded Information

The repository does not include:

- Real inspection records;
- Real employee names;
- Real employee registrations;
- Corporate email addresses;
- Internal plant or site names;
- Internal segments, areas, lines, or workstations;
- Production SharePoint URLs;
- Tenant, client, site, or list identifiers;
- Client secrets or access tokens;
- Databricks workspace identifiers;
- Production catalogs, schemas, volumes, or tables;
- Proprietary questionnaire wording;
- Actual quality-failure records;
- Production Power BI workspace information.

---

## Synthetic Data

The public datasets preserve analytical behavior rather than production values.

Synthetic files reproduce:

- Table grains;
- Column structures;
- Relationship keys;
- Methodology categories;
- Date coverage;
- Safe and unsafe quantities;
- Conditional field behavior;
- Questionnaire categories;
- Employee-role references;
- Dashboard totals and interactions.

The Power BI mock contains:

```text
72 inspection headers
952 question-level responses
119 questionnaire records
1,904 classification rows
365 calendar dates
93 synthetic employee registrations
```

These quantities exist only for the public demonstration.

---

## Employee Information

Public employee registrations use synthetic numeric values. Public names follow a clearly synthetic pattern such as:

```text
Funcionário Sintético 001
```

The public employee dimension does not attempt to reproduce a real employee directory.

The auxiliary employee notebook supports optional enrichment from an authorized private source, but the private source is not configured in the public version.

```python
EMPLOYEE_MASTER_TABLE = None
```

Names and organizational attributes must only be retrieved from an authorized source in an approved environment.

---

## Credentials and Configuration

The repository contains only:

```text
graph_api_config.example.json
```

with placeholders:

```json
{
  "graph_api": {
    "tenant_id": "<tenant-id>",
    "client_id": "<client-id>",
    "client_secret": "<client-secret>",
    "site_id": "<sharepoint-site-id>"
  }
}
```

The real configuration must remain outside the repository in an approved protected location.

Files such as these must not be uploaded:

```text
graph_api_config.json
shared_graph_api.json
.env
credentials.json
secrets.json
```

---

## GitHub Web Upload Consideration

The project is uploaded through the GitHub web interface rather than a locally synchronized Git repository.

A `.gitignore` remains useful as documentation and future protection, but it does not replace manual review before a web upload. Every selected file was manually checked for credentials, production connections, real records, and local paths.

---

## Notebook Privacy Controls

Public notebooks use placeholders for:

```text
<catalog>
<schema>
<volume>
<sharepoint-site-id>
<authorized-employee-table>
```

The notebooks avoid printing:

- Client secrets;
- Access tokens;
- Private configuration values;
- Production list identifiers;
- Real source records.

Environment-specific mappings remain configuration-driven.

---

## Power BI Privacy Controls

The public PBIX contains only imported synthetic data.

Before distribution, the report do not contain:

- Hidden production queries;
- Corporate URLs;
- Production workspace references;
- Real parameters;
- Cached production rows;
- Real names or registrations;
- Local paths containing sensitive information;
- Production dataset or gateway connections.

The public screenshots use synthetic data and exclude account, tenant, workspace, and file-path details.

---

## Identifiers

Identifiers are treated according to their analytical purpose.

### Inspection identifiers

Public inspection identifiers are synthetic and preserve only the structural pattern required by the report.

### Employee registrations

Registrations are synthetic and stored as text in Power BI, even when they contain only digits.

### Stable analytical keys

Keys such as `response_key` and `classification_key` are generated from synthetic identifiers and do not reveal production keys.

---

## Operational Hierarchy

Public operational values use neutral identifiers such as:

```text
SITE_A
SEGMENT_A_01
AREA_A_001
LINE_A_001
POSTO_A_001
```

They demonstrate hierarchy and filtering without reproducing actual organizational structures.

---

## Questionnaire Content

The public questionnaire contains synthetic descriptions designed to reproduce:

- Themes;
- Categories;
- Question order;
- Contextual relationships;
- Safe and unsafe quantities;
- Drill-through details.

It does not publish proprietary requirements or production wording.

---

## Data Minimization

Only fields required to explain the architecture, relationships, transformations, quality controls, or dashboard behavior are included.

Potential employee-directory attributes remain optional:

```text
employee_name
employee_status
employee_job_title
employee_department
```

A production implementation should include only authorized attributes that are necessary for the intended analysis.

---

## Public Portfolio Boundary

The repository demonstrates technical capability, not access to production environments. Public documentation should explain architecture and decisions without enabling reconstruction of confidential systems, operational structures, or inspection history.

---

## Related Documentation

```text
business-context.md
inspection-methodologies.md
technical-decisions.md
limitations.md
../data-engineering/config/graph_api_config.example.json
../power-bi/README.md
```
