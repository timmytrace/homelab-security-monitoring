# Homelab Security Monitoring

A hands-on SOC homelab built to practice security monitoring, log analysis, cloud audit ingestion, endpoint visibility, and detection engineering.

The lab centers on a dedicated Ubuntu Server running Wazuh in Docker and integrates local endpoints, Microsoft 365 audit activity, uptime monitoring, and self-hosted services. The project is intentionally documented as a practical blue-team portfolio project rather than a production deployment.

## Current lab status

- Ubuntu Server host operational
- Docker and Portainer operational
- Wazuh single-node stack upgraded to **4.14.5**
- Wazuh manager, indexer, and dashboard healthy
- Ubuntu and Windows endpoints reporting to Wazuh
- Microsoft 365 Business Standard audit activity integrated into Wazuh
- Microsoft Purview unified auditing confirmed enabled
- Office 365 Management Activity API integration authenticated and producing alerts
- Uptime Kuma monitoring the public website
- Nextcloud, Jellyfin, and Home Assistant running as additional self-hosted services
- Wi-Fi connectivity configured on Ubuntu Server using Netplan and a USB wireless adapter
- Active Directory lab planned as the next major expansion

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
|  Portainer | Uptime Kuma | Nextcloud           |
|  Jellyfin  | Home Assistant                    |
+------------------------------------------------+
                |                   |
                | Wazuh agents      | Browser/API
                v                   v
        Windows endpoint       SOC analyst
        Ubuntu endpoint        workstation

Future expansion:
Windows Server -> Active Directory -> Windows client -> Wazuh
```

## Why I built this

The goal is to move beyond isolated tutorials and build an environment where I can practice the full monitoring lifecycle:

1. deploy infrastructure,
2. onboard endpoints and cloud sources,
3. generate security-relevant events,
4. ingest and normalize telemetry,
5. investigate alerts,
6. troubleshoot broken pipelines,
7. document findings and remediation.

This lab is also being used to support my development toward a Cybersecurity Analyst / SOC Analyst role.

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

## Microsoft 365 monitoring

The Microsoft 365 integration uses a dedicated Entra application registration with the **Office 365 Management APIs** `ActivityFeed.Read` application permission.

Wazuh is configured to collect these content types:

```text
Audit.AzureActiveDirectory
Audit.Exchange
Audit.SharePoint
Audit.General
```

The client secret is stored in a protected file mounted into the Wazuh manager container and referenced with `client_secret_path`. No client secret, tenant secret, Wi-Fi password, Wazuh password, or cluster key is committed to this repository.

The integration was validated end-to-end by confirming:

- Wazuh Office 365 module starts successfully
- Microsoft authentication succeeds
- Office 365 JSON events appear in `alerts.json`
- events are indexed into `wazuh-alerts-*`
- events are visible in the Wazuh dashboard

## Detection and investigation examples

The lab currently supports investigations involving:

- Microsoft 365 / Entra activity
- SharePoint and OneDrive audit events
- Windows event logs
- Linux host telemetry
- endpoint configuration findings
- vulnerability detections
- file integrity monitoring
- Wazuh rule groups and threat hunting
- uptime and service availability

Useful Wazuh dashboard areas include:

- Threat Hunting
- Vulnerability Detection
- File Integrity Monitoring
- Configuration Assessment
- MITRE ATT&CK
- Cloud Security / Office 365

## Repository structure

```text
.
├── README.md
├── docs/
│   ├── architecture.md
│   ├── microsoft-365-wazuh.md
│   ├── networking.md
│   ├── troubleshooting.md
│   └── roadmap.md
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

Any example configuration uses placeholders instead of real credentials.

## Lessons learned

The biggest lesson from this lab has been that building a SOC is not just about installing tools. Most of the work is in understanding data flow, authentication, networking, permissions, persistence, and troubleshooting.

Examples from this project include:

- resolving Wazuh manager/agent version mismatches,
- validating Filebeat-to-indexer communication,
- repairing Microsoft 365 authentication,
- confirming audit ingestion at the JSON and index layers,
- recovering from credential-rotation issues without deleting Wazuh data,
- configuring Ubuntu Server Wi-Fi with Netplan,
- separating cloud-side configuration from SIEM-side configuration.

## Next phase

The next major addition is an Active Directory lab using a dedicated Windows Server system. Planned work includes:

- Windows Server domain controller
- Active Directory Domain Services
- DNS
- organizational units
- users and security groups
- Group Policy
- domain-joined Windows client
- Wazuh agent on the domain controller
- authentication and privilege-change detections
- Kerberos-focused monitoring

See [docs/roadmap.md](docs/roadmap.md) for the planned expansion.
