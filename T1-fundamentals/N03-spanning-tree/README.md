# Project N03: Spanning Tree Protocol (PVST+)

## What This Proves
I can prevent Layer 2 loops by configuring Spanning Tree root bridge placement and verify convergence after link failure.

## Topology
- **3x Cisco 2960 Switches** (Triangle: Switch0 → Switch1 → Switch2 → Switch0)
- **3x PCs** (one per switch for testing)
- **Redundant links** (creates loop without STP)

## Design Goal
- Make Switch0 the **Root Bridge** for VLAN 1
- Observe default blocking port, then manually control root election
- Verify convergence when root bridge link fails

## IP Addressing Plan
| Device | Interface | IP Address | Subnet Mask |
|--------|-----------|------------|-------------|
| PC0 | Fa0 | 192.168.1.10 | 255.255.255.0 |
| PC1 | Fa0 | 192.168.1.11 | 255.255.255.0 |
| PC2 | Fa0 | 192.168.1.12 | 255.255.255.0 |

*All PCs on same subnet (VLAN 1 default) – no router needed*

## Step-by-Step Configuration

### 1. Find Default Blocking Port
```
Switch> enable
Switch# show spanning-tree
```
**Expected:** One switch has a port showing `Altn BLK`

### 2. Configure Root Bridge (Switch0)
```
Switch0> enable
Switch0# configure terminal
Switch0(config)# spanning-tree vlan 1 root primary
Switch0(config)# end
Switch0# write memory
```

### 3. Verify Root Bridge
```
Switch0# show spanning-tree
```
**Expected:** `This bridge is the root`

### 4. Configure Secondary Root (Switch1) – Optional
```
Switch1> enable
Switch1# configure terminal
Switch1(config)# spanning-tree vlan 1 root secondary
Switch1(config)# end
Switch1# write memory
```

### 5. Test Convergence (Failure Simulation)
```
! On PC2 Command Prompt
ping 192.168.1.11 -n 1000

! While ping runs, on Switch0
Switch0# configure terminal
Switch0(config)# interface gigabitEthernet 0/2
Switch0(config-if)# shutdown
```
**Expected:** Pings stop, then resume after 30-50 seconds

### 6. Find New Blocking Port
Look for red triangle on physical topology view (Packet Tracer does not show `Altn BLK` in CLI after reconvergence)

## Verification Commands Summary
| Command | What It Shows |
|---------|----------------|
| `show spanning-tree` | Root bridge, port roles |
| `show spanning-tree vlan 1` | STP info for VLAN 1 |
| `show spanning-tree root` | Root bridge ID |

## Packet Tracer Note
**Important:** Packet Tracer does NOT always show blocking ports (`Altn BLK`) in `show spanning-tree` output after link failure. Instead, look for **red triangles** on physical ports in the topology view.

## Expected Results
| Switch | Role | Blocking Indicator |
|--------|------|---------------------|
| Switch0 | Root Bridge | No red triangles |
| Switch1 | Designated Bridge | No red triangles |
| Switch2 | Non-Root | Red triangle on Gi0/1 or Gi0/2 |

## Convergence Time
- **Observed in test:** ___ seconds (count `Request timed out` messages)

## Files in This Folder
-  [N03-spanning-tree.pkt](https://github.com/manroc49/network-engineering-portfolio/blob/main/T1-fundamentals/N03-spanning-tree/N03-spanning-tree.pkt)
-  [switch0-config.txt](https://github.com/manroc49/network-engineering-portfolio/blob/main/T1-fundamentals/N03-spanning-tree/switch0-config.txt)
-  [switch1-config.txt](https://github.com/manroc49/network-engineering-portfolio/blob/main/T1-fundamentals/N03-spanning-tree/switch2-config.txt)
-  [switch2-config.txt](https://github.com/manroc49/network-engineering-portfolio/blob/main/T1-fundamentals/N03-spanning-tree/switch3-config.txt)

## Troubleshooting
| Problem | Solution |
|---------|----------|
| No red triangles | Check triangle cabling |
| Pings fail | Verify IP addresses on PCs |
| Port missing from `show spanning-tree` | Check physical red triangle – Packet Tracer bug |
| Pings don't resume | Wait up to 50 seconds |

## Screenshots 
-  [Topology 1](/screenshots/topology-1.png)
-  [Topology 2](/screenshots/topolgy-2.png)
-  [Initial blocking port](/screenshots/stp-initial-blocking.png)
-  [Root bridge verification](/screenshots/stp-root-verified.png)
-  [Ping loss during failover](/screenshots/ping-failure-convergence.png)
-  [New blocking port (red triangle)](/sreenshots/stp-new-blocking.png)

## Time to Complete
25 minutes
