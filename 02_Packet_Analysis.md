# 02 — Packet Analysis

## 1. What Is Packet Analysis?

**Packet analysis** is the process of examining network packets to understand communication between devices.

Every time devices communicate over a network, data is transmitted in smaller units. These units contain information about the source, destination, protocol, and actual data being transmitted.

Using Wireshark, analysts can inspect these packets to understand:

- Which devices are communicating.
- Which protocols are being used.
- Which ports are involved.
- How data travels across network layers.
- Whether communication is successful or abnormal.
- Whether traffic contains indicators of suspicious activity.

Packet analysis is an important skill in network troubleshooting, SOC operations, threat hunting, and digital forensics.

## 2. What Is a Network Packet?

A network packet is a formatted unit of data exchanged over a network.

A simplified packet structure looks like this:

```text
+----------------------------+
| Ethernet Header            |
+----------------------------+
| IP Header                  |
+----------------------------+
| TCP / UDP Header           |
+----------------------------+
| Application Data           |
+----------------------------+
| Ethernet Trailer (if used) |
+----------------------------+
```

Each layer adds its own information to the data.

For example, when a device sends an HTTP request over TCP and IPv4, the data is encapsulated with TCP, IP, and Ethernet information.

**Note:** This is a simplified Ethernet II and IPv4 example. Actual packet structures vary depending on the protocols, encapsulation, and network technology involved.

## 3. Understanding Encapsulation

**Encapsulation** is the process of adding protocol information to data as it moves down the network stack before transmission.

The receiving device performs the reverse process, called **decapsulation**, to interpret the data.

### Encapsulation Process

```text
Application Data
       |
       v
Transport Layer
       |
       v
Network Layer
       |
       v
Data Link Layer
       |
       v
Physical Transmission
```

### Common Protocol Data Units (PDUs)

| Layer | PDU Name | Example |
|---|---|---|
| Application | Data / Message | HTTP request |
| Transport | Segment (TCP) / Datagram (UDP) | TCP segment |
| Network | Packet | IPv4 packet |
| Data Link | Frame | Ethernet frame |
| Physical | Bits | Electrical, optical, or radio signals |

**Remember:** TCP uses segments, UDP uses datagrams, the IP layer uses packets, and Ethernet uses frames.

## 4. OSI Model and Packet Analysis

The OSI model divides network communication into seven layers.

| Layer | Name | Examples | What to Analyze |
|---|---|---|---|
| 7 | Application | HTTP, DNS, SMTP | Application requests and responses |
| 6 | Presentation | Data encoding, encryption | Data representation and encryption |
| 5 | Session | Session management | Communication sessions |
| 4 | Transport | TCP, UDP | Ports, flags, sequence numbers |
| 3 | Network | IPv4, IPv6, ICMP | IP addresses and routing information |
| 2 | Data Link | Ethernet, ARP | MAC addresses and frame information |
| 1 | Physical | Cables, radio signals | Physical transmission |

Wireshark primarily analyzes captured packets and decoded protocol information. It does not directly show every physical-layer signal or the complete internal behavior of every OSI layer.

### Practical Example

When opening an HTTPS website, you may observe:

1. DNS traffic used to resolve a domain name.
2. TCP connection establishment, if TCP is used.
3. TLS handshake information.
4. Encrypted application data.

Modern HTTPS can also use HTTP/3 over QUIC, which runs over UDP rather than TCP.

## 5. Ethernet Frame Analysis

Ethernet operates at the Data Link layer.

It provides local network delivery using MAC addresses.

### Ethernet II Frame Structure

| Field | Size | Purpose |
|---|---:|---|
| Destination MAC | 6 bytes | Receiving interface address |
| Source MAC | 6 bytes | Sending interface address |
| EtherType | 2 bytes | Identifies the encapsulated protocol |
| Payload | Variable | Carries the next protocol's data |
| FCS | 4 bytes | Frame error detection |

The Ethernet header is normally 14 bytes without VLAN tags.

The FCS may not be visible in captures because many network adapters remove it before passing packets to the operating system.

### Important EtherType Values

| EtherType | Protocol |
|---|---|
| `0x0800` | IPv4 |
| `0x0806` | ARP |
| `0x86DD` | IPv6 |
| `0x8100` | IEEE 802.1Q VLAN |

### Wireshark Filter

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

Replace the example MAC address with one from your capture.

## 6. IP Header Analysis

The Internet Protocol operates at the Network layer.

It provides logical addressing and routing so packets can travel between networks.

### IPv4 Header Fields

| Field | Purpose |
|---|---|
| Version | Identifies IPv4 |
| IHL | Header length |
| Total Length | Total IPv4 packet length |
| Identification | Helps identify fragments |
| Flags | Controls fragmentation behavior |
| Fragment Offset | Indicates fragment position |
| TTL | Limits packet lifetime |
| Protocol | Identifies the next protocol |
| Header Checksum | Checks IPv4 header integrity |
| Source Address | Sender's IPv4 address |
| Destination Address | Receiver's IPv4 address |

The minimum IPv4 header size is 20 bytes when no options are present.

### Important Wireshark Filters

Show IPv4 packets:

```text
ip
```

Filter by source address:

```text
ip.src == 192.168.1.10
```

Filter by destination address:

```text
ip.dst == 192.168.1.20
```

Filter packets involving an address:

```text
ip.addr == 192.168.1.10
```

Find packets with a low TTL:

```text
ip.ttl <= 5
```

A low TTL may be normal in some network conditions. It should not automatically be treated as malicious.

### IPv6 Basics

IPv6 uses 128-bit addresses and a fixed 40-byte base header.

Important fields include:

- Source Address
- Destination Address
- Traffic Class
- Flow Label
- Payload Length
- Next Header
- Hop Limit

Useful filters:

```text
ipv6
```

```text
ipv6.src == 2001:db8::1
```

```text
ipv6.dst == 2001:db8::2
```

The `2001:db8::/32` range is reserved for documentation and examples.

## 7. TCP and UDP Header Analysis

TCP and UDP operate at the Transport layer.

They provide communication between applications using port numbers.

### TCP Header

Important TCP fields:

| Field | Purpose |
|---|---|
| Source Port | Sending application's port |
| Destination Port | Receiving application's port |
| Sequence Number | Tracks byte positions in the stream |
| Acknowledgment Number | Indicates the next expected byte |
| Header Length | Size of the TCP header |
| Flags | Control connection behavior |
| Window Size | Advertises the receive window |
| Checksum | Detects errors in the TCP segment |
| Urgent Pointer | Supports urgent-data indication |

Useful filters:

```text
tcp
```

```text
tcp.port == 443
```

```text
tcp.flags.syn == 1
```

```text
tcp.flags.reset == 1
```

### UDP Header

UDP has a simpler header containing four fields:

| Field | Size |
|---|---:|
| Source Port | 2 bytes |
| Destination Port | 2 bytes |
| Length | 2 bytes |
| Checksum | 2 bytes |

The UDP header is 8 bytes long.

Useful filters:

```text
udp
```

```text
udp.port == 53
```

```text
udp.dstport == 53
```

### TCP vs UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Provides reliable ordered delivery | Does not guarantee delivery or ordering |
| Acknowledgments | Yes | No transport-level acknowledgments |
| Header size | Minimum 20 bytes | 8 bytes |
| Common uses | HTTPS over TCP, SSH, SMTP | DNS, DHCP, real-time media |

Some application protocols can operate over different transports. For example, DNS commonly uses UDP but can also use TCP.

## 8. Understanding Packet Length

Packet length represents the size of a captured packet or a protocol unit, depending on the field being inspected.

In Wireshark, the Packet List's **Length** column generally shows the captured packet length in bytes.

For a basic Ethernet II + IPv4 + TCP packet without options:

```text
Ethernet Header = 14 bytes
IPv4 Header     = 20 bytes
TCP Header      = 20 bytes
--------------------------
Total Headers   = 54 bytes
```

The actual frame may be larger when application data, VLAN tags, IP options, or TCP options are present.

Useful display filter:

```text
frame.len > 1000
```

This displays captured frames longer than 1000 bytes.

**Important:** `frame.len`, IP total length, and TCP segment length are different measurements. Do not assume they always have the same value.

## 9. Inspecting a Packet in Wireshark

Follow these steps:

1. Open a capture file or start a live capture.
2. Select any packet in the Packet List.
3. Expand the Frame section.
4. Expand Ethernet II.
5. Expand Internet Protocol Version 4 or IPv6.
6. Expand Transmission Control Protocol or User Datagram Protocol.
7. Expand the application protocol, if identified.

### What to Observe

| Section | Information |
|---|---|
| Frame | Capture metadata and packet length |
| Ethernet II | Source and destination MAC addresses |
| IPv4 / IPv6 | Source and destination IP addresses |
| TCP / UDP | Ports and transport-layer details |
| Application | Protocol-specific fields and data |

### Follow TCP Stream

To reconstruct a TCP conversation:

1. Select a TCP packet.
2. Right-click it.
3. Select **Follow → TCP Stream**.
4. Review the reconstructed conversation.

This is useful for understanding application-level communication when the payload is available and readable.

Encrypted traffic generally cannot be reconstructed into readable application content without suitable decryption material.

## 10. Packet Analysis for Cybersecurity

Packet analysis can support several security investigations.

### A. Identifying Port Scanning

Repeated connection attempts to many ports or hosts may indicate network scanning.

Example filter:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

This displays initial TCP SYN packets without the ACK flag.

Investigate source IPs, destination ports, timing, and connection outcomes before classifying activity as a scan.

### B. Detecting Unusual DNS Activity

DNS traffic can reveal suspicious domain lookups, unexpected query patterns, and possible command-and-control communication.

Filter:

```text
dns
```

Investigate:

- Unusual domain names.
- Repeated failed lookups.
- Unexpected DNS servers.
- High volumes of queries.
- Unusual record types.

### C. Identifying Connection Resets

TCP RST packets may indicate rejected or terminated connections.

Filter:

```text
tcp.flags.reset == 1
```

A reset is not automatically an attack. It can occur during normal network and application behavior.

### D. Investigating Unusual Ports

Filter for traffic using a particular port:

```text
tcp.port == 4444
```

An unusual port may deserve investigation, but port numbers alone do not identify an application or prove malicious behavior.

## 11. Practical Lab — Analyze a Basic Network Packet

**Objective:** Identify Ethernet, IP, and TCP/UDP information in a captured packet.

### Step 1: Start Wireshark

Select your active network interface and begin capturing.

### Step 2: Generate Network Traffic

Run:

```bash
ping -c 4 1.1.1.1
```

Then generate a DNS lookup:

```bash
nslookup example.com
```

### Step 3: Apply Filters

For ICMP traffic:

```text
icmp
```

For DNS:

```text
dns
```

For IPv4:

```text
ip
```

### Step 4: Inspect a Packet

Select a packet and expand:

```text
Frame
Ethernet II
Internet Protocol Version 4
User Datagram Protocol / Transmission Control Protocol
Application Protocol
```

Some packets will not contain every listed layer. For example, an ICMP packet does not have a TCP or UDP header.

### Step 5: Record Your Observations

| Field | Your Observation |
|---|---|
| Source MAC | |
| Destination MAC | |
| Source IP | |
| Destination IP | |
| Protocol | |
| Source Port | |
| Destination Port | |
| Packet Length | |

### Expected Learning

You should be able to identify the main packet layers, understand their purpose, and locate important header fields in Wireshark.

## 12. Common Mistakes

- Confusing a MAC address with an IP address.
- Treating an Ethernet frame and an IP packet as the same thing.
- Assuming all application traffic is readable.
- Confusing TCP sequence numbers with packet numbers.
- Assuming every DNS packet uses UDP.
- Treating unusual ports or low TTL values as proof of an attack.
- Confusing captured frame length with IP packet length.

## 13. Interview Questions

**Q1. What is packet analysis?**

Packet analysis is the process of inspecting network packets to understand communication, troubleshoot issues, and investigate suspicious activity.

**Q2. What is encapsulation?**

Encapsulation is the process of adding protocol headers and, where applicable, trailers to data as it moves through network layers.

**Q3. What is the difference between a packet and a frame?**

A packet is the Network-layer data unit, while a frame is the Data Link-layer data unit that carries it.

**Q4. What is the purpose of a MAC address?**

A MAC address identifies a network interface at the Data Link layer and is used for local network communication.

**Q5. What is the purpose of TTL?**

TTL limits how long an IPv4 packet can remain in circulation. Routers decrement it, and a packet is discarded when the value reaches zero.

**Q6. What is the difference between TCP and UDP?**

TCP provides reliable, ordered delivery through connection management and acknowledgments. UDP provides connectionless datagram delivery without guaranteeing reliability or ordering.

**Q7. What is an Ethernet header?**

It contains source and destination MAC addresses and an EtherType field identifying the encapsulated protocol.

**Q8. What is a TCP port?**

A TCP port identifies an application endpoint on a host and helps direct incoming data to the appropriate service.

**Q9. What is a protocol dissector?**

A protocol dissector interprets packet bytes and displays protocol fields in a readable format.

**Q10. How can Wireshark help detect suspicious activity?**

It can help identify unusual connection attempts, suspicious DNS patterns, unexpected communication, and abnormal packet behavior.

## 14. Quick Revision

- Packet analysis helps understand network communication.
- Encapsulation adds protocol information to data.
- Decapsulation removes and interprets that information.
- Ethernet operates at the Data Link layer.
- IP operates at the Network layer.
- TCP and UDP operate at the Transport layer.
- MAC addresses support local network delivery.
- IP addresses support logical addressing and routing.
- Ports identify application endpoints.
- Wireshark displays decoded packet information for investigation.

