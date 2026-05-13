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

### Configure Root Bridge (Switch0)
```
Switch0> enable
Switch0# configure terminal
Switch0(config)# spanning-tree vlan 1 root primary
Switch0(config)# end
Switch0# write memory
```

### Configure Secondary Root (Switch1)
```
Switch1> enable
Switch1# configure terminal
Switch1(config)# spanning-tree vlan 1 root secondary
Switch1(config)# end
Switch1# write memory
```

## Verification Commands
```
! Verify root bridge
show spanning-tree | include "Root|Bridge"

! Verify blocking port
show spanning-tree | include "Altn|BLK"

! Show STP on specific interface
show spanning-tree interface gigabitEthernet 0/1
```

## Failure Simulation (Convergence Test)
```
! On root bridge, shut down trunk port
Switch0# configure terminal
Switch0(config)# interface gigabitEthernet 0/1
Switch0(config-if)# shutdown

! From PC2, ping PC1 continuously to observe convergence
ping 192.168.1.11
```

## Expected Results
| Switch | Role | Blocking Port? |
|--------|------|----------------|
| Switch0 | Root Bridge | No |
| Switch1 | Designated Bridge | No |
| Switch2 | Non-Root | Yes (one port in Altn/BLK) |

## Convergence Time
- **Default STP:** ~50 seconds
- **With root primary configured:** ~30-50 seconds
- **Observed in test:** ___ seconds

## STP Port States
| State | Time | Action |
|-------|------|--------|
| Blocking | 20 sec | Listens for BPDUs only |
| Listening | 15 sec | Elects root/designated ports |
| Learning | 15 sec | Learns MACs, no traffic |
| Forwarding | - | Normal operation |

## Troubleshooting
| Problem | Solution |
|---------|----------|
| No blocking port | Check triangle cabling |
| Pings fail | Verify IP addresses on PCs |
| Pings don't resume after shutdown | Wait up to 50 seconds |

## Files in This Folder
| File | Purpose |
|------|---------|
| `N03-spanning-tree.pkt` | Packet Tracer topology |
| `switch0-config.txt` | Root bridge config |
| `switch1-config.txt` | Secondary root config |
| `switch2-config.txt` | Non-root config |
| `screenshots/N03-stp-initial-blocking.png` | Blocking port before config |
| `screenshots/N03-stp-root-verified.png` | Root bridge verification |
| `screenshots/N03-ping-failure-convergence.png` | Ping loss during failover |
| `screenshots/N03-stp-new-blocking.png` | New blocking port after convergence |

## Time to Complete
20 minutes
