# Break-Fix — Wazuh Indexer Authentication Failure

## Scenario

A Wazuh `admin` / indexer password rotation was attempted in a Docker-based Wazuh 4.14.5 deployment.

The new password was updated in Docker Compose and a new bcrypt hash was generated for `internal_users.yml`. However, the OpenSearch security configuration was not successfully applied before the manager and dashboard began using the new credential.

## Symptoms

The Wazuh indexer logs showed repeated authentication failures for the `admin` account.

The dashboard and manager could no longer authenticate correctly to the indexer.

A `securityadmin.sh` attempt connected with the admin certificate but failed to retrieve cluster state within the timeout period.

## Investigation

### Check service status

```bash
docker compose ps
```

### Review indexer logs

```bash
docker logs --tail 60 single-node-wazuh.indexer-1
```

Repeated authentication failures indicated that the containers and the indexer security database were using different credentials.

### Key finding

The password rotation created a mismatch:

```text
Docker Compose / manager / dashboard -> NEW password
Indexer security database           -> OLD password
```

Because `securityadmin.sh` never completed successfully, the indexer had not adopted the new hash.

## Root Cause

The credential change was partially applied across the Wazuh stack.

In a multi-component SIEM deployment, changing a shared credential in only some components can interrupt data flow and dashboard access.

## Recovery

Backups had been created before the change:

```text
docker-compose.yml.before-password-rotation
internal_users.yml.before-password-rotation
```

The known-good files were restored:

```bash
cp docker-compose.yml.before-password-rotation docker-compose.yml
cp config/wazuh_indexer/internal_users.yml.before-password-rotation config/wazuh_indexer/internal_users.yml
```

The Compose configuration was validated:

```bash
docker compose config >/dev/null && echo "Compose OK"
```

The stack was recreated without deleting data volumes:

```bash
docker compose down
docker compose up -d
```

> `docker compose down -v` was intentionally avoided because `-v` would remove named volumes and could destroy Wazuh data.

## Verification

### Confirm containers

```bash
docker compose ps
```

### Review indexer logs

```bash
docker logs --tail 20 single-node-wazuh.indexer-1
```

The previous Wazuh dashboard password worked again and the authentication failure flood stopped.

### Confirm Microsoft 365 module survived recovery

```bash
docker exec single-node-wazuh.manager-1 sh -c 'grep -i office365 /var/ossec/logs/ossec.log | tail -n 30'
```

The module reported that it started successfully.

### Confirm cloud alerts

```bash
docker exec single-node-wazuh.manager-1 sh -c 'grep -i office365 /var/ossec/logs/alerts/alerts.json | tail -n 10'
```

Microsoft 365 JSON alerts appeared again, proving that the SIEM data pipeline had recovered.

## Lessons Learned

1. Back up configuration before security changes.
2. Shared credentials must be updated consistently across all dependent components.
3. A running container does not prove the application inside it is healthy.
4. Logs are more useful than repeatedly retrying a failing command.
5. Roll back to a known-good state before attempting a second change.
6. Validate the complete telemetry path after recovery, not only the login page.
7. Never delete SIEM data volumes during troubleshooting unless data loss is intentional.

## Interview Talking Point

> During a Wazuh indexer credential rotation, I created a mismatch between the Docker services and the OpenSearch security database. I identified repeated authentication failures in the indexer logs, restored known-good configuration from backups, validated container and indexer health, and confirmed that Microsoft 365 events were again reaching Wazuh `alerts.json` and the dashboard.

## Security Note

No real passwords, hashes, client secrets, cluster keys, or private certificates are stored in this document.
