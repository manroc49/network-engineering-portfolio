# N14: VRF Lite – One Router, Multiple Routing Tables

## What This Proves

I can explain VRF Lite (Virtual Routing and Forwarding) and how it creates multiple isolated routing tables on a single router. RED VRF and BLUE VRF operate as completely separate routers. Traffic from RED cannot reach BLUE even though they share the same physical hardware. This is used in MPLS VPNs and multi-tenant environments.

**Note:** Packet Tracer does not support VRF commands on any router model (tested on 2811, 1941, 1841). The configuration below is the correct Cisco IOS syntax for real hardware or GNS3/EVE-NG. This lab documents my understanding of VRF Lite despite the simulator limitation.

## Topology (Planned)

- 1x Router (2811 or 1941) - VRF capable in real IOS
- 2x PCs (PC1 in RED VRF, PC2 in BLUE VRF)
- RED VRF: 10.0.0.0/24 network on FastEthernet0/0
- BLUE VRF: 10.1.0.0/24 network on FastEthernet0/1

## IP Addressing Plan

| Device | Interface | VRF | IP Address | Subnet Mask | Connected To |
|--------|-----------|-----|------------|-------------|--------------|
| R1 | FastEthernet0/0 | RED | 10.0.0.1 | 255.255.255.0 | PC1 |
| R1 | FastEthernet0/1 | BLUE | 10.1.0.1 | 255.255.255.0 | PC2 |
| PC1 | Fa0 | RED | 10.0.0.10 | 255.255.255.0 | R1 Fa0/0 |
| PC2 | Fa0 | BLUE | 10.1.0.10 | 255.255.255.0 | R1 Fa0/1 |

## VRF Configuration (Correct Syntax for Real IOS)

| VRF Name | RD | Route-Target Export | Route-Target Import | Interfaces |
|----------|----|--------------------|--------------------|------------|
| RED | 65001:1 | 65001:1 | 65001:1 | FastEthernet0/0 |
| BLUE | 65001:2 | 65001:2 | 65001:2 | FastEthernet0/1 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - VRF RED and VRF BLUE commands (syntax only, not executed in PT)

## Step-by-Step Configuration (Real IOS Syntax)

### 1. Build Topology (Real Hardware or GNS3/EVE-NG)

1. Connect PC1 to R1 FastEthernet0/0
2. Connect PC2 to R1 FastEthernet0/1

### 2. Configure VRFs on R1

    enable
    configure terminal
    ip vrf RED
    rd 65001:1
    route-target export 65001:1
    route-target import 65001:1
    exit
    ip vrf BLUE
    rd 65001:2
    route-target export 65001:2
    route-target import 65001:2
    exit
    end
    write memory

### 3. Assign Interfaces to VRFs and Configure IPs

    configure terminal
    interface fastEthernet 0/0
    ip vrf forwarding RED
    ip address 10.0.0.1 255.255.255.0
    no shutdown
    exit
    interface fastEthernet 0/1
    ip vrf forwarding BLUE
    ip address 10.1.0.1 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 4. Configure PCs

PC1:
- IP: 10.0.0.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 10.0.0.1

PC2:
- IP: 10.1.0.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 10.1.0.1

## Verification Commands (Real IOS)

    show vrf
    show ip route vrf RED
    show ip route vrf BLUE

## Expected Results (Real IOS)

| Test | Command | Expected Result |
|------|---------|-----------------|
| VRF creation | `show vrf` | RED and BLUE VRFs listed |
| RED routing table | `show ip route vrf RED` | 10.0.0.0/24 directly connected |
| BLUE routing table | `show ip route vrf BLUE` | 10.1.0.0/24 directly connected |
| Within-VRF ping | `ping 10.0.0.1` from PC1 | Successful |
| Cross-VRF isolation | `ping 10.1.0.10` from PC1 | Fails (no route) |

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| VRF commands not supported in PT | `% Invalid input detected at '^' marker` on `ip vrf RED` | Documented limitation, provided correct syntax for real IOS |
| Tested multiple router models | 2811, 1941, 1841 all fail | Confirmed Packet Tracer does not support VRF |
| No workaround in Packet Tracer | Cannot execute VRF configuration | Use GNS3/EVE-NG for actual VRF labs |

## Packet Tracer Limitation Encountered

| Issue | Impact | Resolution |
|-------|--------|------------|
| VRF commands not supported | Cannot execute VRF configuration | Document syntax and concepts; use GNS3/EVE-NG for actual lab |

## What I'd Do Differently Next Time

- Use GNS3 or EVE-NG with real Cisco IOS images for VRF labs
- Research Packet Tracer limitations before starting advanced labs
- Document the limitation as part of the portfolio (shows awareness of tool constraints)

## Key Commands (Real IOS Syntax)

- `ip vrf RED`
- `rd 65001:1`
- `route-target export 65001:1`
- `route-target import 65001:1`
- `ip vrf forwarding RED`
- `show vrf`
- `show ip route vrf RED`

## What I Learned

- VRF (Virtual Routing and Forwarding) allows a single router to maintain multiple independent routing tables.
- Each VRF has its own route distinguisher (RD) to keep routes unique.
- Route targets control which routes are exported and imported between VRFs.
- VRF Lite is used in multi-tenant environments and MPLS VPNs.
- Packet Tracer does not support VRF commands on any router model.
- For advanced features like VRF, real hardware or GNS3/EVE-NG is required.

## Screenshots

- [VRF command error in Packet Tracer](screenshots/vrf-command-error.png) - Shows `% Invalid input detected` when attempting `ip vrf RED`

## Files in This Folder

| File | Purpose |
|------|---------|
| `N14-vrf-lite.pkt` | Packet Tracer topology (VRF not functional) |
| `R1-config.txt` | R1 config commands (syntax only, not executed) |
| `screenshots/vrf-command-error.png` | Evidence of Packet Tracer limitation |
| `README.md` | This file |

## Time to Complete

15 minutes (including documentation of limitation)
