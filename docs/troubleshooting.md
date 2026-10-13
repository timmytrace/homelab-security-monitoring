# Troubleshooting Log

This document records important issues encountered while building and operating the homelab.

## Wazuh version mismatch

### Symptom

A Windows Wazuh agent could not register against an older manager and reported that the agent version must be lower than or equal to the manager version.

### Resolution

The Wazuh Docker deployment was upgraded to 4.14.5 so the manager matched the newer agent version.

### Lesson

Always confirm the running manager version before onboarding agents. Agent/manager compatibility matters.

---

## Wazuh Docker stack configuration

The lab uses the official single-node Wazuh Docker deployment.

Useful commands:

```bash
cd ~/git/single-node
docker compose ps
docker compose up -d
docker compose down
```

Never use `docker compose down -v` during routine troubleshooting because that can remove persistent volumes.

---

## Microsoft 365 invalid client secret

### Symptom

Wazuh Office 365 logs reported an invalid client secret.

### Investigation

The app registration and permissions were correct, the network path was working, and the Wazuh Office 365 module loaded successfully. The failure was isolated to authentication.

### Resolution

A fresh Entra client secret value was created and carefully placed in the protected secret file mounted into the Wazuh manager.

### Lesson

The **Secret Value** is required, not the Secret ID. Manual transcription of special characters is error-prone.

---

## Office 365 events visible on Ubuntu but not dashboard

### Symptom

Microsoft 365 JSON events were visible in Wazuh alert files, but the dashboard appeared empty.

### Validation path

1. Confirm Office 365 module starts.
2. Confirm Office 365 events exist in `alerts.json`.
3. Test Filebeat output.
4. Confirm daily `wazuh-alerts-*` index exists.
5. Adjust dashboard time range and filters.

Useful commands:

```bash
docker exec single-node-wazuh.manager-1 sh -c 'grep -i office365 /var/ossec/logs/alerts/alerts.json | tail -n 10'
```

```bash
docker exec single-node-wazuh.manager-1 filebeat test output
```

The pipeline was healthy; the final issue was dashboard visibility/filtering.

### Lesson

Troubleshoot SIEM pipelines layer by layer instead of assuming ingestion is broken.

---

## Credential rotation caused authentication failures

### Symptom

After attempting to rotate the Wazuh indexer/admin password, the indexer logs showed repeated authentication failures for `admin`.

### Cause

The Compose configuration and local security files were changed, but the updated security configuration was not successfully applied to the running indexer. Components therefore attempted to authenticate with a password that the indexer had not accepted yet.

### Recovery

Backups created before the change were restored:

```bash
cp docker-compose.yml.before-password-rotation docker-compose.yml
cp config/wazuh_indexer/internal_users.yml.before-password-rotation config/wazuh_indexer/internal_users.yml
```

Then the stack was recreated normally:

```bash
docker compose down
docker compose up -d
```

The previous working password resumed functioning and Microsoft 365 ingestion was revalidated.

### Lesson

Before credential rotation:

- create backups,
- understand which components share the credential,
- validate indexer health before running security update scripts,
- do not assume changing `internal_users.yml` alone changes the live security database.

---

## Ubuntu server offline / no route

### Symptom

Commands such as:

```bash
ping -c 4 1.1.1.1
```

returned `Network is unreachable`.

### Investigation

`ip -br addr` showed Ethernet and Wi-Fi interfaces down, and `ip route` showed only Docker bridge routes with no default route.

### Resolution

The USB Wi-Fi adapter was configured with Netplan and the server obtained a LAN address and default route.

### Lesson

Check interface state and routing before assuming DNS or package-manager problems.

---

## Netplan YAML errors

Errors encountered included:

```text
unknown key
inconsistent indentation
No access points defined
```

The fix was to use spaces consistently and ensure the nesting hierarchy was correct.

A safe workflow is:

```bash
sudo netplan generate
```

before:

```bash
sudo netplan apply
```

---

## Operational rule

When troubleshooting this lab, follow this order:

1. network connectivity,
2. container health,
3. application/module logs,
4. authentication,
5. event generation,
6. alert file,
7. indexer,
8. dashboard filters.

This sequence prevents unnecessary configuration changes and makes failures easier to isolate.
