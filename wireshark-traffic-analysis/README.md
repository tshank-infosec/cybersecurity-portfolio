# Wireshark Traffic Analysis

A hands-on network traffic analysis project using Wireshark to capture, filter, and interpret packet-level data — identifying protocols in use, unusual behavior, and potential indicators of compromise (IOCs).

---

## Objective

The goal of this project was to analyze network traffic at the packet level in order to:

- Identify what protocols and services were active on the network
- Filter and isolate suspicious or anomalous traffic
- Extract metadata (IPs, ports, protocols, payload indicators) to interpret what was actually happening on the wire
- Practice the packet analysis workflow used in SOC and incident response work when investigating potential threats

---

## Environment

| Component | Details |
|---|---|
| Capture Tool | Wireshark |
| Capture Source | _(e.g., live capture on Kali VM / provided .pcap file)_ |
| Network Mode | Host-Only Adapter / NAT Network |
| Hypervisor | Oracle VirtualBox |

> Fill in whether you captured live traffic yourself (e.g., during the Nmap lab) or analyzed a sample/downloaded PCAP file.

---

## Tools Used

- **Wireshark** — packet capture and protocol analysis
- **PCAP files** — recorded traffic for offline analysis
- *(Optional)* **tshark** — command-line packet analysis for scripting/automation

---

## Methodology

### 1. Capture or Load Traffic
- Started a live capture on the relevant interface, or opened an existing `.pcap` file
- Let traffic run long enough to capture a meaningful sample (e.g., during an Nmap scan, web browsing session, or file transfer)

### 2. Filter by Protocol
Used Wireshark display filters to narrow down traffic by type:
```
tcp
udp
http
dns
smb2
icmp
```

### 3. Identify Suspicious or Notable Packets
Looked for patterns indicating scanning, unencrypted credentials, unusual ports, or repeated connection attempts:
```
tcp.flags.syn == 1 and tcp.flags.ack == 0
http.request
dns.qry.name contains "<suspicious domain>"
tcp.port == 445
```

### 4. Extract Metadata
For packets of interest, recorded:
- Source and destination IP addresses
- Source and destination ports
- Protocol
- Payload snippets (where visible/unencrypted)
- Timing/frequency of packets (e.g., rapid SYNs suggesting a scan)

### 5. Follow Streams
Used **Follow → TCP Stream** (or UDP/HTTP Stream) on key conversations to reconstruct the full exchange and understand context.

---

## Findings

> Replace this section with your actual capture results.

| # | Source IP | Destination IP | Protocol | Port | Observation |
|---|---|---|---|---|---|
| 1 | 192.168.56.101 | 192.168.56.102 | TCP (SYN) | 1–65535 | Rapid sequential SYN packets to multiple ports — consistent with a port scan |
| 2 | 192.168.56.102 | 192.168.56.101 | SMB | 445 | SMB negotiation traffic observed |
| 3 | ... | ... | ... | ... | ... |

### Screenshots

> Add Wireshark screenshots for each stage of analysis.

- `screenshots/01-capture-overview.png`
- `screenshots/02-protocol-filter.png`
- `screenshots/03-suspicious-packets.png`
- `screenshots/04-followed-stream.png`

---

## Analysis

> Replace with analysis based on what you actually observed. Example structure below:

**Rapid SYN packets across many ports (no completed handshake)**
A high volume of SYN packets sent to sequential or numerous ports from a single source, without completed three-way handshakes, is a classic signature of a port scan (e.g., an Nmap SYN scan). This matches the scanning activity performed in the companion Nmap lab and demonstrates how scanning activity appears "on the wire" versus from the scanning tool's own output.

**SMB traffic on port 445**
Observed SMB negotiation traffic confirms the target was running the SMB service identified during the Nmap phase. Analyzing this traffic in Wireshark helps confirm findings from port scanning and can reveal additional details, such as SMB version or authentication attempts, that a port scan alone would not show.

**Unencrypted protocol traffic (if observed)**
Any plaintext protocols observed (HTTP, FTP, Telnet) should be flagged, since credentials or sensitive data sent over these protocols can be captured and read directly from the packet payload — a significant risk in any real network.

---

## Skills Demonstrated

- Packet-level traffic analysis using Wireshark
- Protocol identification and filtering (TCP, UDP, DNS, HTTP, SMB, ICMP)
- Recognizing attack patterns (e.g., scanning behavior) at the network layer
- Correlating traffic analysis with tool-based reconnaissance (Nmap) findings
- Extracting and interpreting packet metadata to support threat identification

---

## Lessons Learned

- Traffic analysis and active scanning are two sides of the same investigation — Wireshark shows what a scan actually looks like on the network, complementing what Nmap reports from the scanning side.
- Display filters are essential for cutting through noise; without filtering, even a short capture can contain thousands of irrelevant packets.
- Following a full TCP stream provides far more context than looking at individual packets in isolation.
- Recognizing protocol-level patterns (repeated SYNs, plaintext credentials, unusual ports) is a foundational skill for both defensive monitoring and offensive reconnaissance validation.

---

## Disclaimer

This project was performed entirely within an isolated, self-owned virtual lab environment for educational purposes. No traffic was captured or analyzed from systems or networks without explicit authorization.
