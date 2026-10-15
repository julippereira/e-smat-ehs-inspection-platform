# SharePoint Data Ingestion

## Overview

The E-SMAT ingestion process extracts inspection and questionnaire data from Microsoft SharePoint Lists through Microsoft Graph API and stores the source-aligned records in Bronze Delta tables.

The ingestion layer is responsible only for securely retrieving, validating, standardizing, and persisting the source data. Business transformations, analytical enrichment, and reporting logic are intentionally handled in later pipeline stages.

The implementation is available in:

[`notebooks/02_sharepoint_ingestion.ipynb`](notebooks/02_sharepoint_ingestion.ipynb)

Reusable functions are maintained separately in:

[`notebooks/01_shared_functions.ipynb`](notebooks/01_shared_functions.ipynb)

> Production credentials, SharePoint identifiers, internal field names, catalog names, schema names, and storage paths are not included in the public repository.

---

## Ingestion Scope

The ingestion notebook processes three SharePoint datasets:

| Dataset | Purpose | Bronze output |
|---|---|---|
| Inspection header | Stores the inspection-level operational context and complementary answers | `inspection_header_bronze` |
| Inspection responses | Stores one record per inspection question | `inspection_responses_bronze` |
| Inspection requirements | Stores questionnaire configuration and question metadata | `inspection_requirements_bronze` |

The ingestion process does not load the public CSV files from the `data/sample/` folder. Those files are synthetic portfolio assets that reproduce the logical structure of the production datasets.

---

## High-Level Flow

```text
Protected configuration
        ↓
OAuth 2.0 authentication
        ↓
Microsoft Graph API
        ↓
Discover SharePoint Lists
        ↓
Extract paginated records
        ↓
Read configured fields
        ↓
Normalize source representations
        ↓
Validate record schema
        ↓
Create Spark DataFrames
        ↓
Standardize column names
        ↓
Add ingestion metadata
        ↓
Write Bronze Delta tables
```

---

## Notebook Responsibilities

The ingestion notebook performs the following operations:

1. Loads the shared utility notebook;
2. Reads API configuration from a protected location;
3. Authenticates with Microsoft Graph API;
4. Defines source-list and target-table mappings;
5. Validates the ingestion configuration;
6. Discovers the available SharePoint Lists;
7. Resolves the configured list names to list identifiers;
8. Retrieves all list items using pagination;
9. Extracts and formats the configured fields;
10. Validates the record schema before DataFrame creation;
11. Creates source-aligned Spark DataFrames;
12. Renames source fields to standardized analytical names;
13. Adds ingestion metadata;
14. Writes the Bronze Delta tables;
15. Displays extraction and load summaries.

---

## Configuration Management

### Public configuration example

The repository contains a configuration template:

[`config/graph_api_config.example.json`](config/graph_api_config.example.json)

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

This example file contains placeholders only and can be safely versioned.

### Private production configuration

The production configuration must remain outside the Git repository. The notebook expects the private JSON file to be stored in an approved protected location, such as a Unity Catalog Volume.

Conceptually:

```python
CONFIG_PATH = Path(
    "/Volumes/<catalog>/<schema>/<volume>/graph_api_config.json"
)
```

The private file provides:

- Microsoft Entra tenant identifier;
- Application client identifier;
- Application client secret;
- SharePoint site identifier.

The notebook does not print the credentials or the access token.

---

## Authentication

The notebook uses the OAuth 2.0 client-credentials flow to obtain an application access token for Microsoft Graph API.

```text
Tenant ID
Client ID
Client Secret
      ↓
OAuth 2.0 token endpoint
      ↓
Application access token
      ↓
Authorized Microsoft Graph requests
```

The requested scope is:

```text
https://graph.microsoft.com/.default
```

The authentication request includes a timeout and raises a controlled error when the token request is unsuccessful or the response does not contain an access token.

The token and credentials remain in memory only for the active notebook session.

---

## Dataset Configuration

The notebook uses a configuration-driven structure instead of repeating the extraction logic for each SharePoint List.

Each dataset configuration contains:

```text
sharepoint_list_name
      ↓
Name used to discover the source list

target_table
      ↓
Fully qualified Bronze Delta table

columns
      ↓
Source fields expected from SharePoint

rename_mapping
      ↓
Mapping from source fields to standardized names

field_aliases
      ↓
Optional mapping for SharePoint internal field names
```

A simplified example is:

```python
LISTS_CONFIG = {
    "inspection_responses": {
        "sharepoint_list_name": "<inspection-responses-list>",
        "target_table": (
            "<catalog>.<bronze-schema>.inspection_responses_bronze"
        ),
        "columns": [
            "Title",
            "Site",
            "QuestionId",
            "SafeAct",
            "UnsafeAct",
            "Description",
            "InspectionDate",
            "Author",
        ],
        "rename_mapping": {
            "Title": "inspection_id",
            "Site": "site",
            "QuestionId": "question_id",
            "SafeAct": "safe_act",
            "UnsafeAct": "unsafe_act",
            "Description": "description",
            "InspectionDate": "inspection_date",
            "Author": "created_by",
        },
        "field_aliases": {},
    }
}
```

Production list names and internal field aliases remain only in the private implementation.

---

## Configuration Validation

Before connecting to SharePoint, the notebook validates the ingestion configuration.

The checks include:

- The configuration dictionary is not empty;
- Every dataset has the required configuration keys;
- Every dataset has at least one configured source column;
- Source columns are not duplicated;
- Rename mappings reference configured fields only;
- Internal-field aliases reference configured fields only;
- Target tables use a fully qualified three-part name;
- Multiple datasets do not write to the same Bronze table.

Configuration errors stop the notebook before any extraction request is made.

---

## SharePoint List Discovery

The notebook first retrieves the SharePoint Lists available in the configured site.

```text
GET /sites/{site_id}/lists
```

The response is converted into a mapping between:

```text
displayName → list ID
```

The configured list names are then matched against the discovered lists.

If a configured list is not found:

- The dataset is marked as `not_found`;
- The dataset is included in the extraction summary;
- The remaining configured lists can continue to be processed.

If no configured dataset can be processed, the notebook stops before the Bronze write stage.

---

## Pagination

Both list discovery and item extraction support Microsoft Graph pagination.

The shared function follows the `@odata.nextLink` value until the final page is reached:

```text
Initial request
      ↓
Read current page
      ↓
Append returned values
      ↓
Is @odata.nextLink available?
      │
      ├── Yes → Request next page
      └── No  → Finish extraction
```

This prevents the pipeline from silently loading only the first page of a SharePoint List.

The reusable pagination logic is maintained in `01_shared_functions.ipynb`:

```python
def get_paginated_graph_values(
    initial_url,
    request_headers,
    timeout_seconds,
):
    ...
```

---

## SharePoint Field Extraction

Each item returned by the Graph API contains a `fields` object. The pipeline extracts only the fields configured for the current dataset.

The field lookup supports:

- Exact field-name matching;
- Case-insensitive matching;
- Explicit internal-field aliases;
- Normalized matching without non-alphanumeric characters;
- Common representations of person and author fields.

Conceptually:

```text
Graph list item
      ↓
fields object
      ↓
Configured source field
      ↓
Flexible field lookup
      ↓
Extracted raw value
```

Field aliases are passed as configuration rather than hard-coded into the shared functions.

---

## Source Value Normalization

Before DataFrame creation, common SharePoint and Microsoft Graph representations are converted to stable text values.

### Complex values

Dictionary and list values are reduced to readable content using common keys such as:

```text
displayName
Title
name
email
lookupValue
text
```

### Boolean values

Boolean source values are initially represented as localized text:

```text
true  → Sim
false → Não
```

The Silver transformation notebook later converts these values to Spark Boolean types.

### Dates

Supported date representations are initially normalized to:

```text
DD/MM/YYYY
```

The date parser supports common Portuguese, Spanish, and English month names, as well as numeric ISO and localized formats.

### Long text and HTML

Description, observation, and question fields receive:

- HTML entity decoding;
- HTML tag removal;
- Whitespace normalization.

This removes formatting artifacts without applying business interpretation to the source content.

---

## Schema Validation Before DataFrame Creation

Schema validation occurs before the Spark DataFrame is created.

The pipeline verifies:

- Records are provided as a list;
- Every record is a dictionary;
- The configured source schema is not empty;
- Source columns are unique;
- Rename mappings are valid;
- Standardized target names are unique;
- Every record contains all expected fields;
- Records do not contain unexpected fields;
- Fully null columns are identified.

Fully null fields generate a warning rather than automatically stopping the pipeline, because optional SharePoint columns may legitimately contain no values in a specific extraction.

```text
Extracted records
      ↓
Validate configured schema
      ↓
Validate every record
      ↓
Identify fully null columns
      ↓
Create Spark DataFrame
```

This order prevents malformed structures from being silently accepted by Spark.

---

## Spark DataFrame Creation

The Bronze ingestion layer initially creates all source fields as nullable strings.

```python
StructField(column_name, StringType(), True)
```

This design reduces ingestion failures caused by inconsistent SharePoint representations and postpones explicit data typing to the Silver transformation stage.

The records are converted into ordered tuples based on the configured column sequence. This prevents the DataFrame schema from depending on dictionary-key order.

After creation, the notebook confirms that the generated DataFrame columns exactly match the configured source schema.

---

## Column Standardization

After the source DataFrame is created, the SharePoint fields are renamed to standardized English names.

Examples:

```text
Title           → inspection_id
Site            → site
QuestionId      → question_id
SafeAct         → safe_act
UnsafeAct       → unsafe_act
Description     → description
InspectionDate  → inspection_date
Author          → created_by
```

A second validation confirms that:

- The resulting columns match the expected standardized schema;
- The renaming process did not create duplicate columns.

This produces stable column names for later notebooks without changing the underlying business values.

---

## Ingestion Metadata

Every Bronze DataFrame receives the following operational metadata:

```text
processing_timestamp
source_system
source_list
```

### `processing_timestamp`

Records when the source data was processed by the notebook.

### `source_system`

Identifies the source platform:

```text
sharepoint
```

### `source_list`

Identifies the configured source list associated with the dataset.

These fields support traceability, monitoring, and future troubleshooting.

---

## Bronze Layer

The Bronze layer preserves standardized source values before business transformation.

```text
inspection_header_bronze
inspection_responses_bronze
inspection_requirements_bronze
```

The Bronze tables contain:

- Source-aligned values represented mainly as strings;
- Standardized English column names;
- Cleaned source representations;
- Ingestion metadata;
- No analytical enrichment;
- No final business normalization;
- No Gold-layer calculations.

The Bronze layer separates source acquisition from downstream business logic.

---

## Delta Write Strategy

The portfolio notebook uses:

```python
WRITE_MODE = "overwrite"
OVERWRITE_SCHEMA = "true"
```

The write operation follows this pattern:

```python
(
    dataframe.write.format("delta")
    .mode(WRITE_MODE)
    .option("overwriteSchema", OVERWRITE_SCHEMA)
    .saveAsTable(target_table)
)
```

This represents a full-refresh ingestion strategy. If production requirements change to incremental ingestion, the write pattern can be replaced with an append or Delta `MERGE` strategy without changing the logical dataset configuration.

---

## Extraction and Load Summaries

The notebook creates two execution summaries.

### Extraction summary

Records the source-processing result:

```text
dataset
status
records
target_table
```

Possible statuses include:

```text
processed
empty
not_found
```

### Bronze load summary

Records the persistence result for each processed dataset:

```text
dataset
target_table
records
status
```

The summaries are displayed in the notebook to make the execution outcome immediately visible.

---

## Error Handling

The ingestion notebook stops or reports controlled errors for situations such as:

- Configuration file not found;
- Invalid JSON configuration;
- Missing configuration keys;
- Microsoft Graph authentication failure;
- Missing access token;
- Invalid ingestion configuration;
- Graph API request failure;
- Non-JSON Graph response;
- Schema mismatch;
- Duplicate standardized columns;
- No datasets available for persistence;
- Invalid target-table configuration.

Error messages avoid printing credentials or access tokens.

---

## Security and Privacy

The public ingestion implementation does not include:

- Real tenant identifiers;
- Real client identifiers;
- Client secrets;
- SharePoint site identifiers;
- Production list names;
- SharePoint internal field aliases;
- Unity Catalog names;
- Production schema names;
- Production table names;
- Real records or user identifiers.

The real configuration file must remain excluded from source control.

Recommended `.gitignore` entries:

```gitignore
# Microsoft Graph API credentials
graph_api_config.json
shared_graph_api.json

# Other sensitive configuration
*credentials*.json
*secrets*.json
.env
.env.*

# Databricks local configuration
.databricks/

# Python
__pycache__/
*.pyc
```

---

## Relationship with the Sample Data

The synthetic datasets in [`../data/sample/`](../data/sample/) reproduce the logical structures processed by the ingestion pipeline:

```text
sample-inspection-header.csv
      ↓ represents
inspection_header_bronze

sample-inspection-responses.csv
      ↓ represents
inspection_responses_bronze

sample-requirements.csv
      ↓ represents
inspection_requirements_bronze
```

The CSV files are provided for public inspection and local portfolio demonstrations. The production notebook retrieves data directly from SharePoint rather than reading these CSV files.

---

## Downstream Processing

After the Bronze tables are created, the next notebook reads them independently:

[`notebooks/03_data_transformation.ipynb`](notebooks/03_data_transformation.ipynb)

The transformation stage applies:

- Explicit Spark data types;
- Bilingual value normalization;
- Date conversion;
- Audit-type derivation;
- Stable analytical keys;
- Requirement enrichment;
- Bronze-to-Silver transformation.

```text
SharePoint
      ↓
02_sharepoint_ingestion.ipynb
      ↓
Bronze Delta tables
      ↓
03_data_transformation.ipynb
      ↓
Silver Delta tables
```

---

## Portfolio Scope

This documentation demonstrates the ingestion architecture, authentication pattern, pagination, schema validation, source standardization, metadata strategy, and Bronze persistence approach used by E-SMAT.

The public implementation is intentionally environment-agnostic and cannot connect to the production SharePoint environment without private configuration, authorized application permissions, production list mappings, and infrastructure-specific target names.
