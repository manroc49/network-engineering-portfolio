# N12: GRE Tunnel + IPsec – Secure Site-to-Site VPN

## What This Proves

I can configure a GRE tunnel between two routers (R1 and R2) and secure it with IPsec encryption. The GRE tunnel carries traffic between the two sites. IPsec encrypts the entire GRE tunnel. This creates a secure site-to-site VPN over an untrusted network (simulated by a direct link).

## Topology

- 2x Cisco 1941 routers (R1, R2)
- Simulated Internet connection between R1 and R2 (direct link)
- GRE Tunnel (Tunnel0) between R1 and R2
- IPsec IKEv1 protecting the GRE tunnel
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

## GRE Tunnel + IPsec Configuration Summary

| Router | Tunnel Source | Tunnel Destination | Tunnel IP | ISAKMP Policy | Pre-shared Key | Transform Set | Crypto Map ACL |
|--------|---------------|-------------------|-----------|---------------|----------------|---------------|----------------|
| R1 | 203.0.113.1 | 203.0.113.2 | 10.0.0.1/30 | 10 (aes256, sha256, pre-share, group14) | cisco123 | TSET (esp-aes256 esp-sha256-hmac) | 101 |
| R2 | 203.0.113.2 | 203.0.113.1 | 10.0.0.2/30 | 10 (aes256, sha256, pre-share, group14) | cisco123 | TSET (esp-aes256 esp-sha256-hmac) | 101 |

## Configuration Files

- [R1-config.txt](R1-config.txt) - GRE tunnel + IPsec
- [R2-config.txt](R2-config.txt) - GRE tunnel + IPsec

## Step-by-Step Configuration

### 1. Build Topology

1. Open Packet Tracer → File → New
2. Routers → drag 2x 1941 routers into workspace
3. Rename devices: R1, R2
4. Click lightning bolt → solid black line (Copper Straight-Through)
5. Connect: R1 Gig0/0 → R2 Gig0/0
6. File → Save As → N12-gre-ipsec.pkt

### 2. Configure Basic IP on R1

    enable
    configure terminal
    interface loopback 0
    ip address 10.1.1.1 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 203.0.113.1 255.255.255.252
    no shutdown
    exit
    end
    write memory

### 3. Configure Basic IP on R2

    enable
    configure terminal
    interface loopback 0
    ip address 10.2.2.2 255.255.255.255
    exit
    interface gigabitEthernet 0/0
    ip address 203.0.113.2 255.255.255.252
    no shutdown
    exit
    end
    write memory

### 4. Configure Default Route on R1 (to reach R2)

    configure terminal
    ip route 0.0.0.0 0.0.0.0 203.0.113.2
    end
    write memory

### 5. Configure Default Route on R2 (to reach R1)

    configure terminal
    ip route 0.0.0.0 0.0.0.0 203.0.113.1
    end
    write memory

### 6. Verify Physical Connectivity

On R1:
    ping 203.0.113.2

Expected: 5 replies, 0% loss

### 7. Configure GRE Tunnel on R1

    configure terminal
    interface tunnel 0
    ip address 10.0.0.1 255.255.255.252
    tunnel source 203.0.113.1
    tunnel destination 203.0.113.2
    tunnel mode gre ip
    no shutdown
    exit
    end
    write memory

### 8. Configure GRE Tunnel on R2

    configure terminal
    interface tunnel 0
    ip address 10.0.0.2 255.255.255.252
    tunnel source 203.0.113.2
    tunnel destination 203.0.113.1
    tunnel mode gre ip
    no shutdown
    exit
    end
    write memory

### 9. Verify GRE Tunnel

On R1:
    show interface tunnel 0

Expected: Tunnel0 is up/up

On R2:
    show interface tunnel 0

Expected: Tunnel0 is up/up

### 10. Configure ACL for IPsec (Match GRE traffic)

On R1:
    configure terminal
    access-list 101 permit gre host 203.0.113.1 host 203.0.113.2
    end
    write memory

On R2:
    configure terminal
    access-list 101 permit gre host 203.0.113.2 host 203.0.113.1
    end
    write memory

### 11. Configure ISAKMP Policy on R1

    configure terminal
    crypto isakmp policy 10
    encryption aes 256
    hash sha256
    authentication pre-share
    group 14
    exit
    crypto isakmp key cisco123 address 203.0.113.2
    end
    write memory

### 12. Configure ISAKMP Policy on R2

    configure terminal
    crypto isakmp policy 10
    encryption aes 256
    hash sha256
    authentication pre-share
    group 14
    exit
    crypto isakmp key cisco123 address 203.0.113.1
    end
    write memory

### 13. Configure IPsec Transform Set on R1

    configure terminal
    crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
    end
    write memory

### 14. Configure IPsec Transform Set on R2

    configure terminal
    crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
    end
    write memory

### 15. Configure Crypto Map on R1

    configure terminal
    crypto map CMAP 10 ipsec-isakmp
    set peer 203.0.113.2
    set transform-set TSET
    match address 101
    exit
    interface tunnel 0
    crypto map CMAP
    end
    write memory

### 16. Configure Crypto Map on R2

    configure terminal
    crypto map CMAP 10 ipsec-isakmp
    set peer 203.0.113.1
    set transform-set TSET
    match address 101
    exit
    interface tunnel 0
    crypto map CMAP
    end
    write memory

### 17. Verify IPsec Security Associations

Wait 30 seconds. On R1:

    show crypto isakmp sa
    show crypto ipsec sa

Expected: ISAKMP SA established, IPsec SA shows packets encrypted/decrypted

### 18. Test Tunnel Connectivity

On R1:
    ping 10.0.0.2

Expected: 5 replies, 0% loss

### 19. Test Across Tunnel (Simulated LANs)

On R1:
    ping 10.2.2.2 source 10.1.1.1

Expected: 5 replies, 0% loss

## Verification Commands

    show interface tunnel 0
    show crypto isakmp sa
    show crypto ipsec sa
    show crypto map

## Expected Results (Placeholder)

| Test | Command | Expected Result |
|------|---------|-----------------|
| Tunnel interface | `show interface tunnel 0` | up/up |
| ISAKMP SA | `show crypto isakmp sa` | QM_IDLE state |
| IPsec SA | `show crypto ipsec sa` | packets encrypted/decrypted |
| Tunnel ping | `ping 10.0.0.2` | 5 replies, 0% loss |
| Across tunnel ping | `ping 10.2.2.2 source 10.1.1.1` | 5 replies, 0% loss |

## Files in This Folder

| File | Purpose |
|------|---------|
| `N12-gre-ipsec.pkt` | Packet Tracer topology |
| `R1-config.txt` | R1 running config (GRE + IPsec) |
| `R2-config.txt` | R2 running config (GRE + IPsec) |
| `screenshots/tunnel-interface.png` | `show interface tunnel 0` |
| `screenshots/crypto-isakmp-sa.png` | `show crypto isakmp sa` |
| `screenshots/crypto-ipsec-sa.png` | `show crypto ipsec sa` |
| `screenshots/ping-across-tunnel.png` | `ping 10.2.2.2 source 10.1.1.1` |

## Time to Complete (Estimated)

35 minutes
