# 07 - TLS and HTTPS Analysis with Wireshark

## 1. Introduction to TLS and HTTPS

**TLS (Transport Layer Security)** is a cryptographic protocol designed to secure communication between applications over a network.

It provides three major security properties:

- **Confidentiality:** Encrypts data so unauthorized parties cannot easily read it.
- **Integrity:** Helps detect unauthorized modification of transmitted data.
- **Authentication:** Helps verify the identity of the server using digital certificates.

**HTTPS (Hypertext Transfer Protocol Secure)** is HTTP communication protected by TLS.

For example:

- HTTP: `http://example.com`
- HTTPS: `https://example.com`

HTTPS commonly uses TCP port 443 when running over HTTP/1.1 or HTTP/2 with TLS. HTTP/3 uses QUIC over UDP, typically on port 443.

## 2. Difference Between HTTP and HTTPS

| Feature | HTTP | HTTPS |
|---|---|---|
| Full Form | Hypertext Transfer Protocol | HTTP Secure |
| Encryption | No TLS encryption | Protected by TLS |
| Default Port | 80 | 443 |
| Data Confidentiality | Not provided by HTTP itself | Provided by TLS |
| Integrity | No TLS integrity protection | Protected by TLS |
| Authentication | No server authentication through TLS | Server authentication through certificates |
| Wireshark Visibility | Application content is often readable | Application content is generally encrypted |

**Important:** HTTPS does not hide everything. Depending on the protocol version and configuration, network observers may still see IP addresses, ports, packet sizes, timing, and some TLS handshake metadata.

## 3. Understanding TLS Versions

TLS has evolved to improve security, performance, and cryptographic capabilities.

| Version | Description |
|---|---|
| SSL 2.0 | Obsolete and insecure |
| SSL 3.0 | Obsolete and insecure |
| TLS 1.0 | Deprecated |
| TLS 1.1 | Deprecated |
| TLS 1.2 | Widely supported and still used |
| TLS 1.3 | Modern version with improved security and a faster handshake |

### Why TLS 1.3 Matters

TLS 1.3 introduced several improvements:

- Removes support for many outdated cryptographic algorithms.
- Reduces the number of handshake round trips.
- Encrypts more of the handshake.
- Supports forward-secret key exchange through ephemeral key exchanges.
- Improves connection establishment performance.

**Security note:** SSL is obsolete. Modern systems should use properly configured TLS rather than SSL.

## 4. How HTTPS Communication Works

When a user visits an HTTPS website, the browser and server establish a secure connection before exchanging protected application data.

Simplified process:

```text
Client / Browser                 Web Server
       |                              |
       | ------ ClientHello --------> |
       |                              |
       | <----- ServerHello --------- |
       |                              |
       |     TLS Authentication      |
       |                              |
       |    Session Keys Established  |
       |                              |
       | <==== Encrypted Data ======> |
       |                              |
```

The exact handshake messages differ between TLS 1.2 and TLS 1.3.

## 5. Understanding the TLS Handshake

The TLS handshake is the process used by a client and server to negotiate security settings and establish shared encryption keys.

### TLS 1.2 Handshake

A simplified TLS 1.2 handshake includes:

1. **ClientHello:** The client sends supported TLS versions, cipher suites, and other parameters.
2. **ServerHello:** The server selects the TLS version and cipher suite.
3. **Certificate:** The server provides its certificate.
4. **Key Exchange:** The client and server establish shared secret material.
5. **Finished Messages:** Both sides verify the handshake.
6. **Encrypted Communication:** Protected application data can now be exchanged.

The exact messages depend on the selected cipher suite and key exchange method.

### TLS 1.3 Handshake

A simplified TLS 1.3 handshake includes:

1. **ClientHello:** The client proposes supported versions, cipher suites, and key shares.
2. **ServerHello:** The server selects the parameters and provides its key share.
3. **EncryptedExtensions:** The server sends additional negotiated parameters.
4. **Certificate:** The server provides its certificate when certificate-based authentication is used.
5. **CertificateVerify:** The server proves possession of the certificate's private key.
6. **Finished:** The server and client verify the handshake.
7. **Encrypted Application Data:** Secure communication begins.

Some handshake messages are encrypted in TLS 1.3, so they may not be visible as readable details in a normal packet capture.

## 6. Important TLS Handshake Messages

| Message | Purpose |
|---|---|
| ClientHello | Initiates TLS negotiation |
| ServerHello | Server selects connection parameters |
| EncryptedExtensions | Sends additional negotiated parameters in TLS 1.3 |
| Certificate | Provides the server's digital certificate |
| CertificateVerify | Proves possession of the certificate's private key |
| Finished | Verifies handshake integrity |
| NewSessionTicket | Provides information for future session resumption |
| Alert | Reports TLS warnings or errors |

## 7. TLS Cipher Suites

A cipher suite defines the cryptographic algorithms used to protect a TLS connection.

For TLS 1.2, a cipher suite can specify:
- Key exchange algorithm.
- Authentication algorithm.
- Symmetric encryption algorithm.
- Message authentication algorithm.

For TLS 1.3, cipher suites specify the symmetric encryption and hash algorithms. Key exchange and authentication are negotiated separately.

### Example TLS 1.3 Cipher Suites

```text
TLS_AES_128_GCM_SHA256
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
```

### Important Terms

| Term | Meaning |
|---|---|
| AES | Advanced Encryption Standard |
| GCM | Galois/Counter Mode |
| SHA | Secure Hash Algorithm |
| ChaCha20 | Modern stream cipher |
| Poly1305 | Message authentication algorithm |

## 8. Digital Certificates

A digital certificate is an electronic document used to establish the identity of a website or organization.

HTTPS servers commonly use X.509 certificates.

A certificate generally contains:

- Subject: The identity associated with the certificate.
- Subject Alternative Name (SAN): Names the certificate is valid for.
- Issuer: The Certificate Authority (CA) that issued it.
- Validity period: Start and expiration dates.
- Public key: The certificate's public key.
- Signature: A digital signature from the issuer.

### Certificate Authority (CA)

A Certificate Authority is an organization trusted to issue and sign digital certificates.

Examples include:
- DigiCert
- GlobalSign
- Let's Encrypt

### Certificate Validation

A browser or client generally checks:

1. Whether the certificate is valid for the requested hostname.
2. Whether it is within its validity period.
3. Whether its signature chain leads to a trusted CA.
4. Whether other applicable certificate and revocation checks succeed.

A certificate being present in a packet capture does not automatically mean the connection is trustworthy.

## 9. SNI (Server Name Indication)

**Server Name Indication (SNI)** is a TLS extension that allows a client to indicate the hostname it wants to connect to during the TLS handshake.

It helps a server host multiple HTTPS websites using the same IP address.

Example:

```text
Client connects to: 203.0.113.10

SNI hostname:
example.com
```

The server can use the hostname to select the appropriate certificate and configuration.

### Security Relevance

SNI can help security analysts identify the requested hostname during TLS analysis.

However:
- Traditional SNI is visible in the ClientHello.
- Encrypted ClientHello (ECH), when supported and used, can protect the real ClientHello information, including the inner SNI.
- SNI alone does not prove that a website is malicious.

## 10. TLS Analysis in Wireshark

Wireshark can identify TLS handshakes, certificates, alerts, and encrypted application traffic.

### Useful TLS Display Filters

Display TLS traffic:

```wireshark
tls
```

ClientHello messages:

```wireshark
tls.handshake.type == 1
```

ServerHello messages:

```wireshark
tls.handshake.type == 2
```

Certificate messages:

```wireshark
tls.handshake.type == 11
```

TLS alerts:

```wireshark
tls.record.content_type == 21
```

TLS application data:

```wireshark
tls.app_data
```

TLS handshake messages:

```wireshark
tls.handshake
```

Find packets using TCP port 443:

```wireshark
tcp.port == 443
```

Find TLS traffic to a particular IP:

```wireshark
ip.addr == 192.0.2.10 && tls
```

**Note:** TLS filter fields may vary with protocol versions and Wireshark releases. QUIC traffic may require separate analysis using the `quic` display filter.

## 11. Important TLS Fields

When inspecting a TLS packet, useful fields include:

| Field | Description |
|---|---|
| `tls.handshake.type` | Identifies the handshake message type |
| `tls.handshake.version` | Version information in the handshake |
| `tls.handshake.ciphersuite` | Selected or offered cipher suite |
| `tls.handshake.extensions_server_name` | SNI hostname, when visible |
| `tls.handshake.extensions_supported_version` | Supported TLS versions |
| `tls.handshake.extensions_alpn_str` | Negotiated or offered application protocols |
| `tls.record.content_type` | TLS record content type |
| `tls.alert_message.desc` | TLS alert description, when decoded |

## 12. Analyzing the TLS ClientHello

The ClientHello is one of the most useful messages for initial TLS traffic investigation.

It can reveal information such as:

- Client-supported TLS versions.
- Offered cipher suites.
- Server Name Indication (SNI), when visible.
- Supported extensions.
- Application-Layer Protocol Negotiation (ALPN).
- Key share information.

### How to Inspect ClientHello

1. Open a PCAP or PCAPNG file.
2. Apply the filter:

```wireshark
tls.handshake.type == 1
```

3. Select a ClientHello packet.
4. Expand the Transport Layer Security section.
5. Inspect the handshake details.

### Why It Matters

A SOC analyst can use ClientHello information to investigate:
- Connections to unexpected hostnames.
- Unusual TLS configurations.
- Unexpected TLS versions.
- Suspicious external communication patterns.
- Potential malware command-and-control connections.

These indicators require correlation with endpoint logs, DNS data, threat intelligence, and other evidence.

## 13. TLS Alerts and Handshake Failures

TLS alerts communicate problems or conditions during a secure connection.

Common examples include:

| Alert | Description |
|---|---|
| close_notify | Graceful TLS shutdown |
| unexpected_message | Unexpected protocol message |
| bad_record_mac | Record integrity verification failed |
| handshake_failure | Handshake could not be completed |
| certificate_expired | Certificate validity period has expired |
| unknown_ca | Certificate authority is not trusted |

Some alert descriptions are specific to particular TLS versions. In TLS 1.3, many alert details are encrypted after the ServerHello, so they may not be visible in an ordinary capture.

### Wireshark Filter

```wireshark
tls.record.content_type == 21
```

Repeated handshake failures may indicate:
- Incorrect TLS configuration.
- Certificate problems.
- Incompatible protocol versions.
- Network security device interference.
- Potentially suspicious connection behavior.

A handshake failure is not automatically evidence of an attack.

## 14. Decrypting TLS Traffic in Wireshark

TLS encrypts application data, so a normal packet capture usually cannot display the actual HTTP content.

However, authorized TLS decryption is possible when suitable session secrets are available.

### Using a TLS Key Log File

Modern browsers can generate a TLS key log file for debugging.

General process:

1. Configure the browser to write TLS session secrets to a key log file.
2. Start a fresh browser session.
3. Capture the relevant traffic in Wireshark.
4. Open Wireshark Preferences.
5. Navigate to **Protocols → TLS**.
6. Locate the **(Pre)-Master-Secret log filename** field.
7. Select the key log file.
8. Revisit or reload the website and inspect the capture.

Wireshark can then use the session secrets to decrypt supported traffic.

### Important Security Notes

- Only decrypt traffic you are authorized to inspect.
- Treat key log files as highly sensitive secrets.
- Do not share key logs or decrypted captures publicly.
- A server's private key alone generally cannot decrypt modern forward-secret TLS sessions.
- Decryption depends on having the correct session secrets and a supported protocol capture.

## 15. TLS vs HTTPS

| Feature | TLS | HTTPS |
|---|---|---|
| Meaning | Transport Layer Security | HTTP over TLS |
| Purpose | Provides cryptographic protection | Provides secure web communication |
| Role | Security protocol | Application protocol combination |
| Encryption | Provides encryption | Uses TLS encryption |
| Authentication | Supports certificate-based authentication | Commonly authenticates the web server through TLS |

## 16. TLS Analysis for Cybersecurity

TLS analysis is important in SOC operations and network threat investigations.

### 1. Detecting Suspicious Encrypted Connections

Investigate:
- Connections to unfamiliar external IP addresses.
- Unexpected SNI hostnames.
- Unusual connection frequency.
- Unexpected TLS versions or configurations.
- Unusual communication timing and data volume.

### 2. Investigating Malware Command and Control (C2)

Malware may use encrypted connections to communicate with command-and-control infrastructure.

Possible indicators:
- Repeated connections to an unusual external server.
- Regular beacon-like communication intervals.
- Unexpected TLS sessions from an endpoint.
- Suspicious DNS queries followed by encrypted connections.
- Unusual connection behavior from processes that normally do not access the internet.

TLS encryption does not make communication automatically safe.

### 3. Identifying Certificate Issues

Analysts can inspect:
- Certificate issuer.
- Subject and SAN values.
- Validity period.
- Certificate chain information.
- Unexpected certificate changes.

An unfamiliar issuer or recently observed certificate is not automatically malicious. Investigate it with other evidence.

### 4. Identifying Weak or Outdated TLS Configurations

Security teams can investigate:
- Use of deprecated TLS versions.
- Weak cryptographic configurations.
- Unexpected certificate validation errors.
- Unsupported or misconfigured cipher suites.

### 5. Understanding Encrypted Traffic Behavior

Even when application content is encrypted, analysts can study:
- Source and destination IP addresses.
- Ports and protocol.
- Packet sizes.
- Connection duration.
- Traffic direction.
- Timing and frequency.

These characteristics can help identify unusual communication patterns.

## 17. Practical Lab: Analyze an HTTPS Connection

**Objective:** Capture and inspect a TLS handshake using Wireshark.

### Step 1: Start Wireshark

1. Open Wireshark.
2. Select your active network interface.
3. Start packet capture.

### Step 2: Generate HTTPS Traffic

Open a terminal and run:

```bash
curl -I https://example.com
```

This sends an HTTPS request and displays the response headers.

### Step 3: Find ClientHello

Apply:

```wireshark
tls.handshake.type == 1
```

Inspect:
- Supported TLS versions.
- Offered cipher suites.
- SNI, if visible.
- Supported extensions.

### Step 4: Find ServerHello

Apply:

```wireshark
tls.handshake.type == 2
```

Inspect the server's selected TLS parameters.

### Step 5: Inspect Certificate Information

Apply:

```wireshark
tls.handshake.type == 11
```

If certificate messages are visible, inspect the issuer, subject, validity period, and SAN information.

### Step 6: Identify Encrypted Application Data

Apply:

```wireshark
tls.app_data
```

Observe the encrypted application records.

### Step 7: Save the Capture

Save the capture as:

```text
tls-https-analysis.pcapng
```

### Lab Questions

1. What is TLS?
2. What is the difference between HTTP and HTTPS?
3. What happens during the TLS handshake?
4. What is the purpose of ClientHello?
5. What is a cipher suite?
6. What is a digital certificate?
7. What is SNI?
8. Why is HTTPS application data not normally readable in Wireshark?
9. What is the purpose of a TLS key log file?
10. What information can a SOC analyst investigate even when TLS traffic is encrypted?

## 18. Common TLS Analysis Mistakes

- Assuming TLS 1.2 and TLS 1.3 have identical handshake sequences.
- Treating every TLS alert as a security incident.
- Assuming HTTPS hides all network metadata.
- Confusing a certificate's public key with a TLS session key.
- Assuming SNI is always visible.
- Assuming all encrypted traffic uses TCP port 443.
- Expecting to read HTTPS content without session secrets.
- Assuming an unfamiliar certificate is automatically malicious.
- Sharing TLS key log files or decrypted PCAPs without considering their sensitivity.

## 19. TLS and HTTPS Interview Questions

**Q1. What is TLS?**

TLS is a cryptographic protocol that provides confidentiality, integrity, and authentication for network communication.

**Q2. What is HTTPS?**

HTTPS is HTTP communication protected by TLS.

**Q3. What is the default port for HTTPS?**

The commonly used default port is TCP 443. HTTP/3 also commonly uses UDP 443.

**Q4. What is a TLS handshake?**

It is the process through which a client and server negotiate security parameters and establish shared session keys.

**Q5. What is ClientHello?**

It is the initial TLS handshake message sent by a client to begin negotiation.

**Q6. What is a cipher suite?**

It is a set of cryptographic algorithms used to protect a TLS connection.

**Q7. What is SNI?**

Server Name Indication is a TLS extension that allows a client to indicate the hostname it wants to connect to.

**Q8. What is a digital certificate?**

It is an electronic document that binds an identity to a public key and is digitally signed by a certificate authority or another trusted issuer.

**Q9. Can Wireshark decrypt HTTPS traffic?**

Yes, when suitable TLS session secrets are available and the traffic is supported by Wireshark's decryption capabilities.

**Q10. Why is TLS 1.3 more secure than older versions?**

It removes many outdated cryptographic options, improves handshake protection, and supports modern key exchange and encryption mechanisms.

**Q11. What is forward secrecy?**

Forward secrecy helps ensure that compromise of a server's long-term private key does not expose previously recorded sessions that used properly implemented ephemeral key exchange.

**Q12. What can a SOC analyst investigate without decrypting HTTPS?**

The analyst can examine IP addresses, ports, visible TLS handshake metadata, connection timing, packet sizes, traffic volume, and other available network indicators.

## 20. Quick Revision

| Concept | Key Point |
|---|---|
| TLS | Cryptographic protocol for secure communication |
| HTTPS | HTTP protected by TLS |
| TLS 1.2 | Widely supported TLS version |
| TLS 1.3 | Modern TLS version |
| ClientHello | Starts TLS negotiation |
| ServerHello | Server selects connection parameters |
| Cipher Suite | Defines TLS encryption and hash algorithms |
| Digital Certificate | Binds an identity to a public key |
| CA | Certificate Authority |
| SNI | Indicates the requested hostname |
| TLS Alert | Reports a TLS condition or error |
| TLS Application Data | Protected application communication |
| Key Log File | Contains session secrets for authorized decryption |
| Forward Secrecy | Protects past sessions from later compromise of long-term keys |

---

**Next File:** `08_Advanced_Traffic_Analysis.md`

Next, we will explore advanced Wireshark techniques, traffic behavior, protocol anomalies, network performance indicators, and deeper packet-level investigation.
