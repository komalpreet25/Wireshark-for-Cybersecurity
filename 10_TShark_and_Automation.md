# 10. TShark and Automation

## 1. What is TShark?

**TShark** is the command-line version of Wireshark. It allows security analysts to capture, read, filter, and analyze network traffic directly from a terminal.

While Wireshark provides a graphical interface, TShark performs similar packet-analysis tasks through commands.

### Why use TShark?

- Capture network traffic without opening a graphical interface.
- Analyze existing PCAP and PCAPNG files.
- Apply capture and display filters.
- Extract specific packet fields.
- Generate network statistics.
- Process multiple capture files.
- Automate repetitive packet-analysis tasks.

**Cybersecurity relevance:** TShark is useful in SOC environments, network troubleshooting, incident response, and automated traffic analysis.

---

## 2. TShark vs Wireshark

| Feature | Wireshark | TShark |
|---|---|---|
| Interface | Graphical | Command-line |
| Packet capture | Yes | Yes |
| Packet filtering | Yes | Yes |
| Packet inspection | Visual | Terminal output |
| Statistics | GUI graphs and tools | Command options |
| Automation | Limited through GUI | Suitable for scripts |
| Best use | Interactive analysis | Command-line and batch analysis |

Both tools use the same underlying packet-dissector engine.

---

## 3. Installing and Checking TShark

TShark is commonly installed alongside Wireshark.

### Linux (Kali / Parrot)

```bash
sudo apt update
sudo apt install tshark
```

### Check the installed version

```bash
tshark --version
```

### Display available command options

```bash
tshark --help
```

---

## 4. Listing Network Interfaces

Before capturing packets, identify the available network interfaces.

```bash
tshark -D
```

Example output:

```text
1. eth0
2. wlan0
3. lo
```

Interface names depend on the operating system and network configuration.

### Start capturing on an interface

```bash
sudo tshark -i eth0
```

- `-i` specifies the network interface.
- `eth0` is the selected interface.
- `sudo` may be required for packet-capture permissions.

Press `Ctrl + C` to stop the capture.

---

## 5. Capturing Network Traffic

### Capture a limited number of packets

```bash
sudo tshark -i eth0 -c 100
```

The `-c` option stops the capture after the specified number of packets.

### Save captured packets to a file

```bash
sudo tshark -i eth0 -w capture.pcapng
```

The `-w` option writes captured packets to a file.

### Capture traffic for a specific port

```bash
sudo tshark -i eth0 -f "port 53"
```

This captures traffic matching the capture filter for port 53, commonly used by DNS.

### Capture and save DNS traffic

```bash
sudo tshark -i eth0 -f "port 53" -w dns_capture.pcapng
```

**Important:** Capture filters are applied while packets are being collected. Packets that do not match are not saved.

---

## 6. Reading Existing Capture Files

TShark can analyze previously saved PCAP or PCAPNG files.

### Read a capture file

```bash
tshark -r capture.pcapng
```

The `-r` option reads packets from an existing file.

### Display only the first 20 packets

```bash
tshark -r capture.pcapng -c 20
```

### Read packets using a display filter

```bash
tshark -r capture.pcapng -Y "dns"
```

### Display TCP traffic

```bash
tshark -r capture.pcapng -Y "tcp"
```

### Display traffic from a specific IP

```bash
tshark -r capture.pcapng -Y "ip.addr == 192.168.1.10"
```

---

## 7. Capture Filters vs Display Filters in TShark

TShark supports two important filtering options.

| Option | Filter type | Purpose |
|---|---|---|
| `-f` | Capture filter | Filters packets during capture |
| `-Y` | Display filter | Filters packets during analysis or reading |

### Capture filter example

```bash
sudo tshark -i eth0 -f "tcp port 443"
```

Captures TCP traffic on port 443.

### Display filter example

```bash
tshark -r capture.pcapng -Y "tcp.port == 443"
```

Displays packets matching the TCP port 443 filter.

**Remember:** Capture filters use BPF syntax, while display filters use Wireshark display-filter syntax.

---

## 8. Controlling Packet Output

TShark provides several options to control how packet information is displayed.

| Option | Purpose |
|---|---|
| `-V` | Displays detailed packet information |
| `-x` | Displays packet data in hexadecimal and ASCII |
| `-c` | Limits the number of packets |
| `-q` | Suppresses the normal per-packet output |
| `-T` | Selects an output format |

### Display detailed packet information

```bash
tshark -r capture.pcapng -V
```

### Display packet bytes

```bash
tshark -r capture.pcapng -x
```

### Display a summary without normal packet-by-packet output

```bash
tshark -r capture.pcapng -q
```

### Select JSON output

```bash
tshark -r capture.pcapng -T json
```

JSON output can be useful when packet information needs to be processed by other tools or scripts.

---

## 9. Extracting Specific Packet Fields

Instead of displaying complete packet details, TShark can extract selected fields.

Important options:

- `-T fields`: Selects field-based output.
- `-e`: Specifies a field to extract.
- `-E`: Controls field-output formatting.

### Extract source and destination IP addresses

```bash
tshark -r capture.pcapng -T fields -e ip.src -e ip.dst
```

### Extract TCP ports

```bash
tshark -r capture.pcapng -T fields -e tcp.srcport -e tcp.dstport
```

### Extract DNS query names

```bash
tshark -r capture.pcapng -Y "dns.flags.response == 0" -T fields -e dns.qry.name
```

### Extract HTTP host information

```bash
tshark -r capture.pcapng -Y "http.request" -T fields -e http.host
```

### Use tab-separated output

```bash
tshark -r capture.pcapng -T fields -E separator=, -e ip.src -e ip.dst -e tcp.dstport
```

This produces comma-separated field output that can be redirected into a CSV file. For complex data, use a proper CSV-processing method rather than assuming every field value is free of commas.

---

## 10. Generating Network Statistics

TShark can generate statistics that help analysts understand network activity.

### I/O statistics

```bash
tshark -r capture.pcapng -q -z io,stat,1
```

Displays packet and byte statistics in one-second intervals.

### TCP conversations

```bash
tshark -r capture.pcapng -q -z conv,tcp
```

Displays communication statistics for TCP conversations.

### IP endpoints

```bash
tshark -r capture.pcapng -q -z endpoints,ip
```

Displays IP endpoints observed in the capture.

### DNS statistics

```bash
tshark -r capture.pcapng -q -z dns,tree
```

Displays DNS statistics in a tree format.

**Why statistics matter:** They help identify heavily communicating hosts, unusual traffic volumes, and communication patterns that deserve further investigation.

---

## 11. Following TCP Streams

A TCP stream represents a sequence of TCP packets exchanged between communicating endpoints.

TShark can follow a TCP stream from a capture file.

### Display a TCP stream in ASCII format

```bash
tshark -r capture.pcapng -q -z follow,tcp,ascii,0
```

### Display a TCP stream in raw format

```bash
tshark -r capture.pcapng -q -z follow,tcp,raw,0
```

The final number identifies the stream index. Stream indexes can be inspected using TCP conversation information.

**Security use:** Following a stream can help investigate application communication, exposed credentials in unencrypted protocols, suspicious commands, or unexpected data exchanges.

Encrypted TLS application data generally cannot be read as normal plaintext simply by following the stream.

---

## 12. Exporting HTTP Objects

TShark can export supported objects from HTTP traffic in a capture file.

```bash
tshark -r capture.pcapng --export-objects http,./exported_files
```

This attempts to extract HTTP objects into the specified directory.

### Important points

- The destination directory should exist or be created beforehand.
- Export depends on the captured traffic and supported protocol dissection.
- HTTPS traffic is encrypted. Exporting HTTP objects does not automatically reveal encrypted HTTPS content.
- Exported files should be handled carefully because they may contain malicious content.

---

## 13. Basic TShark Automation

Automation means using commands or scripts to perform repeated analysis tasks with less manual effort.

### Example 1: Save DNS traffic automatically

```bash
#!/bin/bash

sudo tshark -i eth0 -f "port 53" -c 100 -w dns_traffic.pcapng

echo "DNS packet capture completed."
```

This script captures up to 100 packets matching the DNS port filter and saves them.

### Example 2: Extract IP addresses from a capture

```bash
tshark -r capture.pcapng \
-T fields \
-e ip.src \
-e ip.dst > ip_addresses.txt
```

This extracts source and destination IP addresses into a text file.

### Example 3: Extract DNS queries

```bash
tshark -r capture.pcapng \
-Y "dns.flags.response == 0" \
-T fields \
-e dns.qry.name > dns_queries.txt
```

This extracts DNS query names from the capture.

### Example 4: Analyze multiple capture files

```bash
for file in *.pcapng
do
    echo "Analyzing: $file"
    tshark -r "$file" -q -z conv,tcp
done
```

This processes every PCAPNG file in the current directory and displays its TCP conversation statistics.

---

## 14. Practical Cybersecurity Applications

TShark can support different types of security investigations.

| Security task | TShark application |
|---|---|
| Network monitoring | Capture and inspect live traffic |
| DNS investigation | Extract DNS queries and responses |
| Port analysis | Identify observed TCP and UDP ports |
| Traffic investigation | Examine IP addresses and conversations |
| Incident response | Analyze saved PCAP files |
| Data extraction | Extract selected packet fields |
| Threat hunting | Search for suspicious communication patterns |
| Automation | Process multiple capture files using scripts |

### Example: Investigating suspicious DNS activity

An analyst notices an unusually high number of DNS queries from a workstation.

The analyst can:

1. Read the captured traffic.
2. Apply a DNS display filter.
3. Extract requested domain names.
4. Identify unusual query patterns.
5. Compare the results with known threat intelligence.
6. Document findings for further investigation.

A high number of DNS queries alone does not prove malicious activity. The analyst must investigate the domain names, timing, frequency, host behavior, and surrounding evidence.

---

## 15. Practical Lab: Analyze a PCAP File Using TShark

**Objective:** Use TShark to extract useful network information from a saved packet capture.

### Step 1: Check TShark

```bash
tshark --version
```

### Step 2: Read the capture file

```bash
tshark -r sample.pcapng
```

### Step 3: Identify DNS traffic

```bash
tshark -r sample.pcapng -Y "dns"
```

### Step 4: Extract DNS query names

```bash
tshark -r sample.pcapng \
-Y "dns.flags.response == 0" \
-T fields \
-e dns.qry.name
```

### Step 5: Identify TCP conversations

```bash
tshark -r sample.pcapng -q -z conv,tcp
```

### Step 6: Extract source and destination IP addresses

```bash
tshark -r sample.pcapng \
-T fields \
-e ip.src \
-e ip.dst
```

### Step 7: Save the findings

Document:

- Protocols observed.
- Source and destination IP addresses.
- DNS query names.
- TCP conversations.
- Any unusual communication patterns.
- Limitations of the available capture.

**Expected learning outcome:** Understand how TShark can extract network evidence and generate useful statistics without relying on the Wireshark graphical interface.

---

## 16. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Using `-Y` during live capture as a capture filter | Use `-f` for capture filtering |
| Using `-f` to filter an existing capture file | Use `-Y` when reading and filtering saved traffic |
| Forgetting `-r` | Use `-r` to read an existing capture |
| Forgetting `-w` | Use `-w` to save captured packets |
| Assuming HTTPS content is readable | TLS encryption protects application data |
| Treating every unusual packet as malicious | Investigate context and supporting evidence |
| Running captures without permission | Capture only traffic you are authorized to monitor |

---

## 17. Interview Questions

**Q1. What is TShark?**

TShark is the command-line version of Wireshark used to capture, filter, and analyze network packets.

**Q2. What is the difference between Wireshark and TShark?**

Wireshark provides a graphical interface, while TShark performs packet analysis through command-line instructions.

**Q3. Which command lists available network interfaces?**

```bash
tshark -D
```

**Q4. Which option is used to capture packets?**

The `-i` option specifies the network interface for packet capture.

**Q5. What is the purpose of `-r`?**

It reads packets from an existing capture file.

**Q6. What is the difference between `-f` and `-Y`?**

`-f` applies capture filters, while `-Y` applies display filters.

**Q7. Which option saves captured packets to a file?**

The `-w` option.

**Q8. How can you extract source and destination IP addresses?**

```bash
tshark -r capture.pcapng -T fields -e ip.src -e ip.dst
```

**Q9. What is the purpose of `-z conv,tcp`?**

It displays TCP conversation statistics from a capture file.

**Q10. Why is TShark useful in a SOC?**

It helps analysts investigate network traffic, extract packet information, generate statistics, and automate repetitive analysis tasks.

---

## 18. Quick Revision

| Command | Purpose |
|---|---|
| `tshark --version` | Check version |
| `tshark -D` | List interfaces |
| `tshark -i eth0` | Capture traffic | 
| `tshark -i eth0 -c 100` | Capture 100 packets |
| `tshark -i eth0 -w file.pcapng` | Save captured packets |
| `tshark -r file.pcapng` | Read a capture file |
| `tshark -Y "dns"` | Filter DNS packets |
| `tshark -f "port 53"` | Capture DNS-port traffic |
| `tshark -V` | Show detailed packet information |
| `tshark -x` | Show packet bytes |
| `tshark -T fields` | Extract selected fields |
| `tshark -q -z conv,tcp` | Show TCP conversations |

**Key takeaway:** TShark brings Wireshark's packet-analysis capabilities to the command line, making it useful for network investigations, incident response, and basic security automation.
