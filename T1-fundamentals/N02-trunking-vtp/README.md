# N02: 802.1Q Trunking & VTP

## What This Proves

I can connect multiple switches using trunk links and use VTP (VLAN Trunking Protocol) to automatically push VLANs from one server switch to all client switches. No more logging into each switch to create VLANs manually.

## Topology

- 3x Cisco 2960 switches (Switch0 = VTP Server, Switch1 = VTP Client, Switch2 = VTP Client)
- 3x PCs (one connected to each switch)

## My VTP Setup

| Switch | VTP Mode | Domain | Password | Version |
|--------|----------|--------|----------|---------|
| Switch0 | Server | NETLAB | cisco123 | 2 |
| Switch1 | Client | NETLAB | cisco123 | 2 |
| Switch2 | Client | NETLAB | cisco123 | 2 |

## The VLANs I Created

| VLAN ID | Name |
|---------|------|
| 10 | Engineering |
| 20 | Sales |
| 30 | Management |
| 99 | Native (unused) |

I set the native VLAN to 99 instead of leaving it on VLAN 1. This prevents VLAN hopping attacks.

## Configuration Files

- [switch0-config.txt](switch0-config.txt) - VTP Server
- [switch1-config.txt](switch1-config.txt) - VTP Client (VLAN 10 access)
- [switch2-config.txt](switch2-config.txt) - VTP Client (VLAN 20 access)

## Verification Screenshots

| Test | Command | Screenshot |
|------|---------|-------------|
| VTP status on server | `show vtp status` | [screenshot](screenshots/vtp-status-server.png) |
| VTP status on client | `show vtp status` | [screenshot](screenshots/vtp-status-client.png) |
| VLANs on server | `show vlan brief` | [screenshot](screenshots/vlan-brief-server.png) |
| VLANs on client | `show vlan brief` | [screenshot](screenshots/vlan-brief-client.png) |
| Trunk status | `show interfaces trunk` | [screenshot](screenshots/trunk-status.png) |

## What I Learned

- VTP is convenient but risky. A higher revision number on a client can wipe all VLANs.
- Native VLAN should never be VLAN 1. I used VLAN 99.
- Trunk allowed VLANs should be explicit, not left as "all".
- Order matters. Configure VTP domain, password, and mode BEFORE creating VLANs.

## Time to Complete

30 minutes
