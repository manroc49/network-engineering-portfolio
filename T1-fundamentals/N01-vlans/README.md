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

## Verification Results (All Passed)

| Test | Command | Result |
|------|---------|--------|
| VLAN assignment | `show vlan brief` | ✅ Fa0/1=VLAN10, Fa0/2=VLAN20, Fa0/3=VLAN30 |
| Trunk status | `show interfaces trunk` | ✅ Gi0/1 trunk, allowed VLANs 10,20,30 |
| Router subinterfaces | `show ip interface brief` | ✅ Gi0/0.10, .20, .30 all up/up |
| Routing table | `show ip route` | ✅ Directly connected routes for .10,.20,.30 |
| Inter-VLAN ping test | `ping 192.168.20.20` and `ping 192.168.30.30` from PC1 | ✅ Both successful |

## Screenshots
- [VLAN assignment](screenshots/vlan-brief.png)
- [Trunk status](screenshots/trunk-status.png)
- [Router interfaces](screenshots/router-interfaces.png)
- [Router routing table](screenshots/router-route.png)
- [Ping results](screenshots/ping-results.png)
- [Topology](screenshots/topology.png)

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| Switch couldn't reach gateway | Ping to 192.168.30.1 failed | Added `ip default-gateway 192.168.30.1` |
| Router subinterfaces down | `show ip interface brief` showed down/down | Forgot `no shutdown` on physical interface Gi0/0 |
| PC1 couldn't ping PC2 | Request timed out | PC2 had wrong default gateway (set to .10.1 instead of .20.1) |

## What I'd Do Differently Next Time

- Use a **Layer 3 switch** instead of router-on-a-stick for faster inter-VLAN routing
- Add **SSH configuration** on the switch instead of telnet
- Document **VLAN numbering strategy** (10=Engineering, 20=Sales, 30=Management) for scalability

## Key Commands Used

### Switch

- show vlan brief
- show interfaces trunk
- show running-config

### Router
- show ip interface brief
- show ip route
- show running-config

### PC
- ping 192.168.10.1
- ping 192.168.20.20
- ping 192.168.30.30


## What I Learned
- Access ports carry **one** VLAN; trunk ports carry **multiple** VLANs (802.1Q)
- Router‑on‑a‑stick uses logical subinterfaces with `encapsulation dot1Q`
- The switch needs an SVI and `ip default-gateway` for remote management
- Without the router, VLANs cannot communicate (L2 isolation)

## Time to Complete
25 minutes (including documentation)
