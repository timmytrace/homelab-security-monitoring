# Lab 01 — Basic LAN, ARP, MAC Learning and ICMP

## Project summary

This lab builds a simple Ethernet LAN in Cisco Packet Tracer and verifies communication between two endpoints through a Cisco 2960 switch.

The exercise is intentionally small so that the behavior of ARP, Ethernet switching, MAC address learning, ICMP, and Layer 2 forwarding can be observed directly before adding routers, VLANs, or security controls.

## Objectives

- Build a two-host switched LAN.
- Configure static IPv4 addresses.
- Verify endpoint-to-endpoint communication with ICMP.
- Observe ARP resolution.
- Inspect the switch MAC address table.
- Explain the difference between an ARP cache and a switch MAC table.
- Establish a baseline for later VLAN, routing, and security labs.

## Topology

```text
PC0                                    PC1
192.168.10.10                          192.168.10.20
MAC 0001.c9e0.9a51                    MAC 000d.ff94.b04d
      |                                      |
      | Fa0                                 | Fa0
      |                                      |
      +---------- Cisco 2960 Switch ----------+
                   Fa0/1       Fa0/2
```

## Devices

- 1 × Cisco 2960-24TT switch
- 2 × Packet Tracer PCs
- 2 × Copper Straight-Through cables

## Addressing plan

| Device | Interface | IPv4 Address | Subnet Mask | Default Gateway |
|---|---|---:|---:|---|
| PC0 | FastEthernet0 | 192.168.10.10 | 255.255.255.0 | Not required |
| PC1 | FastEthernet0 | 192.168.10.20 | 255.255.255.0 | Not required |

Both hosts are in the same subnet:

```text
192.168.10.0/24
```

A default gateway is not required because neither host needs to leave the local subnet.

## Physical connectivity

The PCs were connected to the switch using Copper Straight-Through cables:

```text
PC0 FastEthernet0 -> Switch0 FastEthernet0/1
PC1 FastEthernet0 -> Switch0 FastEthernet0/2
```

The green link indicators confirmed that both physical links were operational.

## Connectivity test

From PC0:

```text
ping 192.168.10.20
```

Result:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

All four ICMP Echo Requests received replies, confirming successful Layer 3 communication across the Layer 2 switch.

## ARP investigation

After the ping, PC0's ARP cache was inspected:

```text
arp -a
```

Observed mapping:

```text
192.168.10.20 -> 000d.ff94.b04d
```

This demonstrates that PC0 resolved PC1's IPv4 address to its Ethernet MAC address before delivering the ICMP traffic.

### ARP sequence

```text
PC0 does not know PC1's MAC
        |
        v
ARP Request:
"Who has 192.168.10.20?"
        |
        v
Switch floods the broadcast
        |
        v
PC1 receives request
        |
        v
ARP Reply:
"192.168.10.20 is at 000d.ff94.b04d"
        |
        v
PC0 stores the mapping
```

## Switch MAC address learning

The switch was inspected with:

```text
enable
show mac address-table
```

Observed dynamic entries:

| VLAN | MAC Address | Type | Port |
|---|---|---|---|
| 1 | 0001.c9e0.9a51 | DYNAMIC | Fa0/1 |
| 1 | 000d.ff94.b04d | DYNAMIC | Fa0/2 |

The switch learned each device's MAC address from the source MAC of frames arriving on the corresponding interface.

## ARP cache vs MAC address table

These two tables solve different problems.

### Endpoint ARP cache

```text
IP address -> MAC address
```

Example:

```text
192.168.10.20 -> 000d.ff94.b04d
```

The endpoint uses this mapping to build an Ethernet frame for a local IPv4 destination.

### Switch MAC address table

```text
MAC address -> switch port
```

Example:

```text
0001.c9e0.9a51 -> Fa0/1
000d.ff94.b04d -> Fa0/2
```

The switch uses this table to decide which interface should receive a unicast Ethernet frame.

## Packet flow

The successful ping can be summarized as:

```text
1. PC0 determines that 192.168.10.20 is in the local /24 network.
2. PC0 resolves PC1's MAC address with ARP.
3. The switch learns source MAC addresses from incoming frames.
4. PC0 sends an ICMP Echo Request in an Ethernet frame.
5. The switch forwards the frame to Fa0/2.
6. PC1 processes the ICMP Echo Request.
7. PC1 sends an ICMP Echo Reply.
8. The switch forwards the reply to Fa0/1.
9. PC0 receives the reply.
```

## Packet-level analysis in Simulation Mode

Packet Tracer Simulation Mode was used to inspect the full communication sequence rather than relying only on successful ping output.

### ARP Request

PC0 broadcast an ARP request to discover the MAC address associated with `192.168.10.20`.

Observed fields:

```text
Ethernet destination: FFFF.FFFF.FFFF
EtherType:            0x0806
ARP opcode:           0x0001
Source MAC:           0001.C9E0.9A51
Source IP:            192.168.10.10
Target MAC:           0000.0000.0000
Target IP:            192.168.10.20
```

The broadcast destination demonstrates that PC0 did not yet know PC1's Layer 2 address.

### ARP Reply

PC1 replied directly to PC0 with its MAC address.

```text
ARP opcode:           0x0002
Source MAC:           000D.FF94.B04D
Source IP:            192.168.10.20
Target MAC:           0001.C9E0.9A51
Target IP:            192.168.10.10
```

Unlike the request, this frame was unicast because PC1 already knew PC0's MAC address.

### ICMP Echo Request

After ARP resolution, PC0 sent the actual ping.

```text
Ethernet source:      0001.C9E0.9A51
Ethernet destination: 000D.FF94.B04D
EtherType:            0x0800

IPv4 source:          192.168.10.10
IPv4 destination:     192.168.10.20
Protocol:             0x01 (ICMP)
TTL:                  128

ICMP Type:            8
ICMP Code:            0
```

ICMP Type 8 identifies an Echo Request.

### ICMP Echo Reply

PC1 returned the successful Echo Reply through the Layer 2 switch.

```text
Ethernet source:      000D.FF94.B04D
Ethernet destination: 0001.C9E0.9A51
EtherType:            0x0800

IPv4 source:          192.168.10.20
IPv4 destination:     192.168.10.10
Protocol:             0x01 (ICMP)
TTL:                  128

ICMP Type:            0
ICMP Code:            0
```

ICMP Type 0 identifies an Echo Reply. The TTL remained unchanged while passing through the Cisco 2960 because the device was switching at Layer 2 rather than routing the packet at Layer 3.

### Encapsulation observed

```text
Ethernet Frame
└── IPv4 Packet
    └── ICMP Message
```

The exercise demonstrated how ARP resolution occurs before a local IPv4 host can construct the unicast Ethernet frame carrying ICMP traffic.

### Full observed sequence

```text
PC0 broadcasts ARP Request
        ↓
Switch floods broadcast
        ↓
PC1 sends ARP Reply
        ↓
PC0 learns PC1 MAC
        ↓
PC0 sends ICMP Echo Request (Type 8)
        ↓
Switch forwards unicast frame
        ↓
PC1 sends ICMP Echo Reply (Type 0)
        ↓
PC0 receives reply
```

### Simulation evidence

The final ICMP Echo Reply PDU screenshot is stored as:

```text
icmp-reply-pdu.png
```

![ICMP Echo Reply PDU analysis](icmp-reply-pdu.png)

## Key learning points

- Devices on the same IPv4 subnet can communicate without a router.
- A switch forwards Ethernet frames using MAC addresses.
- ARP translates a local IPv4 destination into a MAC address.
- Switches automatically learn MAC-to-port mappings.
- ICMP can be used to verify reachability.
- Dynamic MAC table entries can age out after inactivity and be relearned when traffic resumes.
- VLAN 1 is the default VLAN on the switch ports in this initial lab.

## Security relevance

These fundamentals directly support security analysis.

A SOC analyst may encounter:

- ARP spoofing or poisoning
- unexpected MAC address movement
- rogue devices on switch ports
- duplicate IP addresses
- unusual ICMP activity
- VLAN misconfiguration
- lateral movement inside a subnet

Understanding normal ARP, MAC learning, and ICMP behavior is necessary before identifying abnormal network activity.

## Evidence

The lab captured three key pieces of evidence.

### 1. Topology

Two PCs connected to a Cisco 2960 switch using Copper Straight-Through cables.

![Packet Tracer Lab 01 topology](network-topology.jpg)

### 2. Connectivity and ARP

PC0 successfully pinged PC1 with 0% packet loss, and the ARP cache mapped `192.168.10.20` to PC1's MAC address.

![PC0 ping and ARP output](pc0-arp-ping.jpg)

### 3. Switch MAC address table

The Cisco 2960 dynamically learned PC0 on Fa0/1 and PC1 on Fa0/2.

![Switch MAC address table](switch-mac-table.jpg)

## Next exercise

The next step is to use Packet Tracer **Simulation Mode** to observe ARP and ICMP packets frame-by-frame, including the OSI-layer information shown by Packet Tracer.

After that, Lab 02 will introduce a router and a second IPv4 network so that default gateways and routing can be demonstrated.
