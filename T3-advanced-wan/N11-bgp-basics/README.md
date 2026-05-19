# N11: BGP Basics – eBGP Between Two Autonomous Systems

## What This Proves

I can configure eBGP (External Border Gateway Protocol) between two different Autonomous Systems. R1 (AS 65001) and R2 (AS 65002) exchange routing information using eBGP. Each router advertises its local LAN to the other. PCs in different ASes can ping each other using BGP-learned routes.

**Note:** Packet Tracer does not support iBGP (internal BGP) in some versions. This lab uses eBGP only. For iBGP, use GNS3, EVE-NG, or CML.

## Topology

- 2x Cisco 1941 routers (R1, R2)
- 2x PCs (PC1 on R1 LAN, PC2 on R2 LAN)
- R1 (AS 65001) ↔ R2 (AS 65002) - eBGP (10.0.12.0/30)
- Loopback interfaces on each router (router-id and testing)

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | AS | Connected To |
|--------|-----------|------------|-------------|----|--------------|
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | 65001 | - |
| R1 | Gig0/0 | 10.0.12.1 | 255.255.255.252 | 65001 | R2 Gig0/0 |
| R1 | Gig0/1 | 192.168.1.1 | 255.255.255.0 | 65001 | PC1 |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | 65002 | - |
| R2 | Gig0/0 | 10.0.12.2 | 255.255.255.252 | 65002 | R1 Gig0/0 |
| R2 | Gig0/1 | 192.168.2.1 | 255.255.255.0 | 65002 | PC2 |
| PC1 | Fa0 | 192.168.1.10 | 255.255.255.0 | - | R1 Gig0/1 |
| PC2 | Fa0 | 192.168.2.10 | 255.255.255.0 | - | R2 Gig0/1 |

## BGP Configuration Summary

| Router | AS | Neighbor | Neighbor AS | Type | Networks Advertised |
|--------|-----|----------|-------------|------|---------------------|
| R1 | 65001 | 10.0.12.2 | 65002 | eBGP | 192.168.1.0/24, 1.1.1.1/32 |
| R2 | 65002 | 10.0.12.1 | 65001 | eBGP | 192.168.2.0/24, 2.2.2.2/32 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - eBGP to R2, advertises LAN
- [R2-config.txt](R2-config.txt) - eBGP to R1, advertises LAN

## Step-by-Step Configuration

### 1. Configure Basic IP on R1

    enable
    configure terminal
    interface loopback 0
    ip address 1.1.1.1 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 10.0.12.1 255.255.255.252
    no shutdown
    exit
    interface gigabitEthernet 0/1
    ip address 192.168.1.1 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 2. Configure Basic IP on R2

    enable
    configure terminal
    interface loopback 0
    ip address 2.2.2.2 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 10.0.12.2 255.255.255.252
    no shutdown
    exit
    interface gigabitEthernet 0/1
    ip address 192.168.2.1 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 3. Configure PCs

PC1:
- IP: 192.168.1.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 192.168.1.1

PC2:
- IP: 192.168.2.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 192.168.2.1

### 4. Configure eBGP on R1 (AS 65001)

    configure terminal
    router bgp 65001
    bgp router-id 1.1.1.1
    neighbor 10.0.12.2 remote-as 65002
    network 192.168.1.0 mask 255.255.255.0
    network 1.1.1.1 mask 255.255.255.255
    end
    write memory

### 5. Configure eBGP on R2 (AS 65002)

    configure terminal
    router bgp 65002
    bgp router-id 2.2.2.2
    neighbor 10.0.12.1 remote-as 65001
    network 192.168.2.0 mask 255.255.255.0
    network 2.2.2.2 mask 255.255.255.255
    end
    write memory

## Verification Commands

    show ip bgp summary
    show ip bgp
    show ip route bgp

## Verification Results (All Passed)

| Test | Command | Expected Result |
|------|---------|-----------------|
| BGP neighbors on R1 | `show ip bgp summary` | ✅ 10.0.12.2 in Established state |
| BGP neighbors on R2 | `show ip bgp summary` | ✅ 10.0.12.1 in Established state |
| BGP routes on R1 | `show ip bgp` | ✅ 192.168.2.0/24 from AS 65002 |
| BGP routes on R2 | `show ip bgp` | ✅ 192.168.1.0/24 from AS 65001 |
| Routing table on R1 | `show ip route bgp` | ✅ B 192.168.2.0/24 |
| Routing table on R2 | `show ip route bgp` | ✅ B 192.168.1.0/24 |
| Ping across AS | `ping 192.168.2.10` from PC1 | ✅ 4 replies, 0% loss |

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| Packet Tracer does not support iBGP | `% Cisco Packet Tracer does not support internal BGP` error | Removed R3 and used eBGP only |
| BGP neighbor not establishing | `show ip bgp summary` shows Idle/Active | Verified direct connectivity with ping before BGP config |
| Missing `no shutdown` on interfaces | Interfaces administratively down | Added `no shutdown` to all physical interfaces |
| Networks not advertised | `show ip bgp` shows no entries | Added `network` statements with correct subnet masks |
| BGP router-id not set | BGP picked wrong router-id | Added `bgp router-id` command |
| PC ping failed | No route to remote network | Verified BGP routes in routing table with `show ip route bgp` |

## What I'd Do Differently Next Time

- Use GNS3, EVE-NG, or CML for full BGP support (including iBGP)
- Use `show ip bgp neighbors` for more detailed neighbor information
- Add `next-hop-self` if using multi-hop eBGP (not needed for directly connected neighbors)
- Use private AS numbers (64512-65535) for lab environments

## Key Commands Used

### eBGP Configuration
- `router bgp 65001`
- `bgp router-id 1.1.1.1`
- `neighbor 10.0.12.2 remote-as 65002`
- `network 192.168.1.0 mask 255.255.255.0`

### Verification
- `show ip bgp summary`
- `show ip bgp`
- `show ip route bgp`
- `show ip bgp neighbors`

## What I Learned

- eBGP (External BGP) is used between different Autonomous Systems. iBGP (Internal BGP) is used within the same AS.
- eBGP neighbors are typically directly connected. The TTL of eBGP packets is set to 1 by default.
- BGP uses TCP port 179. Neighbors must have IP reachability before BGP can establish.
- `bgp router-id` must be unique within the AS. Loopback addresses are commonly used.
- `network` statements in BGP advertise exact prefixes. The subnet mask must match exactly.
- BGP routes are not automatically installed in the routing table unless they are the best path.
- `show ip bgp summary` shows neighbor state. `Established` means the BGP session is up.
- Packet Tracer has limitations. iBGP is not supported in many versions. Use other simulators for full BGP labs.

## Screenshots
- [Topology](screenshots/topology.png)
- [BGP neighbors on R1](screenshots/bgp-summary-r1.png)
- [BGP neighbors on R2](screenshots/bgp-summary-r2.png)
- [BGP routes on R1](screenshots/bgp-routes-r1.png)
- [Ping from PC1 to PC2](screenshots/ping-pc1-to-pc2.png)

## Time to Complete

25 minutes (including troubleshooting Packet Tracer iBGP limitation)
