# Packet Tracer - Configure and Verify a Site-to-Site IPsec VPN Using CLI

## Topology

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway | Switch Port |
|--------|----------|------------|-------------|----------------|-------------|
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | N/A | S1 F0/1 |
|  | S0/0/0 (DCE) | 10.1.1.2 | 255.255.255.252 | N/A | N/A |
| R2 | G0/0 | 192.168.2.1 | 255.255.255.0 | N/A | S2 F0/2 |
|  | S0/0/0 | 10.1.1.1 | 255.255.255.252 | N/A | N/A |
|  | S0/0/1 (DCE) | 10.2.2.1 | 255.255.255.252 | N/A | N/A |
| R3 | G0/0 | 192.168.3.1 | 255.255.255.0 | N/A | S3 F0/5 |
|  | S0/0/1 | 10.2.2.2 | 255.255.255.252 | N/A | N/A |
| PC-A | NIC | 192.168.1.3 | 255.255.255.0 | 192.168.1.1 | S1 F0/2 |
| PC-B | NIC | 192.168.2.3 | 255.255.255.0 | 192.168.2.1 | S2 F0/1 |
| PC-C | NIC | 192.168.3.3 | 255.255.255.0 | 192.168.3.1 | S3 F0/18 |

## Objectives

- Verify connectivity throughout the network.
- Configure R1 to support a site-to-site IPsec VPN with R3.

## Background / Scenario

The network topology shows three routers. Your task is to configure R1 and R3 to support a site-to-site IPsec VPN when traffic flows between their respective LANs. The IPsec VPN tunnel is from R1 to R3 via R2. R2 acts as a pass-through and has no knowledge of the VPN.

## ISAKMP Phase 1 Policy Parameters

| Parameters | R1 | R3 |
|------------|----|----|
| Encryption Algorithm | AES 256 | AES 256 |
| Hash Algorithm | SHA-1 | SHA-1 |
| Authentication Method | Pre-share | Pre-share |
| Key Exchange | DH 5 | DH 5 |
| IKE SA Lifetime | 86400 | 86400 |
| ISAKMP Key | vpnpa55 | vpnpa55 |

## IPsec Phase 2 Policy Parameters

| Parameters | R1 | R3 |
|------------|----|----|
| Transform Set Name | VPN-SET | VPN-SET |
| ESP Transform Encryption | esp-aes | esp-aes |
| ESP Transform Authentication | esp-sha-hmac | esp-sha-hmac |
| Peer IP Address | 10.2.2.2 | 10.1.1.2 |
| Traffic to be Encrypted | access-list 110 (source 192.168.1.0 dest 192.168.3.0) | access-list 110 (source 192.168.3.0 dest 192.168.1.0) |
| Crypto Map Name | VPN-MAP | VPN-MAP |
| SA Establishment | ipsec-isakmp | ipsec-isakmp |

## Configuration Steps

### Part 1: Configure IPsec Parameters on R1

1. **Test connectivity:** Ping from PC-A to PC-C.
2. **Enable the Security Technology package:**
   ```
   R1(config)# license boot module c1900 technology-package securityk9
   ```
3. **Identify interesting traffic:**
   ```
   R1(config)# access-list 110 permit ip 192.168.1.0 0.0.0.255 192.168.3.0 0.0.0.255
   ```
4. **Configure ISAKMP policy:**
   ```
   R1(config)# crypto isakmp policy 10
   R1(config-isakmp)# encryption aes 256
   R1(config-isakmp)# authentication pre-share
   R1(config-isakmp)# group 5
   R1(config-isakmp)# exit
   R1(config)# crypto isakmp key vpnpa55 address 10.2.2.2
   ```
5. **Configure IPsec policy:**
   ```
   R1(config)# crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
   R1(config)# crypto map VPN-MAP 10 ipsec-isakmp
   R1(config-crypto-map)# set peer 10.2.2.2
   R1(config-crypto-map)# set transform-set VPN-SET
   R1(config-crypto-map)# match address 110
   R1(config-crypto-map)# exit
   ```
6. **Bind crypto map to interface:**
   ```
   R1(config)# interface s0/0/0
   R1(config-if)# crypto map VPN-MAP
   ```

### Part 2: Configure IPsec Parameters on R3

1. **Enable Security Technology package:** Verify and enable if necessary.
2. **Configure reciprocating ACL:**
   ```
   R3(config)# access-list 110 permit ip 192.168.3.0 0.0.0.255 192.168.1.0 0.0.0.255
   ```
3. **Configure ISAKMP policy:**
   ```
   R3(config)# crypto isakmp policy 10
   R3(config-isakmp)# encryption aes 256
   R3(config-isakmp)# authentication pre-share
   R3(config-isakmp)# group 5
   R3(config-isakmp)# exit
   R3(config)# crypto isakmp key vpnpa55 address 10.1.1.2
   ```
4. **Configure IPsec policy:**
   ```
   R3(config)# crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
   R3(config)# crypto map VPN-MAP 10 ipsec-isakmp
   R3(config-crypto-map)# set peer 10.1.1.2
   R3(config-crypto-map)# set transform-set VPN-SET
   R3(config-crypto-map)# match address 110
   R3(config-crypto-map)# exit
   ```
5. **Bind crypto map to interface:**
   ```
   R3(config)# interface s0/0/1
   R3(config-if)# crypto map VPN-MAP
   ```

### Part 3: Verify the IPsec VPN

1. **Check tunnel before traffic:**
   ```
   show crypto ipsec sa
   ```
2. **Generate interesting traffic:** Ping from PC-A to PC-C.
3. **Check tunnel after traffic:** Re-run `show crypto ipsec sa` to verify packet encryption.
4. **Generate uninteresting traffic:** Ping from PC-A to PC-B.
5. **Verify that uninteresting traffic is not encrypted.**
6. **Check results:** Ensure completion percentage is 100%.

---

© 2015 Cisco and/or its affiliates. All rights reserved.

