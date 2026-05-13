# N05: Static Routing

## What This Proves

I can manually configure routes on three routers in a chain so they can reach each other's remote LANs. No dynamic routing protocols. Just `ip route` commands pointing to next-hop addresses.

## Topology

- 3x Cisco 1941 routers (R1, R2, R3 in a chain)
- 2x PCs (PC1 on R1, PC3 on R3)
- Point-to-point links between routers (/30 subnets)

**Note:** PC2 was removed from the original topology. It served no purpose since the test is PC1 to PC3 across all three routers.

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

## Step-by-Step Configuration

### 1. Configure Basic IP on R1

Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# ip address 10.0.12.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# end
R1# write memory
text


### 2. Configure Basic IP on R2

Router> enable
Router# configure terminal
Router(config)# hostname R2
R2(config)# interface gigabitEthernet 0/0
R2(config-if)# ip address 10.0.12.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface gigabitEthernet 0/1
R2(config-if)# ip address 10.0.23.1 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# end
R2# write memory
text


### 3. Configure Basic IP on R3

Router> enable
Router# configure terminal
Router(config)# hostname R3
R3(config)# interface gigabitEthernet 0/0
R3(config-if)# ip address 10.0.23.2 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# exit
R3(config)# interface gigabitEthernet 0/1
R3(config-if)# ip address 192.168.3.1 255.255.255.0
R3(config-if)# no shutdown
R3(config-if)# end
R3# write memory
text


### 4. Configure Static Routes on R1

R1# configure terminal
R1(config)# ip route 192.168.3.0 255.255.255.0 10.0.12.2
R1(config)# end
R1# write memory
text


### 5. Configure Static Routes on R2

R2# configure terminal
R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.12.1
R2(config)# ip route 192.168.3.0 255.255.255.0 10.0.23.2
R2(config)# end
R2# write memory
text


### 6. Configure Static Routes on R3

R3# configure terminal
R3(config)# ip route 192.168.1.0 255.255.255.0 10.0.23.1
R3(config)# end
R3# write memory
text


### 7. Configure PCs
- **PC1:** IP 192.168.1.10, Mask 255.255.255.0, Gateway 192.168.1.1
- **PC3:** IP 192.168.3.10, Mask 255.255.255.0, Gateway 192.168.3.1

## Verification Commands

show ip route
show ip route static
ping 192.168.3.10
text


## Expected Results (Placeholder)

| Test | Command | Expected Result |
|------|---------|-----------------|
| Ping across routers | `ping 192.168.3.10` from PC1 | 4 replies, 0% loss |
| Static route on R1 | `show ip route` on R1 | Shows `S 192.168.3.0/24 [1/0] via 10.0.12.2` |
| Static routes on R2 | `show ip route` on R2 | Shows both static routes |

## Files in This Folder

| File | Purpose |
|------|---------|
| `N05-static-routing.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config |
| `R2-config.txt` | R2 running config |
| `R3-config.txt` | R3 running config |
| `screenshots/ping-across-routers.png` | PC1 pinging PC3 |
| `screenshots/show-ip-route-r1.png` | Static routes on R1 |
| `screenshots/show-ip-route-r2.png` | Static routes on R2 |

## Time to Complete (Estimated)

20 minutes
