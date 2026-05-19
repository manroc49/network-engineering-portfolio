# N08: EIGRP Load Balancing – Three Links, Two Routers, One Unequal-Cost Path

## What This Proves

I can configure EIGRP across three parallel serial links between two routers and use the `variance` command to enable unequal-cost load balancing. Traffic is distributed across all three links even when one link has higher bandwidth than the others.

## Topology

- 2x Cisco 2811 routers (R1, R2)
- 3 parallel serial links between R1 and R2
- Loopback interfaces on each router (simulated networks)

**Note:** 2811 routers require HWIC-2T modules for serial ports. Add two HWIC-2T modules to R1 and R2 (each module provides 2 serial ports, 2 modules = 4 ports).

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | Bandwidth | Delay | Connected To |
|--------|-----------|------------|-------------|-----------|-------|--------------|
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | - | - | - |
| R1 | Serial0/0/0 | 10.0.12.1 | 255.255.255.252 | 10000 | 100 | R2 Serial0/0/0 |
| R1 | Serial0/0/1 | 10.0.13.1 | 255.255.255.252 | 100000 | 10 | R2 Serial0/0/1 |
| R1 | Serial0/0/2 | 10.0.14.1 | 255.255.255.252 | 10000 | 100 | R2 Serial0/0/2 |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | - | - | - |
| R2 | Serial0/0/0 | 10.0.12.2 | 255.255.255.252 | - | - | R1 Serial0/0/0 |
| R2 | Serial0/0/1 | 10.0.13.2 | 255.255.255.252 | - | - | R1 Serial0/0/1 |
| R2 | Serial0/0/2 | 10.0.14.2 | 255.255.255.252 | - | - | R1 Serial0/0/2 |

**Note:** Bandwidth and delay only need to be set on R1 for variance to work. EIGRP uses the metric of the outgoing interface. R2 does not need bandwidth/delay set for R1 to load balance.

## EIGRP Configuration

| Router | AS Number | Networks | Variance |
|--------|-----------|----------|----------|
| R1 | 100 | 10.0.0.0, 1.1.1.1/32 | 2 |
| R2 | 100 | 10.0.0.0, 2.2.2.2/32 | - |

## Configuration Files

- [R1-config.txt](R1-config.txt) - EIGRP AS 100, variance 2, bandwidth/delay set on serial links
- [R2-config.txt](R2-config.txt) - EIGRP AS 100, no variance needed

## Step-by-Step Configuration

### 1. Add HWIC-2T Modules to R1 and R2

- Click R1 → Physical tab → turn off router
- Drag two HWIC-2T modules to empty slots
- Turn on router
- Repeat for R2

### 2. Configure Basic IP on R1

    R1> enable
    R1# configure terminal
    R1(config)# interface loopback 0
    R1(config-if)# ip address 1.1.1.1 255.255.255.255
    R1(config-if)# exit
    R1(config)# interface serial 0/0/0
    R1(config-if)# ip address 10.0.12.1 255.255.255.252
    R1(config-if)# no shutdown
    R1(config-if)# clock rate 64000
    R1(config-if)# bandwidth 10000
    R1(config-if)# delay 100
    R1(config-if)# exit
    R1(config)# interface serial 0/0/1
    R1(config-if)# ip address 10.0.13.1 255.255.255.252
    R1(config-if)# no shutdown
    R1(config-if)# clock rate 64000
    R1(config-if)# bandwidth 100000
    R1(config-if)# delay 10
    R1(config-if)# exit
    R1(config)# interface serial 0/0/2
    R1(config-if)# ip address 10.0.14.1 255.255.255.252
    R1(config-if)# no shutdown
    R1(config-if)# clock rate 64000
    R1(config-if)# bandwidth 10000
    R1(config-if)# delay 100
    R1(config-if)# exit
    R1(config)# end
    R1# write memory

### 3. Configure Basic IP on R2

    R2> enable
    R2# configure terminal
    R2(config)# interface loopback 0
    R2(config-if)# ip address 2.2.2.2 255.255.255.255
    R2(config-if)# exit
    R2(config)# interface serial 0/0/0
    R2(config-if)# ip address 10.0.12.2 255.255.255.252
    R2(config-if)# no shutdown
    R2(config-if)# exit
    R2(config)# interface serial 0/0/1
    R2(config-if)# ip address 10.0.13.2 255.255.255.252
    R2(config-if)# no shutdown
    R2(config-if)# exit
    R2(config)# interface serial 0/0/2
    R2(config-if)# ip address 10.0.14.2 255.255.255.252
    R2(config-if)# no shutdown
    R2(config-if)# exit
    R2(config)# end
    R2# write memory

### 4. Configure EIGRP on R1

    R1# configure terminal
    R1(config)# router eigrp 100
    R1(config-router)# network 10.0.0.0
    R1(config-router)# network 1.1.1.1 0.0.0.0
    R1(config-router)# variance 2
    R1(config-router)# end
    R1# write memory

### 5. Configure EIGRP on R2

    R2# configure terminal
    R2(config)# router eigrp 100
    R2(config-router)# network 10.0.0.0
    R2(config-router)# network 2.2.2.2 0.0.0.0
    R2(config-router)# end
    R2# write memory

## Verification Commands

    show ip route
    show ip route 2.2.2.2
    show ip eigrp topology
    show ip eigrp traffic

## Expected Results (Placeholder)

| Test | Command | Expected Result |
|------|---------|-----------------|
| EIGRP neighbors | `show ip eigrp neighbors` | R1 and R2 adjacent |
| Route to R2 loopback | `show ip route 2.2.2.2` | Shows 3 successor routes with different metrics |
| Variance applied | `show ip eigrp topology` | Shows feasible successor (FS) routes |
| Load balancing | `show ip route 2.2.2.2` | Multiple next-hop addresses |

## Files in This Folder

| File | Purpose |
|------|---------|
| `N08-eigrp-load-balancing.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config (variance 2, bandwidth/delay set) |
| `R2-config.txt` | R2 running config |
| `screenshots/eigrp-neighbors.png` | `show ip eigrp neighbors` |
| `screenshots/eigrp-topology.png` | `show ip eigrp topology` showing FS routes |
| `screenshots/ip-route-2.2.2.2.png` | `show ip route 2.2.2.2` showing multiple next-hops |

## Time to Complete (Estimated)

25 minutes
