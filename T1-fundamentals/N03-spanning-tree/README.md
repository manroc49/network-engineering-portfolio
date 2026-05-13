# N03: Spanning Tree Protocol (PVST+)

## What This Proves

I can prevent Layer 2 loops by configuring Spanning Tree root bridge placement and verify convergence after link failure. STP automatically blocks redundant ports to stop broadcast storms, then unblocks them if the active path fails.

## Topology

- 3x Cisco 2960 switches (Switch0, Switch1, Switch2 in a triangle)
- 3x PCs (PC0 on Switch0, PC1 on Switch1, PC2 on Switch2)
- Redundant links (creates physical loop that STP must prevent)

**Note:** Triangle topology means every switch connects to the other two. Without STP, this creates a broadcast storm. With STP, one port is automatically blocked.

## STP Configuration

| Switch | STP Role | Bridge Priority | Blocking Port |
|--------|----------|-----------------|---------------|
| Switch0 | Root Bridge | 24576 (root primary) | None |
| Switch1 | Designated Bridge | 28672 (root secondary) | None |
| Switch2 | Non-Root | 32768 (default) | Yes (Gig0/1 or Gig0/2) |

## IP Addressing (for ping testing)

| Device | IP Address | Subnet Mask | Connected To |
|--------|------------|-------------|--------------|
| PC0 | 192.168.1.10 | 255.255.255.0 | Switch0 Fa0/1 |
| PC1 | 192.168.1.11 | 255.255.255.0 | Switch1 Fa0/1 |
| PC2 | 192.168.1.12 | 255.255.255.0 | Switch2 Fa0/1 |

*All PCs on same subnet (VLAN 1 default) – no router needed*

## Configuration Files

- [switch0-config.txt](switch0-config.txt) - Root bridge (root primary)
- [switch1-config.txt](switch1-config.txt) - Secondary root (root secondary)
- [switch2-config.txt](switch2-config.txt) - Non-root (default STP)

## Verification Results (All Passed)

| Test | Command | Result |
|------|---------|--------|
| Default blocking port | `show spanning-tree` before config | ✅ One switch showed `Altn BLK` |
| Root bridge config | `spanning-tree vlan 1 root primary` on Switch0 | ✅ No errors |
| Root bridge verification | `show spanning-tree` on Switch0 | ✅ "This bridge is the root" |
| Blocking port moved | `show spanning-tree` on Switch1/Switch2 | ✅ Blocking port on non-root switch |
| Ping before failure | `ping 192.168.1.11 -n 4` from PC2 | ✅ 4 replies, 0% loss |
| Ping during failure | `ping 192.168.1.11 -n 1000` + shutdown Switch0 Gig0/2 | ✅ Paused for 8 seconds, then resumed |
| New blocking port after failure | `show spanning-tree` on Switch2 | ✅ Red triangle on different port (Packet Tracer bug) |

## Screenshots

- [Initial blocking port before config](screenshots/N03-stp-initial-blocking.png)
- [Root bridge verification](screenshots/N03-stp-root-verified.png)
- [Ping loss during failover](screenshots/N03-ping-failure-convergence.png)
- [New blocking port after convergence (red triangle)](screenshots/N03-stp-new-blocking.png)

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| No blocking port appears | All ports show `Desg FWD` | Check triangle cabling – all three switches must connect |
| Pings never work | `Request timed out` from start | Forgot to assign IP addresses to PCs |
| Pings never resume after shutdown | Timeouts keep going forever | Wait up to 50 seconds (STP is slow) or check which port now has red triangle |
| `spanning-tree vlan 10 root primary` errors | `% Invalid input detected` | VLAN 10 doesn't exist. Use VLAN 1 only or create VLANs first |
| Blocking port missing from `show spanning-tree` after failure | Port just disappears from output | Packet Tracer bug. Look for red triangle on physical topology instead |
| `| include "Root|Bridge"` doesn't work | `% Invalid input detected at '^' marker` | Packet Tracer doesn't support pipe filtering. Use `show spanning-tree` and read manually |

## What I'd Do Differently Next Time

- Use RSTP (802.1w) instead of classic STP for sub-second convergence
- Configure `spanning-tree portfast` on access ports (PC-facing ports) to skip listening/learning states
- Use `spanning-tree bpduguard` on access ports to prevent rogue switches from becoming root
- In a real network, use `show spanning-tree detail` for more granular timer information
- Document convergence time more precisely using Wireshark to capture BPDU timestamps

## Key Commands Used

### STP Configuration
- `spanning-tree vlan 1 root primary` (force root bridge)
- `spanning-tree vlan 1 root secondary` (backup root)
- `show spanning-tree` (view STP status)
- `show spanning-tree vlan 1` (specific VLAN)
- `show spanning-tree interface gigabitEthernet 0/1` (port-specific)

### Failure Testing
- `ping 192.168.1.11 -n 1000` from PC2 (continuous ping)
- `interface gigabitEthernet 0/2` → `shutdown` (break root bridge link)
- `no shutdown` (restore link)

## What I Learned

- STP prevents loops by electing a root bridge and blocking redundant ports. The root bridge is elected by lowest bridge priority (default 32768), then lowest MAC address.
- Port roles: `Root` (faces root bridge), `Desg` (Designated, forwards away from root), `Altn` (Alternate/Blocking, prevents loop).
- Convergence with default timers takes about 50 seconds (Blocking 20s → Listening 15s → Learning 15s → Forwarding).
- Forcing a root bridge with `spanning-tree vlan 1 root primary` sets priority to 24576, guaranteeing it wins.
- Packet Tracer has a bug: after reconvergence, `show spanning-tree` sometimes omits blocking ports entirely. Look for red triangles on the physical topology view instead.
- When a link fails, STP reconverges and the blocking port transitions to forwarding. Traffic resumes automatically.
- Classic STP is slow (30-50 seconds). Modern networks use RSTP (802.1w) for sub-second failover.

## Time to Complete

25 minutes (including troubleshooting Packet Tracer bugs)
