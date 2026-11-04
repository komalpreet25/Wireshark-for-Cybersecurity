# 03 — Capture and Display Filters

## 1. Introduction to Wireshark Filters

Wireshark captures network traffic that is visible to the selected interface. A busy network can generate thousands or even millions of packets.

Analyzing every packet manually is inefficient.

**Wireshark filters** help analysts narrow down network traffic and focus on specific protocols, IP addresses, ports, or packet characteristics.

Wireshark provides two main types of filters:

1. Capture Filters
2. Display Filters

Understanding the difference between them is essential for effective packet analysis.

## 2. Capture Filters

A **capture filter** determines which packets Wireshark collects during a live capture.

Packets that do not match the capture filter are discarded from that capture.

Capture filters use the Berkeley Packet Filter (BPF) syntax, commonly provided through libpcap-compatible capture mechanisms.

### Where to Apply Capture Filters

1. Open Wireshark.
2. Select a network interface.
3. Open Capture Options.
4. Locate the Capture Filter field.
5. Enter the filter expression.
6. Start capturing.

### Common Capture Filters

| Capture Filter | Purpose |
|---|---|
| `host 192.168.1.10` | Capture traffic to or from an IP |
| `src host 192.168.1.10` | Capture traffic originating from an IP |
| `dst host 192.168.1.10` | Capture traffic going to an IP |
| `net 192.168.1.0/24` | Capture traffic involving a network |
| `tcp` | Capture TCP traffic |
| `udp` | Capture UDP traffic |
| `icmp` | Capture ICMP traffic |
| `port 80` | Capture traffic using port 80 |
| `port 443` | Capture traffic using port 443 |
| `tcp port 22` | Capture TCP traffic using port 22 |
| `udp port 53` | Capture UDP traffic using port 53 |
| `src port 443` | Capture packets with source port 443 |
| `dst port 53` | Capture packets with destination port 53 |

### Combining Capture Filters

Capture filters support logical operators.

**AND — `and`**

```text
tcp and port 443
```

Captures TCP traffic involving port 443.

**OR — `or`**

```text
tcp or udp
```

Captures TCP or UDP traffic.

**NOT — `not`**

```text
not port 22
```

Excludes traffic involving port 22.

**Combined Example**

```text
host 192.168.1.10 and tcp port 443
```

Captures TCP traffic involving port 443 where the specified host is either the source or destination.

### Important Capture Filter Limitation

Capture filters cannot be applied retrospectively to packets that have already been discarded.

For example, if you capture only TCP traffic, you cannot later recover DNS packets that were excluded from that capture.

Use broad captures when you are unsure which protocols will be relevant.

## 3. Display Filters

A **display filter** controls which packets are shown in Wireshark after they have been captured.

Unlike capture filters, display filters do not delete or remove packets from the original capture.

You can change display filters repeatedly while investigating the same capture file.

### Where to Apply Display Filters

1. Open Wireshark.
2. Start a capture or open an existing PCAP file.
3. Locate the Display Filter bar.
4. Enter a valid filter expression.
5. Press Enter.

### Common Display Filters

| Display Filter | Purpose |
|---|---|
| `tcp` | Show TCP packets |
| `udp` | Show UDP packets |
| `icmp` | Show ICMP packets |
| `arp` | Show ARP packets |
| `dns` | Show DNS packets |
| `http` | Show HTTP packets |
| `tls` | Show TLS packets |
| `ip` | Show IPv4 packets |
| `ipv6` | Show IPv6 packets |
| `tcp.port == 443` | Show TCP packets involving port 443 |
| `udp.port == 53` | Show UDP packets involving port 53 |
| `ip.addr == 192.168.1.10` | Show IPv4 packets involving an address |
| `ip.src == 192.168.1.10` | Show IPv4 packets from a source address |
| `ip.dst == 192.168.1.20` | Show IPv4 packets to a destination address |

## 4. Capture Filters vs Display Filters

| Feature | Capture Filter | Display Filter |
|---|---|---|
| Applied | Before and during capture | After packets are captured |
| Purpose | Controls which packets are collected | Controls which packets are displayed |
| Syntax | BPF / libpcap syntax | Wireshark display-filter syntax |
| Can change during analysis? | No, not for the existing capture | Yes |
| Can recover excluded packets? | No | Only if they were originally captured |
| Example | `tcp port 443` | `tcp.port == 443` |

### Remember the Syntax Difference

Capture filter:

```text
host 192.168.1.10
```

Display filter:

```text
ip.addr == 192.168.1.10
```

Capture filter:

```text
port 53
```

Display filter:

```text
udp.port == 53
```

The two filtering languages are different. Their expressions are not interchangeable.

## 5. Display Filter Syntax

Display filters use protocol fields and comparison operators.

### A. Comparison Operators

| Operator | Meaning | Example |
|---|---|---|
| `==` | Equal to | `tcp.dstport == 443` |
| `!=` | Not equal to | `tcp.dstport != 443` |
| `>` | Greater than | `frame.len > 1000` |
| `<` | Less than | `frame.len < 100` |
| `>=` | Greater than or equal to | `tcp.window_size_value >= 1024` |
| `<=` | Less than or equal to | `ip.ttl <= 10` |

### B. Logical Operators

| Operator | Meaning | Example |
|---|---|---|
| `&&` | AND | `tcp && ip` |
| `||` | OR | `dns || http` |
| `!` | NOT | `!arp` |

Word-based forms such as `and`, `or`, and `not` are also supported in Wireshark display filters.

### C. Membership Operator

The `in` operator checks whether a field matches one of several values.

Example:

```text
tcp.port in {80 443 8080}
```

Displays TCP packets involving any of the listed ports.

### D. Existence Operator

A field can be checked to determine whether it exists in a packet.

Example:

```text
tcp.flags.syn
```

Displays packets containing the TCP SYN flag field, regardless of whether its value is set.

To check whether the SYN flag is set:

```text
tcp.flags.syn == 1
```

This distinction is important when writing precise filters.

## 6. Filtering by IP Address

IP filters are useful when investigating communication between specific devices.

### Source IP

```text
ip.src == 192.168.1.10
```

Displays IPv4 packets originating from the specified address.

### Destination IP

```text
ip.dst == 192.168.1.20
```

Displays IPv4 packets going to the specified address.

### Source or Destination IP

```text
ip.addr == 192.168.1.10
```

Displays IPv4 packets where the address appears as either source or destination.

### Multiple IP Addresses

```text
ip.addr == 192.168.1.10 || ip.addr == 192.168.1.20
```

Displays packets involving either address.

### Exclude an IP Address

```text
!(ip.addr == 192.168.1.10)
```

Displays packets that do not involve the specified IPv4 address.

### Filter by IPv6

```text
ipv6.addr == 2001:db8::1
```

Displays IPv6 packets involving the specified address.

## 7. Filtering by Port Number

Ports identify application endpoints and are useful for investigating specific services.

### Common Ports

| Port | Protocol / Service |
|---:|---|
| 20 | FTP Data |
| 21 | FTP Control |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 67 / 68 | DHCP |
| 80 | HTTP |
| 110 | POP3 |
| 123 | NTP |
| 143 | IMAP |
| 161 | SNMP |
| 443 | HTTPS |
| 445 | SMB |
| 3389 | RDP |

These are commonly assigned ports. A service can also operate on a non-standard port.

### Filter by Destination Port

```text
tcp.dstport == 443
```

### Filter by Source Port

```text
tcp.srcport == 443
```

### Filter by Either Source or Destination Port

```text
tcp.port == 443
```

### Filter Multiple Ports

```text
tcp.port in {22 80 443 3389}
```

### Exclude a Port

```text
tcp.port != 443
```

This filters packets where the TCP source and destination ports are not equal to 443.

For precise exclusion of all packets involving that port, use:

```text
!(tcp.port == 443)
```

## 8. TCP Filtering

TCP filters help analysts investigate connection establishment, termination, retransmissions, and abnormal communication.

### Show TCP Traffic

```text
tcp
```

### SYN Packets

```text
tcp.flags.syn == 1
```

### Initial SYN Packets

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

These commonly represent connection initiation attempts.

### SYN-ACK Packets

```text
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

### FIN Packets

```text
tcp.flags.fin == 1
```

### RST Packets

```text
tcp.flags.reset == 1
```

### Retransmissions

```text
tcp.analysis.retransmission
```

### Duplicate ACKs

```text
tcp.analysis.duplicate_ack
```

### Zero Window

```text
tcp.analysis.zero_window
```

These analysis flags can help identify network performance or communication issues. Their presence should be interpreted in context.

## 9. DNS Filtering

DNS translates domain names into IP addresses and is frequently examined during security investigations.

### Show DNS Traffic

```text
dns
```

### DNS Queries

```text
dns.flags.response == 0
```

### DNS Responses

```text
dns.flags.response == 1
```

### DNS Errors

```text
dns.flags.rcode != 0
```

### Specific Domain

```text
dns.qry.name == "example.com"
```

### DNS A Records

```text
dns.qry.type == 1
```

### DNS AAAA Records

```text
dns.qry.type == 28
```

### Investigating DNS Activity

Look for:

- Repeated queries to unusual domains.
- Unexpected DNS servers.
- Large numbers of failed lookups.
- Unusual query types.
- Suspicious domain-generation patterns.

Not all unusual DNS behavior is malicious. Validate findings using domain reputation, endpoint logs, and other available evidence.

## 10. HTTP and TLS Filtering

### HTTP Filters

Show HTTP traffic:

```text
http
```

Show HTTP GET requests:

```text
http.request.method == "GET"
```

Show HTTP POST requests:

```text
http.request.method == "POST"
```

Show HTTP responses:

```text
http.response
```

Show a particular host header:

```text
http.host == "example.com"
```

These filters work when traffic is decoded as HTTP and the relevant fields are available.

### TLS Filters

Show TLS traffic:

```text
tls
```

Show TLS handshake messages:

```text
tls.handshake
```

Show Client Hello messages:

```text
tls.handshake.type == 1
```

Show Server Hello messages:

```text
tls.handshake.type == 2
```

Show packets containing a TLS handshake server name:

```text
tls.handshake.extensions_server_name
```

TLS metadata can help analysts investigate connections, but encrypted application content is not normally readable without appropriate decryption material.

## 11. Advanced Display Filter Combinations

Combining multiple conditions helps narrow down investigations.

### A. Find HTTPS TCP Traffic from a Specific Host

```text
ip.src == 192.168.1.10 && tcp.dstport == 443
```

### B. Find DNS Traffic to a Specific Server

```text
ip.dst == 8.8.8.8 && dns
```

### C. Find Initial TCP Connection Attempts

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### D. Find Large TCP Frames

```text
tcp && frame.len > 1000
```

### E. Find TCP Retransmissions to a Specific Host

```text
tcp.analysis.retransmission && ip.addr == 192.168.1.10
```

### F. Exclude ARP and ICMP

```text
!arp && !icmp
```

### G. Find Traffic on Common Remote Access Ports

```text
tcp.port in {22 3389}
```

### H. Find DNS Queries for a Particular Domain

```text
dns.flags.response == 0 && dns.qry.name == "example.com"
```

## 12. Useful Filters for SOC Analysts

These filters are useful starting points for common network investigations.

| Investigation | Display Filter |
|---|---|
| Initial TCP connections | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| TCP resets | `tcp.flags.reset == 1` |
| TCP retransmissions | `tcp.analysis.retransmission` |
| DNS queries | `dns.flags.response == 0` |
| DNS errors | `dns.flags.rcode != 0` |
| HTTP POST requests | `http.request.method == "POST"` |
| TLS handshakes | `tls.handshake` |
| Large frames | `frame.len > 1000` |
| Traffic involving SSH | `tcp.port == 22` |
| Traffic involving SMB | `tcp.port == 445` |
| Traffic involving RDP | `tcp.port == 3389` |
| ICMP traffic | `icmp` |

**Security note:** A POST request, TCP SYN packet, retransmission, or connection to a remote access port is not automatically malicious. Investigate the source, destination, timing, frequency, and surrounding activity.

## 13. Capture Filter Examples for Practical Use

These examples use capture-filter syntax, not display-filter syntax.

### Capture Only DNS Traffic

```text
port 53
```

This captures traffic involving port 53, including DNS over UDP or TCP.

### Capture Only TCP Traffic

```text
tcp
```

### Capture Traffic from One Host

```text
src host 192.168.1.10
```

### Capture Traffic to a Specific Network

```text
dst net 192.168.1.0/24
```

### Capture HTTP and HTTPS Traffic

```text
tcp port 80 or tcp port 443
```

### Exclude SSH Traffic

```text
not tcp port 22
```

## 14. Practical Lab — Investigating Suspicious Traffic with Filters

**Objective:** Practise narrowing down network traffic using multiple display filters.

### Step 1: Obtain a Capture File

Use a PCAP or PCAPNG file from your own lab or an authorized training source.

### Step 2: Open the Capture

Open Wireshark and select:

**File → Open → Your Capture File**

### Step 3: Identify Protocols

Apply:

```text
dns || tcp || udp
```

Observe the different types of communication.

### Step 4: Investigate TCP Connections

Apply:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Observe the source and destination addresses and ports.

### Step 5: Investigate Retransmissions

Apply:

```text
tcp.analysis.retransmission
```

Check whether repeated retransmissions involve the same communication endpoints.

### Step 6: Investigate DNS Queries

Apply:

```text
dns.flags.response == 0
```

Observe queried domain names and query types.

### Step 7: Investigate Large Packets

Apply:

```text
frame.len > 1000
```

Inspect the protocols and endpoints associated with larger frames.

### Lab Observation Table

| Investigation | What Did You Observe? |
|---|---|
| DNS queries | |
| Initial TCP connections | |
| Retransmissions | |
| Unusual ports | |
| Large packets | |
| Suspicious patterns | |

### Expected Learning

After completing this lab, you should be able to:

- Differentiate capture and display filters.
- Filter traffic by protocol, IP, and port.
- Combine multiple conditions.
- Investigate TCP and DNS activity.
- Use filters to narrow down suspicious traffic.

## 15. Common Filtering Mistakes

| Mistake | Correct Approach |
|---|---|
| Using `tcp.port = 443` | Use `tcp.port == 443` |
| Using `ip.addr = 192.168.1.10` | Use `ip.addr == 192.168.1.10` |
| Entering a display filter in Capture Options | Use BPF capture-filter syntax |
| Using `http` to find encrypted HTTPS content | Inspect TLS metadata instead |
| Forgetting parentheses in complex expressions | Group conditions clearly |
| Assuming an empty result means no traffic exists | Check interface, capture scope, and protocol visibility |
| Treating every match as suspicious | Validate the surrounding packet context |

## 16. Interview Questions

**Q1. What are capture filters in Wireshark?**

Capture filters determine which packets are collected during a live capture using BPF-compatible syntax.

**Q2. What are display filters?**

Display filters control which already captured packets are shown during analysis.

**Q3. Can display filters modify the original capture?**

No. They change the displayed packet set, not the original captured data.

**Q4. What happens when a capture filter excludes a packet?**

That packet is not stored in that capture, so it cannot be recovered by changing a display filter later.

**Q5. What is the difference between `ip.addr` and `ip.src`?**

`ip.addr` matches an address appearing as either source or destination. `ip.src` matches only the source address.

**Q6. How can you find initial TCP connection attempts?**

Use:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

**Q7. How can you identify DNS queries?**

Use:

```text
dns.flags.response == 0
```

**Q8. How can you identify TCP retransmissions?**

Use:

```text
tcp.analysis.retransmission
```

**Q9. What is BPF?**

Berkeley Packet Filter is a mechanism and filter language used by packet-capture systems to select network packets.

**Q10. Why are filters important for SOC analysts?**

They help reduce packet noise, focus on relevant communication, investigate suspicious behavior, and improve the efficiency of network analysis.

## 17. Quick Revision

- Capture filters control which packets are collected.
- Display filters control which captured packets are displayed.
- Capture filters commonly use BPF syntax.
- Display filters use Wireshark protocol fields.
- `&&` means AND.
- `||` means OR.
- `!` means NOT.
- `ip.addr` matches source or destination.
- `tcp.flags.syn == 1 && tcp.flags.ack == 0` identifies initial SYN packets.
- `tcp.analysis.retransmission` identifies packets Wireshark marks as retransmissions.
- Filters help analysts investigate traffic more efficiently.
