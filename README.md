# Network Engineering Portfolio – Layers 1-3


Network engineering focused on routing, switching, and protocol analysis. This portfolio demonstrates 14 hands-on projects from VLANs to BGP, all documented with Packet Tracer topologies, configuration files, and Wireshark captures.

## What This Repo Contains

**15 complete projects across three skill levels:**

| Level | Focus | Projects |
|-------|-------|----------|
| **T1: Fundamentals** | VLANs, trunking, STP, EtherChannel, static routing | N01-N05 |
| **T2: Dynamic Routing** | OSPF, EIGRP, route redistribution, HSRP | N06-N10 |
| **T3: Advanced WAN** | BGP, GRE/IPsec, QoS, VRF, SPAN/Wireshark | N11-N14 |

## Project Index

### T1: Fundamentals: Layer 2 & Static Routing

| # | Project | What I Proved | Folder |
|---|---------|---------------|--------|
| N01 | VLAN Segmentation | Segmented /24 into VLANs 10,20,30, inter-VLAN routing via router-on-a-stick | [/T1-fundamentals/N01-vlans/](T1-fundamentals/N01-vlans/) |
| N02 | 802.1Q Trunking & VTP | Trunk between 3 switches, VTP server/client propagating VLANs | [/T1-fundamentals/N02-trunking-vtp/](T1-fundamentals/N02-trunking-vtp/) |
| N03 | Spanning Tree (PVST+) | Root bridge election, port priorities, 6-second convergence | [/T1-fundamentals/N03-spanning-tree/](T1-fundamentals/N03-spanning-tree/) |
| N04 | EtherChannel (LACP) | Bundled 3 ports into single logical link with load balancing | [/T1-fundamentals/N04-etherchannel/](T1-fundamentals/N04-etherchannel/) |
| N05 | Static Routing | Configured static routes across 3 routers, default routes, recursive lookups | [/T1-fundamentals/N05-static-routing/](T1-fundamentals/N05-static-routing/) |

### T2: Dynamic Routing

| # | Project | What I Proved | Folder |
|---|---------|---------------|--------|
| N06 | OSPF Single Area | OSPF neighbors, LSDB exchange, SPF calculation | [/T2-dynamic-routing/N06-ospf-single-area/](T2-dynamic-routing/N06-ospf-single-area/) |
| N07 | OSPF Multi-Area | Area 0,1,2 with ABR, route summarization, LSA types | [/T2-dynamic-routing/N07-ospf-multi-area/](T2-dynamic-routing/N07-ospf-multi-area/) |
| N08 | EIGRP Load Balancing | Unequal-cost load balancing with variance 2, feasible successor | [/T2-dynamic-routing/N08-eigrp-load-balancing/](T2-dynamic-routing/N08-eigrp-load-balancing/) |
| N09 | Route Redistribution | OSPF ↔ EIGRP redistribution, seed metrics, route filtering | [/T2-dynamic-routing/N09-route-redistribution/](T2-dynamic-routing/N09-route-redistribution/) |
| N10 | HSRP Gateway Redundancy | Active/Standby gateway with preemption, 99.9% availability | [/T2-dynamic-routing/N10-hsrp-gateway-redundancy/](T2-dynamic-routing/N10-hsrp-gateway-redundancy/) |

### T3: Advanced WAN

| # | Project | What I Proved | Folder |
|---|---------|---------------|--------|
| N11 | BGP Basics (eBGP/iBGP) | AS numbers, neighbor relationships, path selection (weight, local-pref, AS-path) | [/T3-advanced-wan/N11-bgp-basics/](T3-advanced-wan/N11-bgp-basics/) |
| N12 | GRE Tunnels + IPsec | Tunnel interfaces, encapsulation, crypto maps for site-to-site VPN | [/T3-advanced-wan/N12-gre-ipsec/](T3-advanced-wan/N12-gre-ipsec/) |
| N13 | QoS (CBWFQ, LLQ) | Class maps, policy maps, bandwidth guarantees, priority queuing | [/T3-advanced-wan/N13-qos/](T3-advanced-wan/N13-qos/) |
| N14 | VRF Lite | Multiple routing tables on single router, route isolation | [/T3-advanced-wan/N14-vrf-lite/](T3-advanced-wan/N14-vrf-lite/) |
| N15 | SPAN Port + Wireshark | Mirrored switch port traffic, captured BPDUs, CDP, VTP, OSPF hellos | [/T3-advanced-wan/N15-span-port-wireshark/](T3-advanced-wan/N15-span-port-wireshark/) |

## How Hiring Managers Can Verify My Work

| Path | Time | What You'll See |
|------|------|-----------------|
| **Quickest** (screenshots) | 30 sec | Annotated `show` command outputs |
| **Medium** (open .pkt in Packet Tracer) | 2 min | Live topology with working configs |
| **Deep** (Wireshark captures) | 5 min | Protocol analysis of OSPF, EIGRP, BGP, STP |
| **Full reproduction** | 15 min/project | Re-run configs in your own lab |

## Technologies Demonstrated

| Category | Technologies |
|----------|--------------|
| **Layer 2** | VLANs, 802.1Q Trunking, VTP, STP (PVST+), EtherChannel (LACP), CDP |
| **Layer 3** | Static Routing, OSPF (single/multi-area), EIGRP, BGP (eBGP/iBGP), Route Redistribution, HSRP, VRF Lite |
| **WAN/VPN** | GRE Tunnels, IPsec (crypto maps, ISAKMP, ESP/AH) |
| **QoS** | CBWFQ, LLQ, Class Maps, Policy Maps, Bandwidth/Police/Priority |
| **Troubleshooting** | SPAN Ports, Wireshark (BPDUs, CDP, VTP, OSPF hellos, EIGRP hellos, BGP updates) |
| **Tools** | Cisco Packet Tracer, GNS3, Wireshark, CLI |

## Cost

All projects run on **free software** (Cisco Packet Tracer + Wireshark). No cloud costs.
