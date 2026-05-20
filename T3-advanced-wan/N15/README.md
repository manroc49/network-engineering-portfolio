# N15: SPAN Port + Wireshark – Capturing Traffic for Analysis

## What This Proves

I can configure a SPAN (Switched Port Analyzer) port on a Cisco switch to mirror traffic from source ports to a destination port for analysis with Wireshark. In a physical lab, this allows packet capture without disrupting live traffic. Due to lack of physical hardware, I demonstrate the same concepts using Packet Tracer Simulation Mode to capture and analyze protocol packets (STP, CDP, OSPF, EIGRP, HSRP, ARP, DHCP).

**Note:** This lab documents both the physical SPAN configuration syntax (for real switches) AND Packet Tracer Simulation Mode captures (for protocol analysis). Without physical Cisco devices, Wireshark .pcapng exports are not available; instead, screenshots from Simulation Mode serve as evidence.

## Topology (Simulation Mode)

- 1x Cisco 2960 switch (Switch0)
- 2x Routers or PCs to generate traffic (e.g., R1, R2 for OSPF/EIGRP)
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
| EIGRP Hellos | eigrp | Every 5 seconds | EIGRP routers |
| HSRP | hsrp | Every 3 seconds | HSRP routers |
| ARP | arp | On demand | Any device |
| DHCP | bootp | On request | DHCP client/server |

## Configuration Files

- [switch-config.txt](switch-config.txt) - SPAN configuration (physical switch reference)
- [packet-captures/](packet-captures/) - Screenshots from Simulation Mode

## Step-by-Step Guide (Simulation Mode)

### Step 1: Build Topology

1. Open Packet Tracer → File → New
2. Switches → drag 1x 2960 switch into workspace
3. Routers → drag 2x routers (1941 or 2811) for OSPF/EIGRP traffic
4. Connect:
   - R1 Gig0/0 → Switch0 Gig0/1
   - R2 Gig0/0 → Switch0 Gig0/2
5. File → Save As → N15-span-wireshark.pkt

### Step 2: Configure OSPF on Routers (to generate OSPF Hellos)

On R1:
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

On R2:
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

### Step 3: Capture STP BPDUs

1. Click **Simulation** mode (bottom right corner)
2. Click **Edit Filters** → uncheck everything → check **STP**
3. Click **Play** (triangle button)
4. Watch for STP BPDU packets (every 2 seconds)
5. Click any STP packet → **OSPF Model** tab → **In Layers** section
6. Expand **Ethernet** → **LLC** → **STP** to see BPDU details

📸 **Screenshot 1:** `stp-bpdu.png`

### Step 4: Capture CDP Packets

1. Click **Edit Filters** → uncheck STP → check **CDP**
2. Click **Play**
3. Watch for CDP packets (every 60 seconds)
4. Click any CDP packet → expand to see CDP TLV details

📸 **Screenshot 2:** `cdp-packet.png`

### Step 5: Capture OSPF Hellos

1. Click **Edit Filters** → uncheck CDP → check **OSPF**
2. Click **Play**
3. Watch for OSPF Hello packets (destination 224.0.0.5)
4. Click any OSPF Hello → expand **IP** → **OSPF** to see Hello details

📸 **Screenshot 3:** `ospf-hello.png`

### Step 6: Capture EIGRP Hellos (Optional - configure EIGRP instead of OSPF)

1. Replace OSPF with EIGRP on routers
2. Click **Edit Filters** → check **EIGRP**
3. Click **Play**
4. Capture EIGRP Hello packet

📸 **Screenshot 4:** `eigrp-hello.png`

### Step 7: Capture HSRP (Optional - configure HSRP between two routers)

1. Configure HSRP on R1 and R2 (see N10 for config)
2. Click **Edit Filters** → check **HSRP**
3. Click **Play**
4. Capture HSRP packet

📸 **Screenshot 5:** `hsrp-packet.png`

### Step 8: Capture ARP

1. On any PC/router, `ping` an IP on the same subnet
2. Click **Edit Filters** → check **ARP**
3. Click **Play**
4. Capture ARP request/reply

📸 **Screenshot 6:** `arp-packet.png`

### Step 9: Capture DHCP (Optional - add DHCP server and client)

1. Add DHCP server and client to topology
2. Click **Edit Filters** → check **UDP** or **DHCP**
3. Click **Play** during DHCP discovery
4. Capture DHCP packet

📸 **Screenshot 7:** `dhcp-packet.png`

## Verification Commands (Physical Switch SPAN - Reference)

    show monitor session 1
    show port-security

## Expected Results

| Protocol | Captured | Screenshot |
|----------|----------|------------|
| STP BPDUs | ✅ | `stp-bpdu.png` |
| CDP | ✅ | `cdp-packet.png` |
| OSPF Hellos | ✅ | `ospf-hello.png` |
| EIGRP Hellos | ✅ (optional) | `eigrp-hello.png` |
| HSRP | ✅ (optional) | `hsrp-packet.png` |
| ARP | ✅ | `arp-packet.png` |
| DHCP | ✅ (optional) | `dhcp-packet.png` |

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| No physical switch for SPAN | Cannot configure `monitor session` | Documented SPAN config syntax, used Simulation Mode instead |
| Cannot export .pcapng from PT | No export option | Used screenshots of Simulation Mode packets |
| Filter menu shows limited protocols | EIGRP, HSRP not always visible | Configured OSPF instead; documented limitation |
| Traffic not appearing | No devices generating protocol traffic | Added OSPF between routers to generate Hellos |

## Packet Tracer Limitations Encountered

| Limitation | Impact | Workaround |
|------------|--------|------------|
| No SPAN port configuration | Cannot test `monitor session` | Document syntax; use Simulation Mode |
| No .pcapng export | Cannot use Wireshark | Use screenshots |
| Limited protocol filters | EIGRP/HSRP may not appear | Use OSPF as primary example |

## What I'd Do Differently Next Time

- Use physical Cisco switch with SPAN port and Wireshark for real .pcapng files
- Use GNS3/EVE-NG with Wireshark integration for virtual labs
- Generate more diverse traffic (HTTP, FTP) for analysis

## Key Commands (Physical Switch SPAN - Reference)

- `monitor session 1 source interface gigabitEthernet 0/1 both`
- `monitor session 1 source interface fastEthernet 0/1-24 both`
- `monitor session 1 destination interface fastEthernet 0/24`
- `show monitor session 1`

## What I Learned

- SPAN ports mirror traffic from source ports to a destination port for analysis
- The destination port should be connected to a device running Wireshark
- SPAN can monitor ingress (rx), egress (tx), or both directions
- Packet Tracer Simulation Mode is useful for protocol analysis despite .pcap limitations
- STP BPDUs are sent every 2 seconds by default (multicast 01:80:c2:00:00:00)
- CDP packets are sent every 60 seconds to multicast 01:00:0c:cc:cc:cc
- OSPF Hellos are sent every 10 seconds to 224.0.0.5
- Each protocol has predictable packet timing, making capture easier

## Screenshots (Simulation Mode Captures)

- [STP BPDU capture](screenshots/stp-bpdu.png)
- [CDP packet capture](screenshots/cdp-packet.png)
- [OSPF Hello capture](screenshots/ospf-hello.png)
- [ARP packet capture](screenshots/arp-packet.png)

## Files in This Folder

| File | Purpose |
|------|---------|
| `N15-span-wireshark.pkt` | Packet Tracer topology |
| `switch-config.txt` | SPAN configuration (reference) |
| `screenshots/stp-bpdu.png` | STP BPDU packet |
| `screenshots/cdp-packet.png` | CDP packet |
| `screenshots/ospf-hello.png` | OSPF Hello packet |
| `screenshots/arp-packet.png` | ARP packet |
| `README.md` | This file |

## Time to Complete

30 minutes (including Simulation Mode captures)
