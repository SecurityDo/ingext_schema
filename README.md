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

- `AbnormalSecurity` — Abnormal Security threat events imported via the Threats API (Email/Inbound Email Security):
  - `AbnormalThreat` — one detected threat campaign (`threatId`), holding a `messages[]` array with one element per malicious email in the campaign. Each message carries the sender/recipient (`fromAddress`/`senderDomain`/`senderIpAddress`/`recipientAddress`/`toAddresses[]`), subject, Abnormal's verdict (`attackType`/`attackStrategy`/`attackVector`/`attackedParty`/`impersonatedParty`/`summaryInsights[]`), content (`urls[]`/`links[]`/`attachmentNames[]`), and remediation state (`remediationStatus`/`autoRemediated`/`remediationTimestamp`). The array is dynamic — `mv-expand messages` and reach in with dot notation; `timestamp` from the top-level `timestamp` (`messages.receivedTime` for delivery time)
- `AzureAudit`
- `BlackKite` — Black Kite third-party cyber-risk findings, with a schema per resource type:
  - `blackkiteFinding` — attack-surface vulnerability findings (affected domain/IP/product, CVE with CVSS/CWE/EPSS and CISA KEV status, severity/status, linked tickets)
- `CloudTrail` — AWS CloudTrail audit events (control-plane and object-level activity across an AWS account), stored as a dynamic JSON document:
  - `CloudTrail` — one API-level event: the actor (`userIdentity`, a polymorphic object keyed by `type` — IAMUser/Root/AssumedRole/AWSService/AWSAccount/FederatedUser/IdentityCenterUser/WebIdentityUser), the operation (`eventName` on `eventSource`), region/account, source IP and user-agent, outcome (`responseElements`/`errorCode`), and service-specific `requestParameters`/`responseElements`. Four `eventCategory` classes — Management (control plane), Data (S3/Lambda/DynamoDB object ops), NetworkActivity (VPC endpoint), Insight (anomaly detection); `eventType` distinguishes AwsApiCall / AwsConsoleSignIn / AwsServiceEvent. `timestamp` from `eventTime`
- `CrowdStrike` — CrowdStrike Falcon EDR alerts imported via the Falcon Alerts API:
  - `FalconAlert` — Falcon alerts on an endpoint, in two shapes selected by the `type` field: `automated-lead` (the AI-generated summary/lead — risk `score`, aggregated `mitre_attack[]`, and a `threatgraph_indicators[]` rollup of the correlated detections) and `signal` (a supporting/contextual detection with full process detail — triggering `cmdline`/hashes, `parent_details`/`grandparent_details` lineage, MITRE tactic/technique, prevention disposition, and behavioral context arrays). A signal links to its lead via `aggregate_id == lead.lead_id`; `timestamp` from the top-level `timestamp` (`created_timestamp` for detection recency)
- `CrowdStrikeResources` — CrowdStrike Falcon resource snapshots imported via the Falcon API:
  - `falconAgent` — Falcon host/device inventory (identity/hostname/domain/OU/site, OS and hardware/BIOS, network addresses, sensor version and posture — RFM/containment/RTR, first/last-seen and last-login activity, and the assigned Falcon policies as a keyed `device_policies` object plus a `policies[]` array); `_key` from `device_id` (join key to `FalconAlert.agent_id`), `timestamp` is the capture time (use `last_seen`/`modified_timestamp` for host recency)
- `Cortex` — PaloAlto Cortex XDR detections imported via the Cortex XDR API, with a schema per resource type:
  - `CortexAlert` — individual XDR alerts (identity/classification, severity, affected agent/host, triggering rule, resolution/matching status, and a nested `events` causality-graph array; `timestamp` from `detection_timestamp`, joined to an incident via `case_id`)
  - `CortexIncident` — alert groups/incidents (status, effective severity and per-severity alert counts, aggregated/predicted/manual scores, affected hosts and users, sources, and console deep link; `timestamp` from `creation_time`)
- `F5BigIPLTM` — F5 BIG-IP Local Traffic Manager (LTM) HTTP access logs:
  - `F5BigIPLTM` — per-request load-balancer transactions (front-end virtual server and VIP, back-end pool and pool-member IP:port, client IP/user-agent/referer, request line and URI, response status/size/time; `timestamp` from `timestamp`)
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
- `Zscaler` — Zscaler Internet Access (ZIA) NSS streaming log feeds, with a schema per feed type:
  - `ZscalerWeb` — web proxy transactions (user/department/location/device, URL/host/method/user-agent, proxy action and rule, response code/sizes/timing, URL category class, cloud-app name/class, malware/threat, DLP, risk score; `timestamp` from `time`)
  - `ZscalerFirewall` — cloud firewall sessions (user/device/tunnel, client and server source/dest IP:port across NAT, action and matched rule, network service/application, byte counts and durations, IP category, threat, destination country; `timestamp` from `datetime`)
  - `ZscalerDNS` — DNS control requests (user/device/location, query name/type and answer, client and Zscaler-resolver addresses, domain category, and separate request/response actions and rules; `timestamp` from `datetime`)

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
