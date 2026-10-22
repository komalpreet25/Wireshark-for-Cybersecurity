# Wireshark-for-Cybersecurity

### Advanced Network Traffic Analysis, Threat Hunting & Digital Forensics

![Wireshark](https://img.shields.io/badge/Tool-Wireshark-1679A7?style=for-the-badge)
![Domain](https://img.shields.io/badge/Domain-Cybersecurity-red?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Network%20Analysis-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Learning%20%26%20Research-blue?style=for-the-badge)

Welcome to **Wireshark for Cybersecurity**, a hands-on learning repository dedicated to mastering network traffic analysis using Wireshark and TShark.

This repository covers network protocol fundamentals, advanced packet analysis, capture and display filters, threat hunting, network forensics, automation, and practical cybersecurity investigations.

The goal is to develop the practical skills needed to understand network communication, troubleshoot complex problems, identify suspicious traffic patterns, and support security incident investigations.

---

## Table of Contents

- [Objectives](#objectives)
- [What You Will Learn](#what-you-will-learn)
- [Repository Structure](#repository-structure)
- [Tools and Technologies](#tools-and-technologies)
- [Getting Started](#getting-started)
- [Practical Learning Methodology](#practical-learning-methodology)
- [Sample Display Filters](#sample-display-filters)
- [Hands-On Labs](#hands-on-labs)
- [Security and Ethical Guidelines](#security-and-ethical-guidelines)
- [Learning Roadmap](#learning-roadmap)
- [References](#references)
- [Contributing](#contributing)
- [License](#license)

---

## Objectives

This repository aims to:

- Understand how network packets are structured and transmitted.
- Learn Wireshark architecture and protocol dissection.
- Analyze Ethernet, IPv4, IPv6, TCP, UDP, DNS, HTTP, and TLS traffic.
- Master capture filters and display filters.
- Investigate TCP handshakes, retransmissions, resets, and connection failures.
- Identify indicators of potentially malicious network activity.
- Perform basic network forensic investigations using PCAP files.
- Automate packet analysis using TShark and Python.
- Develop practical skills relevant to SOC analysis and incident response.
- Document technical findings through reproducible labs and investigation reports.

## What You Will Learn

| Learning Area | Topics |
|---|---|
| Fundamentals | Packet anatomy, encapsulation, interfaces, capture architecture |
| Protocol Analysis | Ethernet, ARP, IPv4, IPv6, TCP, UDP, DNS |
| Application Protocols | HTTP, TLS, HTTP/2, QUIC, SMB |
| Filtering | Capture filters, display filters, advanced expressions |
| Network Troubleshooting | TCP handshakes, retransmissions, resets, latency |
| Threat Hunting | Port scanning, suspicious DNS, unusual outbound traffic |
| Digital Forensics | PCAP investigation, stream analysis, timelines, evidence integrity |
| Automation | TShark, field extraction, Python-based analysis |
| SOC Skills | Incident investigation, technical reporting, scenario-based analysis |

## Repository Structure

```text
Wireshark-for-Cybersecurity/
│
├── README.md
├── LICENSE
├── CONTRIBUTING.md
│
├── 01_Network_Analysis_Fundamentals/
├── 02_Advanced_Display_Filters/
├── 03_Protocol_Deep_Dive/
├── 04_Threat_Hunting/
├── 05_Digital_Forensics/
├── 06_TShark_and_Automation/
├── 07_Practical_Labs/
├── 08_Interview_Preparation/
│
└── resources/
    └── sample-pcaps-and-references.md
```

The folders will be populated progressively as the learning material and practical labs are developed.

## Tools and Technologies

The repository focuses on the following tools and technologies:

- **Wireshark:** Graphical network protocol analyzer.
- **TShark:** Command-line packet analyzer.
- **Dumpcap:** Packet capture utility.
- **Npcap / libpcap:** Packet capture libraries and interfaces.
- **Python:** Optional automation and packet-analysis scripts.
- **PCAP / PCAPNG:** Network packet capture formats.
- **Linux and Windows:** Practical analysis environments.

### Prerequisites

Basic knowledge of networking is recommended, including:

- IP addresses and subnetting.
- MAC addresses and Ethernet.
- TCP and UDP.
- Common network ports.
- DNS and HTTP.
- Basic Linux or Windows command-line usage.

Beginners can still follow the repository sequentially.

## Getting Started

### 1. Install Wireshark

Download Wireshark from its official website:

https://www.wireshark.org/download.html

Install the appropriate capture components for your operating system.

### 2. Verify the Installation

Check the Wireshark version:

```bash
wireshark --version
```

Check TShark:

```bash
tshark --version
```

List available capture interfaces:

```bash
tshark -D
```

These commands assume the executables are available in your system PATH.

### 3. Open a Capture File

Open a PCAP or PCAPNG file in Wireshark using **File → Open**.

Alternatively, analyze it from the command line:

```bash
tshark -r capture.pcapng
```

Replace `capture.pcapng` with the path to your own capture file.

### 4. Apply a Display Filter

Try the following filters in Wireshark's display filter bar:

```text
dns
```

```text
tcp
```

```text
tcp.flags.syn == 1
```

```text
tcp.analysis.retransmission
```

These filters help isolate DNS packets, TCP traffic, SYN packets, and detected TCP retransmissions.

## Practical Learning Methodology

Every major topic will aim to include:

1. **Concepts:** Technical explanations of the protocol or feature.
2. **Architecture:** How the underlying mechanisms work.
3. **Commands:** Useful Wireshark and TShark examples.
4. **Filters:** Practical expressions with explanations.
5. **Security Applications:** Relevant defensive use cases.
6. **Hands-On Labs:** Controlled exercises with clear objectives.
7. **Troubleshooting:** Common errors and diagnostic methods.
8. **Interview Preparation:** Technical questions and realistic scenarios.
9. **References:** Official documentation and relevant standards.

The focus is on understanding why a technique works rather than memorizing commands.

## Sample Display Filters

| Purpose | Display Filter |
|---|---|
| DNS traffic | `dns` |
| TCP traffic | `tcp` |
| UDP traffic | `udp` |
| IPv4 traffic | `ip` |
| IPv6 traffic | `ipv6` |
| TCP SYN packets | `tcp.flags.syn == 1` |
| TCP reset packets | `tcp.flags.reset == 1` |
| TCP retransmissions | `tcp.analysis.retransmission` |
| HTTP requests | `http.request` |
| TLS handshake messages | `tls.handshake` |
| Traffic involving a specific host | `ip.addr == 192.168.1.10` |

**Note:** These are display filters, not capture filters. Filter availability and decoded fields depend on the packet contents and Wireshark version.

## Hands-On Labs

Planned practical exercises include:

- DNS troubleshooting and response analysis.
- TCP three-way handshake investigation.
- TCP retransmission and connection-reset analysis.
- Port-scanning indicator investigation.
- Suspicious DNS traffic analysis.
- HTTP request and response inspection.
- TLS handshake metadata analysis.
- PCAP-based incident investigation.
- TShark-based field extraction.
- Evidence integrity and incident reporting.

Labs should use authorized traffic, local test environments, or appropriately licensed sample captures.

## Security and Ethical Guidelines

This repository is intended for **education, defensive security, network troubleshooting, and authorized investigations**.

- Capture traffic only on networks and systems you are authorized to monitor.
- Avoid collecting unnecessary personal or confidential information.
- Never publish PCAP files containing credentials, session tokens, or sensitive communications.
- Use isolated lab environments for suspicious files and untrusted traffic.
- Preserve original evidence when performing forensic analysis.
- Document assumptions, limitations, and supporting evidence.
- Treat unusual traffic as an investigative indicator rather than automatic proof of an attack.

## Learning Roadmap

The repository follows a progressive learning path:

1. Network analysis fundamentals.
2. Advanced capture and display filters.
3. Protocol-level packet analysis.
4. Threat hunting and suspicious traffic investigation.
5. Digital forensics and PCAP analysis.
6. TShark and Python automation.
7. Practical investigation labs.
8. SOC analyst scenarios and interview preparation.

## References

- [Wireshark Official Website](https://www.wireshark.org/)
- [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html/)
- [Wireshark Developer's Guide](https://www.wireshark.org/docs/wsdg_html_chunked/)
- [Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- [Wireshark Source Code](https://github.com/wireshark/wireshark)
- [RFC Editor](https://www.rfc-editor.org/)

## Contributing

Contributions that improve technical accuracy, explanations, practical labs, and documentation are welcome.

Please read [CONTRIBUTING.md](CONTRIBUTING.md) before submitting changes.

## License

This repository's licensing terms are described in [LICENSE](LICENSE).

---

**Learning through packets. Investigating through evidence. Building stronger cybersecurity skills.**
