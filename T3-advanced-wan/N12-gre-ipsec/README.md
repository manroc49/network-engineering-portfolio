# N12: GRE Tunnel + IPsec – Step-by-Step Hand-Holding Guide

## STEP 1: Build Topology

1. Open Packet Tracer → File → New
2. Routers → drag 2x 1941 routers into workspace
3. Rename devices: R1, R2
4. Click lightning bolt → solid black line (Copper Straight-Through)
5. Connect: R1 Gig0/0 → R2 Gig0/0
6. File → Save As → N12-gre-ipsec.pkt

## STEP 2: Configure Basic IP on R1

R1 CLI:

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

## STEP 3: Configure Basic IP on R2

R2 CLI:

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

## STEP 4: Configure Default Routes

On R1:
    configure terminal
    ip route 0.0.0.0 0.0.0.0 203.0.113.2
    end
    write memory

On R2:
    configure terminal
    ip route 0.0.0.0 0.0.0.0 203.0.113.1
    end
    write memory

## STEP 5: Verify Physical Connectivity

On R1:
    ping 203.0.113.2

Expected: 5 replies, 0% loss

📸 Screenshot 1 (optional): Not required but good for troubleshooting

## STEP 6: Configure GRE Tunnel on R1

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

## STEP 7: Configure GRE Tunnel on R2

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

## STEP 8: Verify GRE Tunnel

On R1:
    show interface tunnel 0

Expected: Tunnel0 is up/up

📸 Screenshot 1: tunnel-interface.png (capture from R1 or R2)

## STEP 9: Configure ACL for IPsec

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

## STEP 10: Configure ISAKMP Policy on R1

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

## STEP 11: Configure ISAKMP Policy on R2

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

## STEP 12: Configure IPsec Transform Set on R1

    configure terminal
    crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
    end
    write memory

## STEP 13: Configure IPsec Transform Set on R2

    configure terminal
    crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
    end
    write memory

## STEP 14: Configure Crypto Map on R1

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

## STEP 15: Configure Crypto Map on R2

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

## STEP 16: Verify IPsec Security Associations

Wait 30 seconds. On R1:

    show crypto isakmp sa

Expected: ISAKMP SA established (QM_IDLE state)

    show crypto ipsec sa

Expected: IPsec SA shows packets encrypted/decrypted

📸 Screenshot 2: crypto-isakmp-sa.png
📸 Screenshot 3: crypto-ipsec-sa.png

## STEP 17: Test Tunnel Connectivity

On R1:
    ping 10.0.0.2

Expected: 5 replies, 0% loss

## STEP 18: Test Across Tunnel (Simulated LANs)

On R1:
    ping 10.2.2.2 source 10.1.1.1

Expected: 5 replies, 0% loss

📸 Screenshot 4: ping-across-tunnel.png

## STEP 19: Save Everything

- Packet Tracer: File → Save
- R1: show running-config → copy → R1-config.txt
- R2: show running-config → copy → R2-config.txt

## SCREENSHOT CHECKLIST

| Screenshot | When | What to capture |
|------------|------|-----------------|
| tunnel-interface.png | Step 8 | show interface tunnel 0 (up/up) |
| crypto-isakmp-sa.png | Step 16 | show crypto isakmp sa (QM_IDLE) |
| crypto-ipsec-sa.png | Step 16 | show crypto ipsec sa (encrypted/decrypted packets) |
| ping-across-tunnel.png | Step 18 | ping 10.2.2.2 source 10.1.1.1 successful |
