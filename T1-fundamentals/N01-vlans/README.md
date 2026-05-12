# N01: VLAN Segmentation (Router-on-a-Stick)

## What This Proves
I can take a flat /24 network and segment it into three VLANs (10, 20, 30) on a single switch, block inter-VLAN traffic at Layer 2, and restore connectivity between VLANs using a router‑on‑a‑stick.

## Topology
- 1x Cisco 2960 switch
- 1x Cisco 1941 router
- 3x PCs (one per VLAN)

## IP Addressing Table
| Device | VLAN             | Interface | IP Address      | Gateway       |
|--------|------------------|-----------|-----------------|---------------|
| PC1    | 10 (Engineering) | Fa0/1     | 192.168.10.10/24| 192.168.10.1  |
| PC2    | 20 (Sales)       | Fa0/2     | 192.168.20.20/24| 192.168.20.1  |
| PC3    | 30 (Management)  | Fa0/3     | 192.168.30.30/24| 192.168.30.1  |
| R1     | subif .10        | Gi0/0.10  | 192.168.10.1/24 | N/A           |
| R1     | subif .20        | Gi0/0.20  | 192.168.20.1/24 | N/A           |
| R1     | subif .30        | Gi0/0.30  | 192.168.30.1/24 | N/A           |
| SW1    | VLAN 30 (Mgmt)   | VLAN 30   | 192.168.30.2/24 | 192.168.30.1  |

## Configuration Files
- [switch-config.txt](switch-config.txt)
- [router-config.txt](router-config.txt)

## Verification Screenshots
| Test | Command | Result |
|------|---------|--------|
| VLAN assignment | `show vlan brief` | [screenshot](screenshots/vlan-brief.png) |
| Trunk status | `show interfaces trunk` | [screenshot](screenshots/trunk-status.png) |
| Router subinterfaces | `show ip interface brief` | [screenshot](screenshots/router-interfaces.png) |
| Routing table | `show ip route` | [screenshot](screenshots/router-route.png) |
| Ping PC1 → PC2 (inter‑VLAN) | `ping 192.168.20.20` from PC1 | [screenshot](screenshots/ping-results.png) |
| Ping PC1 → PC3 (inter‑VLAN) | `ping 192.168.30.30` from PC1 | [screenshot](screenshots/ping-results.png) |

## What I Learned
- Access ports carry **one** VLAN; trunk ports carry **multiple** VLANs (802.1Q).
- Router‑on‑a‑stick uses logical subinterfaces with `encapsulation dot1Q`.
- The switch needs an SVI and `ip default-gateway` for remote management.
- Without the router, VLANs cannot communicate (L2 isolation).

## Time to Complete
25 minutes (including documentation)
