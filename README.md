# ARP Analysis in Cisco Packet Tracer

A hands-on lab exploring how the Address Resolution Protocol (ARP) maps IPv4 addresses to MAC addresses on a local network, and how it behaves when traffic leaves that network.

## Objectives

- Capture and examine an ARP request and reply in Packet Tracer Simulation mode
- Observe how switches build their MAC address tables
- Understand how ARP works for remote destinations (via the default gateway)

## Topology

![Topology](screenshots/topology.png)

| Segment | Devices | Gateway |
|---------|---------|---------|
| 172.16.31.0/24 | PCs 172.16.31.2, .3, .4 connected to Switch1 | Router1 (172.16.31.1) |
| 10.10.10.0/24 | Laptops 10.10.10.2, .3 connected wirelessly to an Access Point, which connects to Switch0 | Router0 |

Router0 and Router1 are linked by a serial connection.

## Tools

- Cisco Packet Tracer (Simulation and Realtime modes)
- Commands: `arp -a`, `arp -d`, `ping`, `show mac-address-table`, `show arp`

---

## Part 1: Examining an ARP Request

**Setup:** On 172.16.31.2, I cleared the ARP cache with `arp -d`, switched to Simulation mode (with ARP events visible in the filter), and ran `ping 172.16.31.3`.

**Observations**

- The ICMP packet was held back while the PC sent an **ARP request** to find the destination MAC.
- The request used the broadcast destination address **FF-FF-FF-FF-FF-FF**.
- **Switch1 flooded the broadcast out 3 ports** (every port except the one it arrived on): to 172.16.31.3, 172.16.31.4 and Router1.
- Only **172.16.31.3** accepted the request. The other devices discarded it because the target IP was not theirs, and the router did not forward the broadcast.
- The **ARP reply** was a unicast: the source and destination MAC addresses were swapped compared with the request, and Switch1 sent only **1 copy**, back to 172.16.31.2.
- The waiting ICMP packet was then sent with the correct destination MAC.

**ARP table result** (`arp -a` on 172.16.31.2):

```
Internet Address      Physical Address      Type
172.16.31.3           0060.7036.2849        dynamic
```

The entry is `dynamic` because it was learned through ARP rather than configured by hand.

![ARP broadcast capture](screenshots/arp-broadcast-capture.png)
![ARP table](screenshots/arp-table.png)

**When does a device send an ARP request?** When it needs to send to an IP address (or its default gateway) and has no MAC address mapping for it in its ARP table.

---

## Part 2: Switch MAC Address Tables

I generated traffic with `ping 172.16.31.4` from 172.16.31.2 and `ping 10.10.10.3` from 10.10.10.2, then ran `show mac-address-table` on both switches straight afterwards.

**Observations**

- Both pings returned **4 sent, 4 received**. (The first packet can be slower or lost while ARP resolves.)
- **Switch1** learned one MAC address per PC port, plus the router's MAC on its uplink port.
- **Switch0** showed **two MAC addresses on a single port**, the one leading to the Access Point. Both laptops reach the switch through that one link, so the switch learns both addresses on the same port.
- Entries are dynamic and age out after a few minutes, so a table checked too late can look incomplete. Generating fresh traffic repopulates it.

![Switch1 MAC table](screenshots/switch1-mac-table.png)
![Switch0 MAC table](screenshots/switch0-mac-table.png)

---

## Part 3: ARP in Remote Communication

I pinged 10.10.10.1 from 172.16.31.2, which is on a different network.

**Observations**

- The new ARP entry was for **172.16.31.1**, the default gateway, not for 10.10.10.1.
- In Simulation mode, the ARP request targeted **172.16.31.1**. The PC sees that the destination is on another network, so it frames the packet for the router's MAC address and lets the router forward it.
- Routers do not use MAC address tables, so `show mac-address-table` is not meaningful on Router1. `show arp` shows its ARP table, which includes an entry for 172.16.31.2.
- The first ping can time out because ARP resolution must finish before the ICMP packet can be sent. Later pings succeed.

---

## Key Takeaways

- ARP requests are **broadcasts**; ARP replies are **unicasts**.
- Switches flood broadcasts out all ports except the incoming one. Routers do not forward them.
- Hosts ARP for the **default gateway** when the destination is remote.
- ARP entries are cached temporarily in RAM and expire.
- IPv6 uses ICMPv6 Neighbor Discovery instead of ARP.

## Security Note

ARP has no built-in authentication, so a malicious device can answer a request with its own MAC address (**ARP spoofing / poisoning**) and redirect traffic. Common defenses include **Dynamic ARP Inspection (DAI)** on managed switches, port security, and static ARP entries for critical devices.

## Repository Contents

```
├── README.md
├── screenshots/
│   ├── topology.png
│   ├── arp-broadcast-capture.png
│   ├── arp-table.png
│   ├── switch1-mac-table.png
│   └── switch0-mac-table.png
└── (optional) completed Packet Tracer file
```

## What I Learned

Add your own notes here: what surprised you, what was difficult, and how this connects to what you are studying.

---

*This is a personal write-up of a lab exercise. The lab concept is based on a Cisco Networking Academy activity; all observations and screenshots are from my own work.*
