# N06: OSPF Single Area – Hello Neighbors, Let's Share Routes

## What This Proves
I can configure OSPF in a single area (Area 0) across four routers, establish neighbor adjacencies, and let the routers automatically learn each other's routes. No more manual static routes. OSPF does the work for me.

## Topology
- 4x Cisco 1941 routers (R1, R2, R3, R4 in a line)
- Loopback interfaces on each router (simulated networks)
- Point-to-point links between routers (/30 subnets)

**Note:** Original project suggested a square topology. I used a line instead. Simpler for beginners, no physical loops to confuse OSPF network types.

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | Connected To |
|--------|-----------|------------|-------------|--------------|
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | - |
| R1 | Gig0/0 | 10.0.12.1 | 255.255.255.252 | R2 Gig0/0 |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | - |
| R2 | Gig0/0 | 10.0.12.2 | 255.255.255.252 | R1 Gig0/0 |
| R2 | Gig0/1 | 10.0.23.1 | 255.255.255.252 | R3 Gig0/0 |
| R3 | Loopback0 | 3.3.3.3 | 255.255.255.255 | - |
| R3 | Gig0/0 | 10.0.23.2 | 255.255.255.252 | R2 Gig0/1 |
| R3 | Gig0/1 | 10.0.34.1 | 255.255.255.252 | R4 Gig0/0 |
| R4 | Loopback0 | 4.4.4.4 | 255.255.255.255 | - |
| R4 | Gig0/0 | 10.0.34.2 | 255.255.255.252 | R3 Gig0/1 |

## Router IDs

| Router | Router ID |
|--------|-----------|
| R1 | 1.1.1.1 |
| R2 | 2.2.2.2 |
| R3 | 3.3.3.3 |
| R4 | 4.4.4.4 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - OSPF Area 0, router-id 1.1.1.1
- [R2-config.txt](R2-config.txt) - OSPF Area 0, router-id 2.2.2.2
- [R3-config.txt](R3-config.txt) - OSPF Area 0, router-id 3.3.3.3
- [R4-config.txt](R4-config.txt) - OSPF Area 0, router-id 4.4.4.4

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
text


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
R2(config)# end
R2# write memory
text


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
text


### 4. Configure Basic IP on R4

R4> enable
R4# configure terminal
R4(config)# interface loopback 0
R4(config-if)# ip address 4.4.4.4 255.255.255.255
R4(config-if)# exit
R4(config)# interface gigabitEthernet 0/0
R4(config-if)# ip address 10.0.34.2 255.255.255.252
R4(config-if)# no shutdown
R4(config-if)# exit
R4(config)# end
R4# write memory
text


### 5. Configure OSPF on R1

R1# configure terminal
R1(config)# router ospf 1
R1(config-router)# router-id 1.1.1.1
R1(config-router)# passive-interface loopback 0
R1(config-router)# network 10.0.12.0 0.0.0.3 area 0
R1(config-router)# network 1.1.1.1 0.0.0.0 area 0
R1(config-router)# end
R1# write memory
text


### 6. Configure OSPF on R2

R2# configure terminal
R2(config)# router ospf 1
R2(config-router)# router-id 2.2.2.2
R2(config-router)# passive-interface loopback 0
R2(config-router)# network 10.0.12.0 0.0.0.3 area 0
R2(config-router)# network 10.0.23.0 0.0.0.3 area 0
R2(config-router)# network 2.2.2.2 0.0.0.0 area 0
R2(config-router)# end
R2# write memory
text


### 7. Configure OSPF on R3

R3# configure terminal
R3(config)# router ospf 1
R3(config-router)# router-id 3.3.3.3
R3(config-router)# passive-interface loopback 0
R3(config-router)# network 10.0.23.0 0.0.0.3 area 0
R3(config-router)# network 10.0.34.0 0.0.0.3 area 0
R3(config-router)# network 3.3.3.3 0.0.0.0 area 0
R3(config-router)# end
R3# write memory
text


### 8. Configure OSPF on R4

R4# configure terminal
R4(config)# router ospf 1
R4(config-router)# router-id 4.4.4.4
R4(config-router)# passive-interface loopback 0
R4(config-router)# network 10.0.34.0 0.0.0.3 area 0
R4(config-router)# network 4.4.4.4 0.0.0.0 area 0
R4(config-router)# end
R4# write memory
text


## Verification Commands

show ip ospf neighbor
show ip ospf interface
show ip route ospf
show ip protocols
text


## Expected Results (Placeholder)

| Test | Command | Expected Result |
|------|---------|-----------------|
| OSPF neighbors on R2 | `show ip ospf neighbor` | R1 and R3 in FULL state |
| OSPF routes on R1 | `show ip route ospf` | Routes to 2.2.2.2, 3.3.3.3, 4.4.4.4, 10.0.23.0, 10.0.34.0 |
| OSPF routes on R4 | `show ip route ospf` | Routes to 1.1.1.1, 2.2.2.2, 3.3.3.3, 10.0.12.0, 10.0.23.0 |

## Files in This Folder

| File | Purpose |
|------|---------|
| `N06-ospf-single-area.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config |
| `R2-config.txt` | R2 running config |
| `R3-config.txt` | R3 running config |
| `R4-config.txt` | R4 running config |
| `screenshots/ospf-neighbors.png` | `show ip ospf neighbor` output |
| `screenshots/ospf-routes-r1.png` | `show ip route ospf` on R1 |
| `screenshots/ospf-simulation.png` | OSPF hellos in Simulation mode |

## Time to Complete (Estimated)
25 minutes
