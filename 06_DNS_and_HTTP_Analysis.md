# 06 - DNS and HTTP Analysis with Wireshark

## 1. Introduction

DNS (Domain Name System) and HTTP (Hypertext Transfer Protocol) are two important protocols used in everyday network communication.

- **DNS** translates domain names into IP addresses.
- **HTTP** enables communication between web browsers and web servers.

In cybersecurity, analyzing DNS and HTTP traffic helps security analysts investigate suspicious domains, phishing websites, malware communication, and unusual web requests.

## 2. Understanding DNS

DNS stands for **Domain Name System**.

It translates human-readable domain names into IP addresses that computers use to communicate.

For example:

```text
User enters: example.com

DNS Resolution:

example.com
     |
     v
DNS Server
     |
     v
93.184.216.34
```

Without DNS, users would need to remember IP addresses instead of domain names.

### How DNS Resolution Works

1. A user enters a domain name in a browser.
2. The device checks its local DNS cache.
3. If the address is not cached, it sends a DNS query to a DNS resolver.
4. The resolver obtains the required DNS information.
5. The resolver returns the result to the client.
6. The browser uses the returned IP address to connect to the website.

The resolver may answer from its cache or query other DNS servers, depending on the situation.

## 3. Common DNS Record Types

DNS records provide different types of information about a domain.

| Record | Purpose |
|---|---|
| A | Maps a domain to an IPv4 address |
| AAAA | Maps a domain to an IPv6 address |
| CNAME | Creates an alias for another domain name |
| MX | Identifies mail servers for a domain |
| NS | Identifies authoritative name servers |
| TXT | Stores text information, often used for verification and email security |
| PTR | Maps an IP address to a domain name |
| SOA | Contains administrative information about a DNS zone |

### Example

```text
example.com       A       93.184.216.34
example.com       AAAA    2606:2800:220:1:248:1893:25c8:1946
example.com       MX      mail.example.com
```

These are illustrative record examples.

## 4. DNS Queries and Responses

DNS communication usually uses UDP port 53 for standard queries and responses. TCP port 53 is also used in situations such as large responses and zone transfers.

### DNS Query

A DNS query is a request asking for information about a domain.

Example:

```text
What is the IPv4 address of example.com?
```

### DNS Response

The DNS server responds with the requested information, such as an IP address.

A response may also contain:
- Answer records.
- Authority records.
- Additional records.
- An error or response code.

### Important DNS Response Codes

| Response Code | Meaning |
|---|---|
| NOERROR | Query completed without a DNS error |
| NXDOMAIN | Requested domain name does not exist |
| SERVFAIL | Server failed to process the query |
| REFUSED | Server refused the query |

## 5. DNS Analysis in Wireshark

Wireshark allows analysts to inspect DNS queries and responses.

### Useful DNS Filters

Display all DNS packets:

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

DNS traffic using UDP port 53:

```wireshark
udp.port == 53
```

DNS traffic using TCP port 53:

```wireshark
tcp.port == 53
```

Find DNS errors:

```wireshark
dns.flags.rcode != 0
```

Find NXDOMAIN responses:

```wireshark
dns.flags.rcode == 3
```

Find queries for a specific domain:

```wireshark
dns.qry.name == "example.com"
```

Find A record queries:

```wireshark
dns.qry.type == 1
```

Find AAAA record queries:

```wireshark
dns.qry.type == 28
```

### Important DNS Fields

| Field | Description |
|---|---|
| `dns.qry.name` | Requested domain name |
| `dns.qry.type` | Type of DNS record requested |
| `dns.flags.response` | Identifies query or response |
| `dns.flags.rcode` | DNS response code |
| `dns.a` | IPv4 address in a DNS answer |
| `dns.aaaa` | IPv6 address in a DNS answer |
| `dns.id` | DNS transaction identifier |

## 6. DNS Security Analysis

DNS traffic can provide useful indicators during security investigations.

### 1. Suspicious Domain Queries

Analysts may investigate:
- Random-looking domain names.
- Unusually long domain names.
- Newly observed or unfamiliar domains.
- Domains associated with suspicious activity.
- Repeated queries to unexpected external domains.

These patterns can sometimes be associated with malware or phishing infrastructure, but they are not proof of malicious activity by themselves.

### 2. DNS Tunneling

DNS tunneling is a technique in which data is encoded inside DNS queries and responses to communicate through DNS infrastructure.

Possible indicators:
- Unusually long DNS query names.
- Large numbers of queries to one domain.
- Unusual subdomain patterns.
- High volumes of TXT record queries.
- Repeated DNS requests at unusual intervals.

Useful filters:

```wireshark
dns.qry.name
```

```wireshark
dns.qry.type == 16
```

Record type 16 represents TXT records.

**Security note:** DNS tunneling detection requires traffic context and statistical analysis. TXT queries alone do not indicate tunneling.

### 3. DNS Spoofing

DNS spoofing involves providing false DNS information to redirect a user toward an incorrect destination.

Possible indicators:
- Unexpected IP addresses in DNS responses.
- Different answers for repeated queries.
- DNS responses from unexpected servers.
- Suspicious changes in DNS behavior.

Analysts should compare DNS responses with trusted DNS information and other network evidence.

## 7. Understanding HTTP

HTTP stands for **Hypertext Transfer Protocol**.

It is an application-layer protocol used for communication between clients and web servers.

For example:

1. A user opens a website.
2. The browser sends an HTTP request.
3. The web server processes the request.
4. The server sends an HTTP response.
5. The browser displays the content.

HTTP commonly uses TCP port 80.

Unlike HTTPS, ordinary HTTP traffic is not encrypted, so its contents may be visible in a packet capture.

## 8. HTTP Request Methods

HTTP methods describe the action a client wants to perform.

| Method | Purpose |
|---|---|
| GET | Retrieves a resource |
| POST | Submits data to a server |
| PUT | Creates or replaces a resource |
| PATCH | Partially updates a resource |
| DELETE | Requests deletion of a resource |
| HEAD | Requests response headers without the response body |
| OPTIONS | Requests communication options supported by a resource |

### Example HTTP Request

```http
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
```

Important parts:

- `GET` — request method.
- `/index.html` — requested resource.
- `HTTP/1.1` — HTTP version.
- `Host` — destination hostname.
- `User-Agent` — identifies the client software.

## 9. HTTP Response and Status Codes

An HTTP response is sent by the server after processing a request.

Example:

```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256
```

### Common HTTP Status Codes

| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 301 | Moved Permanently |
| 302 | Found / Temporary Redirect |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |

### Status Code Categories

| Category | Range | Meaning |
|---|---|---|
| 1xx | 100–199 | Informational |
| 2xx | 200–299 | Successful |
| 3xx | 300–399 | Redirection |
| 4xx | 400–499 | Client errors |
| 5xx | 500–599 | Server errors |

## 10. HTTP Analysis in Wireshark

Wireshark can display HTTP requests, responses, headers, and readable payloads when traffic is unencrypted.

### Useful HTTP Filters

Display HTTP traffic:

```wireshark
http
```

HTTP GET requests:

```wireshark
http.request.method == "GET"
```

HTTP POST requests:

```wireshark
http.request.method == "POST"
```

HTTP responses:

```wireshark
http.response
```

Find HTTP 404 responses:

```wireshark
http.response.code == 404
```

Find HTTP 200 responses:

```wireshark
http.response.code == 200
```

Find requests to a specific host:

```wireshark
http.host == "example.com"
```

Find requests containing a specific URI:

```wireshark
http.request.uri contains "login"
```

Find HTTP traffic on port 80:

```wireshark
tcp.port == 80
```

### Important HTTP Fields

| Field | Description |
|---|---|
| `http.request.method` | HTTP request method |
| `http.host` | Requested hostname |
| `http.request.uri` | Requested resource path |
| `http.response.code` | HTTP response status code |
| `http.user_agent` | Client software information |
| `http.content_type` | Type of returned content |
| `http.cookie` | Cookie information |
| `http.authorization` | Authorization header, when present |

## 11. HTTP Stream Analysis

Wireshark provides the **Follow HTTP Stream** feature through its TCP stream analysis.

### Steps

1. Open a packet capture containing HTTP traffic.
2. Apply the filter:

```wireshark
http
```

3. Select an HTTP packet.
4. Right-click the packet.
5. Choose **Follow → HTTP Stream** if available, or use **Follow → TCP Stream**.
6. Examine the request and response.

You may be able to observe:
- Requested URLs and paths.
- HTTP methods.
- Request headers.
- Server response headers.
- Readable response content.

The availability of readable content depends on the protocol, capture completeness, and whether the data is encrypted.

## 12. HTTP Security Analysis

HTTP analysis is useful when investigating web-based threats.

### 1. Detecting Suspicious HTTP Requests

Look for:
- Unexpected POST requests.
- Requests to unfamiliar domains.
- Unusual URI paths.
- Suspicious query parameters.
- Unexpected file downloads.
- Repeated requests to unusual endpoints.

### 2. Investigating Possible Data Exposure

Unencrypted HTTP can expose sensitive information if applications transmit it without appropriate protection.

Analysts may inspect:
- Login requests.
- Authorization headers.
- Cookies.
- Form submissions.
- Response content.

Never share captured credentials or personal information. Handle packet captures as sensitive evidence.

### 3. Identifying Suspicious User-Agent Values

The User-Agent header provides information about the client software.

A suspicious or unusual User-Agent may deserve investigation, especially when combined with other indicators.

Filter:

```wireshark
http.user_agent
```

### 4. Investigating File Downloads

Analysts can inspect HTTP responses and content types to investigate unexpected downloads.

Useful filter:

```wireshark
http.response
```

Examine:
- Content-Type.
- Content-Length.
- Requested URI.
- Source and destination addresses.
- Download timing.

A file extension or content type alone does not establish that a file is malicious.

## 13. DNS vs HTTP

| Feature | DNS | HTTP |
|---|---|---|
| Full Form | Domain Name System | Hypertext Transfer Protocol |
| Main Purpose | Domain name resolution | Web communication |
| OSI Layer | Application | Application |
| Common Port | 53 | 80 |
| Transport | Usually UDP; also TCP | Usually TCP |
| Main Function | Resolves domain names | Transfers web resources |
| Security Use | Investigating suspicious domains | Investigating suspicious web requests |

## 14. DNS and HTTP Practical Lab

**Objective:** Capture DNS and HTTP traffic and understand how domain resolution and web communication work.

### Step 1: Start Wireshark

1. Open Wireshark.
2. Select your active network interface.
3. Start packet capture.
4. Generate normal DNS and web traffic.

### Step 2: Generate DNS Traffic

Open a terminal and run:

```bash
nslookup example.com
```

This sends a DNS query through the configured resolver.

Apply this filter:

```wireshark
dns
```

Observe:
- Query name.
- Query type.
- DNS server address.
- Response address.
- DNS response code.

### Step 3: Generate HTTP Traffic

Use a test website that supports plain HTTP:

```bash
curl -I http://neverssl.com
```

This sends an HTTP request and displays response headers.

Apply:

```wireshark
http
```

Observe:
- HTTP method.
- Host header.
- URI.
- Status code.
- Server response headers.

If no HTTP traffic appears, check whether the request was redirected, whether the capture interface is correct, and whether the traffic was actually unencrypted HTTP.

### Step 4: Investigate DNS Errors

Apply:

```wireshark
dns.flags.rcode != 0
```

Look for failed DNS responses and identify their response codes.

### Step 5: Investigate HTTP Requests

Apply:

```wireshark
http.request
```

Examine request methods, hostnames, and requested paths.

### Step 6: Save the Capture

1. Stop packet capture.
2. Select File → Save As.
3. Save the file in PCAPNG format.
4. Use a descriptive filename such as:

```text
dns-http-analysis.pcapng
```

### Lab Questions

1. What is DNS, and why is it required?
2. What is the difference between A and AAAA records?
3. Which port is commonly used by DNS?
4. What does NXDOMAIN mean?
5. What is the difference between HTTP GET and POST?
6. What does HTTP status code 404 indicate?
7. Why is unencrypted HTTP traffic a security concern?
8. What is DNS tunneling?
9. How can Wireshark identify DNS queries?
10. What is the purpose of the HTTP Host header?

## 15. Common DNS and HTTP Analysis Mistakes

- Assuming every NXDOMAIN response indicates malicious activity.
- Treating every unfamiliar domain as a threat.
- Confusing DNS queries with DNS responses.
- Assuming DNS always uses UDP.
- Confusing HTTP status codes with TCP status or flags.
- Assuming HTTPS content is readable in a normal packet capture.
- Treating every POST request as suspicious.
- Ignoring the source and destination IP addresses.
- Forgetting to examine the complete request-response sequence.
- Assuming a suspicious User-Agent proves malware activity.

## 16. DNS and HTTP Interview Questions

**Q1. What is DNS?**

DNS is a naming system that translates domain names into IP addresses.

**Q2. What is the difference between A and AAAA records?**

A records map domain names to IPv4 addresses, while AAAA records map them to IPv6 addresses.

**Q3. What is an NXDOMAIN response?**

It indicates that the requested domain name does not exist.

**Q4. What is DNS tunneling?**

It is a technique that carries encoded data inside DNS queries and responses.

**Q5. What is HTTP?**

HTTP is an application-layer protocol used for communication between web clients and servers.

**Q6. What is the difference between GET and POST?**

GET is generally used to retrieve resources, while POST is generally used to submit data for processing.

**Q7. What does HTTP 403 mean?**

It means the server understood the request but refused access.

**Q8. Why is HTTP considered less secure than HTTPS?**

Ordinary HTTP does not encrypt application data during transmission, making it potentially readable by parties able to capture the traffic.

**Q9. How can you find HTTP GET requests in Wireshark?**

Use:

```wireshark
http.request.method == "GET"
```

**Q10. How can you identify DNS queries in Wireshark?**

Use:

```wireshark
dns.flags.response == 0
```

## 17. Quick Revision

| Concept | Key Point |
|---|---|
| DNS | Converts domain names into IP addresses |
| A Record | IPv4 address |
| AAAA Record | IPv6 address |
| CNAME | Domain alias |
| MX | Mail server information |
| DNS Port | 53 |
| NXDOMAIN | Domain does not exist |
| DNS Tunneling | Data carried through DNS traffic |
| HTTP | Web communication protocol |
| HTTP Port | 80 |
| GET | Retrieves a resource |
| POST | Submits data |
| HTTP 200 | Successful response |
| HTTP 404 | Resource not found |
| HTTP 500 | Internal server error |
| Follow TCP Stream | Reconstructs TCP communication |

