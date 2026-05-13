markdown

# Project N04: EtherChannel (LACP)

## What This Proves
I can bundle multiple physical links into a single logical link for redundancy and increased bandwidth using LACP.

## Topology
- **2x Cisco 2960 Switches** (connected by 2 parallel Gigabit links)
- **2x PCs** (one per switch for testing)

## Design Goal
- Bundle Gig0/1 and Gig0/2 into Port-Channel 1 using LACP mode `active`
- Configure port-channel as trunk
- Verify fault tolerance by shutting down one physical link

## IP Addressing Plan
| Device | Interface | IP Address | Subnet Mask |
|--------|-----------|------------|-------------|
| PC0 | Fa0 | 192.168.1.10 | 255.255.255.0 |
| PC1 | Fa0 | 192.168.1.11 | 255.255.255.0 |

*Both PCs on same subnet – no router needed*

## Step-by-Step Configuration

### 1. Configure EtherChannel on Switch0

Switch0> enable
Switch0# configure terminal
Switch0(config)# interface range gigabitEthernet 0/1-2
Switch0(config-if-range)# channel-group 1 mode active
Switch0(config-if-range)# no shutdown
Switch0(config-if-range)# exit
Switch0(config)# interface port-channel 1
Switch0(config-if)# switchport mode trunk
Switch0(config-if)# exit
Switch0(config)# end
Switch0# write memory
text


### 2. Configure EtherChannel on Switch1

Switch1> enable
Switch1# configure terminal
Switch1(config)# interface range gigabitEthernet 0/1-2
Switch1(config-if-range)# channel-group 1 mode active
Switch1(config-if-range)# no shutdown
Switch1(config-if-range)# exit
Switch1(config)# interface port-channel 1
Switch1(config-if)# switchport mode trunk
Switch1(config-if)# exit
Switch1(config)# end
Switch1# write memory
text


## Verification Commands

Switch0# show etherchannel summary
text


## Expected Output

Group Port-channel Protocol Ports
1 Po1(SU) LACP Gig0/1(P) Gig0/2(P)
text


## Failure Tolerance Test

! On PC0
ping 192.168.1.11 -n 1000

! On Switch0 while ping runs
Switch0# configure terminal
Switch0(config)# interface gigabitEthernet 0/1
Switch0(config-if)# shutdown
text


## Packet Tracer Note
Cisco 2960 switches only have 2 Gigabit ports (Gig0/1 and Gig0/2). The original project called for 3 links, but use 2 links – EtherChannel works with 2 or more ports.

## Troubleshooting
| Problem | Solution |
|---------|----------|
| Ports show (I) instead of (P) | LACP mode mismatch – ensure both switches use `active` |
| `interface range gig0/1-3` fails | 2960 only has gig0/1 and gig0/2. Use `-2` |
| Ping fails | Verify IP addresses on PCs |
| No (P) on any port | Check cables and `no shutdown` on physical ports |

## Files in This Folder
| File | Purpose |
|------|---------|
| `N04-etherchannel.pkt` | Packet Tracer topology |
| `switch0-config.txt` | Switch0 running config |
| `switch1-config.txt` | Switch1 running config |
| `screenshots/N04-etherchannel-summary.png` | Both ports bundled |
| `screenshots/N04-ping-during-failure.png` | Ping continues after shutdown |
| `screenshots/N04-etherchannel-after-failure.png` | One port shows (D) |

## Time to Complete
15 minutes
