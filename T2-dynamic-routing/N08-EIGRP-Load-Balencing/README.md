# N08: EIGRP Load Balancing – Three Links, Two Routers, One Unequal-Cost Path

## What This Proves

I can configure EIGRP across three parallel serial links between two routers and use the `variance` command to enable unequal-cost load balancing. Traffic is distributed across all three links even when one link has higher bandwidth than the others.

## Topology

- 2x Cisco 2811 routers (R1, R2)
- 3 parallel serial links between R1 and R2
- Loopback interfaces on each router (simulated networks)

**Note:** 2811 routers require HWIC-2T modules for serial ports. Add two HWIC-2T modules to R1 and R2 (each module provides 2 serial ports, 2 modules = 4 ports). Interface names may vary based on slot placement (Serial0/0/0, Serial0/0/1, Serial0/2/0, etc.).

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | Bandwidth | Delay | Connected To |
|--------|-----------|------------|-------------|-----------|-------|--------------|
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | - | - | - |
| R1 | Serial0/0/0 | 10.0.12.1 | 255.255.255.252 | 10000 | 100 | R2 Serial0/0/0 |
| R1 | Serial0/0/1 | 10.0.13.1 | 255.255.255.252 | 100000 | 10 | R2 Serial0/0/1 |
| R1 | Serial0/2/0 | 10.0.14.1 | 255.255.255.252 | 10000 | 100 | R2 Serial0/2/0 |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | - | - | - |
| R2 | Serial0/0/0 | 10.0.12.2 | 255.255.255.252 | - | - | R1 Serial0/0/0 |
| R2 | Serial0/0/1 | 10.0.13.2 | 255.255.255.252 | - | - | R1 Serial0/0/1 |
| R2 | Serial0/2/0 | 10.0.14.2 | 255.255.255.252 | - | - | R1 Serial0/2/0 |

**Note:** Bandwidth and delay only need to be set on R1 for variance to work. EIGRP uses the metric of the outgoing interface. R2 does not need bandwidth/delay set for R1 to load balance. Interface names may vary (Serial0/2/0 could be Serial0/1/0 depending on HWIC slot).

## EIGRP Configuration

| Router | AS Number | Networks | Variance |
|--------|-----------|----------|----------|
| R1 | 100 | 10.0.0.0, 1.1.1.1/32 | 3 |
| R2 | 100 | 10.0.0.0, 2.2.2.2/32 | - |

## Configuration Files

- [R1-config.txt](R1-config.txt) - EIGRP AS 100, variance 3, bandwidth/delay set on serial links
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
    R1(config)# interface serial 0/2/0
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
    R2(config)# interface serial 0/2/0
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
    R1(config-router)# variance 3
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

    show ip eigrp neighbors
    show ip eigrp topology
    show ip route 2.2.2.2
    show ip route

## Verification Results (All Passed)

| Test | Command | Result |
|------|---------|--------|
| EIGRP neighbors on R1 | `show ip eigrp neighbors` | ✅ 3 neighbors (10.0.12.2, 10.0.13.2, 10.0.14.2) |
| EIGRP topology | `show ip eigrp topology` | ✅ 1 successor (10.0.13.2), 2 feasible successors |
| Route to R2 loopback (variance 2) | `show ip route 2.2.2.2` | ✅ Only 1 next-hop (10.0.13.2) |
| Route to R2 loopback (variance 3) | `show ip route 2.2.2.2` | ✅ 3 next-hops (unequal-cost load balancing) |
| Connectivity test | `ping 2.2.2.2 source 1.1.1.1` | ✅ 5 replies, 0% loss |

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| No serial ports on 2811 | `show ip interface brief` shows no Serial interfaces | Added HWIC-2T modules in Physical tab |
| Interface naming different than expected | Serial0/0/2 didn't exist | Used Serial0/2/0 (slot 2, port 0) from second HWIC module |
| Serial link down/down | Protocol shows down | Added `clock rate 64000` on R1 (DCE side) for all serial interfaces |
| Interface administratively down | Status shows down | Added `no shutdown` to all serial interfaces |
| EIGRP neighbors not forming | `show ip eigrp neighbors` shows nothing | Verified network statements include 10.0.0.0 |
| Unequal-cost load balancing not working | Only 1 next-hop in routing table | Increased variance from 2 to 3 (metric ratio was 2.62:1) |
| Forgot `bandwidth` and `delay` on R1 | EIGRP metrics used default serial bandwidth | Added bandwidth 10000/100000 and delay 100/10 on R1 interfaces |

## What I'd Do Differently Next Time

- Calculate the exact metric ratio before setting variance. The ratio was 409600 / 156160 = 2.62, requiring variance 3.
- Document interface names after adding HWIC modules. They vary based on which slot you use.
- Use `bandwidth` command on both sides for consistency, though only outgoing metric matters.
- Test with `debug ip eigrp` to see exactly which routes are being considered.

## Key Commands Used

### EIGRP Configuration
- `router eigrp 100`
- `network 10.0.0.0`
- `network 1.1.1.1 0.0.0.0`
- `variance 3`

### Interface Configuration
- `interface serial 0/0/0`
- `bandwidth 10000`
- `delay 100`
- `clock rate 64000`
- `no shutdown`

### Verification
- `show ip eigrp neighbors`
- `show ip eigrp topology`
- `show ip route 2.2.2.2`

## What I Learned

- EIGRP calculates metric based on bandwidth and delay by default. Formula: metric = (10^7 / bandwidth + delay/10) * 256.
- Higher bandwidth = lower metric. 100000 kbps (100 Mbps) has lower metric than 10000 kbps (10 Mbps).
- The `variance` command enables unequal-cost load balancing. It multiplies the best metric. Any feasible successor with metric less than (best_metric * variance) is installed in the routing table.
- Feasible successors are backup routes that meet the feasibility condition (reported distance < feasible distance). They can become successors if variance allows.
- Without variance, EIGRP only installs successors (best routes) in the routing table.
- HWIC-2T modules add serial ports. Two modules = four ports. Port naming: Serial0/0/0, Serial0/0/1 (first module), Serial0/1/0, Serial0/1/1 (second module), or Serial0/2/0 depending on slot.
- The DCE side of a serial link needs `clock rate`. The DTE side does not. Click the DCE end first when cabling in Packet Tracer.

## Screenshots

- [EIGRP neighbors on R1](screenshots/eigrp-neighbors.png)
- [EIGRP topology table](screenshots/eigrp-topology.png)
- [Route to 2.2.2.2 with variance 3](screenshots/ip-route-2.2.2.2.png)

## Time to Complete

30 minutes
