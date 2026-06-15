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
| `query_samples.html` | Same query samples rendered as HTML (description, query, and an output table) for UI/agents. |
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

### Sanitization requirement

All event samples committed to this repository **must** be sanitized first. Never
commit raw events containing real customer or tenant data. Before adding a sample,
replace every real identifier with a consistent placeholder while keeping the JSON
structure and field types intact:

| Field type                         | Replace with                                              |
| ---------------------------------- | --------------------------------------------------------- |
| User names / email addresses / UPN | `contoso.com` names (e.g. `adele.vance@contoso.com`)      |
| Tenant / organization / context ID | `00000000-0000-0000-0000-000000000001`                    |
| Other GUIDs (session, message, app)| Placeholder GUIDs of the same format                      |
| IP addresses                       | Documentation ranges: `2001:db8::/32` (v6), `192.0.2.0/24` (v4) |
| Domain / organization name         | `contoso.onmicrosoft.com`                                 |
| SIDs                               | `S-1-5-21-1111111111-2222222222-3333333333-1001`          |
| Hostnames, message subjects, geo   | Generic placeholders                                      |

Well-known, non-sensitive constants (such as Microsoft's Graph app ID
`00000003-0000-0000-c000-000000000000`) may be left as-is. After sanitizing, confirm
the sample is still valid JSON and grep for any leftover real identifiers before
committing.
