# 01 — Wireshark Basics

## 1. What Is Wireshark?

**Wireshark** is an open-source network protocol analyzer used to capture and inspect network packets in real time or from previously saved capture files.

It helps you understand how devices communicate, which protocols they use, and what information is visible in network traffic.

### Common Use Cases

- Troubleshooting network connectivity and performance.
- Investigating suspicious network communications.
- Understanding protocols such as TCP, DNS, HTTP, TLS, and ARP.
- Supporting security monitoring and incident response.
- Analyzing packet capture files during forensic investigations.

**Important:** Wireshark helps identify suspicious behavior, but a packet or alert alone does not prove that an attack occurred.

## 2. How Wireshark Works

Wireshark relies on a packet-capture mechanism to receive traffic from a selected network interface.

A simplified workflow is:

```text
Network Interface
       |
       v
Packet Capture Library
   (Npcap / libpcap)
       |
       v
Wireshark Capture Engine
       |
       v
Protocol Dissection
       |
       v
Packet List and Details
       |
       v
Analysis and Investigation
```

### Main Components

| Component | Purpose |
|---|---|
| Network interface | Provides access to traffic visible to the host |
| Npcap / libpcap | Provides packet-capture functionality |
| Capture engine | Receives packets during a capture |
| Protocol dissectors | Interpret packet data according to known protocols |
| Display filters | Select packets for viewing |
| Capture files | Store captured packets for later analysis |

On Windows, Wireshark commonly uses **Npcap**. On Linux and other Unix-like systems, it commonly uses **libpcap**.

## 3. Installing Wireshark

Download Wireshark from the official website:

https://www.wireshark.org/download.html

### Windows

1. Download the official installer.
2. Follow the installation wizard.
3. Install Npcap when prompted if packet capture is required.
4. Open Wireshark and check the available interfaces.

### Linux — Debian / Ubuntu / Kali / Parrot OS

Update package information and install Wireshark:

```bash
sudo apt update
sudo apt install wireshark
```

Verify the installation:

```bash
wireshark --version
```

Check whether TShark is available:

```bash
tshark --version
```

**Security note:** Packet capture may require elevated privileges or specific group permissions. Prefer a properly configured, least-privilege capture setup rather than running the entire graphical application as root.

## 4. Understanding Network Interfaces

A network interface is a connection through which a device sends or receives network traffic.

Common interfaces include:

| Interface | Typical Purpose |
|---|---|
| Ethernet | Wired network connection |
| Wi-Fi | Wireless network connection |
| Loopback | Traffic generated locally on the same device |
| VPN interface | Traffic routed through a VPN tunnel |
| Virtual interface | Traffic associated with a VM, container, or virtual network |

### Selecting an Interface

When Wireshark opens, its welcome screen lists available capture interfaces. Select the interface that corresponds to the traffic you want to investigate.

For example:

- Use Ethernet when investigating a wired connection.
- Use Wi-Fi when examining traffic visible to your wireless adapter.
- Use loopback when investigating local application communications.

Interface names differ between operating systems.

### List Interfaces with TShark

```bash
tshark -D
```

Example output:

```text
1. eth0
2. wlan0
3. lo
```

These are illustrative names; your actual interface list may differ.

Start a short capture on a specific interface:

```bash
sudo tshark -i eth0 -c 20
```

This captures up to 20 packets from `eth0`. Replace `eth0` with an interface shown by `tshark -D`.

## 5. Understanding the Wireshark Interface

The main Wireshark window contains several important areas.

### Packet List

Displays captured packets in rows. Common columns include:

- **No.** — packet number in the current capture.
- **Time** — timestamp relative to the configured time reference.
- **Source** — source address.
- **Destination** — destination address.
- **Protocol** — protocol identified by Wireshark.
- **Length** — packet length in bytes.
- **Info** — a brief protocol-specific summary.

### Packet Details

Displays the selected packet as a protocol tree. Expand individual layers to inspect fields such as addresses, ports, flags, and protocol metadata.

### Packet Bytes

Shows the selected packet's bytes in hexadecimal and ASCII where applicable.

This is useful when validating raw data or examining fields not conveniently exposed in the packet summary.

### Display Filter Bar

Allows you to show only packets matching a display-filter expression without deleting other packets from the capture.

## 6. Capturing Packets

A packet capture records traffic visible to the selected interface.

### Basic Capture Workflow

1. Open Wireshark.
2. Select the appropriate interface.
3. Start capturing.
4. Generate or observe the traffic you need to study.
5. Stop the capture.
6. Apply filters and inspect relevant packets.
7. Save the capture if it is needed for further analysis.

### Generate Traffic for Practice

Use commands on your own device or an authorized lab system.

Test connectivity:

```bash
ping -c 4 1.1.1.1
```

Generate a DNS lookup:

```bash
nslookup example.com
```

Open a website in a browser and observe the resulting DNS, TCP, and TLS traffic where visible.

The exact packets depend on caching, network configuration, the application, and the protocols in use.

### Capture Filter vs Display Filter

These filters serve different purposes.

| Feature | Capture Filter | Display Filter |
|---|---|---|
| When applied | Before and during capture | During analysis |
| Purpose | Limits which packets are captured | Limits which captured packets are displayed |
| Syntax | Capture-filter syntax, generally based on libpcap | Wireshark display-filter syntax |
| Can recover excluded packets? | No | Yes, if they were captured |

Example capture filter:

```text
host 192.168.1.10
```

Example display filter:

```text
ip.addr == 192.168.1.10
```

Do not confuse their syntax. A display filter generally cannot be pasted directly into the capture-filter field.

## 7. Your First Display Filters

Enter these expressions in the display-filter bar.

| Filter | What It Shows |
|---|---|
| `dns` | DNS packets |
| `tcp` | TCP packets |
| `udp` | UDP packets |
| `icmp` | ICMP packets |
| `arp` | ARP packets |
| `http` | Packets decoded as HTTP |
| `tls` | Packets decoded as TLS |
| `ip` | IPv4 packets |
| `ipv6` | IPv6 packets |
| `tcp.port == 443` | TCP packets with source or destination port 443 |
| `ip.addr == 192.168.1.10` | IPv4 packets with the address as source or destination |
| `dns && udp` | Packets matching both DNS and UDP |

**Note:** HTTPS traffic is commonly encrypted using TLS. Filtering for `http` will not normally reveal encrypted HTTPS application contents. Depending on the protocol and capture, useful TLS metadata may still be visible.

## 8. Saving and Opening Capture Files

Wireshark supports common capture formats, including:

- `.pcap`
- `.pcapng`

PCAPNG is the default format in many current Wireshark versions and supports additional capture metadata.

### Save a Capture

1. Stop the capture.
2. Select **File → Save As**.
3. Choose a filename and location.
4. Select the desired supported capture format.

Example filename:

```text
basic_network_traffic.pcapng
```

### Read a Capture File with TShark

```bash
tshark -r basic_network_traffic.pcapng
```

Apply a display filter to the saved file:

```bash
tshark -r basic_network_traffic.pcapng -Y "dns"
```

The `-r` option reads a capture file, while `-Y` applies a display filter.

## 9. Useful Wireshark Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + E` | Start or stop capturing |
| `Ctrl + K` | Open capture options |
| `Ctrl + O` | Open a capture file |
| `Ctrl + S` | Save a capture |
| `Ctrl + F` | Find packets |
| `Ctrl + /` | Focus the display-filter bar in many versions |

Shortcuts can vary by operating system and Wireshark version. Check the menus if a shortcut behaves differently.

## 10. Practical Lab — Capture and Analyze DNS Traffic

**Objective:** Capture DNS traffic and identify the query and response.

### Step 1: Start a Capture

Open Wireshark and select your active network interface.

### Step 2: Generate a DNS Query

Run:

```bash
nslookup example.com
```

### Step 3: Apply a Display Filter

```text
dns
```

### Step 4: Inspect the Packets

Select a DNS packet and expand the DNS section in Packet Details.

Look for:

- Query name
- Query type, such as A or AAAA
- Query/response indicator
- Response code
- Answer records, if present

A query may be answered from a cache or use encrypted DNS, so a visible DNS packet is not guaranteed in every setup.

### Step 5: Investigate the Result

Ask yourself:

1. What domain was queried?
2. Was a response captured?
3. Did the response contain an answer?
4. Was the response successful or did it contain an error?
5. Which IP address, if any, was returned?

### Expected Learning

You should understand how to start a capture, generate traffic, filter packets, inspect protocol details, and save evidence for later analysis.

## 11. Cybersecurity Applications

### Network Troubleshooting

Wireshark can help investigate DNS failures, connection resets, retransmissions, and unexpected communication patterns.

### Suspicious Traffic Investigation

An analyst can examine unfamiliar IP addresses, unusual ports, repeated connection attempts, or unexpected protocol activity.

### Incident Response

A saved PCAP can help reconstruct a sequence of network events and provide supporting evidence for an investigation.

### Security Limitations

- A normal capture usually sees only traffic available to the selected interface.
- Switched networks generally do not expose all other devices' unicast traffic to an ordinary endpoint.
- TLS encryption limits visibility into application payloads unless suitable decryption material and conditions are available.
- NAT, VPNs, proxies, packet loss, and capture placement can affect what you observe.

Use findings alongside endpoint logs, firewall records, DNS logs, and other relevant evidence.

## 12. Common Problems and Troubleshooting

| Problem | Possible Cause | What to Check |
|---|---|---|
| No interfaces appear | Capture driver missing or interface unavailable | Verify Npcap/libpcap and interface status |
| Capture permission denied | Insufficient capture permissions | Review OS permissions and capture configuration |
| No packets appear | Wrong interface or idle network | Generate test traffic and select the active interface |
| DNS filter shows nothing | Cached lookup or encrypted DNS | Generate a fresh query and check the capture interface |
| `http` filter shows nothing | Traffic uses HTTPS or another protocol | Inspect DNS and TLS traffic instead |
| Display filter is invalid | Incorrect syntax or field name | Check spelling and use Wireshark's filter suggestions |
| Important packets are missing | Capture started late or capture filter excluded traffic | Review capture settings and repeat the capture |

## 13. Best Practices

- Capture only traffic you own or are authorized to analyze.
- Use capture filters carefully because excluded packets cannot be recovered from that capture.
- Save original capture files before performing detailed analysis.
- Protect PCAP files because they may contain sensitive information.
- Record the capture time, interface, and purpose of the investigation.
- Avoid running Wireshark with unnecessary administrative privileges.
- Validate suspicious observations before concluding that an attack occurred.

## 14. Interview Questions

**Q1. What is Wireshark?**

Wireshark is an open-source network protocol analyzer used to capture and inspect network packets for troubleshooting, protocol analysis, and security investigations.

**Q2. What is the difference between capture filters and display filters?**

Capture filters determine which packets are collected. Display filters determine which already captured packets are shown.

**Q3. What is a packet capture file?**

It is a file containing captured network packets and associated metadata, commonly stored in PCAP or PCAPNG format.

**Q4. What is a protocol dissector?**

A protocol dissector interprets packet bytes according to a protocol's structure and presents the decoded fields.

**Q5. Why might Wireshark show no packets?**

The wrong interface may be selected, the interface may be idle, permissions may be insufficient, or capture settings may exclude the relevant traffic.

**Q6. Can Wireshark decrypt HTTPS traffic automatically?**

No. HTTPS typically uses TLS encryption. Decryption requires appropriate session secrets or other supported decryption material and the correct capture conditions.

**Q7. Does Wireshark capture all traffic on a network?**

No. It generally captures traffic visible to the selected interface. Network architecture, switching, wireless mode, permissions, and capture location affect visibility.

**Q8. What is TShark?**

TShark is Wireshark's command-line network protocol analyzer. It can capture, read, filter, and extract information from packet traffic.

## 15. Quick Revision

- Wireshark captures and analyzes network packets.
- Npcap and libpcap provide packet-capture functionality on supported systems.
- Protocol dissectors decode packet fields.
- Capture filters limit collected traffic.
- Display filters limit displayed traffic.
- PCAP and PCAPNG store captured network traffic.
- TShark provides command-line analysis.
- Packet visibility depends on interface selection, network design, and capture permissions.

**Next:** [`02_Packet_Analysis.md`](02_Packet_Analysis.md) — learn packet structure, protocol layers, headers, and encapsulation.
