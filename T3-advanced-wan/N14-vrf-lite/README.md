# N14: VRF Lite – One Router, Multiple Routing Tables

## What This Proves

I can configure VRF Lite (Virtual Routing and Forwarding) on a router to create multiple isolated routing tables. RED VRF and BLUE VRF operate as completely separate routers. Traffic from RED cannot reach BLUE even though they share the same physical router. This is used in MPLS VPNs and multi-tenant environments.

## Topology

- 2x Cisco 1941 routers (R1, R2)
- 2x PCs (PC1 in RED VRF, PC2 in BLUE VRF)
- Separate physical links for each VRF between R1 and R2
- RED VRF: 10.0.0.0/24 network
- BLUE VRF: 10.1.0.0/24 network

## IP Addressing Plan

| Device | Interface | VRF | IP Address | Subnet Mask | Connected To |
|--------|-----------|-----|------------|-------------|--------------|
| R1 | Gig0/0 | RED | 10.0.0.1 | 255.255.255.0 | R2 Gig0/0 |
| R1 | Gig0/1 | BLUE | 10.1.0.1 | 255.255.255.0 | R2 Gig0/1 |
| R2 | Gig0/0 | RED | 10.0.0.2 | 255.255.255.0 | R1 Gig0/0 |
| R2 | Gig0/1 | BLUE | 10.1.0.2 | 255.255.255.0 | R1 Gig0/1 |
| PC1 | Fa0 | RED | 10.0.0.10 | 255.255.255.0 | R1 Gig0/0 |
| PC2 | Fa0 | BLUE | 10.1.0.10 | 255.255.255.0 | R1 Gig0/1 |

## VRF Configuration Summary

| VRF Name | RD | Route-Target Export | Route-Target Import | Interfaces on R1 | Interfaces on R2 |
|----------|----|--------------------|--------------------|------------------|------------------|
| RED | 65001:1 | 65001:1 | 65001:1 | Gig0/0 | Gig0/0 |
| BLUE | 65001:2 | 65001:2 | 65001:2 | Gig0/1 | Gig0/1 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - VRF RED and VRF BLUE on R1
- [R2-config.txt](R2-config.txt) - VRF RED and VRF BLUE on R2

## Step-by-Step Configuration

### 1. Build Topology

1. Open Packet Tracer → File → New
2. Routers → drag 2x 1941 routers into workspace
3. End Devices → drag 2x PCs into workspace
4. Rename devices: R1, R2, PC1, PC2
5. Click lightning bolt → solid black line (Copper Straight-Through)
6. Connect:
   - R1 Gig0/0 → R2 Gig0/0
   - R1 Gig0/1 → R2 Gig0/1
   - PC1 FastEthernet0 → R1 Gig0/0
   - PC2 FastEthernet0 → R1 Gig0/1
7. File → Save As → N14-vrf-lite.pkt

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

### 3. Configure VRFs on R2

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

### 4. Configure Interfaces on R1

    configure terminal
    interface gigabitEthernet 0/0
    ip vrf forwarding RED
    ip address 10.0.0.1 255.255.255.0
    no shutdown
    exit
    interface gigabitEthernet 0/1
    ip vrf forwarding BLUE
    ip address 10.1.0.1 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 5. Configure Interfaces on R2

    configure terminal
    interface gigabitEthernet 0/0
    ip vrf forwarding RED
    ip address 10.0.0.2 255.255.255.0
    no shutdown
    exit
    interface gigabitEthernet 0/1
    ip vrf forwarding BLUE
    ip address 10.1.0.2 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 6. Configure PCs

PC1:
- IP: 10.0.0.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 10.0.0.1

PC2:
- IP: 10.1.0.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 10.1.0.1

### 7. Verify VRF Routing Tables on R1

    show ip route vrf RED
    show ip route vrf BLUE

### 8. Verify Cross-VRF Isolation

On PC1:
    ping 10.1.0.10

Expected: No response (VRFs are isolated)

On PC2:
    ping 10.0.0.10

Expected: No response (VRFs are isolated)

## Verification Commands

    show ip route vrf RED
    show ip route vrf BLUE
    show vrf
    ping vrf RED 10.0.0.2
    ping vrf BLUE 10.1.0.2

## Expected Results (Placeholder)

| Test | Command | Expected Result |
|------|---------|-----------------|
| VRF RED routing table | `show ip route vrf RED` | Shows 10.0.0.0/24 directly connected |
| VRF BLUE routing table | `show ip route vrf BLUE` | Shows 10.1.0.0/24 directly connected |
| VRF isolation | `ping 10.1.0.10` from PC1 | No response (0/5) |
| VRF ping RED | `ping vrf RED 10.0.0.2` on R1 | Successful |
| VRF ping BLUE | `ping vrf BLUE 10.1.0.2` on R1 | Successful |

## Files in This Folder

| File | Purpose |
|------|---------|
| `N14-vrf-lite.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config (VRF RED + BLUE) |
| `R2-config.txt` | R2 running config (VRF RED + BLUE) |
| `screenshots/vrf-red-route.png` | `show ip route vrf RED` |
| `screenshots/vrf-blue-route.png` | `show ip route vrf BLUE` |
| `screenshots/vrf-isolation-ping.png` | Ping from PC1 to PC2 (failing) |

## Time to Complete (Estimated)

20 minutes
