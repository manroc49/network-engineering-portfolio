# N04: EtherChannel (LACP)

## What This Proves

I can bundle multiple physical links between two switches into a single logical link using LACP. This increases bandwidth and provides redundancy. If one physical link fails, traffic automatically shifts to the remaining links with no packet loss and no STP reconvergence delay.

## Topology

- 2x Cisco 2960 switches (Switch0, Switch1)
- 2x PCs (PC0 on Switch0, PC1 on Switch1)
- 2 parallel Gigabit links (Gig0/1 and Gig0/2)

**Note:** Cisco 2960 switches only have 2 Gigabit ports. The original project called for 3 links, but 2 links works the same way. EtherChannel supports 2-8 ports.

## EtherChannel Configuration

| Switch | Channel-Group | Protocol | Mode | Port-Channel |
|--------|---------------|----------|------|--------------|
| Switch0 | 1 | LACP | active | Po1 (trunk) |
| Switch1 | 1 | LACP | active | Po1 (trunk) |

## IP Addressing (for ping testing)

| Device | IP Address | Subnet Mask | Connected To |
|--------|------------|-------------|--------------|
| PC0 | 192.168.1.10 | 255.255.255.0 | Switch0 Fa0/1 |
| PC1 | 192.168.1.11 | 255.255.255.0 | Switch1 Fa0/1 |

*Both PCs on same subnet – no router needed*

## Configuration Files

- [switch0-config.txt](switch0-config.txt) - LACP active, port-channel trunk
- [switch1-config.txt](switch1-config.txt) - LACP active, port-channel trunk

## Verification Results (All Passed)

| Test | Command | Result |
|------|---------|--------|
| EtherChannel summary (both ports) | `show etherchannel summary` | ✅ Gig0/1(P) Gig0/2(P) |
| Port-channel status | `show etherchannel port-channel` | ✅ Po1(SU) in use |
| Ping before failure | `ping 192.168.1.11 -n 4` | ✅ 4 replies, 0% loss |
| Ping during link failure | `ping 192.168.1.11 -n 1000` + shutdown Gig0/1 | ✅ No packet loss |
| EtherChannel after failure | `show etherchannel summary` | ✅ Gig0/1(D) Gig0/2(P) |
| Restored port | `no shutdown` on Gig0/1 | ✅ Gig0/1(P) returns |

## Screenshots

- [EtherChannel summary (both ports bundled)](screenshots/N04-etherchannel-summary.png)
- [Ping continuing after port shutdown](screenshots/N04-ping-during-failure.png)
- [EtherChannel after failure (one port down)](screenshots/N04-etherchannel-after-failure.png)

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| Ports show (I) instead of (P) | `show etherchannel summary` shows `(I)` = stand-alone | LACP mode mismatch. Set both switches to `active` |
| `interface range gig0/1-3` fails | `% Invalid input detected` at `3` | 2960 only has gig0/1 and gig0/2. Use `-2` |
| Ping fails after config | `Request timed out` | Forgot to assign IP addresses to PCs |
| No (P) on any port | Both ports show `(I)` | Forgot `no shutdown` on physical ports |
| Port-channel not showing SU | Shows `(SD)` instead of `(SU)` | Forgot `switchport mode trunk` on port-channel interface |
| Ping drops when port shutdown | 1-2 packets lost | Normal. Traffic shifts to remaining link, may drop 1 packet |

## What I'd Do Differently Next Time

- Use 3560 switches if I wanted 3 or 4 links (they have more Gigabit ports)
- Use LACP `passive` on one side and `active` on the other for more controlled negotiation
- Configure `channel-group 1 mode on` for static EtherChannel (no LACP negotiation) if both switches are Cisco
- Add load balancing configuration: `port-channel load-balance src-dst-ip` for better traffic distribution

## Key Commands Used

### Switch0 & Switch1 (LACP Active)
- `show etherchannel summary`
- `show etherchannel port-channel`
- `show interfaces status`
- `show running-config`

### Failure Testing
- `ping -n 1000` (continuous ping)
- `interface gigabitEthernet 0/1` → `shutdown`
- `no shutdown` (restore)

## What I Learned

- EtherChannel bundles physical links into one logical link. STP sees only the port-channel, not the individual links.
- LACP `active` mode initiates negotiation. Both sides need compatible modes (active/active or active/passive).
- The port-channel interface inherits configuration from physical ports. Best practice: configure trunk settings on the port-channel, not the individual ports.
- When a physical link fails, traffic shifts to remaining links with minimal to no packet loss. No 30-50 second STP reconvergence.
- Packet Tracer's 2960 model only has 2 Gigabit ports. Read the hardware specs before building topologies.

## Time to Complete

15 minutes
