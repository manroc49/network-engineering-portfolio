# N05: Static Routing – Sometimes You Just Need To Tell The Router Where To Go

## What This Proves

I can manually configure routes on three routers in a chain so they can reach each other's remote LANs. No dynamic routing protocols. Just `ip route` commands pointing to next-hop addresses. When PC1 sends a ping to PC3, the packet crosses R1 → R2 → R3 and the replies come back the same way.

## Topology

- 3x Cisco 1941 routers (R1, R2, R3 in a chain)
- 2x PCs (PC1 on R1, PC3 on R3)
- Point-to-point links between routers (/30 subnets)

**Note:** The original project had a PC2 connected to R2. I removed it. It served no purpose since the test is PC1 to PC3 across all three routers.

## IP Addressing Plan

| Device | Interface | IP Address | Subnet Mask | Connected To |
|--------|-----------|------------|-------------|--------------|
| R1 | Gig0/0 | 192.168.1.1 | 255.255.255.0 | PC1 |
| R1 | Gig0/1 | 10.0.12.1 | 255.255.255.252 | R2 Gig0/0 |
| R2 | Gig0/0 | 10.0.12.2 | 255.255.255.252 | R1 Gig0/1 |
| R2 | Gig0/1 | 10.0.23.1 | 255.255.255.252 | R3 Gig0/0 |
| R3 | Gig0/0 | 10.0.23.2 | 255.255.255.252 | R2 Gig0/1 |
| R3 | Gig0/1 | 192.168.3.1 | 255.255.255.0 | PC3 |
| PC1 | Fa0 | 192.168.1.10 | 255.255.255.0 | R1 Gig0/0 |
| PC3 | Fa0 | 192.168.3.10 | 255.255.255.0 | R3 Gig0/1 |

## Static Routes Configured

| Router | Destination Network | Next-Hop |
|--------|---------------------|----------|
| R1 | 192.168.3.0/24 | 10.0.12.2 |
| R2 | 192.168.1.0/24 | 10.0.12.1 |
| R2 | 192.168.3.0/24 | 10.0.23.2 |
| R3 | 192.168.1.0/24 | 10.0.23.1 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - Static route to R3 LAN
- [R2-config.txt](R2-config.txt) - Static routes to both LANs
- [R3-config.txt](R3-config.txt) - Static route to R1 LAN

## Verification Results (All Passed)

| Test | Command | Result |
|------|---------|--------|
| Directly connected routes on R1 | `show ip route` | ✅ 192.168.1.0/24 and 10.0.12.0/30 |
| Directly connected routes on R2 | `show ip route` | ✅ 10.0.12.0/30 and 10.0.23.0/30 |
| Directly connected routes on R3 | `show ip route` | ✅ 10.0.23.0/30 and 192.168.3.0/24 |
| Static route on R1 | `show ip route` | ✅ S 192.168.3.0/24 via 10.0.12.2 |
| Static routes on R2 | `show ip route` | ✅ S 192.168.1.0/24 via 10.0.12.1 and S 192.168.3.0/24 via 10.0.23.2 |
| Static route on R3 | `show ip route` | ✅ S 192.168.1.0/24 via 10.0.23.1 |
| PC1 ping gateway | `ping 192.168.1.1` | ✅ 4 replies, 0% loss |
| PC3 ping gateway | `ping 192.168.3.1` | ✅ 4 replies, 0% loss |
| Ping across all routers | `ping 192.168.3.10` from PC1 | ✅ 4 replies, 0% loss |

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| Interfaces on wrong ports | R1 had 192.168.1.1 on Gig0/1 instead of Gig0/0 | Reconfigured interfaces with correct IPs on correct ports |
| Interface administratively down | `show ip interface brief` showed `down` | Added `no shutdown` to the interface |
| PC3 had wrong IP address | PC1 could not ping PC3, PC3 could not ping its gateway | Changed PC3 IP from 192.168.1.10 to 192.168.3.10 |
| Missing static route on R2 | PC1 ping to PC3 failed | Added `ip route 192.168.3.0 255.255.255.0 10.0.23.2` on R2 |
| Missing return path static route | Ping went one way but replies failed | Added `ip route 192.168.1.0 255.255.255.0 10.0.23.1` on R3 |

## What I'd Do Differently Next Time

- Use a /31 subnet for point-to-point links (saves IP address space)
- Add a default route on R2 instead of two static routes (less config)
- Use `ping` with source option to test specific paths: `ping 10.0.23.2 source 192.168.1.1`
- Document IP addresses in a table BEFORE configuring anything

## Key Commands Used

### Basic IP Configuration
- `interface gigabitEthernet 0/0`
- `ip address 192.168.1.1 255.255.255.0`
- `no shutdown`

### Static Routes
- `ip route 192.168.3.0 255.255.255.0 10.0.12.2`

### Verification
- `show ip route`
- `show ip interface brief`
- `ping`

## What I Learned

- Static routes are manual entries in the routing table. Routers don't learn remote networks automatically.
- The syntax is `ip route destination_network subnet_mask next_hop`
- Each router needs to know about remote networks AND how to get back. For PC1 to ping PC3, R3 needs a route back to 192.168.1.0/24. Without it, the ping goes one way but replies never come back.
- `show ip route` shows `C` for directly connected, `L` for local host routes, and `S` for static routes.
- The `[1/0]` in static route output is administrative distance/metric. 1 is the default AD for static routes.
- Always check PC IP addresses first. I wasted 10 minutes troubleshooting routing when PC3 was on the wrong subnet.

## Screenshots
- [Topology](screenshots/topology.png)
- [Static routes on R1](screenshots/show-ip-route-r1.png)
- [Static routes on R2](screenshots/show-ip-route-r2.png)
- [PC1 pinging PC3 across all routers](screenshots/ping-across-routers.png)

## Files in This Folder

| File | Purpose |
|------|---------|
| `N05-static-routing.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config |
| `R2-config.txt` | R2 running config |
| `R3-config.txt` | R3 running config |
| `screenshots/show-ip-route-r1.png` | Static route on R1 |
| `screenshots/show-ip-route-r2.png` | Static routes on R2 |
| `screenshots/ping-across-routers.png` | Successful ping across all routers |

## Time to Complete

25 minutes (including troubleshooting a PC3 IP typo)
