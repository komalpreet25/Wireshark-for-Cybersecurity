# Wireshark-for-Cybersecurity

### Advanced Network Traffic Analysis, Threat Hunting & Digital Forensics

![Wireshark](https://img.shields.io/badge/Tool-Wireshark-1679A7?style=for-the-badge)
![Domain](https://img.shields.io/badge/Domain-Cybersecurity-red?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Network%20Analysis-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Learning%20%26%20Research-blue?style=for-the-badge)

A practical, hands-on learning repository for understanding network traffic, analyzing packets, troubleshooting network issues, and investigating suspicious activity using **Wireshark**.

## 🎯 About This Repository

This repository contains structured notes, practical filters, command-line examples, cybersecurity use cases, and hands-on labs for learning Wireshark from fundamentals to advanced traffic analysis.

The goal is not just to learn Wireshark features, but to understand **what network traffic reveals, how to investigate it, and how to recognize potentially suspicious behavior.**

## 📚 Learning Roadmap

| No. | Topic | What You'll Learn |
|---|---|---|
| 01 | Wireshark Basics | Interface selection, workspace, packet capture, and core features |
| 02 | Packet Analysis | Packet structure, protocol layers, headers, and encapsulation |
| 03 | Capture and Display Filters | Filter syntax, practical examples, and common mistakes |
| 04 | Core Network Protocols | Ethernet, ARP, IPv4/IPv6, UDP, and ICMP |
| 05 | TCP Analysis | Three-way handshake, TCP flags, retransmissions, and resets |
| 06 | DNS and HTTP Analysis | DNS queries, HTTP requests, responses, and suspicious patterns |
| 07 | TLS and HTTPS Analysis | TLS handshakes, certificates, and encrypted traffic analysis |
| 08 | Advanced Traffic Analysis | Conversations, streams, statistics, and network troubleshooting |
| 09 | Threat Hunting | Port scanning indicators, unusual DNS activity, and suspicious connections |
| 10 | TShark and Automation | Command-line analysis, field extraction, and basic automation |
| 11 | PCAP Forensics | PCAP/PCAPNG investigation, evidence handling, and analysis workflow |
| 12 | Practical Labs | Hands-on exercises and investigation scenarios |
| 13 | Top Wireshark Filters | A practical reference of commonly used filters |
| 14 | Interview Questions | Technical questions and SOC analyst scenarios |

## 🛠️ Tools and Technologies

- **Wireshark** — graphical packet analysis
- **TShark** — command-line packet analysis
- **Npcap / libpcap** — packet capture support
- **Python** — basic traffic-analysis automation
- **PCAP / PCAPNG** — packet capture file formats

## 🔍 Cybersecurity Use Cases

Wireshark can help security analysts:

- Investigate suspicious network connections.
- Identify DNS anomalies and unusual traffic patterns.
- Analyze TCP connections, retransmissions, and connection resets.
- Understand network reconnaissance and possible port scanning.
- Examine unencrypted application traffic.
- Inspect TLS handshake metadata and certificate information.
- Support incident response and PCAP-based investigations.

**Important:** Wireshark is a traffic-analysis tool. A suspicious packet or pattern alone does not prove malicious activity; findings should be validated using context and additional evidence.

## 🚀 Getting Started

1. Install Wireshark from the [official download page](https://www.wireshark.org/download.html).
2. Open Wireshark and select an authorized network interface.
3. Start a capture or open an existing `.pcap` or `.pcapng` file.
4. Explore packet details and protocol layers.
5. Practise capture and display filters.
6. Work through the topic files in order.

### Your First Display Filters

```text
dns
tcp
udp
icmp
arp
http
tls
```

These filters display packets matching the selected protocol when that protocol is identified in the captured traffic.

## 🧪 Practical Learning Approach

Each topic focuses on relevant concepts, practical examples, useful filters or commands, common mistakes, and cybersecurity applications.

The practical labs use authorized test environments and sample traffic wherever possible. Do not capture or inspect network traffic without permission.

## 📖 Recommended Learning Order

Start with the basics, learn packet structure and filtering, study individual protocols, and then move into threat hunting, automation, and PCAP forensics.

Understanding **networking fundamentals** will make packet analysis much easier.

## 📌 Repository Goals

- Build strong packet-analysis skills.
- Develop a practical understanding of network protocols.
- Create a useful reference for cybersecurity learners.
- Practise investigation techniques relevant to SOC analysis and incident response.

## ⚖️ Disclaimer

This repository is intended for educational and defensive cybersecurity purposes. Use Wireshark and the techniques described here only on networks and traffic you own or are explicitly authorized to analyze.

## ⭐ Support

If these notes help you learn Wireshark, consider starring the repository and contributing improvements or additional practical examples.

---

**Learning Wireshark one packet at a time.**
