# Site-to-Site IPSec VPN Lab

A Cisco Packet Tracer lab simulating two branch offices connected securely over a public network using a site-to-site IPSec VPN. Traffic between the two private LANs is encrypted as it crosses a simulated ISP, while the ISP itself has no knowledge of the private networks.

Built as hands-on practice for the Cisco CCNA 200-301 while pursuing my Master's in Computer Engineering for IoT Systems at Hochschule Nordhausen.

## Scenario

Two offices, each with its own private LAN, need to communicate securely across the public internet:

- **Office A** (LAN `192.168.10.0/24`) behind router R1
- **Office B** (LAN `192.168.20.0/24`) behind router R2
- An **ISP router** in the middle simulates the public internet — it can route between the public WAN addresses but has no routes to the private LANs

An IPSec tunnel between R1 and R2 encrypts inter-office traffic and makes the two LANs reachable to each other, even though the ISP cannot route private addresses directly.

## Topology

```
  Office A (Berlin)                              Office B (Frankfurt)
  PC-A1                                          PC-B1
  192.168.10.10                                  192.168.20.10
       |                                              |
   [R1]──────────────[ISP]──────────────[R2]
   LAN 192.168.10.1   200.1.1.2/200.2.2.1   LAN 192.168.20.1
   WAN 200.1.1.1                            WAN 200.2.2.2
       └──────── encrypted IPSec tunnel ────────┘
```

## Addressing

| Device | Interface | IP |
|---|---|---|
| R1 | LAN (Gig0/0) | 192.168.10.1/24 |
| R1 | WAN (Gig0/1) | 200.1.1.1/30 |
| R2 | LAN (Gig0/0) | 192.168.20.1/24 |
| R2 | WAN (Gig0/1) | 200.2.2.2/30 |
| ISP | to R1 (Gig0/0) | 200.1.1.2/30 |
| ISP | to R2 (Gig0/1) | 200.2.2.1/30 |
| PC-A1 | LAN A | 192.168.10.10/24 |
| PC-B1 | LAN B | 192.168.20.10/24 |

## How the VPN is built

IPSec on Cisco IOS is configured in five parts on each VPN router (R1 and R2):

1. **IKE Phase 1 (ISAKMP policy)** — establishes a secure, authenticated management channel between the two routers. Settings used: AES-256 encryption, SHA hashing, pre-shared-key authentication, Diffie-Hellman group 2. Both routers must match.
2. **Pre-shared key** — a shared secret bound to the peer's public IP, used to authenticate the two routers to each other.
3. **IKE Phase 2 (transform set)** — defines how the actual user data is encrypted: ESP with AES-256 and SHA-HMAC.
4. **Crypto ACL** — defines the "interesting traffic" to encrypt: traffic from the local LAN to the remote LAN.
5. **Crypto map** — binds the peer, transform set, and crypto ACL together and is applied to the WAN interface.

The 2911 routers require the `securityk9` technology package to be licensed and the router rebooted before any `crypto` commands are available.

Full device configurations are in [`R1-config.txt`](R1-config.txt) and [`R2-config.txt`](R2-config.txt).

## Verification

| Test | Result |
|---|---|
| R1 can ping R2's public WAN (200.2.2.2) through the ISP | ✅ |
| PC-A1 → PC-B1 ping **before** VPN (private traffic) | ❌ blocked (ISP can't route private IPs) |
| PC-A1 → PC-B1 ping **after** VPN tunnel established | ✅ success |
| `show crypto isakmp sa` shows peer in QM_IDLE (Phase 1 up) | ✅ |
| `show crypto ipsec sa` shows non-zero encrypt/decrypt counters | ✅ |

The before/after ping comparison proves the tunnel — not plain routing — is what enables inter-LAN connectivity, and the non-zero encryption counters prove the traffic is genuinely encrypted by IPSec.

## What I learned

- The two-phase IPSec negotiation (IKE Phase 1 for the secure channel, Phase 2 for data encryption) and why both peers must have matching parameters.
- How a crypto ACL defines exactly which traffic is protected, and how the crypto map ties the components together on the WAN interface.
- That IPSec tunnels are built on demand — they only come up when interesting traffic triggers negotiation, which is why the first ping often times out.
- The 2911 `securityk9` licensing requirement before crypto features are usable.
- Reading `show crypto isakmp sa` and `show crypto ipsec sa` to confirm and troubleshoot a tunnel.

## How to open

1. Install [Cisco Packet Tracer](https://www.netacad.com/cisco-packet-tracer) (free with a Cisco Networking Academy account).
2. Open `site-to-site-vpn.pkt`.

## Skills demonstrated

`Cisco IOS` · `IPSec VPN` · `Site-to-Site VPN` · `IKE / ISAKMP` · `Crypto Maps` · `Static Routing` · `Network Security` · `Encryption` · `Troubleshooting`

---

**Author:** Wasif Gull · Master's student in Computer Engineering for IoT Systems @ HS Nordhausen
[LinkedIn](https://www.linkedin.com/in/wasif-gull-257124403/) · [Project 1: Multi-VLAN LAN](https://github.com/WasifGull/multi-vlan-lan-packet-tracer) · [Project 2: OPNsense + Active Directory Home Lab](https://github.com/WasifGull/opnsense-ad-home-lab)
