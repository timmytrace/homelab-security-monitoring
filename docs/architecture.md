# Homelab Architecture

## Overview

This homelab is designed as a small SOC environment where endpoint, infrastructure, and cloud telemetry can be collected into Wazuh for investigation.

## Core host

The main server is a Lenovo ThinkCentre-class desktop running Ubuntu Server.

Current characteristics:

- Intel Core i7-7700
- approximately 8 GB RAM
- approximately 1 TB storage using LVM
- Docker used for most services
- persistent media volume mounted under `/mnt/media`

Because memory is limited, the Ubuntu host is kept focused on Linux services and security monitoring rather than running a Windows Server VM.

## Wazuh deployment

Wazuh is deployed using the official single-node Docker Compose deployment.

Components:

- `wazuh.manager`
- `wazuh.indexer`
- `wazuh.dashboard`

Current version:

```text
4.14.5
```

The Docker Compose project is stored under the Ubuntu user's Git directory and uses persistent Docker volumes for Wazuh data.

## Data flow

```text
Windows endpoint ----+
                     |
Ubuntu endpoint -----+----> Wazuh Manager ----> Filebeat ----> Wazuh Indexer
                     |                                  |
Microsoft 365 -------+                                  v
                                                    Wazuh Dashboard
```

Microsoft 365 events are collected by the Wazuh Office 365 module using Microsoft's Management Activity API.

## Self-hosted services

The same Ubuntu server also hosts:

- Portainer
- Nextcloud
- Jellyfin
- Home Assistant
- Uptime Kuma

These services provide additional infrastructure to monitor and generate realistic administrative and operational events.

## Network model

The server originally used Ethernet and later gained Wi-Fi connectivity through a USB wireless adapter configured using Netplan.

The lab remains on a private home LAN. Administrative interfaces are not intentionally exposed directly to the public Internet.

## Planned Active Directory expansion

The next architecture phase adds a dedicated Windows Server system rather than placing additional virtualization load on the current Ubuntu host.

Planned topology:

```text
                       Home LAN
                          |
          +---------------+----------------+
          |                                |
   Ubuntu SOC Server                Windows Lab Host
          |                                |
      Wazuh SIEM                      Windows Server
          |                           Domain Controller
          |                                |
          +--------- Wazuh agent ----------+
                                           |
                                    Domain-joined client
```

The Windows Server will provide:

- Active Directory Domain Services
- DNS
- Group Policy
- Kerberos authentication
- security event logging

This will allow the lab to generate and investigate realistic identity and privilege-related events.
