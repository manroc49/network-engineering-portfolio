# N07: OSPF Multi-Area 
## What This Proves

I can configure OSPF across multiple areas (Area 0 backbone, Area 1, and Area 2) with an ABR (Area Border Router) handling route summarization and inter-area routing. This reduces LSA flooding and SPF calculations compared to a single-area design.

## Topology

- 4x Cisco 1941 routers (R1, R2, R3, R4)
- R2 is the ABR (Area Border Router) connecting all three areas
- Area 0 (Backbone): R1 and R2
- Area 1: R2 and R3
- Area 2: R2 and R4
- Loopback interfaces on each router (simulated networks)

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | Area | Connected To |
|--------|-----------|------------|-------------|------|--------------|
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | 0 | - |
| R1 | Gig0/0 | 10.0.12.1 | 255.255.255.252 | 0 | R2 Gig0/0 |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | 0 | - |
| R2 | Gig0/0 | 10.0.12.2 | 255.255.255.252 | 0 | R1 Gig0/0 |
| R2 | Gig0/1 | 10.0.23.1 | 255.255.255.252 | 1 | R3 Gig0/0 |
| R2 | Gig0/2 | 10.0.24.1 | 255.255.255.252 | 2 | R4 Gig0/0 |
| R3 | Loopback0 | 3.3.3.3 | 255.255.255.255 | 1 | - |
| R3 | Gig0/0 | 10.0.23.2 | 255.255.255.252 | 1 | R2 Gig0/1 |
| R4 | Loopback0 | 4.4.4.4 | 255.255.255.255 | 2 | - |
| R4 | Gig0/0 | 10.0.24.2 | 255.255.255.252 | 2 | R2 Gig0/2 |

## Area Design

| Area | Routers | Networks |
|------|---------|----------|
| Area 0 (Backbone) | R1, R2 | 10.0.12.0/30, 1.1.1.1/32, 2.2.2.2/32 |
| Area 1 | R2, R3 | 10.0.23.0/30, 3.3.3.3/32 |
| Area 2 | R2, R4 | 10.0.24.0/30, 4.4.4.4/32 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - OSPF Area 0
- [R2-config.txt](R2-config.txt) - ABR connecting Areas 0, 1, and 2 with route summarization
- [R3-config.txt](R3-config.txt) - OSPF Area 1
- [R4-config.txt](R4-config.txt) - OSPF Area 2

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

### 2. Configure Basic IP on R2 (ABR)

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

### 6. Configure OSPF on R2 (ABR - Areas 0, 1, and 2)

    R2# configure terminal
    R2(config)# router ospf 1
    R2(config-router)# router-id 2.2.2.2
    R2(config-router)# passive-interface loopback 0
    R2(config-router)# network 10.0.12.0 0.0.0.3 area 0
    R2(config-router)# network 10.0.23.0 0.0.0.3 area 1
    R2(config-router)# network 10.0.24.0 0.0.0.3 area 2
    R2(config-router)# network 2.2.2.2 0.0.0.0 area 0
    R2(config-router)# area 1 range 10.0.23.0 255.255.255.252
    R2(config-router)# area 2 range 10.0.24.0 255.255.255.252
    R2(config-router)# end
    R2# write memory

### 7. Configure OSPF on R3 (Area 1)

    R3# configure terminal
    R3(config)# router ospf 1
    R3(config-router)# router-id 3.3.3.3
    R3(config-router)# passive-interface loopback 0
    R3(config-router)# network 10.0.23.0 0.0.0.3 area 1
    R3(config-router)# network 3.3.3.3 0.0.0.0 area 1
    R3(config-router)# end
    R3# write memory

### 8. Configure OSPF on R4 (Area 2)

    R4# configure terminal
    R4(config)# router ospf 1
    R4(config-router)# router-id 4.4.4.4
    R4(config-router)# passive-interface loopback 0
    R4(config-router)# network 10.0.24.0 0.0.0.3 area 2
    R4(config-router)# network 4.4.4.4 0.0.0.0 area 2
    R4(config-router)# end
    R4# write memory

## Verification Commands

    show ip ospf neighbor
    show ip ospf border-routers
    show ip ospf database summary
    show ip route ospf

## Expected Results (Placeholder)

| Test | Command | Expected Result |
|------|---------|-----------------|
| OSPF neighbors on R2 | show ip ospf neighbor | R1, R3, R4 all in FULL state |
| ABR status on R2 | show ip ospf border-routers | Shows R2 as ABR |
| Summary routes in database | show ip ospf database summary | Shows summarized routes for Area 1 and Area 2 |
| Inter-area routes on R1 | show ip route ospf | Routes to 3.3.3.3, 4.4.4.4, 10.0.23.0, 10.0.24.0 (marked O IA) |

## Files in This Folder

| File | Purpose |
|------|---------|
| N07-ospf-multi-area.pkt | Packet Tracer topology |
| R1-config.txt | R1 running config (Area 0) |
| R2-config.txt | R2 running config (ABR) |
| R3-config.txt | R3 running config (Area 1) |
| R4-config.txt | R4 running config (Area 2) |
| screenshots/ospf-neighbors-r2.png | show ip ospf neighbor on R2 showing all neighbors |
| screenshots/ospf-border-routers.png | show ip ospf border-routers showing ABR |
| screenshots/ospf-routes-r1.png | show ip route ospf on R1 showing O IA routes |

## Time to Complete (Estimated)

30 minutes
