# Homelab Roadmap

## Completed

- Ubuntu Server deployed
- Docker installed
- Portainer deployed
- Wazuh single-node Docker deployment running on 4.14.5
- Ubuntu Wazuh agent onboarded
- Windows Wazuh agent onboarded
- Microsoft 365 Business Standard audit integration completed
- Microsoft Purview auditing confirmed enabled
- Entra application registration configured for `ActivityFeed.Read`
- Office 365 events visible in Wazuh alerts and dashboard
- Uptime Kuma deployed
- Nextcloud deployed
- Jellyfin deployed
- Home Assistant deployed
- Ubuntu Wi-Fi connectivity configured with Netplan

## Near-term priorities

### 1. Active Directory lab

Planned components:

- dedicated older desktop or mini PC
- Windows Server
- Active Directory Domain Services
- DNS
- Group Policy
- organizational units
- users and security groups
- domain-joined Windows client
- Wazuh agent on the domain controller

Detection goals:

- failed logons
- account lockouts
- user creation and deletion
- group membership changes
- privileged group modifications
- Kerberos authentication events
- PowerShell activity
- suspicious lateral movement patterns

### 2. Detection engineering

Create and document custom detections for:

- repeated authentication failures
- suspicious administrator group changes
- anomalous PowerShell execution
- unusual Microsoft 365 activity
- suspicious file changes
- service creation
- scheduled task creation

Map detections to MITRE ATT&CK where appropriate.

### 3. Network visibility

Potential additions:

- Suricata or Zeek
- network sensor on mirrored or monitored traffic
- DNS logging
- firewall logs

### 4. Dashboard improvements

Build analyst-friendly views for:

- Microsoft 365 activity
- Windows authentication
- vulnerabilities
- FIM changes
- endpoint health
- high-severity alerts

### 5. Incident-response exercises

Generate controlled events and write mini incident reports covering:

- brute-force simulation
- suspicious account creation
- unauthorized group membership change
- PowerShell abuse simulation
- malicious file placement
- Microsoft 365 suspicious sign-in or file activity

## Longer-term ideas

- Splunk side-by-side with Wazuh
- Microsoft Defender integrations if licensing permits
- cloud infrastructure telemetry
- secure remote access / VPN project
- centralized syslog
- backup and recovery automation
- infrastructure-as-code for repeatable deployments

## Portfolio objective

The final goal is a documented SOC lab that demonstrates:

- infrastructure deployment
- log-source onboarding
- SIEM administration
- cloud audit monitoring
- identity monitoring
- detection engineering
- troubleshooting
- incident investigation
- documentation and operational discipline
