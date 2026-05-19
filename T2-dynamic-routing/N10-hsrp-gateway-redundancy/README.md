# N10: HSRP Gateway Redundancy – Two Routers, One Virtual Gateway

## What This Proves

I can configure HSRP (Hot Standby Router Protocol) to provide first-hop redundancy for PCs on a LAN. R1 acts as the active gateway (priority 110), R2 acts as the standby (priority 90). If R1 fails, R2 automatically takes over the virtual IP address. The PCs keep working with zero configuration changes.

## Topology

- 2x Cisco 1941 routers (R1, R2)
- 1x Cisco 2960 switch (Switch0)
- 1-2x PCs (PC1, PC2)
- All devices on same VLAN (default VLAN 1)
- HSRP group 1 with virtual IP 192.168.1.254

## IP Addressing Plan

| Device | Interface | IP Address | Subnet Mask | Default Gateway | HSRP Role | Priority |
|--------|-----------|------------|-------------|-----------------|-----------|----------|
| R1 | Gig0/0 | 192.168.1.1 | 255.255.255.0 | - | Active | 110 |
| R2 | Gig0/0 | 192.168.1.2 | 255.255.255.0 | - | Standby | 90 |
| PC1 | Fa0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.254 | - | - |
| PC2 (optional) | Fa0 | 192.168.1.11 | 255.255.255.0 | 192.168.1.254 | - | - |

## HSRP Configuration

| Router | Priority | Preempt | Virtual IP | Group |
|--------|----------|---------|------------|-------|
| R1 | 110 | Yes | 192.168.1.254 | 1 |
| R2 | 90 | No (default) | 192.168.1.254 | 1 |

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

### 5. Configure PCs

PC1:
- IP: 192.168.1.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 192.168.1.254

PC2 (optional):
- IP: 192.168.1.11
- Subnet Mask: 255.255.255.0
- Default Gateway: 192.168.1.254

### 6. Verify HSRP Status

On R1:
    show standby

Expected: R1 shows state Active, priority 110

On R2:
    show standby

Expected: R2 shows state Standby, priority 90

## Verification Commands

    show standby
    show standby brief
    show ip interface brief

## Expected Results (Placeholder)

| Test | Command | Expected Result |
|------|---------|-----------------|
| HSRP on R1 | `show standby` | State Active, priority 110 |
| HSRP on R2 | `show standby` | State Standby, priority 90 |
| PC ping to gateway | `ping 192.168.1.254` | 5 replies, 0% loss |
| Failover | Shutdown R1 Gig0/0, then `show standby` on R2 | R2 becomes Active |
| Recovery | `no shutdown` on R1 Gig0/0 | R1 preempts back to Active |

## Files in This Folder

| File | Purpose |
|------|---------|
| `N10-hsrp-gateway-redundancy.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config (Active, priority 110, preempt) |
| `R2-config.txt` | R2 running config (Standby, priority 90) |
| `switch-config.txt` | Switch0 running config (default) |
| `screenshots/hsrp-state-r1.png` | `show standby` on R1 showing Active |
| `screenshots/hsrp-state-r2.png` | `show standby` on R2 showing Standby |
| `screenshots/ping-during-failover.png` | Ping continues during failover |

## Time to Complete (Estimated)

20 minutes
