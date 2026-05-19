# N09: Route Redistribution – Making OSPF and EIGRP Talk To Each Other

## What This Proves

I can configure route redistribution between OSPF and EIGRP on a boundary router (R2). OSPF routes are injected into EIGRP, and EIGRP routes are injected into OSPF. Routers in different domains learn about each other's networks automatically. After redistribution, R1 can ping R3 and R4 loopbacks across routing protocol boundaries.

## Topology

- 4x Cisco 1941 routers (R1, R2, R3, R4)
- 1x Cisco 2960 switch (Switch0)
- OSPF Domain (Area 0): R1 and R2 (direct connection)
- EIGRP Domain (AS 100): R2, R3, R4 (connected through Switch0)
- R2 is the redistribution point (runs both OSPF and EIGRP)
- Loopback interfaces on each router (simulated networks)

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | Protocol | Area/AS | Connected To |
|--------|-----------|------------|-------------|----------|---------|--------------|
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | OSPF | Area 0 | - |
| R1 | Gig0/0 | 10.0.12.1 | 255.255.255.252 | OSPF | Area 0 | R2 Gig0/0 |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | Both | - | - |
| R2 | Gig0/0 | 10.0.12.2 | 255.255.255.252 | OSPF | Area 0 | R1 Gig0/0 |
| R2 | Gig0/1 | 10.0.23.1 | 255.255.255.0 | EIGRP | AS 100 | Switch0 |
| R3 | Loopback0 | 3.3.3.3 | 255.255.255.255 | EIGRP | AS 100 | - |
| R3 | Gig0/0 | 10.0.23.2 | 255.255.255.0 | EIGRP | AS 100 | Switch0 |
| R4 | Loopback0 | 4.4.4.4 | 255.255.255.255 | EIGRP | AS 100 | - |
| R4 | Gig0/0 | 10.0.23.3 | 255.255.255.0 | EIGRP | AS 100 | Switch0 |

## Redistribution Configuration on R2

| Direction | Source | Destination | Metric | Type |
|-----------|--------|-------------|--------|------|
| OSPF → EIGRP | OSPF 1 | EIGRP 100 | 10000 100 255 1 1500 | - |
| EIGRP → OSPF | EIGRP 100 | OSPF 1 | 20 | E2 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - OSPF Area 0
- [R2-config.txt](R2-config.txt) - OSPF + EIGRP + redistribution
- [R3-config.txt](R3-config.txt) - EIGRP AS 100
- [R4-config.txt](R4-config.txt) - EIGRP AS 100
- [switch-config.txt](switch-config.txt) - VLAN 10 access ports

## Step-by-Step Configuration

### 1. Configure Switch0 VLANs

    enable
    configure terminal
    vlan 10
    name EIGRP_VLAN
    exit
    interface gigabitEthernet 0/1
    switchport mode access
    switchport access vlan 10
    no shutdown
    exit
    interface fastEthernet 0/1
    switchport mode access
    switchport access vlan 10
    no shutdown
    exit
    interface fastEthernet 0/2
    switchport mode access
    switchport access vlan 10
    no shutdown
    exit
    end
    write memory

### 2. Configure Basic IP on R1

    enable
    configure terminal
    interface loopback 0
    ip address 1.1.1.1 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 10.0.12.1 255.255.255.252
    no shutdown
    exit
    end
    write memory

### 3. Configure Basic IP on R2

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
    ip address 10.0.23.1 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 4. Configure Basic IP on R3

    enable
    configure terminal
    interface loopback 0
    ip address 3.3.3.3 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 10.0.23.2 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 5. Configure Basic IP on R4

    enable
    configure terminal
    interface loopback 0
    ip address 4.4.4.4 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 10.0.23.3 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 6. Configure OSPF on R1

    configure terminal
    router ospf 1
    router-id 1.1.1.1
    passive-interface loopback 0
    network 10.0.12.0 0.0.0.3 area 0
    network 1.1.1.1 0.0.0.0 area 0
    end
    write memory

### 7. Configure OSPF on R2

    configure terminal
    router ospf 1
    router-id 2.2.2.2
    passive-interface loopback 0
    network 10.0.12.0 0.0.0.3 area 0
    network 2.2.2.2 0.0.0.0 area 0
    end
    write memory

### 8. Configure EIGRP on R2

    configure terminal
    router eigrp 100
    network 10.0.23.0 0.0.0.255
    network 2.2.2.2 0.0.0.0
    end
    write memory

### 9. Configure EIGRP on R3

    configure terminal
    router eigrp 100
    network 10.0.23.0 0.0.0.255
    network 3.3.3.3 0.0.0.0
    end
    write memory

### 10. Configure EIGRP on R4

    configure terminal
    router eigrp 100
    network 10.0.23.0 0.0.0.255
    network 4.4.4.4 0.0.0.0
    end
    write memory

### 11. Configure Redistribution on R2 (OSPF → EIGRP)

    configure terminal
    router eigrp 100
    redistribute ospf 1 metric 10000 100 255 1 1500
    end
    write memory

### 12. Configure Redistribution on R2 (EIGRP → OSPF)

    configure terminal
    router ospf 1
    redistribute eigrp 100 metric 20
    end
    write memory

## Verification Commands

    show ip route ospf
    show ip route eigrp
    show ip ospf database external
    show ip eigrp topology

## Verification Results (All Passed)

| Test | Command | Result |
|------|---------|--------|
| OSPF routes on R1 | `show ip route` | ✅ O E2 3.3.3.3, O E2 4.4.4.4, O E2 10.0.23.0 |
| EIGRP routes on R3 | `show ip route eigrp` | ✅ D EX 1.1.1.1 |
| OSPF database on R2 | `show ip ospf database external` | ✅ Type 5 LSAs for 3.3.3.3, 4.4.4.4, 10.0.23.0 |
| Cross-domain ping | `ping 3.3.3.3` from R1 | ✅ 5 replies, 0% loss |

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| 1941 router has no Gig0/2 | Step 3 failed with invalid interface | Used switch for EIGRP domain instead of direct links |
| Switch0 has no Gig0/3 | Interface not found | Used FastEthernet0/1 and FastEthernet0/2 instead |
| R4 Gig0/0 had no IP | `show ip interface brief` showed unassigned | Added `ip address 10.0.23.3 255.255.255.0` |
| R1 and R2 could not ping each other | Direct link down | Used Copper Straight-Through (auto-MDIX works on Gig ports) |
| R3 not advertising loopback | EIGRP missing network statement | Added `network 3.3.3.3 0.0.0.0` on R3 |
| EIGRP neighbor flapping | Hello packets unstable | Ignored - redistribution still worked |
| OSPF redistribution command syntax | `redistribute eigrp 100 subnets metric 20` failed | Used `redistribute eigrp 100 metric 20` (no subnets keyword) |
| R1 could not ping R3 | Link between R1 and R2 not working | Fixed cable type and verified interfaces up/up |

## What I'd Do Differently Next Time

- Use 2811 routers with more Gig ports instead of 1941 to avoid switch complexity
- Document all IPs and VLANs BEFORE touching the CLI
- Verify Layer 2 connectivity with pings before configuring routing protocols
- Use `redistribute eigrp 100 metric 20` without `subnets` keyword in Packet Tracer
- Set EIGRP hello/hold timers to reduce flapping messages

## Key Commands Used

### OSPF Configuration
- `router ospf 1`
- `router-id 1.1.1.1`
- `passive-interface loopback 0`
- `network 10.0.12.0 0.0.0.3 area 0`

### EIGRP Configuration
- `router eigrp 100`
- `network 10.0.23.0 0.0.0.255`
- `network 3.3.3.3 0.0.0.0`

### Redistribution
- `redistribute ospf 1 metric 10000 100 255 1 1500` (under EIGRP)
- `redistribute eigrp 100 metric 20` (under OSPF)

### Switch Configuration
- `vlan 10`
- `switchport mode access`
- `switchport access vlan 10`

## What I Learned

- Route redistribution allows OSPF and EIGRP to exchange routes through a boundary router (ASBR/redistribution point)
- EIGRP requires a seed metric when redistributing from another protocol. The metric order is: bandwidth, delay, reliability, load, MTU.
- OSPF assigns a default metric of 20 to redistributed routes (type E2 by default)
- The `subnets` keyword is not required in Packet Tracer for OSPF redistribution (unlike real IOS)
- A 1941 router only has 2 Gig ports. For connecting to 3 EIGRP routers, you need a switch or serial modules
- EIGRP neighbor flapping can occur but does not always break redistribution
- Always test Layer 2 connectivity with direct interface pings before troubleshooting routing
- OSPF external routes appear as `O E2` in the routing table and Type 5 LSAs in the OSPF database

## Screenshots
- [Topology](screenshots/topology.png)
- [OSPF routes on R1 (O E2 routes)](screenshots/ospf-routes-r1.png)
- [EIGRP routes on R3 (D EX 1.1.1.1)](screenshots/eigrp-routes-r3.png)
- [Cross-domain ping success](screenshots/ping-r1-to-r3.png)


## Time to Complete

45 minutes (including troubleshooting cable type and redistribution syntax)
