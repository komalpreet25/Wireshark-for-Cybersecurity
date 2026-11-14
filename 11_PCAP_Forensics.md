# 11. PCAP Forensics

## 1. What is PCAP Forensics?

**PCAP Forensics** is the process of examining captured network packets to investigate security incidents, suspicious communication, and potential network attacks.

PCAP stands for **Packet Capture**. A PCAP file stores network packets collected during communication between devices.

Common capture file formats include:

- `.pcap` – Packet capture file format.
- `.pcapng` – PCAP Next Generation format, which supports additional metadata and capture information.

Security analysts use PCAP files to reconstruct network activity and understand what happened during a security incident.

### Why is PCAP forensics important?

- Investigating suspicious network connections.
- Identifying communication with potentially malicious IP addresses.
- Detecting unusual DNS queries.
- Investigating possible data exfiltration.
- Understanding malware communication patterns.
- Supporting incident response and digital investigations.

---

## 2. Sources of PCAP Evidence

PCAP files can be collected from different network monitoring sources.

| Source | Description |
|---|---|
| Wireshark | Captures network traffic from a selected interface |
| TShark | Captures packets through command-line tools |
| Network TAP | Provides a copy of network traffic |
| Port Mirroring | Copies traffic from selected switch ports |
| IDS/IPS | May provide packet captures associated with alerts |
| Firewall | May provide packet captures or traffic-related evidence |

**Important:** The available evidence depends on where and how the traffic was captured. A PCAP file may contain only part of a communication session.

---

## 3. PCAP Forensic Investigation Workflow

A structured investigation helps analysts avoid missing important evidence.

<escape>```text
PCAP File
    |
    v
Preserve Original Evidence
    |
    v
Identify Capture Details
    |
    v
Analyze Protocols and Endpoints
    |
    v
Investigate Suspicious Activity
    |
    v
Correlate Findings
    |
    v
Document Evidence
    |
    v
Prepare Investigation Report
```</escape>

### Step 1: Preserve the original evidence

Before analyzing a PCAP file:

- Keep the original capture unchanged.
- Create a working copy for analysis.
- Record the file name, size, and collection time.
- Calculate a cryptographic hash to help verify file integrity.

### Step 2: Identify capture details

Understand the available capture information:

- Capture start and end times.
- Number of packets.
- Capture interfaces, if recorded.
- Protocols present.
- Source and destination addresses.

### Step 3: Analyze network communication

Identify which devices communicated, which protocols were used, and whether any communication patterns appear unusual.

### Step 4: Investigate suspicious activity

Examine indicators such as:

- Unexpected external connections.
- Unusual DNS requests.
- Repeated connection attempts.
- Large outbound data transfers.
- Suspicious HTTP requests.
- Unusual communication timing.

### Step 5: Correlate findings

Compare packet-level evidence with other available information, such as firewall logs, endpoint alerts, DNS logs, and threat intelligence.

### Step 6: Document findings

Record the evidence, analysis, limitations, and conclusions in a clear investigation report.

---

## 4. Preserving PCAP Evidence

Evidence integrity is important during forensic investigations.

### Calculate a SHA-256 hash

On Linux:

```bash
sha256sum capture.pcapng
```

Example output:

```text
a1b2c3d4...  capture.pcapng
```

The displayed value is an illustrative example, not a real hash.

### Verify the file later

```bash
sha256sum capture.pcapng
```

Compare the newly calculated hash with the originally recorded hash.

If the hashes match, the file's contents are consistent with the original hashed version.

**Important:** A matching hash supports integrity verification, but it does not independently prove who collected the file, when it was collected, or whether the original capture was complete.

---

## 5. Initial PCAP Examination

Start by understanding the overall contents of the capture.

### Open a PCAP file

```bash
wireshark capture.pcapng
```

Or use TShark:

```bash
tshark -r capture.pcapng
```

### Display packet count

```bash
capinfos capture.pcapng
```

`capinfos` provides information about capture files, including packet counts, file size, capture duration, and available metadata.

### Identify protocols

```bash
tshark -r capture.pcapng -q -z io,phs
```

This displays a protocol hierarchy summary.

### Identify IP endpoints

```bash
tshark -r capture.pcapng -q -z endpoints,ip
```

### Identify TCP conversations

```bash
tshark -r capture.pcapng -q -z conv,tcp
```

These commands help establish which protocols and endpoints are present before deeper investigation.

---

## 6. Investigating Suspicious IP Addresses

Unexpected communication with external IP addresses may require investigation.

### Display IPv4 traffic

```wireshark
ip
```

### Filter traffic involving a particular IP

```wireshark
ip.addr == 203.0.113.10
```

### Display outbound traffic from an internal host

```wireshark
ip.src == 192.168.1.10
```

### Display traffic to an external destination

```wireshark
ip.dst == 203.0.113.10
```

### What should an analyst investigate?

- Is the destination expected?
- Which protocol and port are being used?
- How frequently does the host communicate with it?
- Is the connection repeated at regular intervals?
- Is the destination associated with known suspicious activity?
- Does the communication match normal business activity?

**Note:** The IP addresses above are reserved documentation examples. They are not real threat indicators.

An unfamiliar IP address does not automatically indicate malicious activity. Validate it using additional evidence.

---

## 7. Investigating Suspicious DNS Activity

DNS analysis can help identify unusual domain lookups and possible malware communication.

### Display DNS traffic

```wireshark
dns
```

### Display DNS queries

```wireshark
dns.flags.response == 0
```

### Display DNS responses

```wireshark
dns.flags.response == 1
```

### Find unusually long DNS query names

```wireshark
dns.qry.name.len > 50
```

### Investigate failed DNS responses

```wireshark
dns.flags.rcode != 0
```

### Possible warning signs

- Random-looking domain names.
- Repeated DNS queries to unusual domains.
- Unexpected domain generation patterns.
- High volumes of DNS requests.
- Unusual query lengths.
- Frequent failed DNS resolutions.

### DNS tunneling investigation

DNS tunneling is a technique in which DNS queries and responses are used to carry data that may otherwise be transmitted through another communication channel.

Analysts may investigate:

- Long or unusual DNS query names.
- High query frequency.
- Repeated subdomain patterns.
- Unusual DNS record types.
- Unexpected volumes of DNS traffic.

These indicators require context. Long domain names or frequent queries can also occur during legitimate services.

---

## 8. Investigating Port Scanning Activity

Port scanning involves checking network ports to identify available services.

### Display TCP SYN packets

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### Display TCP reset packets

```wireshark
tcp.flags.reset == 1
```

### Display traffic to common service ports

```wireshark
tcp.port == 22 || tcp.port == 80 || tcp.port == 443
```

### What should an analyst look for?

- One source contacting many destination ports.
- Repeated connection attempts.
- Multiple unanswered connection requests.
- TCP reset responses.
- Connections to unexpected services.

### Possible scanning patterns

| Pattern | Possible interpretation |
|---|---|
| Many ports contacted on one host | Possible port scan |
| One port contacted across many hosts | Possible service discovery |
| Repeated SYN packets | Possible connection probing |
| Multiple reset responses | Ports may be closed or connections rejected |

A scan may be authorized security testing, routine monitoring, or malicious reconnaissance. Confirm the source and context before drawing conclusions.

---

## 9. Investigating Possible Data Exfiltration

**Data exfiltration** means unauthorized transfer of data from an organization or system to an external destination.

PCAP analysis can help identify communication patterns that may indicate data exfiltration.

### Display outbound traffic

```wireshark
ip.src == 192.168.1.10 && ip.dst != 192.168.1.10
```

This is only a basic example. In a real network, internal and external address ranges should be defined correctly.

### Investigate large packets

```wireshark
frame.len > 1400
```

### Investigate HTTP POST requests

```wireshark
http.request.method == "POST"
```

### Potential warning signs

- Unexpected large outbound transfers.
- Unusual destinations.
- Repeated uploads.
- Communication outside normal working hours.
- Unexpected use of cloud storage or file-sharing services.
- Large or repeated DNS requests.

**Important:** Large packets or outbound traffic do not prove data theft. Legitimate backups, cloud synchronization, software updates, and file transfers can generate similar patterns.

---

## 10. Investigating Suspicious HTTP and TLS Traffic

HTTP and TLS traffic can provide useful information about application communication.

### Display HTTP requests

```wireshark
http.request
```

### Display HTTP POST requests

```wireshark
http.request.method == "POST"
```

### Display unsuccessful HTTP responses

```wireshark
http.response.code >= 400
```

### Display TLS handshake traffic

```wireshark
tls.handshake
```

### Display TLS alerts

```wireshark
tls.alert_message
```

### Investigate the following details

- Requested hostnames.
- HTTP methods and response codes.
- Unexpected user-agent values.
- Suspicious URL paths.
- TLS certificate information.
- Unexpected external connections.
- Unusual connection frequency.

### Limitations of encrypted traffic

HTTPS protects application data using encryption.

Without appropriate decryption keys, analysts generally cannot read the encrypted application content directly from a PCAP.

However, depending on the TLS version and available metadata, analysts may still examine:

- Source and destination IP addresses.
- Ports.
- Connection timing and duration.
- Packet sizes.
- TLS handshake information.
- Certificate details when available.

---

## 11. Identifying Possible Malware Communication

Malware may communicate with external infrastructure to receive commands, transmit information, or download additional files.

This communication is sometimes called **Command and Control (C2)** traffic.

### Possible C2 indicators

- Repeated connections to an unusual destination.
- Communication at regular intervals.
- Unexpected DNS queries.
- Unusual outbound connections.
- Suspicious HTTP or TLS behavior.
- Repeated small data exchanges.

### Investigating beaconing behavior

Beaconing refers to repeated communication between an infected system and a remote destination at regular or near-regular intervals.

For example:

```text
10:00:00  Connection
10:05:00  Connection
10:10:00  Connection
10:15:00  Connection
```

Regular timing can be a useful investigation clue, but it is not proof of malware. Scheduled tasks and legitimate monitoring services can also communicate periodically.

---

## 12. Reconstructing Network Communication

Wireshark provides features that help analysts understand the sequence of communication.

### Follow TCP Stream

1. Open the PCAP file.
2. Select a TCP packet.
3. Right-click the packet.
4. Select **Follow → TCP Stream**.
5. Examine the reconstructed communication.

### Conversation Statistics

Navigate to:

**Statistics → Conversations**

This helps identify communication between network endpoints.

### Endpoint Statistics

Navigate to:

**Statistics → Endpoints**

This displays information about devices observed in the capture.

### Flow Graph

Navigate to:

**Statistics → Flow Graph**

A flow graph shows communication sequences between endpoints and can help explain how packets moved between systems.

---

## 13. Evidence Collection and Documentation

A forensic investigation should be documented clearly so that another analyst can understand and reproduce the findings.

### Important evidence details

| Evidence | Purpose |
|---|---|
| PCAP filename | Identifies the source capture |
| SHA-256 hash | Helps verify file integrity |
| Capture time | Establishes the recorded time period |
| Source IP | Identifies the communicating host |
| Destination IP | Identifies the remote endpoint |
| Protocol | Identifies the communication protocol |
| Source and destination ports | Helps identify services |
| Packet number | Helps locate the relevant packet |
| Timestamp | Helps establish event sequence |
| Observations | Records analyst findings |

### Example investigation note

```text
Incident: Unusual DNS Activity

Source Host: 192.168.1.10
Protocol: DNS
Observation: Repeated DNS queries to an unfamiliar domain.
Evidence: DNS packets observed in the PCAP.
Analysis: Query frequency and domain structure require further review.
Conclusion: Suspicious activity is possible but not confirmed.
Recommended Action: Correlate with endpoint and DNS logs.
```

The example is illustrative. It does not represent a confirmed security incident.

---

## 14. PCAP Forensics Limitations

PCAP analysis is valuable, but it does not provide complete visibility into every security event.

| Limitation | Explanation |
|---|---|
| Encrypted traffic | Application contents may not be readable |
| Incomplete capture | Some packets may be missing |
| Packet loss | Capture systems may fail to record every packet |
| Limited capture point | Traffic outside the monitored interface may not be visible |
| NAT | Private hosts may appear behind a shared public IP |
| Dynamic IP addresses | Address ownership may change over time |
| Missing context | A packet alone may not explain the full incident |
| Spoofed addresses | Source IP information may be misleading in some situations |

**Key principle:** A PCAP provides evidence of observed network traffic, not a complete record of everything that happened on a system.

---

## 15. Practical Lab: Investigating a Suspicious PCAP

**Objective:** Perform a basic forensic investigation using a saved packet capture.

Use a PCAP file from a trusted educational lab or an authorized environment.

### Step 1: Calculate the file hash

```bash
sha256sum sample.pcapng
```

Record the hash before analysis.

### Step 2: Inspect capture information

```bash
capinfos sample.pcapng
```

Record the packet count, duration, and file size.

### Step 3: Identify protocols

```bash
tshark -r sample.pcapng -q -z io,phs
```

### Step 4: Identify IP endpoints

```bash
tshark -r sample.pcapng -q -z endpoints,ip
```

### Step 5: Investigate DNS queries

```bash
tshark -r sample.pcapng -Y "dns.flags.response == 0" -T fields -e dns.qry.name
```

### Step 6: Investigate TCP conversations

```bash
tshark -r sample.pcapng -q -z conv,tcp
```

### Step 7: Check for suspicious HTTP activity

```bash
tshark -r sample.pcapng -Y "http.request" -T fields -e ip.src -e http.host -e http.request.method -e http.request.uri
```

### Step 8: Document your findings

Prepare a short report containing:

1. Capture file details.
2. Protocols identified.
3. Important source and destination IP addresses.
4. Unusual DNS activity, if any.
5. Suspicious communication patterns, if any.
6. Supporting packet numbers and timestamps.
7. Investigation limitations.
8. Recommended next steps.

### Expected learning outcome

Understand how to preserve a packet capture, investigate network communication, identify potential security indicators, and document findings without treating every unusual packet as a confirmed attack.

---

## 16. Interview Questions

**Q1. What is PCAP forensics?**

PCAP forensics is the analysis of captured network packets to investigate security incidents and understand network activity.

**Q2. What is the difference between PCAP and PCAPNG?**

PCAP is a packet capture format, while PCAPNG is a newer format that supports additional metadata and multiple capture interfaces.

**Q3. Why is hashing important in forensic investigations?**

Hashing helps verify that a file has not changed since its hash was originally recorded.

**Q4. What is data exfiltration?**

Data exfiltration is the unauthorized transfer of information from a system or network to an external destination.

**Q5. What is Command and Control traffic?**

C2 traffic is communication between malware or a compromised system and an attacker's remote infrastructure.

**Q6. What is beaconing?**

Beaconing is repeated communication with a remote destination at regular or near-regular intervals.

**Q7. Can Wireshark read HTTPS traffic?**

Wireshark can analyze TLS metadata and encrypted packets, but reading encrypted application content generally requires appropriate decryption keys.

**Q8. What is the purpose of Follow TCP Stream?**

It reconstructs TCP communication to help analysts understand the data exchanged between endpoints.

**Q9. Does a suspicious IP address confirm a cyberattack?**

No. An IP address must be investigated and correlated with other evidence before reaching a conclusion.

**Q10. Why should the original PCAP file be preserved?**

Preserving the original helps maintain evidence integrity and allows other analysts to verify the investigation.

---

## 17. Quick Revision

| Term | Meaning |
|---|---|
| PCAP | Packet Capture |
| PCAPNG | PCAP Next Generation |
| SHA-256 | Cryptographic hash algorithm |
| IOC | Indicator of Compromise |
| C2 | Command and Control |
| DNS tunneling | Using DNS traffic to carry data |
| Data exfiltration | Unauthorized data transfer |
| Beaconing | Repeated communication with a remote destination |
| TCP stream | Sequence of TCP communication |
| Packet forensics | Investigating captured network packets |

### Important Commands

| Command | Purpose |
|---|---|
| `sha256sum file.pcapng` | Calculate SHA-256 hash |
| `capinfos file.pcapng` | Inspect capture information |
| `tshark -r file.pcapng` | Read packets |
| `tshark -r file.pcapng -q -z io,phs` | Show protocol hierarchy |
| `tshark -r file.pcapng -q -z endpoints,ip` | Display IP endpoints |
| `tshark -r file.pcapng -q -z conv,tcp` | Display TCP conversations |

**Key takeaway:** PCAP forensics helps cybersecurity analysts turn captured network traffic into useful investigation evidence. Proper evidence preservation, careful analysis, correlation, and clear documentation are essential for reliable conclusions.
