# 05 - TCP Analysis with Wireshark

## 1. Introduction to TCP Analysis

**Transmission Control Protocol (TCP)** is a connection-oriented transport-layer protocol that provides reliable data delivery between devices.

TCP ensures that:
- Data reaches the destination reliably.
- Packets are delivered in the correct order.
- Lost segments can be retransmitted.
- Data transmission is controlled according to network conditions.

In Wireshark, TCP analysis helps security analysts understand connection behavior, troubleshoot network problems, and investigate suspicious communication.

## 2. Understanding TCP Connections

TCP establishes a connection before transferring application data.

A TCP connection is identified by a combination of:

- Source IP address
- Source port
- Destination IP address
- Destination port

This combination is commonly called a **five-tuple** when the transport protocol is included.

### Important TCP characteristics

| Feature | Description |
|---|---|
| Connection-oriented | Establishes a connection before data transfer |
| Reliable | Uses acknowledgments and retransmissions |
| Ordered delivery | Reassembles data in the correct order |
| Flow control | Prevents the sender from overwhelming the receiver |
| Full-duplex | Allows both devices to send data simultaneously |

## 3. TCP Three-Way Handshake

The TCP three-way handshake establishes a connection between a client and a server.

<pre>
Client                         Server
  |                               |
  | -------- SYN ---------------> |
  |                               |
  | <------ SYN, ACK ------------- |
  |                               |
  | -------- ACK ---------------> |
  |                               |
  |       Connection Established  |
</pre>

### Step 1: SYN

The client sends a SYN packet to request a TCP connection.

- SYN flag is set to 1.
- The client selects an initial sequence number.
- The server receives the connection request.

### Step 2: SYN-ACK

The server responds with SYN and ACK flags.

- SYN acknowledges the connection request process.
- ACK acknowledges the client's initial sequence number.
- The server provides its own initial sequence number.

### Step 3: ACK

The client sends an ACK packet.

- It acknowledges the server's initial sequence number.
- The TCP connection is established.
- Application data can now be exchanged.

### Wireshark filter

```wireshark
tcp.flags.syn == 1
```

To identify SYN packets without the ACK flag:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

To identify SYN-ACK packets:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

**Security relevance:** A large number of SYN requests without completed handshakes may indicate a SYN flood attempt. However, network scans, busy services, packet loss, and server overload can also produce unusual SYN patterns.

## 4. Understanding TCP Flags

TCP flags are control bits in the TCP header. They help manage connection establishment, data transfer, and connection termination.

| Flag | Meaning | Purpose |
|---|---|---|
| SYN | Synchronize | Initiates a TCP connection |
| ACK | Acknowledgment | Confirms received data or sequence numbers |
| FIN | Finish | Initiates an orderly connection termination |
| RST | Reset | Resets or immediately terminates a connection |
| PSH | Push | Requests that received data be passed to the application promptly |
| URG | Urgent | Indicates that the urgent pointer field is significant |
| ECE | ECN Echo | Supports Explicit Congestion Notification |
| CWR | Congestion Window Reduced | Signals a response to congestion notification |

### Useful TCP flag filters

```wireshark
tcp.flags.syn == 1
tcp.flags.fin == 1
tcp.flags.reset == 1
tcp.flags.push == 1
tcp.flags.ack == 1
```

To find packets with both SYN and RST flags:

```wireshark
tcp.flags.syn == 1 && tcp.flags.reset == 1
```

**Important:** TCP flags must be interpreted in context. A RST packet is not automatically malicious; it can occur during normal connection handling or when a service refuses a connection.

## 5. TCP Sequence and Acknowledgment Numbers

TCP uses sequence and acknowledgment numbers to maintain reliable and ordered communication.

### Sequence Number

A sequence number identifies the position of data bytes in a TCP stream.

It helps the receiver:
- Identify missing data.
- Reassemble data in the correct order.
- Detect duplicate segments.

### Acknowledgment Number

An acknowledgment number indicates the next byte the receiver expects to receive.

For example:

- Sender transmits bytes 1000–1499.
- Receiver successfully receives them.
- Receiver sends ACK 1500.

This means the receiver expects byte 1500 next.

### Wireshark fields

- `tcp.seq` — Relative sequence number.
- `tcp.ack` — Relative acknowledgment number.
- `tcp.seq_raw` — Raw sequence number.
- `tcp.ack_raw` — Raw acknowledgment number.

### Filters

```wireshark
tcp.seq == 1000
tcp.ack == 1500
tcp.analysis.duplicate_ack
tcp.analysis.lost_segment
```

**Note:** Wireshark commonly displays relative sequence numbers by default, making TCP analysis easier. Raw sequence numbers show the actual values transmitted in the TCP header.

## 6. TCP Window Size and Flow Control

TCP uses a receive window to control how much data a sender can transmit before receiving further acknowledgments.

This mechanism is called **flow control**.

### Receive Window

The receive window tells the sender how much additional data the receiver is currently prepared to accept.

For example:

- A receiver advertises a window of 65,535 bytes.
- The sender uses this information to control the amount of unacknowledged data in transit.

### Window Scaling

TCP window scaling allows larger receive windows than the original 16-bit TCP window field can represent.

It is negotiated during the TCP handshake.

### Wireshark fields

```wireshark
tcp.window_size_value
tcp.window_size
tcp.window_size_scalefactor
```

### Zero Window

A TCP Zero Window occurs when a receiver advertises a receive window of zero bytes.

This means the receiver temporarily cannot accept more data.

Possible reasons:
- Application is processing data slowly.
- Receiver has limited available buffer space.
- System resources are under pressure.

Filter:

```wireshark
tcp.window_size_value == 0
```

A zero window is not automatically an attack. Repeated or prolonged zero-window conditions may indicate a performance or application issue.

## 7. TCP Connection Termination

TCP normally terminates a connection using a four-step exchange.

<pre>
Client                         Server
  |                               |
  | -------- FIN ---------------> |
  |                               |
  | <-------- ACK --------------- |
  |                               |
  | <-------- FIN --------------- |
  |                               |
  | -------- ACK ---------------> |
  |                               |
  |       Connection Closed      |
</pre>

### FIN Flag

The FIN flag indicates that a device has finished sending data and wants to close its side of the connection.

### RST Flag

The RST flag immediately resets a TCP connection instead of performing an orderly shutdown.

Common reasons include:
- Connection attempts to a closed port.
- Application or server terminating a connection.
- Firewall or network device rejecting a connection.
- Unexpected connection state.

### Wireshark filters

```wireshark
tcp.flags.fin == 1
tcp.flags.reset == 1
```

**Security relevance:** Unexpected RST packets, repeated connection resets, or unusual termination patterns may deserve investigation, especially when combined with other suspicious activity.

## 8. TCP Retransmissions

TCP retransmission occurs when a sender sends data again because it has not received the expected acknowledgment.

Possible reasons:
- Packet loss.
- Network congestion.
- Unstable network connection.
- Receiver or server performance problems.

Wireshark can identify retransmissions through TCP analysis.

### Important TCP analysis indicators

| Indicator | Meaning |
|---|---|
| TCP Retransmission | A segment is retransmitted |
| Fast Retransmission | A segment is retransmitted after duplicate ACKs suggest loss |
| Duplicate ACK | Receiver repeatedly acknowledges the same sequence position |
| Out-of-Order | Segments arrive in an unexpected order |
| Previous Segment Not Captured | A segment expected by the analyzer is missing from the capture |

### Useful filters

```wireshark
tcp.analysis.retransmission
tcp.analysis.fast_retransmission
tcp.analysis.duplicate_ack
tcp.analysis.out_of_order
tcp.analysis.lost_segment
```

**Important:** These indicators do not always prove packet loss or malicious activity. Capture loss, asymmetric routing, reordering, and network conditions can affect Wireshark's analysis.

## 9. TCP Stream Analysis

Wireshark provides a feature called **Follow TCP Stream**.

It allows analysts to view the application data exchanged between two endpoints within a TCP connection.

### How to Follow a TCP Stream

1. Open a packet capture file or start a capture.
2. Apply a TCP display filter if needed.
3. Select a TCP packet.
4. Right-click the packet.
5. Select **Follow → TCP Stream**.
6. Examine the communication between both endpoints.

Wireshark displays the conversation in a reconstructed stream view.

### Why Follow TCP Stream is useful

- Understand client-server communication.
- Inspect readable HTTP requests and responses.
- Identify suspicious commands or text.
- Investigate unexpected application behavior.
- Understand how data was exchanged during a session.

**Limitation:** HTTPS and other encrypted protocols generally do not expose readable application content without appropriate decryption keys or session secrets.

## 10. TCP Stream and Conversation Analysis

Wireshark provides different ways to analyze TCP communication.

### Conversations

Navigate to:

**Statistics → Conversations → TCP**

This provides information such as:
- Source and destination addresses.
- Source and destination ports.
- Number of packets.
- Total bytes transferred.
- Connection duration.

### TCP Stream Graphs

Navigate to:

**Statistics → TCP Stream Graphs**

Depending on the selected graph, you can examine:
- Round-trip time.
- Throughput.
- Time sequence.
- Window scaling behavior.

These graphs help identify performance problems and unusual traffic patterns.

## 11. Important TCP Display Filters

| Purpose | Wireshark Filter |
|---|---|
| All TCP traffic | `tcp` |
| SYN packets | `tcp.flags.syn == 1` |
| SYN without ACK | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| SYN-ACK packets | `tcp.flags.syn == 1 && tcp.flags.ack == 1` |
| FIN packets | `tcp.flags.fin == 1` |
| RST packets | `tcp.flags.reset == 1` |
| Retransmissions | `tcp.analysis.retransmission` |
| Fast retransmissions | `tcp.analysis.fast_retransmission` |
| Duplicate ACKs | `tcp.analysis.duplicate_ack` |
| Out-of-order packets | `tcp.analysis.out_of_order` |
| Zero-window packets | `tcp.window_size_value == 0` |
| TCP traffic on port 443 | `tcp.port == 443` |
| TCP traffic from an IP | `ip.src == 192.168.1.10 && tcp` |
| TCP traffic to a server | `ip.dst == 192.168.1.20 && tcp` |

## 12. TCP Analysis for Cybersecurity

TCP analysis is useful in SOC operations, network troubleshooting, and incident response.

### 1. Detecting Port Scanning

Port scanning tools may generate many connection attempts against different ports.

Possible indicators:
- Multiple SYN packets.
- Connections to many destination ports.
- Repeated unanswered connection attempts.
- Unusual connection patterns from one source IP.

Filter:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### 2. Investigating SYN Flood Activity

A SYN flood attempts to consume server resources by generating a large number of TCP connection requests.

Possible indicators:
- High volume of incoming SYN packets.
- Many incomplete handshakes.
- Unusual source IP distribution.
- Increased connection backlog or service degradation.

A packet capture alone may not establish that a SYN flood is occurring. Correlate traffic with server logs and network monitoring data.

### 3. Investigating Suspicious Connections

Analysts can examine:
- Unusual destination ports.
- Unexpected external IP addresses.
- Repeated connection attempts.
- Abnormally long-lived connections.
- Unexpected data transfer patterns.

### 4. Troubleshooting Network Performance

TCP analysis can help identify:
- Retransmissions.
- Duplicate ACKs.
- Zero-window conditions.
- Connection resets.
- Unexpected delays.

## 13. Practical Lab: Analyze a TCP Handshake

**Objective:** Capture and analyze a TCP connection using Wireshark.

### Step 1: Start Capture

1. Open Wireshark.
2. Select your active network interface.
3. Start packet capture.
4. Generate some TCP traffic by opening a website.

### Step 2: Apply the SYN Filter

```wireshark
tcp.flags.syn == 1
```

Observe the SYN and SYN-ACK packets.

### Step 3: Identify the Three-Way Handshake

Find these three packets:

1. SYN
2. SYN-ACK
3. ACK

Check the source and destination IP addresses and ports.

### Step 4: Analyze TCP Flags

Apply:

```wireshark
tcp.flags.fin == 1 || tcp.flags.reset == 1
```

Observe how connections are terminated.

### Step 5: Check Retransmissions

Apply:

```wireshark
tcp.analysis.retransmission
```

If no packets appear, it simply means Wireshark did not identify retransmissions in the captured traffic.

### Step 6: Follow TCP Stream

1. Select a TCP packet.
2. Right-click it.
3. Choose Follow → TCP Stream.
4. Examine the communication.

### Lab Questions

1. What are the three steps of a TCP handshake?
2. What is the difference between SYN and SYN-ACK?
3. What is the purpose of the ACK flag?
4. What does a TCP retransmission indicate?
5. What is the difference between FIN and RST?
6. What does a Zero Window indicate?

## 14. Common TCP Analysis Mistakes

- Assuming every RST packet indicates an attack.
- Treating every retransmission as proof of network failure.
- Confusing sequence numbers with acknowledgment numbers.
- Assuming SYN packets always indicate malicious scanning.
- Ignoring the source and destination ports.
- Forgetting that encrypted traffic cannot normally be read as plaintext.
- Treating Wireshark's analysis labels as definitive proof without checking packet context.

## 15. TCP Analysis Interview Questions

**Q1. What is TCP?**

TCP is a connection-oriented transport-layer protocol that provides reliable and ordered data delivery.

**Q2. What is the TCP three-way handshake?**

It is the connection establishment process involving SYN, SYN-ACK, and ACK packets.

**Q3. What is the purpose of the SYN flag?**

It initiates a TCP connection and synchronizes sequence numbers.

**Q4. What is the difference between FIN and RST?**

FIN is used for orderly connection termination, while RST immediately resets a connection.

**Q5. What is a TCP retransmission?**

It occurs when TCP sends data again because the expected acknowledgment has not been received.

**Q6. What is a duplicate ACK?**

It is an acknowledgment that repeats the same acknowledgment number, indicating that the receiver is still expecting the same next byte.

**Q7. What is a TCP Zero Window?**

It indicates that the receiver currently has no available receive-window space for additional data.

**Q8. What is Follow TCP Stream?**

It is a Wireshark feature that reconstructs and displays the data exchanged in a TCP connection.

**Q9. How can you identify a SYN scan in Wireshark?**

You can inspect repeated SYN packets without ACK responses and analyze the destination ports and connection patterns.

**Q10. What is the purpose of TCP sequence numbers?**

They identify the position of bytes in a TCP stream and support ordered delivery and retransmission.

## 16. Quick Revision

| Concept | Key Point |
|---|---|
| TCP | Reliable, connection-oriented protocol |
| Three-Way Handshake | SYN → SYN-ACK → ACK |
| SYN | Starts connection establishment |
| ACK | Acknowledges received data |
| FIN | Orderly connection termination |
| RST | Resets a connection |
| Sequence Number | Identifies byte position |
| Acknowledgment Number | Indicates the next expected byte |
| Window Size | Supports flow control |
| Retransmission | Sending data again |
| Duplicate ACK | Repeated acknowledgment number |
| Zero Window | Receiver cannot currently accept more data |
| Follow TCP Stream | Reconstructs TCP communication |

