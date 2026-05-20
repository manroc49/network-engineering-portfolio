# N13: QoS (CBWFQ, LLQ) – Prioritizing Traffic When Bandwidth Is Limited

## What This Proves

I can configure Quality of Service (QoS) using Class-Based Weighted Fair Queuing (CBWFQ) and Low Latency Queuing (LLQ) on a serial link. VoIP traffic (VOICE class) gets priority queuing, video traffic (VIDEO class) gets guaranteed bandwidth, and data traffic (DATA class) gets the remainder. The policy ensures critical traffic is not dropped during congestion.

## Topology

- 2x Cisco 2811 routers (R1, R2)
- Serial link between R1 Serial0/0/0 and R2 Serial0/0/0
- R1 is traffic source, R2 applies QoS policy
- Loopback interfaces for traffic generation

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | DCE/DTE |
|--------|-----------|------------|-------------|---------|
| R1 | Serial0/0/0 | 10.0.0.1 | 255.255.255.252 | DCE (clock rate 64000) |
| R2 | Serial0/0/0 | 10.0.0.2 | 255.255.255.252 | DTE |
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | - |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | - |

## QoS Policy Summary

| Class | Match Criteria | Action | Bandwidth |
|-------|----------------|--------|-----------|
| VOICE | DSCP EF or RTP traffic | Priority (LLQ) | 10% |
| VIDEO | DSCP AF41 or UDP port 554 | Bandwidth | 30% |
| DATA | DSCP AF21 or TCP ports 80/443 | Bandwidth | 50% |
| class-default | All other traffic | Bandwidth | 10% |

## Configuration Files

- [R1-config.txt](R1-config.txt) - Traffic source (IPs, no QoS policy)
- [R2-config.txt](R2-config.txt) - QoS policy applied to Serial0/0/0 output

## Step-by-Step Configuration

### 1. Build Topology

1. Open Packet Tracer → File → New
2. Routers → drag 2x 2811 routers into workspace
3. Rename devices: R1, R2
4. Click lightning bolt → clock icon (Serial DCE)
5. Connect: R1 Serial0/0/0 → R2 Serial0/0/0 (click R1 first for DCE)
6. File → Save As → N13-qos.pkt

### 2. Configure Basic IP on R1

    enable
    configure terminal
    interface loopback 0
    ip address 1.1.1.1 255.255.255.255
    exit
    interface serial 0/0/0
    ip address 10.0.0.1 255.255.255.252
    no shutdown
    clock rate 64000
    exit
    end
    write memory

### 3. Configure Basic IP on R2

    enable
    configure terminal
    interface loopback 0
    ip address 2.2.2.2 255.255.255.255
    exit
    interface serial 0/0/0
    ip address 10.0.0.2 255.255.255.252
    no shutdown
    exit
    end
    write memory

### 4. Verify Serial Connectivity

On R1:
    ping 10.0.0.2

Expected: 5 replies, 0% loss

### 5. Configure Class Maps on R2

    configure terminal
    class-map match-any VOICE
    match ip dscp ef
    match ip rtp
    exit
    class-map match-any VIDEO
    match ip dscp af41
    match udp port 554
    exit
    class-map match-any DATA
    match ip dscp af21
    match tcp port 80
    match tcp port 443
    exit
    end
    write memory

### 6. Configure Policy Map on R2

    configure terminal
    policy-map QOS-POLICY
    class VOICE
    priority percent 10
    class VIDEO
    bandwidth percent 30
    class DATA
    bandwidth percent 50
    class class-default
    bandwidth percent 10
    exit
    end
    write memory

### 7. Apply Policy Map to Interface on R2

    configure terminal
    interface serial 0/0/0
    service-policy output QOS-POLICY
    exit
    end
    write memory

### 8. Verify QoS Policy

On R2:

    show policy-map interface serial 0/0/0

Expected: Shows QOS-POLICY applied to Serial0/0/0 output

    show class-map
    show policy-map

## Verification Commands

    show policy-map
    show policy-map interface serial 0/0/0
    show class-map

## Expected Results (Placeholder)

| Test | Command | Expected Result |
|------|---------|-----------------|
| Class maps created | `show class-map` | VOICE, VIDEO, DATA classes exist |
| Policy map created | `show policy-map` | QOS-POLICY with 4 classes |
| Policy applied to interface | `show policy-map interface serial 0/0/0` | Service-policy output: QOS-POLICY |
| Priority queue | `show policy-map interface` | VOICE class shows priority percent 10 |
| Bandwidth guarantees | `show policy-map interface` | VIDEO and DATA show bandwidth percent |

## Files in This Folder

| File | Purpose |
|------|---------|
| `N13-qos.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config (traffic source) |
| `R2-config.txt` | R2 running config (QoS policy) |
| `screenshots/class-maps.png` | `show class-map` output |
| `screenshots/policy-map.png` | `show policy-map` output |
| `screenshots/service-policy.png` | `show policy-map interface serial 0/0/0` |

## Time to Complete (Estimated)

25 minutes
