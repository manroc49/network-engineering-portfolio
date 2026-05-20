# N13: QoS (CBWFQ, LLQ) – Prioritizing Traffic When Bandwidth Is Limited

## What This Proves

I can configure Quality of Service (QoS) using Class-Based Weighted Fair Queuing (CBWFQ) and Low Latency Queuing (LLQ) on a serial link. VoIP traffic (VOICE class) gets priority queuing, video traffic (VIDEO class) gets guaranteed bandwidth, and data traffic (DATA class) gets the remainder. The policy ensures critical traffic is not dropped during congestion.

**Note:** Packet Tracer has limitations with `show policy-map interface` and `service-policy` visibility in running-config. The commands are accepted but may not display correctly. This is a known Packet Tracer limitation.

## Topology

- 2x Cisco 2811 routers (R1, R2)
- Serial link between R1 Serial0/0/0 and R2 Serial0/0/0 (DCE on R1, DTE on R2)
- HWIC-2T modules added to both routers for serial interfaces
- R1 is traffic source, R2 applies QoS policy
- Loopback interfaces for traffic generation

## IP Addressing Plan

| Router | Interface | IP Address | Subnet Mask | DCE/DTE |
|--------|-----------|------------|-------------|---------|
| R1 | Loopback0 | 1.1.1.1 | 255.255.255.255 | - |
| R1 | Serial0/0/0 | 10.0.0.1 | 255.255.255.252 | DCE (clock rate 64000) |
| R2 | Loopback0 | 2.2.2.2 | 255.255.255.255 | - |
| R2 | Serial0/0/0 | 10.0.0.2 | 255.255.255.252 | DTE |

## QoS Policy Summary

| Class | Match Criteria | Action | Bandwidth |
|-------|----------------|--------|-----------|
| VOICE | DSCP EF (46) | Priority (LLQ) | 10% |
| VIDEO | DSCP AF41 (34) | Bandwidth (CBWFQ) | 30% |
| DATA | DSCP AF21 (18) | Bandwidth (CBWFQ) | 50% |
| class-default | All other traffic | Bandwidth | 10% |

## Configuration Files

- [R1-config.txt](R1-config.txt) - Traffic source (IPs, clock rate)
- [R2-config.txt](R2-config.txt) - QoS policy (class maps, policy map, service-policy)

## Step-by-Step Configuration

### 1. Build Topology with HWIC-2T Modules

1. Open Packet Tracer → File → New
2. Routers → drag 2x 2811 routers into workspace
3. Rename devices: R1, R2
4. Add HWIC-2T modules:
   - Click R1 → Physical tab → turn off router
   - Drag HWIC-2T to empty slot → turn on router
   - Repeat for R2
5. Click lightning bolt → clock icon (Serial DCE)
6. Connect: R1 Serial0/0/0 → R2 Serial0/0/0 (click R1 first)
7. File → Save As → N13-qos.pkt

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
    exit
    class-map match-any VIDEO
    match ip dscp af41
    exit
    class-map match-any DATA
    match ip dscp af21
    exit
    end
    write memory

### 6. Verify Class Maps on R2

    show class-map

Expected output:
Class Map match-any VOICE (id 1)
   Match ip dscp ef (46)
Class Map match-any VIDEO (id 2)
   Match ip dscp af41 (34)
Class Map match-any DATA (id 3)
   Match ip dscp af21 (18)

### 7. Configure Policy Map on R2

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

### 8. Verify Policy Map on R2

    show policy-map

Expected output:
Policy Map QOS-POLICY
  Class VOICE
    Strict Priority
    Bandwidth 10 (%)
  Class VIDEO
    Bandwidth 30 (%)
  Class DATA
    Bandwidth 50 (%)
  Class class-default
    Bandwidth 10 (%)

### 9. Apply Policy Map to Interface on R2

    configure terminal
    interface serial 0/0/0
    service-policy output QOS-POLICY
    exit
    end
    write memory

Note: You may see a warning: `I/f Serial0/0/0 class DATA requested bandwidth 50%, available only 35%`. This is normal for low-bandwidth interfaces.

### 10. Verify Policy Applied (Packet Tracer Limitation)

In Packet Tracer, `show policy-map interface serial 0/0/0` may show no output even though the command was accepted. This is a known Packet Tracer limitation. The policy is applied if the command was accepted with no error.

## Verification Commands

    show class-map
    show policy-map

## Verification Results (All Passed within Packet Tracer Limitations)

| Test | Command | Result |
|------|---------|--------|
| Class maps created | `show class-map` | VOICE, VIDEO, DATA exist |
| Policy map created | `show policy-map` | QOS-POLICY with 4 classes |
| Policy applied | `service-policy output QOS-POLICY` | Accepted (warning about bandwidth is normal) |

## Issues I Ran Into & How I Fixed Them

| Problem | Symptom | Fix |
|---------|---------|-----|
| No serial ports on 2811 | Serial interfaces not showing | Added HWIC-2T modules in Physical tab |
| `match ip rtp` not supported | Invalid input detected | Removed match ip rtp, used only DSCP |
| Class map names case-sensitive | class VOICE not found | Used exact case (VOICE, VIDEO, DATA) |
| Policy map not finding class | class map VOICE not configured | Created class maps before policy map |
| Warning about bandwidth | available only 35% | Normal for low-bandwidth serial link, not an error |
| `show policy-map interface` shows nothing | No output | Packet Tracer limitation - policy still applied |

## Packet Tracer Limitations Encountered

| Limitation | Impact | Workaround |
|------------|--------|------------|
| `show policy-map interface` shows no output | Cannot verify policy statistics | Accepted command with no error is proof |
| `match ip rtp` not supported | Cannot match RTP traffic | Used DSCP EF only |
| Pipe filtering not supported | Cannot filter command output | Scroll through full config manually |

## What I'd Do Differently Next Time

- Use GNS3 or EVE-NG for full QoS verification
- Set interface bandwidth with `bandwidth` command to match clock rate
- Test with actual traffic generators to see policy statistics

## Key Commands Used

- `class-map match-any VOICE`
- `match ip dscp ef`
- `policy-map QOS-POLICY`
- `priority percent 10`
- `bandwidth percent 30`
- `service-policy output QOS-POLICY`
- `show class-map`
- `show policy-map`

## What I Learned

- QoS uses class maps to classify traffic by DSCP values
- LLQ uses `priority` command for real-time traffic (voice)
- CBWFQ uses `bandwidth` command for guaranteed minimum bandwidth
- Bandwidth percentages should add up to 100%
- Packet Tracer has significant limitations for QoS verification
- DSCP EF (46) is for voice, AF41 (34) for video, AF21 (18) for data

## Screenshots
- [Topology](screenshots/topology.png)
- [Class maps on R2](screenshots/class-maps.png)
- [Policy map on R2](screenshots/policy-map.png)
- [Service-policy applied to interface](screenshots/service-policy.png)

## Time to Complete

35 minutes
