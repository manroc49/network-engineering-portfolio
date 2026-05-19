# N11: BGP Basics – eBGP and iBGP in One Lab

## What This Proves

I can configure BGP (Border Gateway Protocol) between two different Autonomous Systems (eBGP) and within the same AS (iBGP). R1 (AS 65001) advertises its LAN to R2 (AS 65002). R2 advertises those routes to R3 via iBGP with next-hop-self. R3 can reach R1's LAN, and R1 can reach R3's LAN. BGP is what makes the internet work.

## Topology

- 3x Cisco 1941 routers (R1, R2, R3)
- 2x PCs (PC1 on R1 LAN, PC3 on R3 LAN)
- R1 (AS 65001) ↔ R2 (AS 65002) - eBGP (10.0.12.0/30)
- R2 (AS 65002) ↔ R3 (AS 65002) - iBGP (10.0.23.0/30)
- Loopback interfaces on each router (router-id and testing)

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | AS | Connected To |
|--------|-----------|------------|-------------|----|--------------|
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | 65001 | - |
| R1 | Gig0/0 | 10.0.12.1 | 255.255.255.252 | 65001 | R2 Gig0/0 |
| R1 | Gig0/1 | 192.168.1.1 | 255.255.255.0 | 65001 | PC1 |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | 65002 | - |
| R2 | Gig0/0 | 10.0.12.2 | 255.255.255.252 | 65002 | R1 Gig0/0 |
| R2 | Gig0/1 | 10.0.23.1 | 255.255.255.252 | 65002 | R3 Gig0/0 |
| R3 | Loopback0 | 3.3.3.3 | 255.255.255.255 | 65002 | - |
| R3 | Gig0/0 | 10.0.23.2 | 255.255.255.252 | 65002 | R2 Gig0/1 |
| R3 | Gig0/1 | 192.168.3.1 | 255.255.255.0 | 65002 | PC3 |
| PC1 | Fa0 | 192.168.1.10 | 255.255.255.0 | - | R1 Gig0/1 |
| PC3 | Fa0 | 192.168.3.10 | 255.255.255.0 | - | R3 Gig0/1 |

## BGP Configuration Summary

| Router | AS | Neighbor | Neighbor AS | Type | Next-Hop Self | Networks Advertised |
|--------|-----|----------|-------------|------|---------------|---------------------|
| R1 | 65001 | 10.0.12.2 | 65002 | eBGP | No | 192.168.1.0/24, 1.1.1.1/32 |
| R2 | 65002 | 10.0.12.1 | 65001 | eBGP | No | - |
| R2 | 65002 | 10.0.23.3 | 65002 | iBGP | Yes | - |
| R3 | 65002 | 10.0.23.1 | 65002 | iBGP | No | 192.168.3.0/24, 3.3.3.3/32 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - eBGP to R2, advertises LAN
- [R2-config.txt](R2-config.txt) - eBGP to R1, iBGP to R3 with next-hop-self
- [R3-config.txt](R3-config.txt) - iBGP to R2, advertises LAN

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
    ip address 10.0.23.1 255.255.255.252
    no shutdown
    exit
    end
    write memory

### 3. Configure Basic IP on R3

    enable
    configure terminal
    interface loopback 0
    ip address 3.3.3.3 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 10.0.23.2 255.255.255.252
    no shutdown
    exit
    interface gigabitEthernet 0/1
    ip address 192.168.3.1 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 4. Configure PCs

PC1:
- IP: 192.168.1.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 192.168.1.1

PC3:
- IP: 192.168.3.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 192.168.3.1

### 5. Configure eBGP on R1 (AS 65001)

    configure terminal
    router bgp 65001
    bgp router-id 1.1.1.1
    neighbor 10.0.12.2 remote-as 65002
    network 192.168.1.0 mask 255.255.255.0
    network 1.1.1.1 mask 255.255.255.255
    end
    write memory

### 6. Configure BGP on R2 (eBGP to R1, iBGP to R3)

    configure terminal
    router bgp 65002
    bgp router-id 2.2.2.2
    neighbor 10.0.12.1 remote-as 65001
    neighbor 10.0.23.3 remote-as 65002
    neighbor 10.0.23.3 next-hop-self
    end
    write memory

### 7. Configure iBGP on R3 (AS 65002)

    configure terminal
    router bgp 65002
    bgp router-id 3.3.3.3
    neighbor 10.0.23.1 remote-as 65002
    network 192.168.3.0 mask 255.255.255.0
    network 3.3.3.3 mask 255.255.255.255
    end
    write memory

## Verification Commands

    show ip bgp summary
    show ip bgp
    show ip route bgp

## Expected Results (Placeholder)

| Test | Command | Expected Result |
|------|---------|-----------------|
| BGP neighbors on R1 | `show ip bgp summary` | 10.0.12.2 in Established state |
| BGP neighbors on R2 | `show ip bgp summary` | 10.0.12.1 and 10.0.23.3 in Established state |
| BGP neighbors on R3 | `show ip bgp summary` | 10.0.23.1 in Established state |
| BGP routes on R1 | `show ip bgp` | 192.168.3.0/24 from AS 65002 |
| BGP routes on R3 | `show ip bgp` | 192.168.1.0/24 from AS 65001 |
| Routing table on R1 | `show ip route bgp` | B 192.168.3.0/24 |
| Routing table on R3 | `show ip route bgp` | B 192.168.1.0/24 |
| Ping across AS | `ping 192.168.3.10` from PC1 | 5 replies, 0% loss |

## Files in This Folder

| File | Purpose |
|------|---------|
| `N11-bgp-basics.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config (eBGP to R2) |
| `R2-config.txt` | R2 running config (eBGP + iBGP with next-hop-self) |
| `R3-config.txt` | R3 running config (iBGP to R2) |
| `screenshots/bgp-summary-r1.png` | `show ip bgp summary` on R1 |
| `screenshots/bgp-summary-r2.png` | `show ip bgp summary` on R2 |
| `screenshots/bgp-routes-r1.png` | `show ip bgp` on R1 |
| `screenshots/ping-pc1-to-pc3.png` | Ping from PC1 to PC3 |

## Time to Complete (Estimated)

30 minutes
