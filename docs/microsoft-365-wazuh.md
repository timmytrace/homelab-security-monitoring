# Microsoft 365 to Wazuh Integration

## Objective

Integrate Microsoft 365 audit activity into Wazuh so cloud identity, SharePoint, OneDrive, Exchange, and general Microsoft 365 events can be investigated alongside endpoint telemetry.

## Licensing and audit readiness

The tenant uses Microsoft 365 Business Standard.

Unified auditing was verified with Exchange Online PowerShell:

```powershell
Get-AdminAuditLogConfig | Format-List UnifiedAuditLogIngestionEnabled
```

Expected result:

```text
UnifiedAuditLogIngestionEnabled : True
```

Purview Audit searches initially returned zero results, so verifying the backend audit state directly was an important troubleshooting step.

## Entra application registration

A dedicated application registration was created for Wazuh.

Configuration:

- single-tenant application
- Office 365 Management APIs
- Application permission: `ActivityFeed.Read`
- admin consent granted tenant-wide

Required values:

- Directory (tenant) ID
- Application (client) ID
- client secret value

The secret is never committed to GitHub.

## Secret handling

Instead of placing the secret directly in `ossec.conf`, a protected file is mounted into the Wazuh manager container and referenced by path.

Example:

```xml
<client_secret_path>/var/ossec/etc/m365_client_secret</client_secret_path>
```

The host-side secret file should be restricted, for example:

```bash
chmod 600 m365_client_secret
```

## Wazuh configuration

Sanitized example:

```xml
<office365>
  <enabled>yes</enabled>
  <interval>1m</interval>
  <curl_max_size>1M</curl_max_size>
  <only_future_events>yes</only_future_events>

  <api_auth>
    <tenant_id>YOUR-TENANT-ID</tenant_id>
    <client_id>YOUR-CLIENT-ID</client_id>
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

## Validation steps

### 1. Validate Wazuh module syntax

```bash
docker exec single-node-wazuh.manager-1 /var/ossec/bin/wazuh-modulesd -t
```

A clean return without a configuration error indicates the module configuration is valid.

### 2. Confirm the secret file exists inside the container

```bash
docker exec single-node-wazuh.manager-1 ls -l /var/ossec/etc/m365_client_secret
```

Check only metadata and file size. Do not print the secret.

### 3. Check module logs

```bash
docker exec single-node-wazuh.manager-1 sh -c 'grep -i office365 /var/ossec/logs/ossec.log | tail -n 30'
```

Expected startup line:

```text
wazuh-modulesd:office365: INFO: Module Office365 started.
```

### 4. Confirm alert generation

```bash
docker exec single-node-wazuh.manager-1 sh -c 'grep -i office365 /var/ossec/logs/alerts/alerts.json | tail -n 10'
```

JSON output here proves Wazuh is generating alerts from Microsoft 365 telemetry.

### 5. Confirm indexing

The expected destination is a daily Wazuh alerts index, for example:

```text
wazuh-alerts-4.x-YYYY.MM.DD
```

## Test activity used

Harmless cloud actions were used to generate audit records, such as:

- opening OneDrive
- creating a small test file
- renaming the file
- deleting the file
- signing in to Microsoft 365 services

Because `only_future_events` is enabled, generating new events after the integration starts is important.

## Dashboard investigation

Useful views:

```text
Cloud Security -> Office 365 -> Events
```

and:

```text
Threat Intelligence -> Threat Hunting
```

Useful query:

```text
rule.groups:office365
```

Also ensure the dashboard time picker includes the event window, such as `Last 24 hours`.

## Troubleshooting notes

### Invalid client secret

A failed authentication attempt produced an invalid-secret error. The root causes to check are:

- copied Secret ID instead of Secret Value
- manually mistyped secret
- extra newline in the secret file
- expired or deleted secret

A fresh client secret value resolved the problem.

### Dashboard blank while Ubuntu shows events

The pipeline was validated layer-by-layer:

1. Office 365 module logs
2. `alerts.json`
3. Filebeat output test
4. Wazuh index existence
5. dashboard filters/time range

This avoided incorrectly changing the Microsoft integration when the issue was only the dashboard view.

### Filebeat test

```bash
docker exec single-node-wazuh.manager-1 filebeat test output
```

Healthy output includes successful DNS, connection, TLS handshake, and server communication.

## Security considerations

Never commit:

- client secret values
- tenant credentials
- Wazuh passwords
- cluster keys
- certificates or private keys

If a secret appears in a screenshot or terminal history, rotate or delete it as appropriate.
