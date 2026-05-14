# N07: OSPF Multi-Area – Splitting The Network So OSPF Doesn't Melt

## What This Proves

I can configure OSPF across multiple areas (Area 0 backbone, Area 1, and Area 2) with an ABR (Area Border Router) handling inter-area routing. This reduces LSA flooding and SPF calculations compared to single-area design.

## Topology

- 4x Cisco 2811 routers (R1, R2, R3, R4)
- R2 is the ABR (Area Border Router) connecting all three areas
- Area 0 (Backbone): R1 and R2 (FastEthernet link)
- Area 1: R2 and R3 (Serial link)
- Area 2: R2 and R4 (Serial link)
- Loopback interfaces on each router (simulated networks)

**Note:** 2811 routers require HWIC-2T modules for serial ports. Add one HWIC-2T to R2, R3, and R4 before cabling.

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | Area | Connected To |
|--------|-----------|------------|-------------|------|--------------|
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | 0 | - |
| R1 | FastEthernet0/0 | 10.0.12.1 | 255.255.255.252 | 0 | R2 Fa0/0 |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | 0 | - |
| R2 | FastEthernet0/0 | 10.0.12.2 | 255.255.255.252 | 0 | R1 Fa0/0 |
| R2 | Serial0/0/0 | 10.0.23.1 | 255.255.255.252 | 1 | R3 Serial0/0/0 |
| R2 | Serial0/0/1 | 10.0.24.1 | 255.255.255.252 | 2 | R4 Serial0/0/0 |
| R3 | Loopback0 | 3.3.3.3 | 255.255.255.255 | 1 | - |
| R3 | Serial0/0/0 | 10.0.23.2 | 255.255.255.252 | 1 | R2 Serial0/0/0 |
| R4 | Loopback0 | 4.4.4.4 | 255.255.255.255 | 2 | - |
| R4 | Serial0/0/0 | 10.0.24.2 | 255.255.255.252 | 2 | R2 Serial0/0/1 |

## Router IDs

| Router | Router ID |
|--------|-----------|
| R1 | 1.1.1.1 |
| R2 | 2.2.2.2 |
| R3 | 3.3.3.3 |
| R4 | 4.4.4.4 |

## Area Design

| Area | Routers | Networks |
|------|---------|----------|
| Area 0 (Backbone) | R1, R2 | 10.0.12.0/30, 1.1.1.1/32, 2.2.2.2/32 |
| Area 1 | R2, R3 | 10.0.23.0/30, 3.3.3.3/32 |
| Area 2 | R2, R4 | 10.0.24.0/30, 4.4.4.4/32 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - OSPF Area 0
- [R2-config.txt](R2-config.txt) - ABR connecting Areas 0, 1, and 2
- [R3-config.txt](R3-config.txt) - OSPF Area 1
- [R4-config.txt](R4-config.txt) - OSPF Area 2

## Step-by-Step Configuration

### 1. Configure Basic IP on R1

    R1> enable
    R1# configure terminal
    R1(config)# interface loopback 0
    R1(config-if)# ip address 1.1.1.1 255.255.255.255
    R1(config-if)# exit
    R1(config)# interface fastEthernet 0/0
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
    R2(config)# interface fastEthernet 0/0
    R2(config-if)# ip address 10.0.12.2 255.255.255.252
    R2(config-if)# no shutdown
    R2(config-if)# exit
    R2(config)# interface serial 0/0/0
    R2(config-if)# ip address 10.0.23.1 255.255.255.252
    R2(config-if)# no shutdown
    R2(config-if)# clock rate 64000
    R2(config-if)# exit
    R2(config)# interface serial 0/0/1
    R2(config-if)# ip address 10.0.24.1 255.255.255.252
    R2(config-if)# no shutdown
    R2(config-if)# clock rate 64000
    R2(config-if)# exit
    R2(config)# end
    R2# write memory

### 3. Configure Basic IP on R3

    R3> enable
    R3# configure terminal
    R3(config)# interface loopback 0
    R3(config-if)# ip address 3.3.3.3 255.255.255.255
    R3(config-if)# exit
    R3(config)# interface serial 0/0/0
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
    R4(config)# interface serial 0/0/0
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

### 6. Configure OSPF on R2 (ABR - Areas 0, 1, 2)

    R2# configure terminal
    R2(config)# router ospf 1
    R2(config-router)# router-id 2.2.2.2
    R2(config-router)# passive-interface loopback 0
    R2(config-router)# network 10.0.12.0 0.0.0.3 area 0
    R2(config-router)# network 10.0.23.0 0.0.0.3 area 1
    R2(config-router)# network 10.0.24.0 0.0.0.3 area 2
    R2(config-router)# network 2.2.2.2 0.0.0.0 area 0
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
    show ip route ospf
    show ip ospf interface
    show ip protocols

## Verification Results (All Passed)

| Test | Command | Result |
|------|---------|--------|
| OSPF neighbors on R2 | `show ip ospf neighbor` | ✅ R1 (1.1.1.1), R3 (3.3.3.3), R4 (4.4.4.4) in FULL state |
| OSPF routes on R1 | `show ip route ospf` | ✅ O IA routes to 3.3.3.3, 4.4.4.4, 10.0.23.0, 10.0.24.0 |
| Inter-area ping | `ping 3.3.3.3 source 1.1.1.1` | ✅ 5 replies, 0% loss |

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| Missing HWIC-2T modules | Serial interfaces not showing up | Added HWIC-2T modules to R2, R3, R4 in Physical tab |
| R2 FastEthernet0/0 had wrong IP | IP was 10.0.23.1 instead of 10.0.12.2 | Changed to 10.0.12.2 |
| R2 Serial0/0/0 administratively down | Interface showed `administratively down` | Added `no shutdown` to Serial0/0/0 |
| Missing clock rate on serial | Serial link down/down | Added `clock rate 64000` on R2 serial interfaces (DCE side) |
| R3 Serial0/0/0 down | Protocol down | Added `no shutdown` on R3 serial interface |
| Forgot `passive-interface loopback 0` | Loopback sending Hellos unnecessarily | Added to all routers |
| OSPF neighbors not forming | Missing network statements | Added `network` statements for all connected subnets |

## What I'd Do Differently Next Time

- Configure OSPF network statements using the interface IP with `0.0.0.0` wildcard mask instead of the whole subnet. More precise.
- Use `router-id` command BEFORE starting OSPF process to avoid dynamic election conflicts.
- Document all IP addresses in a table BEFORE configuring anything.
- Add `bandwidth` commands on serial links for accurate metric calculation.

## Key Commands Used

### OSPF Configuration
- `router ospf 1`
- `router-id 2.2.2.2`
- `passive-interface loopback 0`
- `network 10.0.12.0 0.0.0.3 area 0`
- `network 10.0.23.0 0.0.0.3 area 1`
- `network 10.0.24.0 0.0.0.3 area 2`

### Serial Configuration
- `interface serial 0/0/0`
- `clock rate 64000`
- `no shutdown`

### Verification
- `show ip ospf neighbor`
- `show ip route ospf`
- `show ip interface brief`

## What I Learned

- Multi-area OSPF reduces LSA flooding and SPF calculations. Routers in Area 0 don't need to know the detailed topology of Area 1.
- An ABR (Area Border Router) has interfaces in multiple areas and maintains separate LSDBs for each area.
- Type 3 Summary LSAs are generated by ABRs to advertise networks from one area to another. They show up as `O IA` routes in `show ip route`.
- Serial links require a clock rate on the DCE side. Without it, the link stays `down/down`.
- The HWIC-2T module adds serial ports to 2811 routers. You must add it BEFORE cabling.
- `show ip ospf border-routers` on the ABR itself shows nothing. That command is for viewing other border routers.
- `show ip route ospf` shows intra-area routes as `O` and inter-area routes as `O IA`.
- The area range command (not used in this lab) would summarize routes at the ABR, reducing routing table size.

## Screenshots

- [OSPF neighbors on R2](screenshots/ospf-neighbors-r2.png)
- [OSPF routes on R1](screenshots/ospf-routes-r1.png)

## Files in This Folder

| File | Purpose |
|------|---------|
| `N07-ospf-multi-area.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config |
| `R2-config.txt` | R2 running config (ABR) |
| `R3-config.txt` | R3 running config |
| `R4-config.txt` | R4 running config |
| `screenshots/ospf-neighbors-r2.png` | `show ip ospf neighbor` on R2 |
| `screenshots/ospf-routes-r1.png` | `show ip route ospf` on R1 |

## Time to Complete

30 minutes
