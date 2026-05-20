# N15: SPAN Port + Wireshark

## What This Proves

I can configure a SPAN (Switched Port Analyzer) port on a Cisco switch to mirror traffic from source ports to a destination port for analysis with Wireshark. In a physical lab, this allows packet capture without disrupting live traffic. Due to lack of physical hardware, I demonstrate the same concepts using Packet Tracer Simulation Mode to capture and analyze protocol packets (STP, CDP, OSPF, ARP).

**Note:** This lab documents both the physical SPAN configuration syntax (for real switches) AND Packet Tracer Simulation Mode captures (for protocol analysis). Without physical Cisco devices, Wireshark .pcapng exports are not available; instead, screenshots from Simulation Mode serve as evidence.

## Topology (Simulation Mode)

- 1x Cisco 2960 switch (Switch0)
- 2x 1941 routers (R1, R2) for OSPF traffic generation
- Simulation Mode in Packet Tracer to view captured packets

## SPAN Configuration (Physical Switch - Reference Only)

monitor session 1 source interface gigabitEthernet 0/1 both
monitor session 1 source interface fastEthernet 0/1-24 both
monitor session 1 destination interface fastEthernet 0/24

## Captured Protocols in Simulation Mode

| Protocol | Filter | Frequency | Traffic Source |
|----------|--------|-----------|----------------|
| STP BPDUs | stp | Every 2 seconds | Switch itself |
| CDP | cdp | Every 60 seconds | Switch itself |
| OSPF Hellos | ospf | Every 10 seconds | OSPF routers |
| ARP | arp | On demand | Any device |

## Configuration Files

- [switch-config.txt](switch-config.txt) - SPAN configuration (physical switch reference)
- [screenshots/](screenshots/) - Screenshots from Simulation Mode

## Step-by-Step Guide (Simulation Mode)

### 1. Build Topology

1. Open Packet Tracer → File → New
2. Switches → drag 1x 2960 switch into workspace
3. Routers → drag 2x 1941 routers into workspace
4. Rename devices: Switch0, R1, R2
5. Connect: R1 Gig0/0 → Switch0 Gig0/1, R2 Gig0/0 → Switch0 Gig0/2
6. File → Save As → N15-span-wireshark.pkt

### 2. Configure OSPF on R1

    enable
    configure terminal
    interface loopback 0
    ip address 1.1.1.1 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 10.0.0.1 255.255.255.0
    no shutdown
    exit
    router ospf 1
    router-id 1.1.1.1
    network 10.0.0.0 0.0.0.255 area 0
    network 1.1.1.1 0.0.0.0 area 0
    end
    write memory

### 3. Configure OSPF on R2

    enable
    configure terminal
    interface loopback 0
    ip address 2.2.2.2 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 10.0.0.2 255.255.255.0
    no shutdown
    exit
    router ospf 1
    router-id 2.2.2.2
    network 10.0.0.0 0.0.0.255 area 0
    network 2.2.2.2 0.0.0.0 area 0
    end
    write memory

### 4. Capture STP BPDUs

1. Click Simulation mode
2. Edit Filters → check STP
3. Click Play
4. Click STP packet from Switch0 → expand Ethernet → LLC → STP

📸 Screenshot: stp-bpdu.png

### 5. Capture CDP Packets

1. Edit Filters → uncheck STP → check CDP
2. Click Play
3. Click CDP packet from Switch0 → expand to see CDP TLV details

📸 Screenshot: cdp-packet.png

### 6. Capture OSPF Hellos

1. Edit Filters → uncheck CDP → check OSPF
2. Click Play
3. Click OSPF packet between R1 and Switch0 → expand IP → OSPF

📸 Screenshot: ospf-hello.png

### 7. Capture ARP

1. On R1: clear arp-cache
2. Edit Filters → check ARP
3. On R1: ping 10.0.0.2
4. Click ARP packet → expand to see request/reply details

📸 Screenshot: arp-packet.png

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| No physical switch for SPAN | Cannot configure monitor session | Documented SPAN config syntax, used Simulation Mode instead |
| Cannot export .pcapng from PT | No export option | Used screenshots of Simulation Mode packets |
| STP packets not showing | Looking at router instead of switch | Clicked packets from Switch0 (outbound) |
| ARP not triggering | MAC already in cache | Used `clear arp-cache` before ping |

## Packet Tracer Limitations Encountered

| Limitation | Impact | Workaround |
|------------|--------|------------|
| No SPAN port configuration | Cannot test monitor session | Document syntax; use Simulation Mode |
| No .pcapng export | Cannot use Wireshark | Use screenshots |

## What I'd Do Differently Next Time

- Use physical Cisco switch with SPAN port and Wireshark for real .pcapng files
- Use GNS3/EVE-NG with Wireshark integration for virtual labs

## Key Commands (Physical Switch SPAN - Reference)

- monitor session 1 source interface gigabitEthernet 0/1 both
- monitor session 1 source interface fastEthernet 0/1-24 both
- monitor session 1 destination interface fastEthernet 0/24
- show monitor session 1

## What I Learned

- SPAN ports mirror traffic from source ports to a destination port for analysis
- Packet Tracer Simulation Mode is useful for protocol analysis despite .pcap limitations
- STP BPDUs are sent every 2 seconds from the switch (multicast 01:80:c2:00:00:00)
- CDP packets are sent every 60 seconds to multicast 01:00:0c:cc:cc:cc
- OSPF Hellos are sent every 10 seconds to 224.0.0.5
- ARP cache must be cleared to trigger new ARP requests

## Screenshots
- [Topology](screenshots/topology.png)
- [STP BPDU capture](screenshots/stp-bpdu.png)
- [CDP packet capture](screenshots/cdp-packet.png)
- [OSPF Hello capture](screenshots/ospf-hello.png)
- [ARP packet capture](screenshots/arp-packet.png)



## Time to Complete

30 minutes
