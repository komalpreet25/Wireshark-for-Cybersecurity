# 08 - Advanced Traffic Analysis with Wireshark

## 1. Introduction to Advanced Traffic Analysis

Advanced traffic analysis involves examining network communication beyond individual packets to understand overall network behavior.

Instead of only checking IP addresses and ports, analysts investigate:

- Communication patterns between devices.
- Unusual traffic volume.
- Unexpected external connections.
- Repeated connection attempts.
- Protocol anomalies.
- Unusual packet timing and sizes.
- Potential indicators of malicious activity.

Wireshark provides several tools that help analysts investigate these patterns through packet inspection, statistics, graphs, and display filters.

## 2. Understanding Normal vs Suspicious Traffic

Before identifying suspicious traffic, analysts need to understand what normal network communication looks like.

### Normal Traffic Examples

- A browser connecting to a website over HTTPS.
- A device performing DNS queries.
- A workstation communicating with an internal file server.
- A user accessing an approved application.
- A device receiving updates from trusted services.

### Potentially Suspicious Traffic Examples

- A workstation repeatedly connecting to an unknown external IP.
- Large numbers of connection attempts to different ports.
- Unexpected communication with unfamiliar countries or networks.
- Unusual DNS query patterns.
- Unexpected outbound data transfers.
- Devices communicating over unusual ports.

**Important:** Unusual traffic is not automatically malicious. Always compare observations with normal network behavior, asset information, and security logs.

## 3. Analyzing Network Conversations

A network conversation represents communication between two endpoints.

Wireshark provides a Conversations feature to summarize traffic between endpoints.

### How to Open Conversations

1. Open a PCAP or PCAPNG file.
2. Navigate to **Statistics → Conversations**.
3. Select the required protocol tab:
   - Ethernet
   - IPv4
   - IPv6
   - TCP
   - UDP
4. Review the communication details.

### Important Information

| Field | Description |
|---|---|
| Address A | First endpoint |
| Address B | Second endpoint |
| Packets | Number of packets exchanged |
| Bytes | Total data transferred |
| Duration | Communication duration |
| Bits/s | Approximate traffic rate |

### Why Conversations Matter

Conversation analysis helps identify:

- Devices generating large amounts of traffic.
- Unexpected external communication.
- Repeated connections.
- Unusual data transfer patterns.
- Important endpoints for further investigation.

## 4. Analyzing Endpoints

Endpoints provide a summary of devices and addresses appearing in a capture.

### How to View Endpoints

Navigate to:

**Statistics → Endpoints**

Select the appropriate tab:

- Ethernet
- IPv4
- IPv6
- TCP
- UDP

### What to Investigate

- Which devices are communicating?
- Which IP addresses appear most frequently?
- Are unknown external addresses present?
- Which endpoints exchange the most data?
- Are there unexpected communication partners?

An endpoint appearing frequently is not necessarily suspicious. Consider its role and expected network activity.

## 5. Identifying High-Volume Traffic

High-volume traffic analysis helps identify devices or connections transferring unusually large amounts of data.

Possible reasons include:

- File downloads.
- Software updates.
- Cloud backups.
- Video streaming.
- Large file transfers.
- Unauthorized data transfers.

### Useful Wireshark Features

Open:

**Statistics → Conversations → TCP**

Review the Bytes and Bits/s columns.

You can also inspect individual packets using:

```wireshark
frame.len
```

To identify larger frames:

```wireshark
frame.len > 1400
```

This filter finds frames larger than 1400 bytes. It does not automatically indicate suspicious activity.

### Security Investigation

When you identify high-volume traffic, investigate:

1. Source and destination IP addresses.
2. Communication direction.
3. Destination port.
4. Protocol.
5. Connection duration.
6. Whether the destination is expected.
7. Whether the traffic matches normal business activity.

## 6. Analyzing Unusual Ports

Network services commonly use standard ports, but applications can also use non-standard ports.

Examples:

| Port | Common Service |
|---|---|
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 110 | POP3 |
| 143 | IMAP |
| 443 | HTTPS |
| 445 | SMB |
| 3389 | RDP |

### Useful Filters

Find traffic to a specific port:

```wireshark
tcp.port == 4444
```

Find traffic using UDP port 53:

```wireshark
udp.port == 53
```

Find traffic using ports outside a selected range:

```wireshark
tcp.port < 1024
```

Find traffic on commonly used remote administration ports:

```wireshark
tcp.port == 22 || tcp.port == 3389
```

**Security relevance:** Unexpected services or ports may deserve investigation. However, a non-standard port does not prove malicious activity, and attackers can also use standard ports.

## 7. Detecting Port Scanning Behavior

Port scanning is a technique used to discover open ports and available services on a target system.

Security teams use scanning for authorized assessments, while attackers may use it for reconnaissance.

### Common Scanning Patterns

- Multiple SYN packets sent to different destination ports.
- Repeated connection attempts to one IP address.
- Many unanswered connection requests.
- Multiple RST responses.
- Attempts to discover available services.

### Useful Filter

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

To inspect RST responses:

```wireshark
tcp.flags.reset == 1
```

### Investigation Steps

1. Identify the source IP.
2. Identify the target IP.
3. Review destination ports.
4. Check the number and frequency of connection attempts.
5. Determine whether the activity is authorized.
6. Correlate with firewall and endpoint logs.

A high number of SYN packets alone is not sufficient to confirm port scanning.

## 8. Detecting Possible Data Exfiltration

Data exfiltration means transferring data from an organization or system to an unauthorized destination.

It is a common concern during security incident investigations.

### Possible Indicators

- Unexpected large outbound transfers.
- Communication with unfamiliar external servers.
- Unusual transfer timing.
- Repeated connections to external destinations.
- Large uploads over HTTP or other protocols.
- DNS traffic with unusual query patterns.

### Investigation Process

1. Identify the source endpoint.
2. Identify the external destination.
3. Determine the amount of data transferred.
4. Inspect the protocol and destination port.
5. Review connection duration and timing.
6. Compare activity with normal business operations.
7. Correlate findings with endpoint and security logs.

### Useful Filters

Find outbound traffic from a specific internal address:

```wireshark
ip.src == 192.168.1.10
```

Find traffic to an external destination:

```wireshark
ip.dst == 203.0.113.10
```

Find HTTP POST requests:

```wireshark
http.request.method == "POST"
```

**Note:** Replace example addresses with the actual addresses from your capture. A POST request or large transfer is not automatically evidence of data theft.

## 9. Detecting Beaconing Behavior

Beaconing is a communication pattern in which a device periodically contacts a remote server.

Malware may use beaconing to communicate with command-and-control infrastructure.

### Possible Indicators

- Repeated connections to the same external IP.
- Similar time intervals between connections.
- Repeated requests to the same domain.
- Small, consistent amounts of data exchanged.
- Connections occurring even when the user is inactive.

### Example

```text
10:00:00 → External Server
10:05:00 → External Server
10:10:00 → External Server
10:15:00 → External Server
```

This illustrates periodic communication at five-minute intervals.

### How to Investigate

1. Identify repeated connections.
2. Examine timestamps.
3. Compare the destination IP and port.
4. Check whether the intervals are consistent.
5. Review DNS queries associated with the destination.
6. Correlate the activity with endpoint processes and security alerts.

### Useful Wireshark Features

- Statistics → Conversations.
- Statistics → I/O Graphs.
- Statistics → Flow Graph.

Regular timing alone does not prove malware beaconing. Legitimate applications also communicate periodically.

## 10. Analyzing Packet Timing

Packet timing analysis helps understand when network communication occurs and whether delays or unusual patterns exist.

### Important Concepts

**Inter-Arrival Time:** The time difference between two packets arriving at the capture point.

**Round-Trip Time (RTT):** The time required for a packet to travel to a destination and for the corresponding response to return.

**Latency:** The delay experienced during network communication.

**Jitter:** Variation in packet delay over time.

### Wireshark Fields

| Field | Description |
|---|---|
| `frame.time` | Absolute packet timestamp |
| `frame.time_delta` | Time difference from the previous captured packet |
| `frame.time_delta_displayed` | Time difference from the previous displayed packet |
| `tcp.analysis.ack_rtt` | Estimated TCP acknowledgment round-trip time |

### Useful Filter

```wireshark
tcp.analysis.ack_rtt
```

These measurements help investigate network delays, connection behavior, and performance problems.

## 11. Using I/O Graphs

I/O Graphs visualize traffic volume over time.

### How to Open I/O Graphs

1. Open a capture file.
2. Navigate to **Statistics → I/O Graphs**.
3. Select an appropriate time interval.
4. Add a display filter if needed.
5. Observe the traffic graph.

### What I/O Graphs Help Identify

- Traffic spikes.
- Periods of unusual network activity.
- Sudden increases in packet volume.
- Changes in traffic patterns.
- Repeated communication intervals.

### Example Filters for I/O Graphs

All TCP traffic:

```wireshark
tcp
```

DNS traffic:

```wireshark
dns
```

HTTP traffic:

```wireshark
http
```

TLS traffic:

```wireshark
tls
```

I/O Graphs are useful for understanding traffic trends that may not be obvious when inspecting individual packets.

## 12. Using Flow Graphs

A Flow Graph displays communication between endpoints in chronological order.

### How to Open Flow Graph

1. Open a packet capture.
2. Navigate to **Statistics → Flow Graph**.
3. Select the required flow or endpoints.
4. Review the packet sequence and communication direction.

### Why Flow Graphs Matter

They help analysts understand:

- Which device initiated communication.
- How endpoints exchanged packets.
- TCP handshake sequences.
- Request and response behavior.
- Connection termination.
- Communication between multiple devices.

Flow Graphs are especially useful when investigating complex network conversations.

## 13. Identifying Fragmented IP Packets

IP fragmentation occurs when an IP packet is divided into smaller fragments for transmission.

IPv4 fragmentation can happen when a packet exceeds the supported Maximum Transmission Unit (MTU) and fragmentation is permitted.

### Important IPv4 Fragmentation Fields

| Field | Description |
|---|---|
| Identification | Helps identify fragments belonging to the same original packet |
| Flags | Includes fragmentation control information |
| Fragment Offset | Indicates the position of a fragment |
| Total Length | Total IPv4 packet length |

### Useful Wireshark Filters

Find IPv4 packets with the More Fragments flag set:

```wireshark
ip.flags.mf == 1
```

Find IPv4 packets with a non-zero fragment offset:

```wireshark
ip.frag_offset > 0
```

Find all IPv4 packets with fragmentation indicated:

```wireshark
ip.flags.mf == 1 || ip.frag_offset > 0
```

### Security Relevance

Fragmentation can be involved in:
- Evasion techniques.
- Firewall inspection challenges.
- Packet reassembly problems.
- Unusual network traffic patterns.

Fragmentation is also a normal networking behavior. Its presence alone does not indicate an attack.

## 14. Identifying Retransmissions and Packet Loss Indicators

TCP retransmissions can help identify potential network performance problems.

### Common Indicators

| Indicator | Meaning |
|---|---|
| Retransmission | A TCP segment is sent again |
| Fast Retransmission | A segment is retransmitted after duplicate ACKs suggest loss |
| Duplicate ACK | Receiver repeats an acknowledgment number |
| Out-of-Order | Segments arrive in an unexpected order |
| Zero Window | Receiver temporarily cannot accept more data |

### Useful Filters

```wireshark
tcp.analysis.retransmission
```

```wireshark
tcp.analysis.duplicate_ack
```

```wireshark
tcp.analysis.out_of_order
```

```wireshark
tcp.window_size_value == 0
```

### Investigation

When these indicators appear:

1. Identify the affected TCP conversation.
2. Check source and destination addresses.
3. Examine packet timestamps.
4. Review sequence and acknowledgment numbers.
5. Look for repeated patterns.
6. Compare with network and endpoint logs.

Wireshark analysis flags are clues, not definitive proof of the underlying cause.

## 15. Advanced Display Filter Combinations

Combining display filters makes traffic investigation more efficient.

### Find TCP Traffic to a Specific IP

```wireshark
ip.addr == 192.168.1.10 && tcp
```

### Find DNS Queries from a Specific IP

```wireshark
ip.src == 192.168.1.10 && dns.flags.response == 0
```

### Find HTTP POST Requests

```wireshark
http.request.method == "POST"
```

### Find TLS Traffic to a Specific Server

```wireshark
ip.dst == 203.0.113.10 && tls
```

### Find TCP SYN Packets to a Specific Port

```wireshark
tcp.dstport == 22 && tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### Find Large TCP Frames

```wireshark
tcp && frame.len > 1400
```

### Find Traffic Between Two Specific IP Addresses

```wireshark
ip.addr == 192.168.1.10 && ip.addr == 192.168.1.20
```

## 16. Practical Lab: Advanced Traffic Investigation

**Objective:** Investigate a packet capture using Wireshark's statistics, filters, and analysis tools.

### Step 1: Open a PCAP File

Use a sample capture from an authorized lab or a capture you created yourself.

### Step 2: Analyze Conversations

Navigate to:

**Statistics → Conversations**

Review:
- Most active endpoints.
- Total bytes transferred.
- Communication duration.
- TCP and UDP conversations.

### Step 3: Analyze Endpoints

Navigate to:

**Statistics → Endpoints**

Identify:
- Frequently communicating IP addresses.
- Internal and external endpoints.
- Devices generating significant traffic.

### Step 4: Check TCP Retransmissions

Apply:

```wireshark
tcp.analysis.retransmission
```

Observe whether Wireshark identifies retransmitted segments.

### Step 5: Check DNS Activity

Apply:

```wireshark
dns
```

Review requested domains, DNS response codes, and DNS servers.

### Step 6: Inspect TLS Connections

Apply:

```wireshark
tls.handshake
```

Inspect ClientHello and ServerHello messages where available.

### Step 7: Open I/O Graphs

Navigate to:

**Statistics → I/O Graphs**

Look for:
- Traffic spikes.
- Repeated activity.
- Unusual changes in packet volume.

### Step 8: Create an Investigation Summary

Document:

| Investigation Area | Observation |
|---|---|
| Source IP | Identify the initiating endpoint |
| Destination IP | Identify the communication target |
| Protocol | Identify TCP, UDP, DNS, HTTP, TLS, etc. |
| Port | Identify the destination service port |
| Traffic Volume | Record bytes transferred |
| Timing | Identify repeated or unusual activity |
| Findings | Record observations and supporting evidence |

Do not label traffic malicious unless sufficient evidence supports that conclusion.

## 17. Common Advanced Traffic Analysis Mistakes

- Treating high traffic volume as proof of data exfiltration.
- Assuming every unusual port indicates an attack.
- Confusing normal periodic communication with malware beaconing.
- Ignoring timestamps during investigation.
- Analyzing packets without understanding the complete conversation.
- Treating retransmissions as automatic proof of network failure.
- Ignoring legitimate application behavior.
- Assuming fragmentation is always malicious.
- Drawing conclusions from one indicator without correlating evidence.

## 18. Advanced Traffic Analysis Interview Questions

**Q1. What is advanced traffic analysis?**

It is the investigation of network communication patterns, traffic behavior, and anomalies to understand performance and identify potential security threats.

**Q2. What is a network conversation in Wireshark?**

It is communication between two endpoints that Wireshark summarizes using information such as packets, bytes, and duration.

**Q3. What is data exfiltration?**

It is the unauthorized transfer of data from a system or organization to an external destination.

**Q4. What is beaconing?**

It is periodic communication between a device and a remote server, sometimes used by malware for command-and-control communication.

**Q5. How can you investigate port scanning?**

By examining repeated connection attempts, SYN packets, destination ports, source addresses, and connection responses.

**Q6. What are I/O Graphs?**

They are Wireshark visualizations that display network traffic activity over time.

**Q7. What is a Flow Graph?**

It is a visualization of packet exchanges between endpoints in chronological order.

**Q8. What is IP fragmentation?**

It is the process of dividing an IP packet into smaller fragments for transmission when fragmentation is required and permitted.

**Q9. What is the difference between latency and jitter?**

Latency is communication delay, while jitter is the variation in that delay.

**Q10. How can Wireshark help detect suspicious outbound traffic?**

It can help identify unexpected external destinations, unusual traffic volumes, repeated connections, and suspicious communication patterns.

**Q11. Why is packet timing important?**

It helps identify delays, repeated communication intervals, unusual activity patterns, and potential performance issues.

**Q12. Why should analysts correlate multiple indicators?**

Because individual network indicators can have legitimate explanations. Correlation provides stronger evidence and reduces false positives.

## 19. Quick Revision

| Concept | Key Point |
|---|---|
| Traffic Analysis | Examining network communication behavior |
| Conversations | Summary of communication between endpoints |
| Endpoints | Devices and addresses in a capture |
| High-Volume Traffic | Large amounts of transferred data |
| Port Scanning | Discovering available ports and services |
| Data Exfiltration | Unauthorized data transfer |
| Beaconing | Periodic communication with a remote server |
| Latency | Communication delay |
| Jitter | Variation in packet delay |
| I/O Graphs | Visualize traffic over time |
| Flow Graph | Shows packet communication sequence |
| IP Fragmentation | Divides packets into smaller fragments |
| Retransmission | TCP sends data again |
| Duplicate ACK | Repeated acknowledgment number |
| Zero Window | Receiver temporarily cannot accept more data |

