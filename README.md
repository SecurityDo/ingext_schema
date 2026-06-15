# Ingext Schema

A knowledge base describing the major data types that Fluency/Ingext stores in the
datalake. Its purpose is to give an AI agent an accurate, abstract view of the data
available for search so it can generate correct KQL queries and reports.

The agent works against the YAML schema as an abstraction layer over the data. The
underlying data may live in different physical locations, but all of it is queryable
through KQL.

## Layout

Each data type lives in its own directory under `datatypes/`. A directory may contain:

| File / directory     | Description                                                                                  |
| -------------------- | -------------------------------------------------------------------------------------------- |
| `lake_*.json`        | Parquet schema used by the index engine.                                                     |
| `info_*.yaml`        | YAML schema with descriptions for the major fields, written to help an AI agent build KQL.   |
| `query_samples.txt`  | Collection of sample KQL queries for the data type.                                          |
| `eventSamples/`      | Example raw events in JSON.                                                                   |

Not every data type includes all of these files yet.

## Data types

- `AzureAudit`
- `CloudTrail`
- `Cortex`
- `Fortigate`
- `GSuite`
- `K8sContainerLog`
- `Office365`
- `Office365Resources`
- `WindowsAudit`

## Note on samples

Event samples are sanitized: identifiers such as user names, email addresses, IP
addresses, tenant/organization GUIDs, and geolocation have been replaced with
placeholder values (for example, `contoso.com` and the `2001:db8::/32` documentation
IP range). They illustrate the shape and field set of each event, not real data.
