# Command Reference

A compact reference for recurring homelab administration and troubleshooting.

## Docker / Wazuh

```bash
cd ~/git/single-node
```

```bash
docker compose ps
```

```bash
docker compose up -d
```

```bash
docker compose down
```

Do not use `-v` unless you intentionally want to remove persistent volumes.

### Test Wazuh modules

```bash
docker exec single-node-wazuh.manager-1 /var/ossec/bin/wazuh-modulesd -t
```

### View Office 365 module logs

```bash
docker exec single-node-wazuh.manager-1 sh -c 'grep -i office365 /var/ossec/logs/ossec.log | tail -n 30'
```

### View Office 365 alerts

```bash
docker exec single-node-wazuh.manager-1 sh -c 'grep -i office365 /var/ossec/logs/alerts/alerts.json | tail -n 10'
```

### Test Filebeat output

```bash
docker exec single-node-wazuh.manager-1 filebeat test output
```

### Check indexer logs

```bash
docker logs --tail 60 single-node-wazuh.indexer-1
```

## Networking

```bash
ip -br addr
```

```bash
ip route
```

```bash
ping -c 4 1.1.1.1
```

```bash
ping -c 4 microsoft.com
```

### Netplan validation

```bash
sudo netplan generate
```

```bash
sudo netplan apply
```

### Wi-Fi authentication package

```bash
dpkg -l | grep wpasupplicant
```

## System resources

```bash
free -h
```

```bash
lscpu | grep -E 'Model name|CPU\(s\)'
```

```bash
df -h
```

## Microsoft 365 audit verification

From Exchange Online PowerShell:

```powershell
Get-AdminAuditLogConfig | Format-List UnifiedAuditLogIngestionEnabled
```

Expected result:

```text
UnifiedAuditLogIngestionEnabled : True
```

## Wazuh dashboard search

Useful Office 365 filter:

```text
rule.groups:office365
```

Use a time range such as `Last 24 hours` when validating new events.

## Secret-handling rule

Never run commands that print secret values into screenshots or logs. Prefer checking:

- file existence
- permissions
- file size
- module authentication result

without printing the actual credential.
