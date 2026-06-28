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
- `BlackKite` — Black Kite third-party cyber-risk findings, with a schema per resource type:
  - `blackkiteFinding` — attack-surface vulnerability findings (affected domain/IP/product, CVE with CVSS/CWE/EPSS and CISA KEV status, severity/status, linked tickets)
- `CloudTrail`
- `Cortex`
- `Fluency`
- `Fortigate`
- `GSuite`
- `GSuiteResources` — Google Workspace (GSuite) directory resource snapshots, with a schema per resource type:
  - `gsuiteUser` — directory users (profile/name, primary and alias emails, org unit, account/admin status, 2-Step Verification, recovery info)
  - `gsuiteGroup` — directory groups (name/email, description, aliases, direct member count, expanded member list with role/status/type)
- `K8sContainerLog`
- `Office365`
- `Office365Resources` — Entra ID / Microsoft 365 directory resource snapshots, with a schema per resource type:
  - `office365User` — directory users (profile, licenses, group/role membership, MFA/SSPR registration)
  - `office365Group` — groups (type/security flags, membership, dynamic membership rule)
  - `office365Device` — registered/joined devices (OS, compliance, trust type, last sign-in)
  - `office365Application` — application registrations (credentials, requested API permissions, redirect config)
  - `office365InstalledApp` — service principals / enterprise applications (consent, credentials, exposed permissions)
- `Qualys` — Qualys resource snapshots, with a schema per resource type:
  - `qualysHost` — tracked host assets (identity/FQDN and Qualys host IDs, IP, OS/hardware, last vuln & compliance scan and boot times, and a nested `agentInfo` Cloud Agent object; `timestamp` from `created`, `_key` from `name`)
- `SentinelOne` — SentinelOne endpoint resource snapshots, with a schema per resource type:
  - `sentinelOneAgent` — installed agents/endpoints (hardware/OS, agent version and health, mitigation posture, scan and threat status, site/group placement)
  - `sentinelOneApplication` — installed applications per endpoint (name/version/publisher, signed/risk level, install/update times, denormalized agent fields)
  - `sentinelOneThreat` — detected threats (file/classification, confidence, analyst verdict, mitigation/incident status, behavioral indicators, agent context at detection and real time)
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
