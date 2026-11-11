# 09 - Threat Hunting with Wireshark

## 1. Introduction to Threat Hunting

**Threat Hunting** is a proactive cybersecurity activity in which security analysts search for suspicious activities, hidden threats, and indicators of compromise (IOCs) within systems and networks.

Unlike traditional security monitoring, which often relies on predefined alerts, threat hunting involves actively investigating network and endpoint data to identify threats that may have gone undetected.

Wireshark helps analysts investigate network traffic by examining packets, protocols, communication patterns, and suspicious connections.

### What Can Wireshark Help Investigate?

- Suspicious IP addresses and domains.
- Unusual outbound connections.
- Port scanning activity.
- Possible malware communication.
- DNS tunneling indicators.
- Suspicious HTTP requests.
- Unexpected data transfers.
- Unusual TCP connection patterns.

**Important:** Wireshark is a network protocol analyzer, not a complete threat detection platform. Its findings should be correlated with endpoint logs, SIEM alerts, firewall records, and threat intelligence.

## 2. Understanding Indicators of Compromise (IOCs)

An **Indicator of Compromise (IOC)** is an observable piece of information that may indicate a security incident or malicious activity.

### Common Types of IOCs

| IOC | Description | Example |
|---|---|---|
| IP Address | Address associated with suspicious communication | 203.0.113.10 |
| Domain | Suspicious or malicious domain | suspicious-example.test |
| URL | Potentially malicious web address | http://example.test/login |
| File Hash | Unique fingerprint of a file | SHA-256 hash |
| Email Address | Address associated with suspicious activity | attacker@example.test |
| Network Pattern | Unusual communication behavior | Repeated outbound connections |

The examples above are illustrative and are not real threat indicators.

### Why IOCs Matter

IOCs help analysts:
- Identify suspicious communication.
- Investigate potential malware activity.
- Search for known malicious infrastructure.
- Correlate security events.
- Support incident response.

An IOC should be validated using reliable sources and investigation context before taking action.

## 3. Threat Hunting Methodology

A structured investigation helps analysts avoid random packet inspection.

### Basic Threat Hunting Workflow

```text
Define Investigation Objective
            |
            v
Collect Network Traffic
            |
            v
Apply Wireshark Filters
            |
            v
Identify Suspicious Indicators
            |
            v
Investigate Related Packets
            |
            v
Correlate with Other Evidence
            |
            v
Document Findings
            |
            v
Escalate or Close Investigation
```

### Step 1: Define the Objective

Determine what you are looking for.

Examples:
- Suspicious outbound communication.
- Unexpected DNS requests.
- Possible port scanning.
- Unusual data transfers.

### Step 2: Collect Traffic

Use:
- Live network captures.
- PCAP files.
- Security investigation datasets.
- Authorized lab traffic.

### Step 3: Analyze Traffic

Use display filters, packet details, conversations, and statistics.

### Step 4: Investigate Indicators

Examine suspicious IPs, domains, ports, protocols, and communication patterns.

### Step 5: Correlate Evidence

Compare Wireshark observations with:
- SIEM alerts.
- Firewall logs.
- Endpoint Detection and Response (EDR) alerts.
- DNS logs.
- Threat intelligence sources.

### Step 6: Document Findings

Record the evidence, investigation steps, and conclusion.

## 4. Investigating Suspicious IP Addresses

An IP address can be an important starting point during a network investigation.

Analysts may investigate:
- Unexpected external connections.
- Communication with known malicious infrastructure.
- Repeated connections to unknown destinations.
- Unusual inbound traffic.
- Connections to unexpected geographic regions.

### Useful Wireshark Filters

Find traffic involving a specific IP:

```wireshark
ip.addr == 203.0.113.10
```

Find packets coming from an IP:

```wireshark
ip.src == 203.0.113.10
```

Find packets going to an IP:

```wireshark
ip.dst == 203.0.113.10
```

Find TCP communication with an IP:

```wireshark
ip.addr == 203.0.113.10 && tcp
```

### Investigation Steps

1. Identify the suspicious IP address.
2. Determine whether it is internal or external.
3. Identify the communicating device.
4. Examine the destination port and protocol.
5. Review connection frequency and traffic volume.
6. Check available threat intelligence.
7. Correlate with endpoint and firewall logs.

**Important:** IP reputation alone does not confirm malicious activity. Shared hosting, cloud services, VPNs, and compromised legitimate servers can complicate attribution.

## 5. Investigating Suspicious DNS Activity

DNS analysis can help identify potential malware communication and suspicious domain activity.

### Possible Indicators

- Random-looking domain names.
- Unusually long subdomains.
- Repeated queries to unfamiliar domains.
- Large volumes of DNS requests.
- Unexpected TXT record queries.
- Repeated failed DNS lookups.

### Useful Filters

All DNS traffic:

```wireshark
dns
```

DNS queries:

```wireshark
dns.flags.response == 0
```

DNS responses:

```wireshark
dns.flags.response == 1
```

Failed DNS responses:

```wireshark
dns.flags.rcode != 0
```

NXDOMAIN responses:

```wireshark
dns.flags.rcode == 3
```

TXT record queries:

```wireshark
dns.qry.type == 16
```

Long DNS query names:

```wireshark
dns.qry.name.len > 50
```

### What to Investigate

- Requested domain names.
- Query frequency.
- Source device.
- DNS resolver.
- Response codes.
- Record types.
- Related connections after DNS resolution.

### Security Relevance

Malware may use DNS to locate command-and-control infrastructure or transfer encoded information.

However, long domain names and TXT queries can also be legitimate. Investigate the complete communication pattern before reaching a conclusion.

## 6. Investigating Possible Port Scanning

Port scanning is used to discover open ports and services on a target system.

It can be performed for legitimate security assessments or as reconnaissance before an attack.

### Common Indicators

- Multiple SYN packets from one source.
- Attempts to connect to different destination ports.
- Repeated connection attempts.
- Multiple RST responses.
- Unusual connection patterns.

### Useful Filters

Find TCP SYN packets:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Find TCP reset packets:

```wireshark
tcp.flags.reset == 1
```

Find traffic to a specific destination:

```wireshark
ip.dst == 192.168.1.20 && tcp
```

### Investigation Questions

1. Which device initiated the connection attempts?
2. Which system was targeted?
3. How many destination ports were contacted?
4. Were connections successful?
5. Did the activity occur within a short time?
6. Was the scanning activity authorized?

### Security Relevance

Port scanning may reveal services exposed on a system. Analysts should investigate whether the activity is expected or unauthorized.

## 7. Investigating Possible Malware Communication

Malware may communicate with remote servers to receive commands, send stolen information, download additional payloads, or maintain access.

This communication is commonly associated with **Command and Control (C2)**.

### Possible Network Indicators

- Repeated connections to unfamiliar external IPs.
- Unexpected DNS lookups.
- Periodic communication patterns.
- Unusual destination ports.
- Suspicious HTTP requests.
- Unexpected encrypted connections.
- Large or unusual outbound transfers.

### Useful Filters

HTTP requests:

```wireshark
http.request
```

TLS ClientHello:

```wireshark
tls.handshake.type == 1
```

DNS queries:

```wireshark
dns.flags.response == 0
```

Repeated TCP SYN attempts:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### Investigation Process

1. Identify the endpoint generating the traffic.
2. Examine its external destinations.
3. Review DNS requests associated with those destinations.
4. Check TLS metadata or HTTP requests.
5. Examine connection timing and frequency.
6. Investigate the responsible endpoint process using EDR or system logs.
7. Check threat intelligence for known indicators.

**Note:** Encrypted C2 communication may hide application content. Traffic metadata and endpoint telemetry can still provide useful evidence.

## 8. Investigating Possible Data Exfiltration

Data exfiltration refers to unauthorized transfer of information from an internal system to an external destination.

### Possible Indicators

- Unusually large outbound transfers.
- Repeated communication with unfamiliar external servers.
- Unexpected file uploads.
- Unusual DNS activity.
- Data transfers outside normal working hours.
- Unexpected protocols or destinations.

### Useful Filters

Outbound traffic from an internal IP:

```wireshark
ip.src == 192.168.1.10
```

HTTP POST requests:

```wireshark
http.request.method == "POST"
```

Traffic to a specific external IP:

```wireshark
ip.dst == 203.0.113.10
```

Large frames:

```wireshark
frame.len > 1400
```

### Investigation Questions

- Which endpoint sent the data?
- Where was the data sent?
- How much data was transferred?
- Which protocol was used?
- Was the destination authorized?
- Does the transfer match normal business activity?

**Important:** Large transfers and POST requests can be legitimate. Confirm the destination, application, user activity, and business context before identifying possible exfiltration.

## 9. Investigating Suspicious HTTP Traffic

HTTP traffic may expose request details and application data when it is not protected by TLS.

### Potential Indicators

- Unexpected POST requests.
- Unusual URL paths.
- Suspicious query parameters.
- Unexpected file downloads.
- Requests to unfamiliar domains.
- Unusual User-Agent strings.
- Repeated redirects.

### Useful Filters

All HTTP requests:

```wireshark
http.request
```

GET requests:

```wireshark
http.request.method == "GET"
```

POST requests:

```wireshark
http.request.method == "POST"
```

HTTP redirects:

```wireshark
http.response.code >= 300 && http.response.code < 400
```

HTTP server errors:

```wireshark
http.response.code >= 500
```

### Investigation Steps

1. Identify the source and destination.
2. Examine the requested hostname.
3. Review the URI and HTTP method.
4. Inspect request and response headers.
5. Follow the TCP stream where useful.
6. Identify unexpected downloads or data submissions.
7. Correlate with browser and endpoint logs.

Never expose captured credentials, cookies, or personal information in public reports.

## 10. Investigating Suspicious TLS Traffic

Modern malware frequently uses encrypted communication to make network inspection more difficult.

Even when payloads are encrypted, Wireshark may reveal useful metadata.

### Potential Indicators

- Connections to unfamiliar external destinations.
- Unexpected SNI hostnames.
- Repeated TLS connections.
- Unusual connection timing.
- Unexpected TLS configurations.
- Suspicious certificate information.

### Useful Filters

TLS traffic:

```wireshark
tls
```

ClientHello:

```wireshark
tls.handshake.type == 1
```

ServerHello:

```wireshark
tls.handshake.type == 2
```

TLS alerts:

```wireshark
tls.record.content_type == 21
```

### Investigation Steps

1. Identify the communicating endpoints.
2. Examine visible SNI information.
3. Inspect the TLS version and cipher suites.
4. Review certificate information when available.
5. Analyze connection frequency and duration.
6. Correlate the destination with DNS and threat intelligence.
7. Check the endpoint process responsible for the connection.

An unfamiliar certificate, hostname, or TLS configuration is an investigation lead, not proof of malware.

## 11. Understanding Common Network Attack Indicators

The following table summarizes network patterns that may be relevant during threat hunting.

| Threat | Possible Network Indicators |
|---|---|
| Port Scanning | Repeated SYN requests to multiple ports |
| SYN Flood | Large volume of incomplete TCP handshakes |
| DNS Tunneling | Unusual DNS query patterns and encoded subdomains |
| Malware C2 | Repeated communication with suspicious external infrastructure |
| Data Exfiltration | Unexpected outbound data transfers |
| ARP Spoofing | Conflicting IP-to-MAC address mappings |
| DNS Spoofing | Unexpected or manipulated DNS responses |
| Suspicious Web Activity | Unexpected HTTP requests or downloads |

These indicators are not definitive proof. Validate findings with additional evidence.

## 12. Correlating Network Evidence

A strong investigation uses multiple sources of evidence rather than relying on one packet or filter.

### Example Investigation

Suppose a workstation repeatedly connects to an unfamiliar external IP address.

A useful investigation may include:

1. **Wireshark:** Identify the destination IP and communication pattern.
2. **DNS Logs:** Determine whether the workstation recently resolved an unfamiliar domain.
3. **Firewall Logs:** Check whether the connection was allowed or blocked.
4. **EDR:** Identify the process responsible for the connection.
5. **Threat Intelligence:** Check whether the IP or domain is associated with known threats.
6. **SIEM:** Search for related alerts and events.

### Why Correlation Matters

Correlation helps analysts:
- Reduce false positives.
- Build a clearer incident timeline.
- Identify affected systems.
- Understand the scope of an incident.
- Support evidence-based decisions.

## 13. Practical Lab: Investigate Suspicious Network Traffic

**Objective:** Perform a structured threat-hunting investigation using Wireshark.

Use an authorized sample PCAP from a cybersecurity lab or a capture generated in your own environment.

### Step 1: Open the Capture

1. Start Wireshark.
2. Open the selected PCAP or PCAPNG file.
3. Review the packet list.
4. Note the capture time and available protocols.

### Step 2: Identify Important Endpoints

Navigate to:

**Statistics → Endpoints**

Record:
- Internal IP addresses.
- External IP addresses.
- Frequently communicating endpoints.
- Unusual endpoints.

### Step 3: Review Network Conversations

Navigate to:

**Statistics → Conversations**

Examine:
- Active connections.
- Source and destination addresses.
- Communication duration.
- Total bytes transferred.

### Step 4: Investigate DNS Activity

Apply:

```wireshark
dns
```

Look for:
- Unexpected domains.
- Repeated queries.
- Failed DNS responses.
- Unusual record types.

### Step 5: Investigate TCP Connections

Apply:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Review:
- Source IP.
- Destination IP.
- Destination ports.
- Connection frequency.

### Step 6: Review HTTP and TLS Traffic

HTTP:

```wireshark
http
```

TLS:

```wireshark
tls
```

Inspect available hostnames, requests, handshake metadata, and connection patterns.

### Step 7: Check Retransmissions

```wireshark
tcp.analysis.retransmission
```

Determine whether network performance issues may be affecting communication.

### Step 8: Prepare an Investigation Report

Use this format:

| Field | Details |
|---|---|
| Investigation ID | Assign a unique reference |
| Capture File | PCAP filename |
| Source IP | Initiating endpoint |
| Destination IP | Communication target |
| Protocol | TCP, UDP, DNS, HTTP, TLS, etc. |
| Observed Activity | What was detected |
| Evidence | Packet numbers, timestamps, and filters |
| Assessment | Benign, suspicious, or inconclusive |
| Next Action | Monitor, investigate, or escalate |

Do not classify activity as malicious without sufficient supporting evidence.

## 14. Common Threat Hunting Mistakes

- Assuming every unfamiliar IP address is malicious.
- Treating a single suspicious packet as proof of an attack.
- Ignoring normal application behavior.
- Relying only on IP reputation.
- Confusing scanning activity with confirmed exploitation.
- Assuming encrypted communication is automatically malicious.
- Ignoring DNS activity during investigations.
- Failing to correlate endpoint and firewall logs.
- Making conclusions without recording supporting evidence.
- Sharing sensitive PCAP files containing credentials or personal information.

## 15. Threat Hunting Interview Questions

**Q1. What is threat hunting?**

Threat hunting is the proactive process of searching for suspicious activities and hidden threats within systems and networks.

**Q2. What is an IOC?**

An Indicator of Compromise is an observable piece of information that may indicate malicious activity or a security incident.

**Q3. What is command and control (C2)?**

C2 is communication between an attacker-controlled system and compromised devices to send commands or exchange information.

**Q4. How can Wireshark help with threat hunting?**

It helps analysts inspect packets, investigate suspicious connections, analyze protocols, and identify unusual communication patterns.

**Q5. How can you identify possible port scanning?**

By examining repeated connection attempts, SYN packets, destination ports, and connection responses.

**Q6. What is DNS tunneling?**

It is a technique that carries encoded data through DNS queries and responses.

**Q7. What is data exfiltration?**

It is the unauthorized transfer of data from a system or organization to an external destination.

**Q8. What is network beaconing?**

It is periodic communication between a device and a remote server, sometimes used by malware for C2 communication.

**Q9. Why is threat intelligence useful?**

It provides information about known malicious IP addresses, domains, file hashes, and attack infrastructure that can support investigations.

**Q10. Why should analysts correlate multiple security logs?**

Because a single indicator may have a legitimate explanation. Correlation provides stronger evidence and helps reduce false positives.

**Q11. What is the difference between threat hunting and threat detection?**

Threat detection commonly identifies suspicious activity through monitoring and alerts, while threat hunting proactively searches for threats that may not have triggered existing alerts.

**Q12. What should an analyst document during an investigation?**

The analyst should record the source and destination, timestamps, protocol, observed behavior, packet evidence, filters, assessment, and recommended next steps.

## 16. Quick Revision

| Concept | Key Point |
|---|---|
| Threat Hunting | Proactive search for hidden threats |
| IOC | Indicator of Compromise |
| C2 | Command and Control |
| DNS Tunneling | Encoded communication through DNS |
| Port Scanning | Discovering ports and services |
| Data Exfiltration | Unauthorized data transfer |
| Beaconing | Periodic communication with a remote server |
| Suspicious DNS | Unusual domain queries and response patterns |
| TLS Analysis | Investigating encrypted connection metadata |
| Traffic Correlation | Combining evidence from multiple sources |
| SIEM | Security Information and Event Management |
| EDR | Endpoint Detection and Response |
| PCAP | Packet Capture File |
| False Positive | Benign activity incorrectly identified as suspicious |

