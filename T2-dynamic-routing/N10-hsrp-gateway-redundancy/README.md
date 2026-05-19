# N10: HSRP Gateway Redundancy – Two Routers, One Virtual Gateway

## What This Proves

I can configure HSRP (Hot Standby Router Protocol) to provide first-hop redundancy for PCs on a LAN. R1 acts as the active gateway (priority 110), R2 acts as the standby (priority 90). If R1 fails, R2 automatically takes over the virtual IP address. The PCs keep working with zero configuration changes.

## Topology

- 2x Cisco 1941 routers (R1, R2)
- 1x Cisco 2960 switch (Switch0)
- 1x PC (PC1)
- All devices on same VLAN (default VLAN 1)
- HSRP group 1 with virtual IP 192.168.1.254

## IP Addressing Plan

| Device | Interface | IP Address | Subnet Mask | Default Gateway | HSRP Role | Priority |
|--------|-----------|------------|-------------|-----------------|-----------|----------|
| R1 | Gig0/0 | 192.168.1.1 | 255.255.255.0 | - | Active | 110 |
| R2 | Gig0/0 | 192.168.1.2 | 255.255.255.0 | - | Standby | 90 |
| PC1 | Fa0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.254 | - | - |

## HSRP Configuration Summary

| Router | Priority | Preempt | Virtual IP | Group | Role |
|--------|----------|---------|------------|-------|------|
| R1 | 110 | Yes | 192.168.1.254 | 1 | Active |
| R2 | 90 | No (default) | 192.168.1.254 | 1 | Standby |

## Configuration Files

- [R1-config.txt](R1-config.txt) - HSRP active, priority 110, preempt
- [R2-config.txt](R2-config.txt) - HSRP standby, priority 90
- [switch-config.txt](switch-config.txt) - Default VLAN 1 (no config needed)

## Step-by-Step Configuration

### 1. Configure Basic IP on R1

    enable
    configure terminal
    interface gigabitEthernet 0/0
    ip address 192.168.1.1 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 2. Configure HSRP on R1

    configure terminal
    interface gigabitEthernet 0/0
    standby 1 ip 192.168.1.254
    standby 1 priority 110
    standby 1 preempt
    exit
    end
    write memory

### 3. Configure Basic IP on R2

    enable
    configure terminal
    interface gigabitEthernet 0/0
    ip address 192.168.1.2 255.255.255.0
    no shutdown
    exit
    end
    write memory

### 4. Configure HSRP on R2

    configure terminal
    interface gigabitEthernet 0/0
    standby 1 ip 192.168.1.254
    standby 1 priority 90
    exit
    end
    write memory

### 5. Configure PC1

- IP: 192.168.1.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 192.168.1.254

## Verification Commands

    show standby
    show standby brief
    show ip interface brief

## Verification Results (All Passed)

| Test | Command | Expected Result |
|------|---------|-----------------|
| HSRP on R1 | `show standby` | ✅ State Active, priority 110, virtual IP 192.168.1.254 |
| HSRP on R2 | `show standby` | ✅ State Standby, priority 90, virtual IP 192.168.1.254 |
| PC ping to virtual gateway | `ping 192.168.1.254` | ✅ 4 replies, 0% loss |
| Failover | Shutdown R1 Gig0/0, `show standby` on R2 | ✅ R2 becomes Active |
| Recovery | `no shutdown` on R1 Gig0/0 | ✅ R1 preempts back to Active |

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| HSRP not forming | `show standby` showed Init state | Verified both routers had `no shutdown` on interfaces |
| Both routers showing Standby | No Active router | Checked that HSRP group numbers match (both using `standby 1`) |
| Priority not working | R2 became active instead of R1 | Verified R1 priority 110, R2 priority 90, added `preempt` on R1 |
| PC cannot ping virtual IP | `Request timed out` | Ensured PC gateway was set to 192.168.1.254, not 192.168.1.1 |
| Failover took too long | Multiple lost pings | Default timers (Hello 3 sec, Hold 10 sec) cause 10-15 second failover |
| R1 did not reclaim active role after recovery | R2 stayed Active | Added `standby 1 preempt` on R1 |

## What I'd Do Differently Next Time

- Use `standby 1 timers 1 3` to speed up failover (1 sec hello, 3 sec hold)
- Add a second PC for more realistic testing
- Use `standby 1 track` to track upstream interfaces (not covered in this lab)
- Document HSRP group number in the IP addressing plan

## Key Commands Used

### HSRP Configuration
- `standby 1 ip 192.168.1.254`
- `standby 1 priority 110`
- `standby 1 preempt`

### Verification
- `show standby`
- `show standby brief`
- `show ip interface brief`

## What I Learned

- HSRP allows two routers to share a virtual IP address. The active router handles traffic for that IP.
- Higher priority wins. Default priority is 100. R1 with 110 becomes active, R2 with 90 becomes standby.
- `preempt` allows a router with higher priority to reclaim active role after it recovers from a failure. Without preempt, the standby stays active.
- HSRP Hellos are sent every 3 seconds to multicast 224.0.0.2. Hold timer is 10 seconds.
- Failover takes about 10-15 seconds with default timers. PCs may lose 1-2 pings during failover.
- The virtual IP address must be in the same subnet as the physical interfaces.
- Both routers must be in the same HSRP group (group 1 in this lab).
- Switch does not need any special configuration. Default VLAN 1 works fine.

## Screenshots
- [Topology](screenshots/topology.png)
- [HSRP Active on R1](screenshots/hsrp-state-r1.png)
- [HSRP Standby on R2](screenshots/hsrp-state-r2.png)
- [Ping during failover](screenshots/ping-during-failover.png)

## Time to Complete

20 minutes
