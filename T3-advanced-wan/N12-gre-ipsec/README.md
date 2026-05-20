# N12: GRE Tunnel + IPsec – Secure Site-to-Site VPN

## What This Proves

I can configure a GRE tunnel between two routers (R1 and R2) to carry traffic across a simulated Internet connection. The GRE tunnel encapsulates packets, allowing routing between the two sites. IPsec encryption was attempted but could not be completed due to Packet Tracer limitations. The GRE tunnel itself is fully functional.

**Note:** Packet Tracer does not support applying crypto maps to GRE tunnel interfaces in many versions. IPsec encryption could not be configured in this lab environment. The GRE tunnel itself is functional. For full IPsec VPN labs, use GNS3, EVE-NG, or CML.

## Topology

- 2x Cisco 2901 routers (R1, R2)
- Simulated Internet connection between R1 and R2 (direct link)
- GRE Tunnel (Tunnel0) between R1 and R2
- Loopback interfaces to simulate LAN subnets on each side

## IP Addressing Plan

| Device | Interface | IP Address | Subnet Mask | Purpose |
|--------|-----------|------------|-------------|---------|
| R1 | Loopback0 | 10.1.1.1 | 255.255.255.255 | Simulated HQ LAN |
| R1 | Gig0/0 | 203.0.113.1 | 255.255.255.252 | Physical (simulated Internet) |
| R1 | Tunnel0 | 10.0.0.1 | 255.255.255.252 | GRE tunnel |
| R2 | Loopback0 | 10.2.2.2 | 255.255.255.255 | Simulated Branch LAN |
| R2 | Gig0/0 | 203.0.113.2 | 255.255.255.252 | Physical (simulated Internet) |
| R2 | Tunnel0 | 10.0.0.2 | 255.255.255.252 | GRE tunnel |

## GRE Tunnel Configuration Summary

| Router | Tunnel Source | Tunnel Destination | Tunnel IP |
|--------|---------------|-------------------|-----------|
| R1 | 203.0.113.1 | 203.0.113.2 | 10.0.0.1/30 |
| R2 | 203.0.113.2 | 203.0.113.1 | 10.0.0.2/30 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - GRE tunnel configuration
- [R2-config.txt](R2-config.txt) - GRE tunnel configuration

## Step-by-Step Configuration

### 1. Configure Basic IP on R1

    enable
    configure terminal
    interface loopback 0
    ip address 10.1.1.1 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 203.0.113.1 255.255.255.252
    no shutdown
    exit
    ip route 0.0.0.0 0.0.0.0 203.0.113.2
    end
    write memory

### 2. Configure Basic IP on R2

    enable
    configure terminal
    interface loopback 0
    ip address 10.2.2.2 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 203.0.113.2 255.255.255.252
    no shutdown
    exit
    ip route 0.0.0.0 0.0.0.0 203.0.113.1
    end
    write memory

### 3. Verify Physical Connectivity

On R1:
    ping 203.0.113.2

Expected: 5 replies, 0% loss

### 4. Configure GRE Tunnel on R1

    configure terminal
    interface tunnel 0
    ip address 10.0.0.1 255.255.255.252
    tunnel source gigabitEthernet 0/0
    tunnel destination 203.0.113.2
    tunnel mode gre ip
    no shutdown
    exit
    end
    write memory

### 5. Configure GRE Tunnel on R2

    configure terminal
    interface tunnel 0
    ip address 10.0.0.2 255.255.255.252
    tunnel source gigabitEthernet 0/0
    tunnel destination 203.0.113.1
    tunnel mode gre ip
    no shutdown
    exit
    end
    write memory

### 6. Verify GRE Tunnel

On R1:
    show interface tunnel 0

Expected: Tunnel0 is up, line protocol is up

### 7. Test Tunnel Connectivity

On R1:
    ping 10.0.0.2

Expected: 5 replies, 0% loss

### 8. Test Across Tunnel (Simulated LANs)

On R1:
    ping 10.2.2.2 source 10.1.1.1

Expected: 5 replies, 0% loss

## Verification Commands

    show interface tunnel 0
    show ip route
    ping 10.0.0.2
    ping 10.2.2.2 source 10.1.1.1

## Verification Results (All Passed for GRE Tunnel)

| Test | Command | Expected Result |
|------|---------|-----------------|
| Tunnel interface | `show interface tunnel 0` | ✅ up/up |
| Tunnel connectivity | `ping 10.0.0.2` | ✅ 5 replies, 0% loss |
| Across-tunnel ping | `ping 10.2.2.2 source 10.1.1.1` | ✅ 5 replies, 0% loss |

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| Security license not active | `show version` showed security: disable | Added `license boot module c2900 technology-package securityk9` and reloaded |
| `hash sha256` not supported | `% Invalid input detected at '^' marker` | Changed to `hash sha` (SHA-1) |
| `group 14` not supported | `% Invalid input detected at '^' marker` | Changed to `group 2` (DH Group 2) |
| Crypto map on tunnel not supported | `% Invalid input detected at '^' marker` | Packet Tracer limitation - accepted GRE-only tunnel |
| `show license` incomplete command | `% Incomplete command` | Used `show version` to verify license instead |
| Pipe `|` not working | `% Invalid input detected` | Manually scrolled through `show running-config` |

## Packet Tracer Limitations Encountered

| Limitation | Impact | Workaround |
|------------|--------|------------|
| `crypto map` on tunnel interface not supported | IPsec cannot be applied to GRE tunnel | Use GNS3/EVE-NG for full IPsec VPN labs |
| `hash sha256` not supported | Must use SHA-1 instead | Used `hash sha` |
| `group 14` not supported | Must use lower DH group | Used `group 2` |
| `show license` command incomplete | Cannot view license status directly | Used `show version` to verify |
| Pipe `|` filtering not supported | Cannot filter command output | Manually scroll through output |

## What I'd Do Differently Next Time

- Use GNS3 or EVE-NG for full IPsec VPN labs (these support crypto maps on tunnel interfaces)
- Use a simulator with full IOS feature support
- Test crypto commands in a different environment before committing to Packet Tracer

## Key Commands Used

### GRE Tunnel Configuration
- `interface tunnel 0`
- `ip address 10.0.0.1 255.255.255.252`
- `tunnel source gigabitEthernet 0/0`
- `tunnel destination 203.0.113.2`
- `tunnel mode gre ip`
- `no shutdown`

### Verification
- `show interface tunnel 0`
- `show ip route`
- `ping 10.0.0.2`

## What I Learned

- GRE tunnels encapsulate packets inside IP packets, allowing routing between separate networks over a single physical link.
- The tunnel source and destination are the physical interface IP addresses. The tunnel itself has its own IP address space.
- GRE tunnels do not encrypt traffic by default. Anyone on the path can see the encapsulated packets.
- Packet Tracer has significant limitations for security features like IPsec crypto maps on tunnel interfaces.
- Security licenses on 2901 routers require the `license boot module c2900 technology-package securityk9` command followed by a reload.
- In Packet Tracer, supported ISAKMP parameters are limited to `hash sha` (not sha256) and `group 2` (not group 14).
- Even with an active security license, `crypto map` on tunnel interface is not supported in Packet Tracer.
- For full VPN labs (GRE+IPsec), a different simulator (GNS3, EVE-NG, CML) is required.

## Screenshots
- [Topology](screenshots/topology.png)
- [GRE Tunnel interface up/up](screenshots/tunnel-interface.png)
- [Ping across GRE tunnel](screenshots/tunnel-ping.png)
- [Show IP route with tunnel](screenshots/show-ip-route-tunnel.png)


## Time to Complete

90 minutes (including troubleshooting security license and Packet Tracer limitations)
