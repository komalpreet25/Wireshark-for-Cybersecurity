# 14. Wireshark Interview Questions and Answers

## Introduction

Wireshark is one of the most widely used network protocol analyzers in cybersecurity and network administration.

It helps professionals capture, inspect, and analyze network traffic to troubleshoot connectivity issues, investigate suspicious activity, and understand network communication.

This document contains frequently asked Wireshark interview questions, practical scenarios, command-line questions, and SOC-related questions.

---

# 1. Basic Wireshark Questions

### Q1. What is Wireshark?

Wireshark is an open-source network protocol analyzer used to capture, inspect, and analyze network packets in real time or from previously saved capture files.

### Q2. Why is Wireshark used in cybersecurity?

Wireshark helps security professionals:

- Monitor network traffic.
- Investigate suspicious communication.
- Identify unusual network behavior.
- Analyze protocols.
- Troubleshoot network problems.
- Investigate possible security incidents.

### Q3. Is Wireshark free?

Yes. Wireshark is free and open-source software distributed under the GNU General Public License (GPL).

### Q4. What is packet analysis?

Packet analysis is the process of examining network packets to understand communication between devices, identify protocols, and investigate network behavior.

### Q5. What is packet sniffing?

Packet sniffing is the process of capturing network packets for monitoring, troubleshooting, and security analysis.

### Q6. What is a packet?

A packet is a formatted unit of data transmitted across a network. It contains information such as source and destination addresses, protocol information, and payload data.

### Q7. What is a protocol?

A protocol is a set of rules that defines how devices communicate and exchange data over a network.

Examples include:

- TCP
- UDP
- HTTP
- HTTPS
- DNS
- ARP
- ICMP

### Q8. What is a network interface?

A network interface is a hardware or software component that allows a device to communicate over a network.

Examples include Ethernet, Wi-Fi, and loopback interfaces.

### Q9. What is a PCAP file?

PCAP stands for Packet Capture. It is a file format used to store captured network packets for later analysis.

### Q10. What is the difference between PCAP and PCAPNG?

PCAPNG is a newer packet-capture format that supports additional metadata and multiple capture interfaces. Traditional PCAP has a simpler structure and more limited metadata support.

---

# 2. Wireshark Architecture and Packet Analysis

### Q11. How does Wireshark work?

Wireshark captures network packets through a capture mechanism such as libpcap or Npcap, depending on the operating system. Its protocol dissectors then decode packet contents into readable protocol information.

### Q12. What is a protocol dissector?

A protocol dissector is a component that interprets packet data according to a protocol's rules and displays the information in a readable format.

### Q13. What is packet encapsulation?

Encapsulation is the process of adding protocol headers, and sometimes trailers, as data moves through different network layers.

For example:

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

### Q14. What is the OSI model?

The OSI model is a seven-layer reference model used to understand network communication.

1. Physical
2. Data Link
3. Network
4. Transport
5. Session
6. Presentation
7. Application

### Q15. What is the difference between a frame, packet, and segment?

| Term | Layer | Description |
|---|---|---|
| Frame | Data Link | Data unit at the data-link layer |
| Packet | Network | Data unit at the network layer |
| Segment | Transport | Commonly used for TCP data |
| Datagram | Transport | Commonly used for UDP data |

### Q16. What is the Packet Details pane?

It displays the decoded protocol fields of a selected packet in a hierarchical structure.

### Q17. What is the Packet Bytes pane?

It displays the raw packet data in hexadecimal and ASCII formats.

### Q18. What is Follow TCP Stream?

Follow TCP Stream reconstructs the application data exchanged in a TCP connection, where the available capture and protocol dissection permit it.

### Q19. What is the Conversations feature?

The Conversations feature displays communication statistics between pairs of network endpoints.

### Q20. What is the Endpoints feature?

The Endpoints feature lists network endpoints observed in the capture, such as IP and MAC addresses.

---

# 3. Capture Filters and Display Filters

### Q21. What are capture filters?

Capture filters determine which packets are collected during a packet capture.

Example:

```text
port 53
```

This captures traffic matching port 53.

### Q22. What are display filters?

Display filters determine which packets are displayed from captured traffic.

Example:

```wireshark
dns
```

This displays DNS packets.

### Q23. What is the difference between capture filters and display filters?

| Capture Filters | Display Filters |
|---|---|
| Applied during capture | Applied when displaying packets |
| Use BPF syntax | Use Wireshark display-filter syntax |
| Excluded packets are not collected | Hidden packets remain in the capture |
| Example: `tcp port 443` | Example: `tcp.port == 443` |

### Q24. How do you filter traffic from a specific IP address?

```wireshark
ip.addr == 192.168.1.10
```

### Q25. How do you filter traffic to a specific destination IP?

```wireshark
ip.dst == 192.168.1.10
```

### Q26. How do you filter DNS traffic?

```wireshark
dns
```

### Q27. How do you filter HTTP traffic?

```wireshark
http
```

### Q28. How do you filter HTTPS-related TCP traffic?

```wireshark
tcp.port == 443
```

This filters TCP traffic involving port 443. It does not independently prove that every matching packet contains HTTPS.

### Q29. How do you filter TCP SYN packets?

```wireshark
tcp.flags.syn == 1
```

### Q30. How do you exclude DNS traffic?

```wireshark
!dns
```

---

# 4. Networking Protocol Questions

### Q31. What is TCP?

TCP (Transmission Control Protocol) is a connection-oriented transport protocol that provides reliable, ordered delivery of data.

### Q32. What is UDP?

UDP (User Datagram Protocol) is a connectionless transport protocol that sends datagrams without providing TCP-style delivery guarantees.

### Q33. What is the difference between TCP and UDP?

| TCP | UDP |
|---|---|
| Connection-oriented | Connectionless |
| Reliable, ordered delivery | No built-in guarantee of delivery or ordering |
| Uses acknowledgments and retransmissions | No TCP-style acknowledgments or retransmissions |
| Commonly used for HTTPS and SSH | Commonly used for DNS, streaming, and voice traffic |

### Q34. What is the TCP three-way handshake?

It is the process used to establish a TCP connection.

1. SYN – Client requests a connection.
2. SYN-ACK – Server acknowledges and responds.
3. ACK – Client acknowledges the response.

### Q35. What are TCP flags?

TCP flags are control bits used to manage TCP connections.

Important flags include:

- SYN – Initiates a connection.
- ACK – Acknowledges received data.
- FIN – Indicates connection termination.
- RST – Resets a connection.
- PSH – Requests prompt delivery of buffered data to the application.
- URG – Indicates urgent data.

### Q36. What is a TCP retransmission?

A TCP retransmission occurs when TCP sends data again because the sender believes the original data was not successfully acknowledged.

### Q37. What is ARP?

ARP (Address Resolution Protocol) maps an IPv4 address to a MAC address on a local network.

### Q38. What is DNS?

DNS (Domain Name System) translates domain names into IP addresses and provides other domain-related information.

### Q39. What is ICMP?

ICMP (Internet Control Message Protocol) is used for network error reporting and diagnostic functions.

### Q40. What is the difference between HTTP and HTTPS?

HTTP transfers web data without transport-layer encryption by itself. HTTPS uses HTTP over TLS to protect communication through encryption and authentication.

---

# 5. Advanced Wireshark Questions

### Q41. What is TCP stream analysis?

TCP stream analysis is the examination of communication exchanged between two endpoints during a TCP connection.

### Q42. What is packet fragmentation?

Packet fragmentation occurs when an IP packet is divided into smaller fragments for transmission.

### Q43. What is packet loss?

Packet loss occurs when packets fail to reach their intended destination.

### Q44. What is latency?

Latency is the time taken for data to travel from one point to another.

### Q45. What is jitter?

Jitter is the variation in packet delay over time.

### Q46. What is beaconing?

Beaconing is repeated communication between a system and a remote destination at regular or near-regular intervals. It can be associated with malware Command and Control activity.

### Q47. What is data exfiltration?

Data exfiltration is the unauthorized transfer of information from a system or network to an external destination.

### Q48. What is DNS tunneling?

DNS tunneling is a technique that uses DNS queries and responses to carry data, potentially bypassing normal network restrictions.

### Q49. What is ARP spoofing?

ARP spoofing is an attack in which an attacker sends forged ARP messages to associate their MAC address with another device's IP address.

### Q50. Can Wireshark decrypt HTTPS traffic?

Wireshark can decrypt certain TLS traffic when suitable session secrets or other supported decryption material are available. Without the required keys, application data generally remains encrypted.

---

# 6. TShark Questions

### Q51. What is TShark?

TShark is the command-line version of Wireshark. It is used to capture, filter, and analyze network packets through terminal commands.

### Q52. How do you check the TShark version?

```bash
tshark --version
```

### Q53. How do you list available network interfaces?

```bash
tshark -D
```

### Q54. How do you start a packet capture?

```bash
sudo tshark -i eth0
```

### Q55. How do you save captured packets?

```bash
sudo tshark -i eth0 -w capture.pcapng
```

### Q56. How do you read an existing PCAP file?

```bash
tshark -r capture.pcapng
```

### Q57. How do you display only DNS traffic from a PCAP file?

```bash
tshark -r capture.pcapng -Y "dns"
```

### Q58. How do you extract source and destination IP addresses?

```bash
tshark -r capture.pcapng -T fields -e ip.src -e ip.dst
```

### Q59. What is the difference between Wireshark and TShark?

Wireshark provides a graphical interface for packet analysis, while TShark provides command-line packet analysis and supports scripting and automation.

### Q60. Why is TShark useful in a SOC?

It allows analysts to process capture files, extract packet information, generate statistics, and automate repetitive network investigations.

---

# 7. SOC Analyst L1 Scenario-Based Questions

These questions focus on how an analyst applies Wireshark during a real security investigation.

### Q61. You notice repeated connections from one internal IP to multiple external IP addresses. What would you investigate?

I would identify the source host, destination IP addresses, ports, and protocols. I would examine the connection frequency and timing, then correlate the activity with firewall logs, endpoint alerts, and threat intelligence.

### Q62. You notice repeated DNS queries to an unfamiliar domain. What would you do?

I would investigate the domain name, query frequency, DNS response information, and source host. I would check domain reputation and correlate the activity with endpoint logs before deciding whether escalation is necessary.

### Q63. You observe many TCP SYN packets from one source to different ports. What could this indicate?

It could indicate port scanning or service discovery. I would examine the destination ports, connection responses, source host, and network context to determine whether the activity is authorized or suspicious.

### Q64. You notice a large amount of outbound traffic from a workstation. What would you investigate?

I would identify the destination, protocol, port, transfer duration, and traffic volume. I would compare the activity with normal workstation behavior and investigate whether the transfer is expected.

### Q65. You observe repeated TCP retransmissions. What could be the reason?

Possible reasons include network congestion, packet loss, unstable connections, or routing problems. I would investigate the communication endpoints, timing, and other TCP indicators.

### Q66. You notice repeated connections at regular intervals to the same external IP. What could this indicate?

It could indicate beaconing behavior, including possible malware Command and Control communication. I would investigate the destination, timing pattern, traffic contents where visible, and endpoint activity.

### Q67. A user reports that a website is not loading. How would you troubleshoot using Wireshark?

I would capture the connection attempt and check DNS resolution, TCP connection establishment, TLS handshake, and server responses. This can help identify where the connection is failing.

### Q68. You observe an ARP reply containing an unexpected MAC address. What would you investigate?

I would compare the IP-to-MAC mapping with the expected network information, inspect related ARP packets, and investigate possible ARP spoofing or legitimate address changes.

### Q69. An IDS alert reports suspicious communication. How would you use Wireshark?

I would identify the relevant source and destination, filter the associated traffic, inspect the protocol and connection behavior, and correlate packet evidence with the IDS alert and other security logs.

### Q70. You find a suspicious IP address in a PCAP file. What would you do next?

I would check its communication history, destination ports, protocol, DNS information, and reputation. I would correlate these findings with other evidence and follow the organization's incident response procedure.

---

# 8. Practical Interview Questions

### Q71. How do you start capturing packets in Wireshark?

Open Wireshark, select an active network interface, and start the capture.

### Q72. How do you stop a packet capture?

Click the red Stop Capture button.

### Q73. How do you save a packet capture?

Select File → Save As and save the capture in PCAPNG or another supported format.

### Q74. How do you identify the source and destination IP addresses?

Select a packet and inspect the Internet Protocol section in the Packet Details pane, or use:

```wireshark
ip.src
```

```wireshark
ip.dst
```

### Q75. How do you identify the protocol used in a packet?

Inspect the Protocol column or examine the packet's decoded protocol layers.

### Q76. How do you find DNS queries?

Apply:

```wireshark
dns.flags.response == 0
```

### Q77. How do you find TCP retransmissions?

Apply:

```wireshark
tcp.analysis.retransmission
```

### Q78. How do you inspect a TCP conversation?

Right-click a TCP packet and select Follow → TCP Stream.

### Q79. How do you identify packet timestamps?

Inspect the Time column or the frame.time field in the packet details.

### Q80. How do you export HTTP objects?

Use File → Export Objects → HTTP, where supported by the captured and dissected traffic.

---

# 9. Important Wireshark Commands and Filters

| Command / Filter | Purpose |
|---|---|
| `wireshark` | Launch Wireshark |
| `tshark --version` | Check TShark version |
| `tshark -D` | List interfaces |
| `tshark -i eth0` | Start capture |
| `tshark -r file.pcapng` | Read a capture |
| `tshark -w file.pcapng` | Write captured packets |
| `ip` | Display IPv4 traffic |
| `tcp` | Display TCP traffic |
| `udp` | Display UDP traffic |
| `dns` | Display DNS traffic |
| `http` | Display HTTP traffic |
| `tls` | Display TLS traffic |
| `arp` | Display ARP traffic |
| `icmp` | Display ICMP traffic |
| `tcp.flags.syn == 1` | Find TCP SYN packets |
| `tcp.flags.reset == 1` | Find TCP reset packets |
| `tcp.analysis.retransmission` | Find retransmissions |

---

# 10. Interview Tips for Beginners

If you are preparing for a cybersecurity internship, networking position, or SOC Analyst L1 interview, focus on the following topics:

### Must-Know Topics

- What is Wireshark?
- Packet capture and packet analysis.
- OSI model and TCP/IP model.
- TCP vs UDP.
- TCP three-way handshake.
- Capture filters vs display filters.
- DNS request and response.
- HTTP vs HTTPS.
- ARP and ICMP.
- PCAP and PCAPNG.
- Common ports and protocols.
- TCP retransmissions.
- TShark basics.
- Suspicious network traffic indicators.

### Practical Skills to Demonstrate

1. Capture network traffic.
2. Apply display filters.
3. Identify source and destination IP addresses.
4. Analyze TCP connections.
5. Investigate DNS requests.
6. Inspect HTTP and TLS traffic.
7. Read an existing PCAP file.
8. Explain suspicious traffic indicators.
9. Use basic TShark commands.
10. Prepare a short network investigation report.

### How to Answer Technical Questions

- Start with a simple definition.
- Explain the purpose.
- Give a practical example.
- Mention its cybersecurity relevance.
- Avoid claiming that an indicator alone confirms an attack.

---

# 11. Final Self-Assessment

Test your understanding before attending an interview.

- [ ] I can explain what Wireshark is.
- [ ] I understand packet capture and packet analysis.
- [ ] I can explain the OSI and TCP/IP models.
- [ ] I understand TCP and UDP.
- [ ] I can explain the TCP three-way handshake.
- [ ] I know the difference between capture and display filters.
- [ ] I can identify common network protocols.
- [ ] I can analyze DNS traffic.
- [ ] I can inspect HTTP and TLS traffic.
- [ ] I understand TCP retransmissions.
- [ ] I can identify potential suspicious network patterns.
- [ ] I can read and investigate PCAP files.
- [ ] I can use basic TShark commands.
- [ ] I can document network investigation findings.

---

## Final Takeaway

Wireshark is more than a packet-capturing tool. It helps cybersecurity professionals understand network communication, troubleshoot problems, investigate suspicious activity, and support incident response.

For interviews, focus on understanding the concepts and demonstrating practical skills rather than memorizing filters and definitions.

**Remember:** A good network security analyst does not just identify unusual packets; they investigate the context, correlate evidence, and communicate their findings clearly.
