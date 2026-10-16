# Limitations

## Overview

The public E-SMAT repository is a portfolio representation of a production-inspired solution. It demonstrates architecture, application behavior, data engineering, quality controls, semantic modeling, DAX, and reporting with synthetic data.

The public version is intentionally constrained and should not be interpreted as a deployable copy of the production environment.

---

## Public Data Limitations

The synthetic datasets preserve structure and analytical consistency but do not reproduce real operational distributions.

The mock does not represent:

- Actual inspection volumes;
- Actual site performance;
- Real employee activity;
- Production methodology adoption;
- Real unsafe-condition frequency;
- Actual near-miss trends;
- Production Safety Tag activity;
- Real questionnaire history.

Dashboard results are demonstration values only.

---

## Application Availability

The repository documents the Power Apps behavior but does not provide a production-ready application package connected to public services.

Without the private environment, the public artifacts cannot reproduce:

- Production connectors;
- SharePoint permissions;
- Environment variables;
- Internal list identifiers;
- Corporate authentication;
- Production delegation behavior;
- Organizational application policies.

---

## Offline Capability

The architecture diagram identifies offline capability as a supported design consideration. Actual offline behavior depends on the Power Apps client, connector behavior, cached data, synchronization logic, and organizational configuration.

The public repository does not provide a complete offline synchronization test suite or conflict-resolution implementation.

---

## SharePoint and Graph Configuration

The ingestion notebook cannot connect to a real SharePoint site without:

- An authorized application registration;
- Tenant and client identifiers;
- A valid client secret or approved authentication alternative;
- Site and list identifiers;
- Required Microsoft Graph permissions;
- Private field aliases;
- Network and workspace access.

All public values are placeholders.

---

## Databricks Environment

The notebooks require a configured Databricks environment with Spark, Delta Lake, and permission to create or overwrite the configured tables.

The public notebooks do not include:

- Cluster policies;
- Job definitions;
- Production schedules;
- Unity Catalog grants;
- Secret scopes;
- Alerting configuration;
- Retry policies;
- Environment-specific performance tuning.

---

## Full-Refresh Strategy

The public notebooks use:

```python
WRITE_MODE = "overwrite"
OVERWRITE_SCHEMA = "true"
```

This improves clarity and reproducibility but is not necessarily the preferred production strategy for larger volumes.

Possible production enhancements include:

- Incremental extraction;
- Delta `MERGE`;
- Change tracking;
- Watermarks;
- Batch identifiers;
- Historical quality monitoring;
- Partitioning and optimization.

---

## Data-Quality History

The public quality-monitoring tables represent the latest execution because they use overwrite mode.

The implementation does not retain a long-term quality history unless append mode and an execution identifier are added.

---

## Quality Rules

The 23 public checks cover key structural and business conditions, but they are not an exhaustive corporate data-governance framework.

The public implementation does not include:

- Site-specific thresholds;
- Statistical anomaly detection;
- Volume expectations;
- Freshness service-level targets;
- Automated owner notifications;
- Quarantine workflows;
- Rule ownership metadata;
- Historical failure trends.

---

## Employee Enrichment

The public employee dimension uses synthetic names. The optional authorized employee source is not configured.

Employee attributes such as job title, department, status, and manager depend on source availability, permissions, data minimization, and privacy approval.

The public Power BI clone retains the existing employee-role relationships. The normalized bridge produced by `06_employee_dimension.ipynb` is documented as a future model enhancement rather than fully integrated into the PBIX.

---

## Power BI Source Paths

The PBIX can contain imported synthetic data and open without production access. Refreshing locally may require updating CSV source paths in Power Query to the downloaded `mock-data/` folder.

The repository does not guarantee portable relative file paths across every operating system and local folder structure.

---

## PBIX Version Control

A `.pbix` file is a binary artifact and does not provide readable line-by-line changes in GitHub.

The repository compensates with:

- Screenshots;
- DAX documentation;
- Semantic-model documentation;
- Synthetic CSV sources;
- Theme JSON;
- Key-validation reports.

A future PBIP version could improve source-control transparency but is outside the current scope.

---

## Visual Layout

The public clone preserves the existing layout rather than redesigning every visual for the synthetic cardinality.

The mock contains more segment values than the original filter layout was designed to display. Some segment labels are therefore truncated.

Optional workstation values may appear blank. These conditions are accepted because the objective is to demonstrate the functioning dashboard with synthetic data, not to optimize every visual for the public mock.

---

## Calendar Scope

The public calendar covers only:

```text
01/01/2026 to 31/12/2026
```

The current `periodo` ordering is sufficient for one year. A multi-year model should add a dedicated numeric year-month sort key.

---

## Methodology Documentation

The repository explains how EMAT, SMAT, and ESMAT are handled technically. It does not publish formal corporate methodology definitions, procedural ownership, regulatory interpretation, or proprietary inspection standards.

---

## Architecture Diagrams

The architecture images are explanatory diagrams rather than infrastructure-as-code or deployable environment definitions.

They omit implementation details such as:

- Network topology;
- Identity architecture;
- Firewall rules;
- Capacity sizing;
- Disaster recovery;
- Tenant configuration;
- Deployment pipelines;
- Licensing requirements.

---

## Performance and Scale

The public datasets are small and do not represent production-scale performance.

The repository does not provide benchmark results for:

- Large SharePoint Lists;
- Graph API throttling;
- High-volume Spark transformations;
- Concurrent Power Apps users;
- Large semantic models;
- Power BI refresh duration;
- DirectQuery or composite models.

---

## Testing Scope

The public project includes schema, key, relationship, quantity, and quality validations, but it does not provide a complete automated test framework for every Power Apps formula, notebook branch, DAX measure, visual interaction, or drill-through state.

---

## Security Evaluation

The repository applies public-safety practices and excludes sensitive artifacts, but it is not a formal security assessment, privacy impact assessment, penetration test, or compliance certification.

Production deployment requires appropriate organizational review.

---

## Maintenance

The documentation represents the solution at the time of publication. Future changes to Power Apps, SharePoint, Microsoft Graph, Databricks, Power BI, questionnaires, or methodology rules may require updates to the notebooks, mock data, screenshots, and documentation.

---

## Appropriate Use

The public repository is appropriate for:

- Portfolio review;
- Architecture discussion;
- Technical learning;
- Data-model demonstration;
- Documentation examples;
- Synthetic dashboard exploration.

It is not appropriate as:

- A production deployment package;
- A source of real business performance;
- A substitute for corporate procedures;
- A source of employee information;
- A compliance or regulatory reference;
- A complete enterprise solution template without further engineering.

---

## Related Documentation

```text
business-context.md
inspection-methodologies.md
technical-decisions.md
data-privacy.md
../architecture/README.md
../power-apps/README.md
../data-engineering/README.md
../power-bi/README.md
```
