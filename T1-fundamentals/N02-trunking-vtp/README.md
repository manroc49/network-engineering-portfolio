# N02: 802.1Q Trunking & VTP

## What This Proves

I can connect multiple switches using trunk links and use VTP (VLAN Trunking Protocol) to automatically push VLANs from one server switch to all client switches. No more logging into each switch to create VLANs manually.

## Topology

- 3x Cisco 2960 switches (Switch0 = VTP Server, Switch1 = VTP Client, Switch2 = VTP Client)
- 2x PCs (PC1 on Switch1, PC2 on Switch2)

**Note:** No router in this topology. PC1 and PC2 are in different VLANs and CANNOT ping each other. This project proves VLAN propagation, not inter-VLAN routing.

## VTP Configuration

| Switch | VTP Mode | Domain | Password | Version |
|--------|----------|--------|----------|---------|
| Switch0 | Server | NETLAB | cisco123 | 2 |
| Switch1 | Client | NETLAB | cisco123 | 2 |
| Switch2 | Client | NETLAB | cisco123 | 2 |

## VLANs Created on Server (Propagate to Clients)

| VLAN ID | Name |
|---------|------|
| 10 | Engineering |
| 20 | Sales |
| 30 | Management |
| 99 | Native (unused) |

I set the native VLAN to 99 instead of leaving it on VLAN 1. This prevents VLAN hopping attacks.

## Configuration Files

- [switch0-config.txt](switch0-config.txt) - VTP Server
- [switch1-config.txt](switch1-config.txt) - VTP Client (VLAN 10 access)
- [switch2-config.txt](switch2-config.txt) - VTP Client (VLAN 20 access)

## Verification Results (All Passed)

| Test | Command | Result |
|------|---------|--------|
| VTP status on server | `show vtp status` | ✅ Mode: Server, Domain: NETLAB |
| VTP status on client | `show vtp status` | ✅ Mode: Client, same revision number as server |
| VLANs on server | `show vlan brief` | ✅ VLANs 10,20,30,99 exist |
| VLANs on client | `show vlan brief` | ✅ Same VLANs (proves propagation) |
| Trunk status | `show interfaces trunk` | ✅ Trunks up, native VLAN 99 |

## Screenshots

- [VTP status on server](screenshots/vtp-status-server.png)
- [VTP status on client](screenshots/vtp-status-client.png)
- [VLAN brief on server](screenshots/vlan-brief-server.png)
- [VLAN brief on client](screenshots/vlan-brief-client.png)
- [Trunk status](screenshots/trunk-status.png)
- [Topology](screenshots/topology.png)

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| Native VLAN mismatch | `%CDP-4-NATIVE_VLAN_MISMATCH` error, trunk down | Set `switchport trunk native vlan 99` on both ends of trunk |
| STP blocking trunk port | `%SPANTREE-2-BLOCK_PVID_LOCAL` error | Fixed native VLAN mismatch, STP recovered automatically |
| VTP domain case mismatch | VLANs not propagating | Used EXACT same case: `NETLAB` on all switches |
| `vtp pruning` command invalid | `% Invalid input detected` | Removed it (not supported on 2960 in Packet Tracer) |
| Trunk kept flapping | Port up/down repeatedly | Reset switch with `write erase` and `delete flash:vlan.dat` |

## What I'd Do Differently Next Time

- In a real production network, I probably wouldn't use VTP at all. Most companies disable it because the revision number problem is too dangerous.
- Use VTP version 3 if I had to use VTP (has better safety features like primary server concept)
- Manually configure trunks and VLANs on each switch instead of relying on VTP propagation

## Key Commands Used

### Switch0 (VTP Server)
- show vtp status
- show vlan brief
- show interfaces trunk
- show running-config

### Switch1 / Switch2 (VTP Clients)
- show vtp status
- show vlan brief
- show interfaces trunk
- show running-config

## What I Learned

- Native VLAN must match on both ends of a trunk. If one side is VLAN 1 and the other is VLAN 99, the trunk won't work.
- Order matters when configuring trunks. Set native VLAN first, then trunk mode.
- VTP domain names and passwords are case-sensitive. `NETLAB` is different from `Netlab`.
- Sometimes you just need to reset the switch. `write erase` and `delete flash:vlan.dat` followed by reload is faster than hunting down hidden config errors.
- The 2960 switch in Packet Tracer doesn't support `vtp pruning`.

## Time to Complete

45 minutes (including troubleshooting)
