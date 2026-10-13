# Homelab Security Monitoring

A hands-on SOC homelab built to practice security monitoring, log analysis, cloud audit ingestion, detection engineering, incident investigation, and break-fix troubleshooting.

The lab centers on a dedicated Ubuntu Server running Wazuh in Docker and integrates local endpoints, Microsoft 365 audit activity, uptime monitoring, and self-hosted services. The goal is not only to deploy tools, but to understand how telemetry flows through the SIEM, intentionally break components, troubleshoot failures, recover safely, and document the lessons as a blue-team portfolio.

## Current lab status

- Ubuntu Server operational
- Docker and Portainer operational
- Wazuh single-node stack running **4.14.5**
- Wazuh manager, indexer, and dashboard healthy
- Ubuntu and Windows endpoints reporting to Wazuh
- Microsoft 365 Business Standard audit activity integrated into Wazuh
- Microsoft Purview unified auditing confirmed enabled
- Office 365 Management Activity API authenticated and producing alerts
- Uptime Kuma monitoring the public website
- Nextcloud, Jellyfin, and Home Assistant running as additional self-hosted services
- Wi-Fi connectivity configured on Ubuntu Server using Netplan and a USB wireless adapter

## Architecture

```text
                    Microsoft 365
               Purview / Entra audit
                       |
          Office 365 Management Activity API
                       |
                       v
+------------------------------------------------+
| Ubuntu Server                                  |
|                                                |
|  Docker                                        |
|  +---------------- Wazuh -------------------+  |
|  | Manager | Indexer | Dashboard           |  |
|  +------------------------------------------+  |
|                                                |
|  Portainer | Uptime Kuma | Nextcloud          |
|  Jellyfin  | Home Assistant                   |
+------------------------------------------------+
                |                   |
                | Wazuh agents      | Browser/API
                v                   v
        Windows endpoint       SOC analyst
        Ubuntu endpoint        workstation
```

## Portfolio method

Every major exercise follows the same workflow:

```text
Build -> Verify -> Break -> Investigate -> Fix -> Verify -> Document -> Commit
```

This makes the repository a record of both successful implementation and real troubleshooting.

## Why I built this

The goal is to move beyond isolated tutorials and practice the full monitoring lifecycle:

1. deploy infrastructure,
2. onboard endpoints and cloud sources,
3. generate security-relevant events,
4. ingest and normalize telemetry,
5. investigate alerts,
6. write and tune detections,
7. intentionally break components,
8. troubleshoot and recover,
9. document findings and remediation.

This lab supports my development toward Cybersecurity Analyst / SOC Analyst work.

## Key technologies

- Ubuntu Server
- Docker / Docker Compose
- Portainer
- Wazuh 4.14.5
- OpenSearch / Wazuh Indexer
- Filebeat
- Microsoft Entra ID
- Microsoft Purview Audit
- Office 365 Management Activity API
- Microsoft 365 Business Standard
- Uptime Kuma
- Nextcloud
- Jellyfin
- Home Assistant
- Git / GitHub

## Featured labs

### Lab 03 — Microsoft 365 Audit Monitoring with Wazuh

Documents the complete Microsoft 365 integration, including Purview audit verification, Entra application registration, `ActivityFeed.Read`, protected client-secret handling, Wazuh Office 365 configuration, ingestion validation, and troubleshooting.

[Open Lab 03](labs/03-microsoft365-integration/README.md)

### Lab 04 — Office 365 Alert Investigation

A repeatable SOC investigation template for analyzing real Microsoft 365 alerts in Wazuh, beginning with mailbox-permission activity.

[Open Lab 04](labs/04-office365-investigation/README.md)

## Break-fix case studies

### Wazuh Indexer Authentication Failure

A credential rotation created a mismatch between Docker services and the OpenSearch security database. The case documents log analysis, root-cause identification, rollback, and end-to-end validation of Microsoft 365 telemetry after recovery.

[Open the break-fix case](break-fix/indexer-authentication-failure/README.md)

## Microsoft 365 monitoring

The integration uses a dedicated Entra application registration with the Office 365 Management APIs `ActivityFeed.Read` application permission.

Wazuh collects:

```text
Audit.AzureActiveDirectory
Audit.Exchange
Audit.SharePoint
Audit.General
```

The client secret is stored in a protected file mounted into the Wazuh manager container and referenced with `client_secret_path`. No client secret, Wi-Fi password, Wazuh password, or cluster key is committed to this repository.

The integration was validated by confirming:

- Wazuh Office 365 module starts successfully
- Microsoft authentication succeeds
- Office 365 JSON events appear in `alerts.json`
- events are indexed into `wazuh-alerts-*`
- events are visible in the Wazuh dashboard

## Detection and investigation focus

The current learning path is intentionally focused on becoming grounded in Wazuh before adding another SIEM platform.

Planned exercises include:

- Microsoft 365 alert triage
- mailbox permission investigations
- Linux authentication investigations
- Windows endpoint investigations
- custom Wazuh rules
- custom decoders
- File Integrity Monitoring
- vulnerability detection
- Security Configuration Assessment
- false-positive tuning
- threat-intelligence enrichment
- Active Response
- dashboard and reporting workflows

## Repository structure

```text
.
├── README.md
├── .gitignore
├── docs/
│   ├── architecture.md
│   ├── microsoft-365-wazuh.md
│   ├── networking.md
│   ├── troubleshooting.md
│   └── roadmap.md
├── labs/
│   ├── 03-microsoft365-integration/
│   └── 04-office365-investigation/
├── break-fix/
│   └── indexer-authentication-failure/
├── notes/
│   └── command-reference.md
└── screenshots/
    └── README.md
```

## Security note

This repository is intentionally sanitized. Secrets and sensitive values are excluded, including:

- passwords
- Microsoft client secret values
- Wazuh cluster keys
- Wi-Fi credentials
- API tokens
- private certificates

Example configuration uses placeholders instead of real credentials. A `.gitignore` is included to reduce the risk of accidentally committing common secret and backup files.

## Lessons learned

The biggest lesson from this lab is that building a SOC is not just about installing tools. The real learning comes from understanding data flow, authentication, networking, permissions, persistence, filtering, and recovery.

Examples include:

- resolving Wazuh manager/agent version mismatches,
- validating Filebeat-to-indexer communication,
- repairing Microsoft 365 authentication,
- confirming audit ingestion at the JSON and index layers,
- recovering from credential-rotation issues without deleting Wazuh data,
- configuring Ubuntu Server Wi-Fi with Netplan,
- separating cloud-side configuration from SIEM-side configuration,
- identifying dashboard filters that hide otherwise healthy telemetry.

## Current learning roadmap

The immediate priority is Wazuh depth rather than adding more infrastructure:

1. investigate real alerts,
2. build custom detections,
3. tune noisy rules,
4. practice file-integrity and endpoint monitoring,
5. work with vulnerability and configuration findings,
6. create repeatable break-fix exercises,
7. document every result as a portfolio case study.
