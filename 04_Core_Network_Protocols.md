# 04 — Core Network Protocols

## 1. Introduction to Network Protocols

A **network protocol** is a set of rules that defines how devices communicate and exchange data over a network.

Protocols define how information is formatted, addressed, transmitted, and received.

In cybersecurity, understanding network protocols helps analysts investigate communication patterns, troubleshoot connectivity, and identify suspicious network behavior.

### Protocols Covered in This File

| Protocol | Layer | Main Purpose |
|---|---|---|
| Ethernet | Data Link | Communication within a local network |
| ARP | Network access / Link layer | Resolves IPv4 addresses to MAC addresses |
| IPv4 | Network | Logical addressing and packet routing |
| IPv6 | Network | Next-generation IP addressing and routing |
| UDP | Transport | Connectionless data delivery |
| ICMP | Network | Network diagnostics and error reporting |

## 2. Ethernet Protocol

**Ethernet** is a widely used technology for wired local area networks (LANs). It defines frame transmission and MAC addressing at the Data Link layer.

When devices communicate within the same Ethernet network, frames contain source and destination MAC addresses.

### Ethernet Frame Structure

```text
+-----------------------------+
| Destination MAC  (6 Bytes)  |
+-----------------------------+
| Source MAC       (6 Bytes)  |
+-----------------------------+
| EtherType        (2 Bytes)  |
+-----------------------------+
| Payload          (Variable) |
+-----------------------------+
| FCS              (4 Bytes)  |
+-----------------------------+
```

The Ethernet header is 14 bytes without VLAN tags. The FCS is commonly removed by the network adapter before the packet reaches the capture software.

### Important Ethernet Fields

| Field | Description |
|---|---|
| Destination MAC | Identifies the receiving interface |
| Source MAC | Identifies the sending interface |
| EtherType | Identifies the protocol carried in the payload |
| Payload | Contains the encapsulated network-layer data |
| FCS | Provides frame error detection |

### Common EtherType Values

| EtherType | Protocol |
|---|---|
| `0x0800` | IPv4 |
| `0x0806` | ARP |
| `0x86DD` | IPv6 |
| `0x8100` | IEEE 802.1Q VLAN |

### Wireshark Filters

Show Ethernet frames:

```text
eth
```

Filter by source MAC:

```text
eth.src == aa:bb:cc:dd:ee:ff
```

Filter by destination MAC:

```text
eth.dst == aa:bb:cc:dd:ee:ff
```

Find broadcast Ethernet frames:

```text
eth.dst == ff:ff:ff:ff:ff:ff
```

### Cybersecurity Relevance

Ethernet analysis helps identify:

- Unknown devices communicating on a local network.
- Unexpected MAC addresses.
- Broadcast traffic patterns.
- Potential MAC spoofing indicators.
- Unusual communication between local devices.

A MAC address alone cannot reliably establish the identity of a device because MAC addresses can be changed or spoofed.

## 3. ARP — Address Resolution Protocol

**ARP (Address Resolution Protocol)** is used in IPv4 local networks to resolve an IPv4 address into the corresponding MAC address.

When a device wants to communicate with another device on the same local network, it needs the destination MAC address to construct the Ethernet frame.

### How ARP Works

Suppose:

- Device A: `192.168.1.10`
- Device B: `192.168.1.20`

Device A wants to communicate with Device B but does not know its MAC address.

The process is:

```text
Device A
   |
   | ARP Request (Broadcast)
   | Who has 192.168.1.20?
   v
Local Network
   |
   v
Device B
   |
   | ARP Reply (Unicast)
   | 192.168.1.20 is at MAC-B
   v
Device A
```

### ARP Request

An ARP request is normally sent as an Ethernet broadcast because the sender does not yet know the destination MAC address.

Example:

```text
Who has 192.168.1.20?
Tell 192.168.1.10
```

### ARP Reply

The device owning the requested IPv4 address responds with its MAC address.

Example:

```text
192.168.1.20 is at aa:bb:cc:dd:ee:20
```

### Important ARP Fields

| Field | Purpose |
|---|---|
| Hardware Type | Identifies the hardware network type |
| Protocol Type | Identifies the network protocol |
| Hardware Size | Length of the hardware address |
| Protocol Size | Length of the protocol address |
| Opcode | Identifies request or reply |
| Sender MAC | MAC address of the sender |
| Sender IP | IPv4 address of the sender |
| Target MAC | Target MAC address |
| Target IP | Target IPv4 address |

Common ARP opcodes:

| Opcode | Meaning |
|---|---|
| 1 | ARP Request |
| 2 | ARP Reply |

### Wireshark ARP Filters

Show all ARP packets:

```text
arp
```

Show ARP requests:

```text
arp.opcode == 1
```

Show ARP replies:

```text
arp.opcode == 2
```

Find gratuitous ARP packets:

```text
arp.isgratuitous
```

Inspect ARP traffic in Wireshark by expanding:

```text
Frame
Ethernet II
Address Resolution Protocol
```

### ARP Cache

Devices temporarily store IP-to-MAC mappings in an ARP cache to avoid repeatedly broadcasting ARP requests.

On Linux:

```bash
ip neigh
```

On Windows:

```cmd
arp -a
```

### ARP Spoofing / ARP Poisoning

**ARP spoofing** is an attack in which an attacker sends forged ARP messages to associate their MAC address with another device's IP address, often the default gateway.

This can allow the attacker to position themselves between communicating devices.

### Example Attack Scenario

```text
Victim Device
      |
      | Believes gateway IP belongs to attacker MAC
      v
Attacker Device
      |
      | Forwards or intercepts traffic
      v
Network Gateway
```

This can enable a Man-in-the-Middle (MITM) attack when the network and other conditions permit it.

### Indicators to Investigate

- Multiple MAC addresses claiming the same IP address.
- Unexpected changes in gateway MAC mappings.
- Repeated unsolicited ARP replies.
- Unusual ARP traffic patterns.

**Important:** Duplicate IP configurations, failover systems, and legitimate network changes can also cause unexpected ARP behavior. Validate before identifying an attack.

## 4. IPv4 — Internet Protocol Version 4

**IPv4** is a Network-layer protocol that provides logical addressing and routing between networks.

It uses 32-bit addresses, commonly written in dotted-decimal notation.

Example:

```text
192.168.1.10
```

### IPv4 Header Structure

```text
+--------------------------------+
| Version | IHL | DSCP | Length  |
+--------------------------------+
| Identification | Flags | Offset|
+--------------------------------+
| TTL | Protocol | Header Checksum|
+--------------------------------+
| Source IP Address              |
+--------------------------------+
| Destination IP Address         |
+--------------------------------+
| Options (Optional)              |
+--------------------------------+
```

The minimum IPv4 header size is 20 bytes.

### Important IPv4 Fields

| Field | Purpose |
|---|---|
| Version | Identifies IPv4 |
| IHL | Indicates IPv4 header length |
| DSCP / ECN | Traffic classification and congestion notification |
| Total Length | Total size of the IPv4 packet |
| Identification | Helps identify fragments |
| Flags | Controls fragmentation behavior |
| Fragment Offset | Indicates fragment position |
| TTL | Limits packet lifetime |
| Protocol | Identifies the encapsulated transport or control protocol |
| Header Checksum | Detects errors in the IPv4 header |
| Source IP | Sender's IPv4 address |
| Destination IP | Receiver's IPv4 address |

### Common IPv4 Protocol Numbers

| Protocol Number | Protocol |
|---:|---|
| 1 | ICMP |
| 6 | TCP |
| 17 | UDP |
| 41 | IPv6 encapsulation |
| 47 | GRE |
| 50 | ESP |
| 51 | AH |
| 89 | OSPF |

### IPv4 Private Address Ranges

| Range | Address Space |
|---|---|
| `10.0.0.0/8` | Private |
| `172.16.0.0/12` | Private |
| `192.168.0.0/16` | Private |

Private addresses are commonly used inside local networks and are not directly routable over the public Internet.

### Wireshark IPv4 Filters

Show IPv4 traffic:

```text
ip
```

Filter by source IP:

```text
ip.src == 192.168.1.10
```

Filter by destination IP:

```text
ip.dst == 192.168.1.20
```

Find packets with a low TTL:

```text
ip.ttl <= 5
```

Find IPv4 fragments:

```text
ip.flags.mf == 1 || ip.frag_offset > 0
```

### IPv4 Fragmentation

When an IPv4 packet is too large for the next network link's Maximum Transmission Unit (MTU), fragmentation may occur if permitted.

Important fragmentation fields:

- Identification
- More Fragments (MF) flag
- Fragment Offset

Wireshark can help identify fragments and reassemble them when sufficient packets are available.

### Cybersecurity Relevance

IPv4 analysis helps investigate:

- Unexpected external connections.
- Unusual source or destination addresses.
- IP fragmentation behavior.
- Suspicious communication patterns.
- Potential IP spoofing indicators.

## 5. IPv6 — Internet Protocol Version 6

**IPv6** was developed to address limitations of IPv4, especially the shortage of available IPv4 addresses.

IPv6 uses 128-bit addresses and supports a much larger address space.

Example:

```text
2001:db8::1
```

### IPv6 Header Structure

The IPv6 base header has a fixed size of 40 bytes.

| Field | Size | Purpose |
|---|---:|---|
| Version | 4 bits | Identifies IPv6 |
| Traffic Class | 8 bits | Traffic priority and congestion information |
| Flow Label | 20 bits | Identifies traffic flows |
| Payload Length | 16 bits | Length of the payload after the base header |
| Next Header | 8 bits | Identifies the next header or upper-layer protocol |
| Hop Limit | 8 bits | Limits packet forwarding |
| Source Address | 128 bits | Sender's IPv6 address |
| Destination Address | 128 bits | Receiver's IPv6 address |

Unlike IPv4, IPv6 has no header checksum in its base header.

### Important IPv6 Address Types

| Address Type | Purpose |
|---|---|
| Global Unicast | Routable public IPv6 address |
| Link-Local | Communication on the local link |
| Unique Local | Private-style internal addressing |
| Multicast | Communication with a group of interfaces |
| Loopback | Local device communication (`::1`) |

IPv6 does not use broadcast addressing. Multicast and anycast serve related communication purposes.

### Important IPv6 Ranges

| Range | Purpose |
|---|---|
| `2000::/3` | Global Unicast |
| `fe80::/10` | Link-Local |
| `fc00::/7` | Unique Local |
| `ff00::/8` | Multicast |
| `::1/128` | Loopback |
| `2001:db8::/32` | Documentation and examples |

### Wireshark IPv6 Filters

Show IPv6 packets:

```text
ipv6
```

Filter by source address:

```text
ipv6.src == 2001:db8::1
```

Filter by destination address:

```text
ipv6.dst == 2001:db8::2
```

Find ICMPv6 traffic:

```text
icmpv6
```

### IPv6 Security Considerations

Analysts should understand:

- Unexpected IPv6 communication.
- Rogue Router Advertisements.
- Neighbor Discovery Protocol (NDP) behavior.
- IPv6 tunneling.
- Differences between IPv4 and IPv6 firewall policies.

IPv6 can operate alongside IPv4 in dual-stack networks. Monitoring only IPv4 traffic can leave important communication unobserved.

## 6. UDP — User Datagram Protocol

**UDP (User Datagram Protocol)** is a connectionless Transport-layer protocol.

It allows applications to send datagrams without establishing a TCP-style connection.

UDP does not guarantee delivery, ordering, or retransmission at the transport layer.

### UDP Header Structure

```text
+----------------------+----------------------+
| Source Port          | Destination Port     |
+----------------------+----------------------+
| Length               | Checksum             |
+----------------------+----------------------+
| Application Data                            |
+---------------------------------------------+
```

UDP uses a fixed 8-byte header.

### UDP Header Fields

| Field | Size | Purpose |
|---|---:|---|
| Source Port | 16 bits | Identifies the sending application endpoint |
| Destination Port | 16 bits | Identifies the receiving application endpoint |
| Length | 16 bits | Length of the UDP header and data |
| Checksum | 16 bits | Error detection |

### Common UDP Services

| Port | Service |
|---:|---|
| 53 | DNS |
| 67 / 68 | DHCP |
| 69 | TFTP |
| 123 | NTP |
| 161 / 162 | SNMP |
| 500 | IKE |
| 514 | Syslog |
| 443 | QUIC / HTTP/3 commonly uses UDP 443 |

Port numbers identify common service assignments; actual application behavior depends on the traffic and configuration.

### Wireshark UDP Filters

Show UDP packets:

```text
udp
```

Filter by destination port:

```text
udp.dstport == 53
```

Filter by source port:

```text
udp.srcport == 123
```

Filter by either source or destination port:

```text
udp.port == 53
```

Find UDP packets larger than 1000 bytes at the captured-frame level:

```text
udp && frame.len > 1000
```

### Cybersecurity Relevance

UDP traffic analysis can help investigate:

- DNS tunneling indicators.
- Unexpected UDP communication.
- Suspicious high-volume traffic.
- NTP abuse or unusual time synchronization.
- Potential UDP-based denial-of-service activity.
- QUIC and HTTP/3 communication.

UDP traffic should be analyzed with its application context because many legitimate services use it.

## 7. ICMP — Internet Control Message Protocol

**ICMP (Internet Control Message Protocol)** is used for network diagnostics, error reporting, and operational information.

It is commonly used by tools such as `ping` and `traceroute`.

ICMP is carried inside IP packets and does not use TCP or UDP port numbers.

### Common ICMP Message Types

| Type | Code | Meaning |
|---:|---:|---|
| 0 | 0 | Echo Reply |
| 3 | Varies | Destination Unreachable |
| 5 | Varies | Redirect |
| 8 | 0 | Echo Request |
| 11 | Varies | Time Exceeded |
| 12 | Varies | Parameter Problem |

### How Ping Works

When a device sends a ping request, it normally sends an ICMP Echo Request.

The destination may respond with an ICMP Echo Reply.

```text
Source Device
      |
      | ICMP Echo Request
      v
Destination Device
      |
      | ICMP Echo Reply
      v
Source Device
```

### Linux Ping Command

```bash
ping -c 4 8.8.8.8
```

This sends four ICMP Echo Requests.

### Wireshark ICMP Filters

Show ICMP traffic:

```text
icmp
```

Show Echo Requests:

```text
icmp.type == 8
```

Show Echo Replies:

```text
icmp.type == 0
```

Show Destination Unreachable messages:

```text
icmp.type == 3
```

Show Time Exceeded messages:

```text
icmp.type == 11
```

### ICMP and Traceroute

Traceroute helps identify the network hops between a source and destination.

On Linux:

```bash
traceroute 8.8.8.8
```

Depending on the implementation and options, traceroute may use UDP, ICMP, or TCP probes.

When a probe's TTL or Hop Limit expires at an intermediate router, that router may return an ICMP Time Exceeded message.

### Cybersecurity Relevance

ICMP analysis can help identify:

- Network discovery activity.
- Repeated ping requests.
- Destination unreachable errors.
- Unusual ICMP traffic volumes.
- Potential ICMP tunneling indicators.
- Possible network reconnaissance.

ICMP is also essential for legitimate network diagnostics, so its presence alone is not suspicious.

## 8. ARP vs IPv4 vs IPv6

| Feature | ARP | IPv4 | IPv6 |
|---|---|---|---|
| Purpose | IPv4-to-MAC resolution | Logical addressing and routing | Logical addressing and routing |
| Address size | Uses IPv4 and MAC fields | 32 bits | 128 bits |
| Broadcast | Uses broadcast requests on Ethernet | Supports broadcast | No broadcast |
| Local neighbor resolution | ARP | Uses ARP on Ethernet | Uses Neighbor Discovery |
| Wireshark filter | `arp` | `ip` | `ipv6` |

## 9. TCP vs UDP vs ICMP

| Feature | TCP | UDP | ICMP |
|---|---|---|---|
| OSI layer | Transport | Transport | Network |
| Uses ports | Yes | Yes | No |
| Connection-oriented | Yes | No | No |
| Main purpose | Reliable data transport | Connectionless datagram transport | Diagnostics and error reporting |
| Common use | SSH, HTTP over TCP | DNS, DHCP, QUIC | Ping, error reporting |
| Wireshark filter | `tcp` | `udp` | `icmp` |

## 10. Practical Lab — Analyze Core Network Protocols

**Objective:** Capture and identify Ethernet, ARP, IPv4, IPv6, UDP, and ICMP traffic.

### Step 1: Start a Capture

Open Wireshark and select an active network interface.

### Step 2: Generate ICMP Traffic

On Linux:

```bash
ping -c 4 8.8.8.8
```

Apply:

```text
icmp
```

Observe Echo Requests and Echo Replies, if replies are received.

### Step 3: Generate DNS Traffic

Run:

```bash
nslookup example.com
```

Apply:

```text
dns
```

Observe the DNS query and response, if visible.

### Step 4: Inspect ARP

Apply:

```text
arp
```

If your device needs to resolve a local IPv4 address, you may observe ARP requests and replies.

You can inspect the neighbor cache using:

```bash
ip neigh
```

### Step 5: Inspect Ethernet Frames

Apply:

```text
eth
```

Select a packet and inspect:

- Source MAC address.
- Destination MAC address.
- EtherType.

### Step 6: Inspect IPv4 and IPv6

Apply:

```text
ip
```

Then:

```text
ipv6
```

Observe the source and destination addresses and compare their header fields.

IPv6 packets will appear only if IPv6 traffic is available in the capture.

### Step 7: Inspect UDP Traffic

Apply:

```text
udp
```

Observe source ports, destination ports, and UDP length.

### Lab Observation Table

| Protocol | What to Identify |
|---|---|
| Ethernet | Source MAC, destination MAC, EtherType |
| ARP | Request, reply, sender IP, target IP |
| IPv4 | Source IP, destination IP, TTL |
| IPv6 | Source address, destination address, Hop Limit |
| UDP | Source port, destination port, length |
| ICMP | Message type and code |

### Expected Learning

After completing this lab, you should be able to recognize core network protocols and identify their important fields inside captured packets.

## 11. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Assuming ARP resolves IPv6 addresses | IPv6 uses Neighbor Discovery Protocol |
| Assuming all Ethernet traffic is IPv4 | Ethernet can carry IPv4, IPv6, ARP, and other protocols |
| Assuming UDP guarantees delivery | UDP does not guarantee delivery or ordering |
| Assuming ICMP uses port numbers | ICMP does not use TCP or UDP ports |
| Assuming IPv6 uses broadcast | IPv6 uses multicast and anycast instead |
| Treating every ARP reply as suspicious | ARP replies are part of normal network communication |
| Assuming all ICMP traffic is malicious | ICMP supports legitimate diagnostics and error reporting |
| Assuming all IP packets are unfragmented | IPv4 fragmentation and IPv6 extension headers require additional analysis |

## 12. Interview Questions

**Q1. What is Ethernet?**

Ethernet is a widely used LAN technology that defines frame transmission and MAC addressing.

**Q2. What is ARP?**

ARP resolves an IPv4 address into a MAC address on a local network.

**Q3. What is ARP spoofing?**

ARP spoofing is an attack in which forged ARP messages associate an attacker's MAC address with another device's IP address.

**Q4. What is the difference between IPv4 and IPv6?**

IPv4 uses 32-bit addresses, while IPv6 uses 128-bit addresses and provides a much larger address space.

**Q5. What is TTL?**

TTL limits the lifetime of an IPv4 packet by decreasing at each router hop.

**Q6. What is the difference between UDP and TCP?**

TCP provides reliable, ordered delivery. UDP provides connectionless delivery without transport-level guarantees of reliability or ordering.

**Q7. What is ICMP used for?**

ICMP is used for network diagnostics, error reporting, and operational information.

**Q8. Does ICMP use port numbers?**

No. ICMP does not use TCP or UDP port numbers.

**Q9. What is the purpose of an Ethernet MAC address?**

It identifies a network interface for communication on a local network.

**Q10. How does IPv6 resolve neighboring devices?**

IPv6 uses Neighbor Discovery Protocol (NDP), which operates through ICMPv6 messages.

**Q11. What is an ICMP Echo Request?**

It is a message commonly sent by the ping utility to test whether a destination can respond.

**Q12. What is the difference between ARP and NDP?**

ARP resolves IPv4 addresses to MAC addresses, while IPv6 Neighbor Discovery performs neighbor resolution and other link-local discovery functions.

## 13. Quick Revision

- Ethernet uses MAC addresses for local frame delivery.
- ARP resolves IPv4 addresses to MAC addresses.
- ARP requests are normally broadcast on Ethernet.
- ARP spoofing can support Man-in-the-Middle attacks.
- IPv4 uses 32-bit addresses.
- IPv6 uses 128-bit addresses.
- IPv4 uses TTL; IPv6 uses Hop Limit.
- UDP is connectionless and has an 8-byte header.
- ICMP supports network diagnostics and error reporting.
- IPv6 uses NDP instead of ARP.
- Wireshark helps inspect protocol fields and investigate network behavior.

