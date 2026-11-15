# 12. Practical Wireshark Labs

## Introduction

Practical labs help cybersecurity learners understand how network communication works and how to investigate suspicious traffic using Wireshark and TShark.

This section contains hands-on exercises covering:

- Packet capture and inspection.
- Capture and display filters.
- Ethernet, ARP, IP, and TCP analysis.
- DNS and HTTP investigation.
- TLS and HTTPS analysis.
- Suspicious traffic identification.
- PCAP investigation and reporting.

**Lab safety:** Perform packet captures only on your own devices, authorized networks, or educational lab environments. Never capture other people's private traffic without permission.

---

# Lab 1: Basic Packet Capture

<badge color="success">Beginner</badge>

### Objective

Learn how to capture live network traffic using Wireshark.

### Requirements

- Wireshark installed.
- An active network connection.
- Permission to capture traffic on the selected interface.

### Steps

**Step 1: Open Wireshark**

Launch Wireshark and identify the available network interfaces.

**Step 2: Select an interface**

Choose the interface currently carrying network traffic, such as:

- Ethernet
- Wi-Fi
- `eth0`
- `wlan0`

**Step 3: Start capturing**

Double-click the selected interface or click the Start Capture button.

**Step 4: Generate network traffic**

Open a browser and visit a website you are authorized to access.

**Step 5: Stop capturing**

Click the Stop button.

**Step 6: Save the capture**

Save the file as:

```text
basic_capture.pcapng
```

### Tasks

1. Identify the number of captured packets.
2. Find the source and destination IP addresses.
3. Identify at least three protocols.
4. Open a packet and inspect its protocol layers.

### Expected Learning Outcome

Understand how to start and stop packet capture, save PCAPNG files, and inspect basic packet information.

---

# Lab 2: Capture Filters and Display Filters

<badge color="success">Beginner</badge>

### Objective

Understand the difference between capture filters and display filters.

### Part A: Display Filters

Open your saved capture file.

Apply these filters one by one:

**1. Display DNS traffic**

```wireshark
dns
```

**2. Display TCP traffic**

```wireshark
tcp
```

**3. Display UDP traffic**

```wireshark
udp
```

**4. Display IPv4 traffic**

```wireshark
ip
```

**5. Display traffic from a specific IP**

```wireshark
ip.addr == 192.168.1.10
```

Replace the example address with an IP address actually present in your capture.

### Part B: Capture Filters

Start a new capture using these capture-filter examples.

**1. Capture DNS-port traffic**

```text
port 53
```

**2. Capture TCP traffic**

```text
tcp
```

**3. Capture traffic from a specific host**

```text
host 192.168.1.10
```

**4. Capture HTTPS-port traffic**

```text
tcp port 443
```

### Tasks

1. Apply at least five display filters.
2. Perform a new capture using a capture filter.
3. Compare the number of packets captured with and without filtering.
4. Explain why capture filters and display filters are different.

### Expected Learning Outcome

Learn to control which packets are collected and which packets are displayed during analysis.

---

# Lab 3: Ethernet, ARP, and IP Analysis

<badge color="success">Beginner</badge>

### Objective

Understand how devices communicate at the network and data-link layers.

### Steps

**Step 1: Generate local network traffic**

On Linux, use:

```bash
ping -c 4 192.168.1.1
```

Replace the example address with your actual default gateway.

**Step 2: Capture the traffic**

Start a Wireshark capture before running the command.

**Step 3: Apply the ARP filter**

```wireshark
arp
```

**Step 4: Apply the ICMP filter**

```wireshark
icmp
```

**Step 5: Inspect an IP packet**

```wireshark
ip
```

### Tasks

1. Identify an ARP request and response, if present.
2. Identify the sender and target IP addresses in an ARP packet.
3. Identify the source and destination MAC addresses.
4. Identify the IPv4 source and destination addresses.
5. Observe the ICMP Echo Request and Echo Reply packets.

### Expected Learning Outcome

Understand ARP resolution, MAC addresses, IP addressing, and basic ICMP communication.

---

# Lab 4: TCP Three-Way Handshake Analysis

<badge color="warning">Intermediate</badge>

### Objective

Understand how TCP establishes a connection between two devices.

### Steps

**Step 1: Start a Wireshark capture**

Select your active network interface.

**Step 2: Generate TCP traffic**

Use a browser to access a website or connect to an authorized test service.

**Step 3: Apply the TCP filter**

```wireshark
tcp
```

**Step 4: Identify TCP flags**

Look for these three packets:

| Packet | TCP flag | Purpose |
|---|---|---|
| First | SYN | Client requests a connection |
| Second | SYN, ACK | Server acknowledges and responds |
| Third | ACK | Client acknowledges the response |

**Step 5: Apply a SYN filter**

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

**Step 6: Apply a reset filter**

```wireshark
tcp.flags.reset == 1
```

### Tasks

1. Identify a TCP connection attempt.
2. Find the SYN, SYN-ACK, and ACK packets where available.
3. Record their source and destination ports.
4. Inspect sequence and acknowledgment numbers.
5. Identify any retransmissions or reset packets.

### Expected Learning Outcome

Understand TCP connection establishment, flags, acknowledgments, and connection behavior.

---

# Lab 5: DNS Traffic Analysis

<badge color="warning">Intermediate</badge>

### Objective

Understand how DNS translates domain names into IP addresses.

### Steps

**Step 1: Start a capture**

Open Wireshark and select your active network interface.

**Step 2: Generate DNS traffic**

On Linux:

```bash
nslookup example.com
```

On Windows:

```cmd
nslookup example.com
```

**Step 3: Apply the DNS filter**

```wireshark
dns
```

**Step 4: Display DNS queries**

```wireshark
dns.flags.response == 0
```

**Step 5: Display DNS responses**

```wireshark
dns.flags.response == 1
```

**Step 6: Inspect DNS record information**

Look for:

- Query name.
- Query type.
- Response code.
- Answer records.
- Resolved IP addresses.

### Tasks

1. Identify the DNS query for the requested domain.
2. Identify the DNS response.
3. Record the query type.
4. Identify the returned IP address, if available.
5. Explain the relationship between the query and response.

### Expected Learning Outcome

Understand DNS queries, responses, record types, and domain resolution.

---

# Lab 6: HTTP Traffic Analysis

<badge color="warning">Intermediate</badge>

### Objective

Understand how HTTP requests and responses appear in packet captures.

### Important

Use an authorized educational HTTP test environment. Avoid submitting real passwords, personal information, or sensitive data.

### Steps

**Step 1: Start a capture**

Open Wireshark.

**Step 2: Generate HTTP traffic**

Visit a permitted HTTP test website or use a local test server.

**Step 3: Apply the HTTP filter**

```wireshark
http
```

**Step 4: Display HTTP requests**

```wireshark
http.request
```

**Step 5: Display HTTP responses**

```wireshark
http.response
```

**Step 6: Inspect HTTP methods**

```wireshark
http.request.method
```

### Tasks

1. Identify an HTTP request.
2. Find the request method.
3. Identify the requested hostname, if available.
4. Inspect the response status code.
5. Examine HTTP headers.
6. Use Follow TCP Stream to understand the communication.

### Expected Learning Outcome

Understand HTTP request and response structures and how unencrypted web traffic can expose application data.

---

# Lab 7: TLS and HTTPS Analysis

<badge color="warning">Intermediate</badge>

### Objective

Understand how encrypted web communication appears in Wireshark.

### Steps

**Step 1: Start a capture**

Open Wireshark and select your active network interface.

**Step 2: Generate HTTPS traffic**

Visit a website using HTTPS.

**Step 3: Apply the TLS filter**

```wireshark
tls
```

**Step 4: Inspect the TLS handshake**

```wireshark
tls.handshake
```

**Step 5: Inspect Client Hello packets**

```wireshark
tls.handshake.type == 1
```

**Step 6: Inspect Server Hello packets**

```wireshark
tls.handshake.type == 2
```

### Tasks

1. Identify a TLS Client Hello.
2. Identify a Server Hello, if present.
3. Inspect the negotiated TLS version where available.
4. Examine certificate information where available.
5. Identify the source and destination IP addresses.
6. Explain why encrypted application data cannot normally be read directly.

### Expected Learning Outcome

Understand TLS handshakes, encryption, certificates, and the visibility limitations of HTTPS traffic.

---

# Lab 8: Network Scanning Traffic Analysis

<badge color="warning">Intermediate</badge>

### Objective

Understand how network scanning activity can appear in packet captures.

### Lab environment

Use only your own virtual machine or an explicitly authorized lab target.

### Steps

**Step 1: Start a Wireshark capture**

Select the interface connected to your authorized lab network.

**Step 2: Perform a basic TCP connection scan**

Using Nmap against your own test machine:

```bash
nmap -sT -p 22,80,443 <TARGET_IP>
```

Replace `<TARGET_IP>` with the IP address of your authorized lab machine.

**Step 3: Apply the SYN filter**

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

**Step 4: Inspect TCP reset packets**

```wireshark
tcp.flags.reset == 1
```

**Step 5: Examine connection attempts**

Observe how the source host contacts different destination ports.

### Tasks

1. Identify the scanning source IP.
2. Identify the target IP.
3. Record the destination ports contacted.
4. Observe SYN and RST packets.
5. Explain how scanning can help discover exposed services.

### Expected Learning Outcome

Understand how basic TCP scanning behavior can be recognized through packet analysis.

---

# Lab 9: Suspicious DNS Activity Investigation

<badge color="danger">Advanced</badge>

### Objective

Investigate unusual DNS patterns that may require security attention.

### Steps

**Step 1: Open a provided educational PCAP**

Use a trusted training capture containing DNS traffic.

**Step 2: Display DNS queries**

```wireshark
dns.flags.response == 0
```

**Step 3: Find long query names**

```wireshark
dns.qry.name.len > 50
```

**Step 4: Identify unusual query frequency**

Use:

**Statistics → I/O Graphs**

Look for repeated or unusually high DNS activity.

**Step 5: Examine DNS query names**

Check for:

- Random-looking subdomains.
- Repeated patterns.
- Unusual query lengths.
- Unexpected domain names.
- High query frequency.

### Tasks

1. Identify the source host generating DNS queries.
2. Record unusual domain names.
3. Compare query frequency over time.
4. Identify any repeated patterns.
5. Explain whether the activity requires further investigation.

### Expected Learning Outcome

Learn how to investigate possible DNS tunneling and suspicious domain-resolution patterns without assuming that unusual DNS traffic confirms an attack.

---

# Lab 10: PCAP Forensic Investigation

<badge color="danger">Advanced</badge>

### Objective

Perform a structured investigation of a saved PCAP file.

### Requirements

- Wireshark.
- TShark.
- `capinfos`.
- A trusted educational PCAP file.

### Steps

**Step 1: Calculate the file hash**

```bash
sha256sum sample.pcapng
```

Record the hash before analysis.

**Step 2: Inspect capture details**

```bash
capinfos sample.pcapng
```

**Step 3: Identify the protocol hierarchy**

```bash
tshark -r sample.pcapng -q -z io,phs
```

**Step 4: Identify IP endpoints**

```bash
tshark -r sample.pcapng -q -z endpoints,ip
```

**Step 5: Identify TCP conversations**

```bash
tshark -r sample.pcapng -q -z conv,tcp
```

**Step 6: Investigate DNS traffic**

```bash
tshark -r sample.pcapng -Y "dns"
```

**Step 7: Investigate HTTP requests**

```bash
tshark -r sample.pcapng -Y "http.request"
```

**Step 8: Prepare an investigation report**

Document the findings and supporting packet numbers.

### Tasks

1. Record the PCAP file details.
2. Identify the protocols observed.
3. Identify important network endpoints.
4. Investigate unusual DNS or HTTP activity.
5. Identify any suspicious communication patterns.
6. Record supporting evidence.
7. Document investigation limitations.
8. Write a conclusion based on the available evidence.

### Expected Learning Outcome

Understand how to conduct a basic network forensic investigation using Wireshark and TShark.

---

# Lab 11: TShark Command-Line Analysis

<badge color="danger">Advanced</badge>

### Objective

Use TShark to perform packet analysis without relying on the graphical interface.

### Steps

**Step 1: List network interfaces**

```bash
tshark -D
```

**Step 2: Read a PCAP file**

```bash
tshark -r sample.pcapng
```

**Step 3: Extract source and destination IP addresses**

```bash
tshark -r sample.pcapng -T fields -e ip.src -e ip.dst
```

**Step 4: Extract DNS query names**

```bash
tshark -r sample.pcapng -Y "dns.flags.response == 0" -T fields -e dns.qry.name
```

**Step 5: Generate TCP conversation statistics**

```bash
tshark -r sample.pcapng -q -z conv,tcp
```

**Step 6: Export the extracted information**

```bash
tshark -r sample.pcapng -T fields -e ip.src -e ip.dst > extracted_ips.txt
```

### Tasks

1. Read a PCAP file through TShark.
2. Extract IP addresses.
3. Extract DNS query names.
4. Generate TCP conversation statistics.
5. Save extracted information to a text file.
6. Explain how command-line analysis can support automation.

### Expected Learning Outcome

Develop basic command-line packet-analysis skills and understand how TShark supports repeatable investigations.

---

# Lab 12: Build a Mini Network Investigation Report

<badge color="danger">Advanced</badge>

### Objective

Combine the skills learned in previous labs to prepare a professional network investigation report.

### Scenario

A security team receives an alert about unusual outbound communication from a workstation.

You are assigned to investigate the provided PCAP file and identify whether the observed traffic requires further investigation.

### Investigation checklist

- [ ] Record PCAP filename and SHA-256 hash.
- [ ] Record capture duration and packet count.
- [ ] Identify protocols observed.
- [ ] Identify source and destination IP addresses.
- [ ] Inspect DNS queries and responses.
- [ ] Review TCP conversations.
- [ ] Examine unusual ports and connection attempts.
- [ ] Investigate suspicious HTTP or TLS metadata.
- [ ] Record relevant packet numbers and timestamps.
- [ ] Document limitations and possible explanations.
- [ ] Prepare a final investigation report.

### Report Template

```text
NETWORK INVESTIGATION REPORT

1. Investigation Title:
2. Objective:
3. PCAP File Name:
4. SHA-256 Hash:
5. Capture Duration:
6. Total Packets:

7. Protocols Identified:
8. Source IP Addresses:
9. Destination IP Addresses:
10. Important Ports:
11. DNS Findings:
12. TCP Findings:
13. HTTP/TLS Findings:

14. Suspicious Observations:
15. Supporting Evidence:
16. Investigation Limitations:
17. Conclusion:
18. Recommended Next Steps:
```

### Expected Learning Outcome

Demonstrate the ability to analyze network traffic, identify potential security indicators, preserve evidence, and communicate findings in a structured report.

---

## 13. Lab Completion Tracker

Use this checklist to track your progress.

- [ ] Lab 1 – Basic Packet Capture
- [ ] Lab 2 – Capture and Display Filters
- [ ] Lab 3 – Ethernet, ARP, and IP Analysis
- [ ] Lab 4 – TCP Three-Way Handshake
- [ ] Lab 5 – DNS Traffic Analysis
- [ ] Lab 6 – HTTP Traffic Analysis
- [ ] Lab 7 – TLS and HTTPS Analysis
- [ ] Lab 8 – Network Scanning Traffic Analysis
- [ ] Lab 9 – Suspicious DNS Investigation
- [ ] Lab 10 – PCAP Forensic Investigation
- [ ] Lab 11 – TShark Command-Line Analysis
- [ ] Lab 12 – Network Investigation Report

---

## 14. Skills Developed Through These Labs

| Skill | Practical application |
|---|---|
| Packet Capture | Collect network traffic |
| Packet Inspection | Understand packet structure |
| Filtering | Find relevant network packets |
| TCP Analysis | Investigate connection behavior |
| DNS Analysis | Investigate domain-resolution activity |
| HTTP Analysis | Inspect unencrypted web communication |
| TLS Analysis | Examine encrypted connections and handshake metadata |
| Threat Investigation | Identify potential suspicious patterns |
| PCAP Forensics | Preserve and investigate packet evidence |
| TShark | Perform command-line analysis |
| Documentation | Prepare investigation reports |

## Final Takeaway

Practical packet analysis is an important skill for cybersecurity professionals, especially SOC analysts and network security investigators.

The goal is not just to identify packets but to understand communication patterns, recognize unusual behavior, correlate evidence, and explain findings accurately.

**Always remember:** Suspicious traffic is an investigation lead, not automatically proof of a security incident.
