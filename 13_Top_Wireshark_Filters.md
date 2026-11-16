# 13. Top Wireshark Filters

## Introduction

Wireshark filters help cybersecurity analysts find specific packets within large amounts of network traffic.

There are two main types of filters:

1. **Capture Filters:** Select which packets Wireshark collects during a capture.
2. **Display Filters:** Select which packets are shown from an existing capture or live capture.

Understanding both filter types is essential for packet analysis, network troubleshooting, SOC operations, and threat hunting.

---

# 1. Basic Display Filters

| Filter | Purpose |
|---|---|
| `frame` | Display all captured frames |
| `ip` | Display IPv4 traffic |
| `ipv6` | Display IPv6 traffic |
| `tcp` | Display TCP packets |
| `udp` | Display UDP packets |
| `icmp` | Display ICMP traffic |
| `arp` | Display ARP packets |
| `dns` | Display DNS traffic |
| `http` | Display HTTP traffic |
| `tls` | Display TLS traffic |
| `dhcp` | Display DHCP traffic |
| `ssh` | Display SSH traffic |
| `ftp` | Display FTP traffic |
| `smtp` | Display SMTP traffic |

---

# 2. IP Address Filters

These filters help identify communication between network devices.

| Filter | Purpose |
|---|---|
| `ip.addr == 192.168.1.10` | Traffic involving a specific IP |
| `ip.src == 192.168.1.10` | Traffic originating from an IP |
| `ip.dst == 192.168.1.10` | Traffic destined for an IP |
| `ip.addr == 192.168.1.10 && tcp` | TCP traffic involving an IP |
| `ip.src == 192.168.1.10 && ip.dst == 192.168.1.20` | Traffic between two specific IPs |
| `ip.ttl < 10` | IPv4 packets with TTL below 10 |
| `ip.flags.mf == 1` | IPv4 packets with the More Fragments flag set |
| `ip.frag_offset > 0` | IPv4 packets with a nonzero fragment offset |

Replace example addresses with addresses from your own capture.

---

# 3. TCP Filters

TCP filters help investigate connections, handshakes, retransmissions, and connection problems.

| Filter | Purpose |
|---|---|
| `tcp` | All TCP traffic |
| `tcp.port == 80` | TCP traffic on port 80 |
| `tcp.port == 443` | TCP traffic on port 443 |
| `tcp.srcport == 22` | TCP packets from source port 22 |
| `tcp.dstport == 80` | TCP packets to destination port 80 |
| `tcp.flags.syn == 1` | TCP packets with SYN flag set |
| `tcp.flags.ack == 1` | TCP packets with ACK flag set |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Initial SYN packets |
| `tcp.flags.fin == 1` | TCP packets with FIN flag set |
| `tcp.flags.reset == 1` | TCP reset packets |
| `tcp.analysis.retransmission` | TCP retransmissions |
| `tcp.analysis.duplicate_ack` | Duplicate acknowledgments |
| `tcp.analysis.lost_segment` | Packets indicating a possible missing segment |
| `tcp.analysis.window_full` | TCP window-full events |
| `tcp.stream == 0` | Packets belonging to TCP stream 0 |

### TCP Three-Way Handshake

The TCP three-way handshake establishes a connection between two endpoints.

```text
Client                    Server
  |                          |
  |-------- SYN ------------>|
  |                          |
  |<----- SYN + ACK ----------|
  |                          |
  |-------- ACK ------------>|
  |                          |
  |     Connection Ready     |
```

Useful filters:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

```wireshark
tcp.flags.ack == 1 && tcp.flags.syn == 0
```

The last filter matches many ACK packets, not exclusively the final handshake packet. Use packet sequence and connection context to identify the handshake.

---

# 4. UDP Filters

UDP filters are useful for analyzing DNS, DHCP, streaming, and other connectionless traffic.

| Filter | Purpose |
|---|---|
| `udp` | All UDP traffic |
| `udp.port == 53` | UDP traffic on port 53 |
| `udp.port == 67` | UDP traffic on port 67 |
| `udp.port == 68` | UDP traffic on port 68 |
| `udp.srcport == 53` | UDP packets from source port 53 |
| `udp.dstport == 53` | UDP packets to destination port 53 |
| `udp.length > 1000` | UDP datagrams longer than 1000 bytes |

---

# 5. DNS Filters

DNS filters are particularly useful for investigating domain resolution, suspicious queries, and possible DNS tunneling.

| Filter | Purpose |
|---|---|
| `dns` | All DNS traffic |
| `dns.flags.response == 0` | DNS queries |
| `dns.flags.response == 1` | DNS responses |
| `dns.qry.name` | Packets containing a DNS query name |
| `dns.qry.name == "example.com"` | Queries for a specific domain |
| `dns.qry.name contains "google"` | Query names containing a string |
| `dns.qry.name.len > 50` | DNS query names longer than 50 characters |
| `dns.flags.rcode != 0` | DNS responses with a nonzero response code |
| `dns.count.answers == 0` | DNS packets with zero answer records |
| `dns.qry.type == 1` | A-record queries |
| `dns.qry.type == 28` | AAAA-record queries |
| `dns.qry.type == 16` | TXT-record queries |

### Security Investigation

Possible warning signs include:

- Long or unusual domain names.
- Repeated queries to unfamiliar domains.
- High DNS query frequency.
- Unexpected TXT-record activity.
- Repeated DNS failures.

These patterns may justify further investigation but do not independently prove malicious activity.

---

# 6. HTTP Filters

HTTP filters help inspect unencrypted web communication.

| Filter | Purpose |
|---|---|
| `http` | HTTP traffic |
| `http.request` | HTTP requests |
| `http.response` | HTTP responses |
| `http.request.method == "GET"` | GET requests |
| `http.request.method == "POST"` | POST requests |
| `http.response.code == 200` | HTTP 200 responses |
| `http.response.code >= 400` | HTTP client/server error responses |
| `http.host` | Packets containing an HTTP Host field |
| `http.request.uri` | Packets containing an HTTP request URI |
| `http.user_agent` | Packets containing a User-Agent field |
| `http.content_type` | Packets containing a Content-Type field |

### Security Investigation

Analysts may investigate:

- Unexpected HTTP POST requests.
- Suspicious URL paths.
- Unusual user-agent values.
- Unexpected file downloads.
- Unencrypted sensitive information.

HTTP fields are only visible when the traffic is dissected as HTTP. HTTPS application content is normally encrypted.

---

# 7. TLS and HTTPS Filters

TLS filters help analyze encrypted web connections and handshake activity.

| Filter | Purpose |
|---|---|
| `tls` | TLS traffic |
| `tls.handshake` | TLS handshake packets |
| `tls.handshake.type == 1` | Client Hello |
| `tls.handshake.type == 2` | Server Hello |
| `tls.handshake.type == 11` | Certificate handshake message |
| `tls.alert_message` | TLS alert messages |
| `tls.handshake.extensions_server_name` | Packets containing a TLS server-name extension |
| `tls.record.content_type == 23` | TLS application-data records |

### Security Investigation

Inspect:

- TLS handshake information.
- Certificate details, when available.
- Negotiated protocol versions.
- Unexpected external connections.
- Repeated TLS alerts.
- Connection timing and traffic volume.

**Note:** Encrypted TLS application data cannot normally be read as plaintext without suitable decryption keys. Server Name Indication (SNI) may also be unavailable or protected in some connections.

---

# 8. ARP and ICMP Filters

### ARP Filters

| Filter | Purpose |
|---|---|
| `arp` | All ARP traffic |
| `arp.opcode == 1` | ARP requests |
| `arp.opcode == 2` | ARP replies |
| `arp.src.proto_ipv4 == 192.168.1.10` | ARP packets with a specific sender IPv4 address |
| `arp.dst.proto_ipv4 == 192.168.1.1` | ARP packets with a specific target IPv4 address |

### ICMP Filters

| Filter | Purpose |
|---|---|
| `icmp` | All IPv4 ICMP traffic |
| `icmp.type == 8` | ICMP Echo Requests |
| `icmp.type == 0` | ICMP Echo Replies |
| `icmp.type == 3` | Destination Unreachable |
| `icmp.type == 11` | Time Exceeded |
| `icmpv6` | IPv6 ICMP traffic |

### Security Relevance

These filters can help investigate:

- ARP resolution activity.
- Possible ARP spoofing.
- Network connectivity problems.
- Ping activity.
- Unreachable destinations.
- Network discovery behavior.

---

# 9. Port-Based Filters

Port filters help identify services communicating over the network.

| Filter | Purpose |
|---|---|
| `tcp.port == 21` | FTP |
| `tcp.port == 22` | SSH |
| `tcp.port == 23` | Telnet |
| `tcp.port == 25` | SMTP |
| `tcp.port == 53` | TCP DNS |
| `tcp.port == 80` | HTTP |
| `tcp.port == 110` | POP3 |
| `tcp.port == 143` | IMAP |
| `tcp.port == 443` | HTTPS |
| `tcp.port == 445` | SMB |
| `tcp.port == 3389` | RDP |
| `udp.port == 53` | UDP DNS |
| `udp.port == 67 || udp.port == 68` | DHCP |

**Important:** Port numbers indicate commonly assigned services, not guaranteed application identity. Applications can use nonstandard ports.

---

# 10. Suspicious Traffic Investigation Filters

These filters can help identify traffic that deserves closer examination.

| Filter | Investigation purpose |
|---|---|
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Initial TCP connection attempts |
| `tcp.flags.reset == 1` | Connection resets |
| `tcp.analysis.retransmission` | Retransmission investigation |
| `dns.qry.name.len > 50` | Unusually long DNS query names |
| `http.request.method == "POST"` | HTTP data-submission requests |
| `frame.len > 1400` | Large captured frames |
| `ip.ttl <= 2` | Very low IPv4 TTL values |
| `tcp.port == 4444` | Traffic involving a commonly observed test or alternate port |
| `tls.alert_message` | TLS alert investigation |
| `icmp` | ICMP activity investigation |

**Remember:** These are investigation filters, not automatic attack-detection rules. Their results need context and correlation.

---

# 11. Combined Display Filters

Combined filters use logical operators to narrow down results.

| Operator | Meaning |
|---|---|
| `&&` | AND |
| `||` | OR |
| `!` | NOT |
| `==` | Equal to |
| `!=` | Not equal to |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal to |
| `<=` | Less than or equal to |

### Examples

**1. DNS or HTTP traffic**

```wireshark
dns || http
```

**2. TCP traffic on port 80 or 443**

```wireshark
tcp.port == 80 || tcp.port == 443
```

**3. Traffic from a specific IP using TCP**

```wireshark
ip.src == 192.168.1.10 && tcp
```

**4. Exclude DNS traffic**

```wireshark
!dns
```

**5. HTTP POST requests with an error response**

```wireshark
http.request.method == "POST" && http.response.code >= 400
```

This filter matches packets that contain both fields. Since requests and responses are generally separate packets, it will not necessarily show the POST request and its corresponding error response together. Analyze the related stream or request/response context to connect them.

---

# 12. Top Capture Filters

Capture filters use Berkeley Packet Filter (BPF) syntax.

Unlike display filters, they determine which packets are collected.

| Capture Filter | Purpose |
|---|---|
| `tcp` | Capture TCP packets |
| `udp` | Capture UDP packets |
| `icmp` | Capture ICMP packets |
| `arp` | Capture ARP packets |
| `host 192.168.1.10` | Capture traffic to or from a host |
| `src host 192.168.1.10` | Capture traffic from a host |
| `dst host 192.168.1.10` | Capture traffic to a host |
| `port 53` | Capture traffic on port 53 |
| `tcp port 443` | Capture TCP traffic on port 443 |
| `udp port 53` | Capture UDP traffic on port 53 |
| `net 192.168.1.0/24` | Capture traffic to or from a network |
| `not port 22` | Exclude traffic on port 22 |
| `tcp and port 80` | Capture TCP traffic on port 80 |
| `udp and port 53` | Capture UDP traffic on port 53 |

### Capture Filter Examples

**Capture traffic from a specific host:**

```text
host 192.168.1.10
```

**Capture only TCP traffic:**

```text
tcp
```

**Capture DNS traffic:**

```text
port 53
```

**Exclude SSH traffic:**

```text
not port 22
```

**Capture traffic from a network:**

```text
net 192.168.1.0/24
```

---

# 13. Useful Filters for SOC Analysts

SOC analysts use filters to investigate alerts and understand network behavior.

| Investigation | Suggested Filter |
|---|---|
| Identify DNS queries | `dns.flags.response == 0` |
| Investigate TCP handshakes | `tcp.flags.syn == 1` |
| Investigate TCP resets | `tcp.flags.reset == 1` |
| Find retransmissions | `tcp.analysis.retransmission` |
| Inspect HTTP requests | `http.request` |
| Inspect TLS handshakes | `tls.handshake` |
| Identify ICMP traffic | `icmp` |
| Investigate ARP traffic | `arp` |
| Find large frames | `frame.len > 1400` |
| Inspect traffic to an IP | `ip.addr == 192.168.1.10` |
| Inspect a TCP stream | `tcp.stream == 0` |

### Example SOC Investigation Workflow

1. Identify the source IP from a security alert.
2. Filter traffic involving that IP.
3. Identify the protocols and ports used.
4. Investigate DNS queries and external destinations.
5. Review TCP connections and retransmissions.
6. Examine HTTP or TLS metadata where available.
7. Correlate findings with endpoint and firewall logs.
8. Document supporting evidence and conclusions.

---

# 14. Important Wireshark Filter Fields

Some useful fields to remember:

| Field | Information |
|---|---|
| `frame.time` | Packet timestamp |
| `frame.len` | Frame length |
| `frame.number` | Frame number |
| `eth.src` | Ethernet source MAC |
| `eth.dst` | Ethernet destination MAC |
| `ip.src` | Source IPv4 address |
| `ip.dst` | Destination IPv4 address |
| `ip.ttl` | IPv4 Time To Live |
| `tcp.srcport` | TCP source port |
| `tcp.dstport` | TCP destination port |
| `tcp.flags` | TCP flags |
| `udp.srcport` | UDP source port |
| `udp.dstport` | UDP destination port |
| `dns.qry.name` | DNS query name |
| `http.host` | HTTP Host field |
| `http.request.method` | HTTP request method |
| `tls.handshake.type` | TLS handshake message type |

---

# 15. Common Filter Mistakes

| Mistake | Correct approach |
|---|---|
| Using `tcp.port = 443` | Use `tcp.port == 443` |
| Using `tcp.port == 443` as a capture filter | Use `tcp port 443` |
| Using `port 53` as a display filter | Use `udp.port == 53` or `tcp.port == 53` |
| Confusing `ip.src` with `ip.dst` | Check the direction of communication |
| Assuming port 443 always means HTTPS | Verify protocol dissection and connection details |
| Assuming every SYN packet is a scan | Analyze connection patterns and context |
| Assuming encrypted data is readable | Understand TLS encryption limitations |

---

# 16. Quick Revision: Most Useful 20 Filters

These are useful filters to remember for regular packet analysis.

| No. | Filter | Purpose |
|---:|---|---|
| 1 | `ip` | IPv4 traffic |
| 2 | `ipv6` | IPv6 traffic |
| 3 | `tcp` | TCP traffic |
| 4 | `udp` | UDP traffic |
| 5 | `dns` | DNS traffic |
| 6 | `http` | HTTP traffic |
| 7 | `tls` | TLS traffic |
| 8 | `arp` | ARP traffic |
| 9 | `icmp` | ICMP traffic |
| 10 | `ip.addr == 192.168.1.10` | Traffic involving an IP |
| 11 | `tcp.port == 443` | TCP port 443 |
| 12 | `udp.port == 53` | UDP port 53 |
| 13 | `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Initial SYN packets |
| 14 | `tcp.flags.reset == 1` | TCP reset packets |
| 15 | `tcp.analysis.retransmission` | Retransmissions |
| 16 | `dns.flags.response == 0` | DNS queries |
| 17 | `http.request` | HTTP requests |
| 18 | `tls.handshake` | TLS handshake |
| 19 | `frame.len > 1400` | Large frames |
| 20 | `tcp.stream == 0` | TCP stream 0 |

---

## Final Takeaway

Wireshark filters are essential for efficient packet analysis. Instead of manually inspecting thousands of packets, analysts can use specific filters to identify relevant communication, investigate network behavior, and support security investigations.

**Learning tip:** Practice writing filters yourself instead of only memorizing them. Understanding what each field represents will make packet analysis much easier.
