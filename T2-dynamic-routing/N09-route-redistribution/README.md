# N09: Route Redistribution

## What This Proves

I can configure route redistribution between OSPF and EIGRP on a boundary router (R2). OSPF routes are injected into EIGRP, and EIGRP routes are injected into OSPF. Routers in different domains learn about each other's networks automatically.

## Topology

- 4x Cisco 1941 routers (R1, R2, R3, R4)
- OSPF Domain (Area 0): R1 and R2
- EIGRP Domain (AS 100): R2, R3, R4
- R2 is the redistribution point (runs both OSPF and EIGRP)
- Loopback interfaces on each router (simulated networks)

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | Protocol | Area/AS | Connected To |
|--------|-----------|------------|-------------|----------|---------|--------------|
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | OSPF | Area 0 | - |
| R1 | Gig0/0 | 10.0.12.1 | 255.255.255.252 | OSPF | Area 0 | R2 Gig0/0 |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | Both | - | - |
| R2 | Gig0/0 | 10.0.12.2 | 255.255.255.252 | OSPF | Area 0 | R1 Gig0/0 |
| R2 | Gig0/1 | 10.0.23.1 | 255.255.255.252 | EIGRP | AS 100 | R3 Gig0/0 |
| R2 | Gig0/2 | 10.0.24.1 | 255.255.255.252 | EIGRP | AS 100 | R4 Gig0/0 |
| R3 | Loopback0 | 3.3.3.3 | 255.255.255.255 | EIGRP | AS 100 | - |
| R3 | Gig0/0 | 10.0.23.2 | 255.255.255.252 | EIGRP | AS 100 | R2 Gig0/1 |
| R3 | Gig0/1 | 10.0.34.1 | 255.255.255.252 | EIGRP | AS 100 | R4 Gig0/1 |
| R4 | Loopback0 | 4.4.4.4 | 255.255.255.255 | EIGRP | AS 100 | - |
| R4 | Gig0/0 | 10.0.24.2 | 255.255.255.252 | EIGRP | AS 100 | R2 Gig0/2 |
| R4 | Gig0/1 | 10.0.34.2 | 255.255.255.252 | EIGRP | AS 100 | R3 Gig0/1 |

## Router IDs

| Router | OSPF Router ID | EIGRP Router ID |
|--------|----------------|-----------------|
| R1 | 1.1.1.1 | - |
| R2 | 2.2.2.2 | 2.2.2.2 |
| R3 | - | 3.3.3.3 |
| R4 | - | 4.4.4.4 |

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

## Step-by-Step Configuration

### 1. Configure Basic IP on R1

    R1> enable
    R1# configure terminal
    R1(config)# interface loopback 0
    R1(config-if)# ip address 1.1.1.1 255.255.255.255
    R1(config-if)# exit
    R1(config)# interface gigabitEthernet 0/0
    R1(config-if)# ip address 10.0.12.1 255.255.255.252
    R1(config-if)# no shutdown
    R1(config-if)# exit
    R1(config)# end
    R1# write memory

### 2. Configure Basic IP on R2

    R2> enable
    R2# configure terminal
    R2(config)# interface loopback 0
    R2(config-if)# ip address 2.2.2.2 255.255.255.255
    R2(config-if)# exit
    R2(config)# interface gigabitEthernet 0/0
    R2(config-if)# ip address 10.0.12.2 255.255.255.252
    R2(config-if)# no shutdown
    R2(config-if)# exit
    R2(config)# interface gigabitEthernet 0/1
    R2(config-if)# ip address 10.0.23.1 255.255.255.252
    R2(config-if)# no shutdown
    R2(config-if)# exit
    R2(config)# interface gigabitEthernet 0/2
    R2(config-if)# ip address 10.0.24.1 255.255.255.252
    R2(config-if)# no shutdown
    R2(config-if)# exit
    R2(config)# end
    R2# write memory

### 3. Configure Basic IP on R3

    R3> enable
    R3# configure terminal
    R3(config)# interface loopback 0
    R3(config-if)# ip address 3.3.3.3 255.255.255.255
    R3(config-if)# exit
    R3(config)# interface gigabitEthernet 0/0
    R3(config-if)# ip address 10.0.23.2 255.255.255.252
    R3(config-if)# no shutdown
    R3(config-if)# exit
    R3(config)# interface gigabitEthernet 0/1
    R3(config-if)# ip address 10.0.34.1 255.255.255.252
    R3(config-if)# no shutdown
    R3(config-if)# exit
    R3(config)# end
    R3# write memory

### 4. Configure Basic IP on R4

    R4> enable
    R4# configure terminal
    R4(config)# interface loopback 0
    R4(config-if)# ip address 4.4.4.4 255.255.255.255
    R4(config-if)# exit
    R4(config)# interface gigabitEthernet 0/0
    R4(config-if)# ip address 10.0.24.2 255.255.255.252
    R4(config-if)# no shutdown
    R4(config-if)# exit
    R4(config)# interface gigabitEthernet 0/1
    R4(config-if)# ip address 10.0.34.2 255.255.255.252
    R4(config-if)# no shutdown
    R4(config-if)# exit
    R4(config)# end
    R4# write memory

### 5. Configure OSPF on R1 (Area 0)

    R1# configure terminal
    R1(config)# router ospf 1
    R1(config-router)# router-id 1.1.1.1
    R1(config-router)# passive-interface loopback 0
    R1(config-router)# network 10.0.12.0 0.0.0.3 area 0
    R1(config-router)# network 1.1.1.1 0.0.0.0 area 0
    R1(config-router)# end
    R1# write memory

### 6. Configure OSPF on R2 (Area 0)

    R2# configure terminal
    R2(config)# router ospf 1
    R2(config-router)# router-id 2.2.2.2
    R2(config-router)# passive-interface loopback 0
    R2(config-router)# network 10.0.12.0 0.0.0.3 area 0
    R2(config-router)# network 2.2.2.2 0.0.0.0 area 0
    R2(config-router)# end
    R2# write memory

### 7. Configure EIGRP on R2 (AS 100)

    R2# configure terminal
    R2(config)# router eigrp 100
    R2(config-router)# network 10.0.0.0
    R2(config-router)# network 2.2.2.2 0.0.0.0
    R2(config-router)# end
    R2# write memory

### 8. Configure EIGRP on R3 (AS 100)

    R3# configure terminal
    R3(config)# router eigrp 100
    R3(config-router)# network 10.0.0.0
    R3(config-router)# network 3.3.3.3 0.0.0.0
    R3(config-router)# end
    R3# write memory

### 9. Configure EIGRP on R4 (AS 100)

    R4# configure terminal
    R4(config)# router eigrp 100
    R4(config-router)# network 10.0.0.0
    R4(config-router)# network 4.4.4.4 0.0.0.0
    R4(config-router)# end
    R4# write memory

### 10. Configure Redistribution on R2 (OSPF → EIGRP)

    R2# configure terminal
    R2(config)# router eigrp 100
    R2(config-router)# redistribute ospf 1 metric 10000 100 255 1 1500
    R2(config-router)# end
    R2# write memory

### 11. Configure Redistribution on R2 (EIGRP → OSPF)

    R2# configure terminal
    R2(config)# router ospf 1
    R2(config-router)# redistribute eigrp 100 subnets metric 20
    R2(config-router)# end
    R2# write memory

## Verification Commands

    show ip route ospf
    show ip route eigrp
    show ip ospf database
    show ip eigrp topology

## Expected Results (Placeholder)

| Test | Command | Expected Result |
|------|---------|-----------------|
| OSPF routes on R1 | `show ip route ospf` | O 2.2.2.2, O E2 3.3.3.3, O E2 4.4.4.4 |
| EIGRP routes on R3 | `show ip route eigrp` | D 1.1.1.1, D 2.2.2.2 |
| Redistributed routes in OSPF | `show ip ospf database external` | Type 5 External LSAs |
| Ping across domains | `ping 3.3.3.3 source 1.1.1.1` | 5 replies, 0% loss |

## Files in This Folder

| File | Purpose |
|------|---------|
| `N09-route-redistribution.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config (OSPF only) |
| `R2-config.txt` | R2 running config (OSPF + EIGRP + redistribution) |
| `R3-config.txt` | R3 running config (EIGRP only) |
| `R4-config.txt` | R4 running config (EIGRP only) |
| `screenshots/ospf-routes-r1.png` | `show ip route ospf` on R1 showing redistributed EIGRP routes |
| `screenshots/eigrp-routes-r3.png` | `show ip route eigrp` on R3 showing redistributed OSPF routes |
| `screenshots/ping-r1-to-r3.png` | Ping from R1 loopback to R3 loopback |

## Time to Complete (Estimated)

35 minutes
