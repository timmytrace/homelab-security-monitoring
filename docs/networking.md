# Networking Notes

## Purpose

Document the network changes made while bringing the Ubuntu SOC server online over Wi-Fi without disturbing Docker or Wazuh.

## Original state

The Ubuntu server was initially configured only for Ethernet using Netplan.

The wired interface was configured for DHCP, while a USB Wi-Fi adapter was detected by the kernel but remained down because no wireless configuration existed.

## Wi-Fi adapter discovery

Useful commands:

```bash
ip -br addr
ip route
```

The wireless interface appeared as a `wlx...` device, confirming that Ubuntu recognized the USB adapter.

## Netplan configuration

A wireless section was added under the existing `network:` block.

Sanitized example:

```yaml
network:
  version: 2

  ethernets:
    enp1s0:
      dhcp4: true
      dhcp6: true
      match:
        macaddress: XX:XX:XX:XX:XX:XX
      set-name: enp1s0

  wifis:
    wlxYOURADAPTER:
      dhcp4: true
      optional: true
      access-points:
        "YOUR_WIFI_SSID":
          password: "YOUR_WIFI_PASSWORD"
```

## Validation workflow

Before applying Netplan changes:

```bash
sudo netplan generate
```

This catches YAML and indentation errors without applying the configuration.

Then:

```bash
sudo netplan apply
```

Connectivity checks:

```bash
ip -br addr
ip route
ping -c 4 1.1.1.1
ping -c 4 microsoft.com
```

## Troubleshooting encountered

### Unknown key `wifi` / `wifis`

The issue was not the keyword itself. `wifis:` is valid, but it must be correctly indented under `network:`.

### Inconsistent YAML indentation

Netplan is strict about indentation. Spaces were used consistently and `version: 2`, `ethernets:`, and `wifis:` were aligned under `network:`.

### `No access points defined`

The `access-points:` block was initially indented at the wrong level. It must be nested under the wireless interface.

### Missing `wpa_supplicant`

The package was checked with:

```bash
dpkg -l | grep wpasupplicant
```

The package was present and Netplan could use it for WPA authentication.

## Routing behavior

When both Ethernet and Wi-Fi were connected, the Ethernet route had the lower metric and was preferred. Wi-Fi remained available as another route.

For this homelab, no manual route manipulation was required once connectivity was restored.

## Security note

Never publish:

- Wi-Fi passwords
- private SSIDs if you consider them sensitive
- router administration credentials
- public IP information unnecessarily

Screenshots used for portfolio purposes should be reviewed and redacted before upload.
