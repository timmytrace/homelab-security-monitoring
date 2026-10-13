# Lab 03 — Microsoft 365 Audit Monitoring with Wazuh

## Objective

Integrate Microsoft 365 Business Standard audit activity into Wazuh so cloud identity, Exchange, SharePoint, OneDrive, and general Microsoft 365 events can be investigated alongside endpoint telemetry.

## Environment

- Ubuntu Server
- Docker / Docker Compose
- Wazuh 4.14.5 single-node deployment
- Microsoft 365 Business Standard
- Microsoft Purview Audit
- Microsoft Entra ID
- Office 365 Management Activity API

## Architecture

```text
Microsoft 365
    |
    v
Microsoft Purview Audit
    |
    v
Office 365 Management Activity API
    |
    v
Entra application registration
(ActivityFeed.Read)
    |
    v
Wazuh Office 365 module
    |
    v
Wazuh Manager -> Filebeat -> Wazuh Indexer -> Dashboard
```

## Build

### 1. Verify Microsoft 365 auditing

Unified audit ingestion was verified through Exchange Online PowerShell:

```powershell
Get-AdminAuditLogConfig | Format-List UnifiedAuditLogIngestionEnabled
```

Expected result:

```text
UnifiedAuditLogIngestionEnabled : True
```

### 2. Create an Entra application

A dedicated application registration was created for Wazuh.

Permission configured:

```text
Office 365 Management APIs
Application permission: ActivityFeed.Read
Admin consent: Granted
```

The application used:

- Directory (tenant) ID
- Application (client) ID
- Client secret value

No secret values are stored in this repository.

### 3. Store the client secret outside ossec.conf

The Microsoft client secret was placed in a protected host-side file and mounted read-only into the Wazuh manager container.

Example Docker bind mount:

```yaml
- ./config/wazuh_cluster/m365_client_secret:/var/ossec/etc/m365_client_secret:ro
```

The actual secret file is excluded from source control.

### 4. Configure the Wazuh Office 365 module

Sanitized example:

```xml
<office365>
  <enabled>yes</enabled>
  <interval>1m</interval>
  <curl_max_size>1M</curl_max_size>
  <only_future_events>yes</only_future_events>

  <api_auth>
    <tenant_id>REDACTED</tenant_id>
    <client_id>REDACTED</client_id>
    <client_secret_path>/var/ossec/etc/m365_client_secret</client_secret_path>
    <api_type>commercial</api_type>
  </api_auth>

  <subscriptions>
    <subscription>Audit.AzureActiveDirectory</subscription>
    <subscription>Audit.Exchange</subscription>
    <subscription>Audit.SharePoint</subscription>
    <subscription>Audit.General</subscription>
  </subscriptions>
</office365>
```

### 5. Validate the Wazuh module

```bash
docker exec single-node-wazuh.manager-1 /var/ossec/bin/wazuh-modulesd -t
```

The configuration test completed without an error.

## Verification

### Confirm the secret mount

```bash
docker exec single-node-wazuh.manager-1 ls -l /var/ossec/etc/m365_client_secret
```

### Confirm Office 365 module activity

```bash
docker exec single-node-wazuh.manager-1 sh -c 'grep -i office365 /var/ossec/logs/ossec.log | tail -n 30'
```

### Confirm alerts are generated

```bash
docker exec single-node-wazuh.manager-1 sh -c 'grep -i office365 /var/ossec/logs/alerts/alerts.json | tail -n 10'
```

JSON Office 365 alerts were observed in `alerts.json`.

### Confirm indexer connectivity

```bash
docker exec single-node-wazuh.manager-1 filebeat test output
```

The Filebeat-to-indexer connection completed successfully.

### Dashboard validation

Office 365 activity became visible in the Wazuh dashboard after adjusting the time range and removing an overly restrictive `rule.level` filter.

Useful filter:

```text
rule.groups:office365
```

## Break / Fix Exercise

During setup, the integration produced an `Invalid client secret` authentication error.

### Symptoms

- Wazuh Office 365 module loaded.
- Microsoft authentication failed.
- No new Microsoft 365 audit events appeared.

### Root cause

The client secret supplied to Wazuh did not exactly match the Entra client secret value.

### Fix

A new client secret was created in Entra, entered into the protected secret file, and the Wazuh manager container was recreated.

### Verification

Wazuh subsequently reported a successful Microsoft 365 tenant connection, and Office 365 JSON events appeared in `alerts.json`.

## SOC Lessons Learned

- A SIEM integration must be verified at multiple layers, not only in the dashboard.
- Authentication success does not prove event ingestion.
- Event ingestion does not prove indexing.
- Indexing does not guarantee a dashboard filter will display the data.
- Secret handling should be separated from main configuration files.
- Microsoft cloud audit events may have ingestion latency.
- `only_future_events` affects what appears immediately after enabling the integration.

## Security Notes

Never commit:

- client secret values
- Wazuh passwords
- Wi-Fi passwords
- cluster keys
- API tokens
- private certificates

All credentials in this repository must be replaced with placeholders.
