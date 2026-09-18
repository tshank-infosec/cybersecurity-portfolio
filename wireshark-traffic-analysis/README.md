# Wireshark Traffic Analysis

A hands-on network traffic analysis project using Wireshark to capture, filter, and interpret packet-level data generated during an Nmap SYN scan — identifying scan behavior at the protocol level and confirming open ports directly from the wire.

---

## Objective

The goal of this project was to capture and analyze the network traffic generated during an active Nmap scan in order to:

- Observe what an Nmap SYN scan actually looks like at the packet level
- Confirm open ports independently of Nmap's own reported output
- Understand ARP's role in host discovery on a local network
- Practice the packet analysis workflow used in SOC and incident response work when investigating scanning or reconnaissance activity

---

## Environment

| Component | Details |
|---|---|
| Capture Tool | Wireshark (live capture on `eth0`) |
| Attacker / Scanning Host | Kali Linux — `192.168.56.101` (MAC `08:00:27:5a:87:bc`) |
| Target Host | Windows 11 — `192.168.56.102` (MAC `08:00:27:b9:e1:e4`) |
| Network Mode | VirtualBox Host-Only Adapter |
| Hypervisor | Oracle VirtualBox |
| Capture File | `nmapscancap.pcapng` |

---

## Tools Used

- **Wireshark** — live packet capture and protocol analysis
- **Nmap** — traffic generator (`sudo nmap -sS -p- -T4 192.168.56.102`), run from the companion Nmap lab

---

## Methodology

### 1. Start the Capture
Started a live Wireshark capture on interface `eth0` before launching the scan, so ARP resolution and the full scan would be recorded from the first packet.

### 2. Generate Traffic
Ran the full-port SYN scan against the Windows target from Kali, in a separate terminal, while the capture was running:
```bash
sudo nmap -sS -p- -T4 192.168.56.102
```

### 3. Stop and Save the Capture
Stopped the capture once the scan completed (23.58 seconds, 131,261 packets) and saved it as `nmapscancap.pcapng`.

### 4. Filter and Isolate Traffic
Applied display filters to break the capture down by protocol and behavior:
```
tcp
arp
tcp.stream eq 5
```

### 5. Follow a TCP Stream
Used **Follow → TCP Stream** on the exchange for port 139 to see the complete three-packet sequence for that port in isolation.

---

## Findings

### Nmap Scan Output (for reference)
```
Nmap scan report for 192.168.56.102
Host is up (0.0012s latency).
Not shown: 65524 closed tcp ports (reset)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5040/tcp  open  unknown
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
MAC Address: 08:00:27:B9:E1:E4 (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 23.58 seconds
```

### Packet-Level Observations

| # | Source | Destination | Protocol | Port(s) | Observation |
|---|---|---|---|---|---|
| 1 | 192.168.56.101 | 192.168.56.102 | ARP | — | "Who has 192.168.56.102? Tell 192.168.56.101" — attacker resolving the target's MAC address before scanning |
| 2 | 192.168.56.102 | 192.168.56.101 | ARP | — | Reply: "192.168.56.102 is at 08:00:27:b9:e1:e4" |
| 3 | 192.168.56.101 | 192.168.56.102 | TCP | 54160 → various | Rapid sequential SYN packets from a single fixed source port (54160) to hundreds of destination ports in order — the signature of Nmap's SYN scan |
| 4 | 192.168.56.102 | 192.168.56.101 | TCP | various → 54160 | Immediate `RST, ACK` replies on closed ports (e.g., 256, 1720, 199, 143, 113, 554, 443, 53, 587) |
| 5 | 192.168.56.102 | 192.168.56.101 | TCP | 139 → 54160 | `SYN, ACK` reply — confirms port 139 is genuinely open, independent of Nmap's own report |
| 6 | 192.168.56.101 | 192.168.56.102 | TCP | 54160 → 139 | Immediate `RST` sent by Kali right after receiving the SYN-ACK — Nmap tearing down the connection without completing the handshake |

### Screenshots

**1. Live capture during the Nmap scan**
Shows the capture running on `eth0` in real time, alongside the terminal output of the completed Nmap scan. The packet list shows the flood of SYN packets from `192.168.56.101:54160` to sequential ports on `192.168.56.102`, followed by red-highlighted `RST, ACK` responses (Wireshark's default coloring for reset packets).

![Live capture during Nmap scan](screenshots/01-live-capture-nmap-scan.png)

**2. Full scan filtered by `tcp`**
The saved capture (`nmapscancap.pcapng`) filtered to `tcp`, showing the scan from the beginning: SYN packets sent to ports 256, 1720, 199, 143, 113, 139, 554, 443, 53, and 587 in rapid succession, each from the same source port (54160). Note packet #22 — a `SYN, ACK` from port 139, standing out as the one genuine "open port" response among a run of resets.

![Full scan filtered by tcp](screenshots/02-tcp-filter-full-scan.png)

**3. ARP resolution filtered by `arp`**
Shows the ARP exchange that occurred before scanning began: Kali (`192.168.56.101`) asking who has `192.168.56.102`, and the target replying with its MAC address (`08:00:27:b9:e1:e4`). A second, later ARP exchange shows the target resolving Kali's MAC address (`08:00:27:5a:87:bc`) in return.

![ARP resolution](screenshots/03-arp-resolution.png)

**4. Isolated TCP stream for port 139 (`tcp.stream eq 5`)**
Filtering to this single stream isolates the exact three packets for port 139: the SYN from Kali, the SYN-ACK from the Windows target confirming the port is open, and the RST Kali sends back immediately — never completing a full connection.

![TCP stream for port 139](screenshots/04-tcp-stream-port139.png)

---

## Analysis

**ARP resolution precedes scanning**
Before any TCP traffic was sent, Kali and the Windows target exchanged ARP requests to resolve each other's MAC addresses. This is expected behavior on a local Ethernet segment — IP-based scanning can't begin until the sender knows the destination's hardware address, so ARP traffic is a normal and necessary precursor to any local network scan.

**The SYN scan is visible as a clear, repeatable packet pattern**
Every scanned port followed the same three-part pattern: a SYN from Kali on a single fixed source port (54160), followed by either a `RST, ACK` (closed port) or a `SYN, ACK` (open port) from the target. The consistent source port and the sheer sequential volume of destination ports is a strong, recognizable signature of a scanning tool — a real device browsing the network normally would not generate this pattern.

**Open ports were confirmed independently from Nmap's own output**
Rather than relying only on Nmap's summary, the capture shows a `SYN, ACK` reply from the target on port 139 — direct packet-level proof that the port is open, independent of what the scanning tool reported. This is the core value of traffic analysis: it validates tool output using raw evidence from the wire.

**Only TCP (and ARP) traffic appears — no application-layer protocols**
No SMB, HTTP, or DNS traffic appears anywhere in the capture. This is expected and consistent with how a SYN scan works: Nmap deliberately sends a `RST` immediately after receiving a `SYN, ACK`, rather than completing the handshake. Because the underlying TCP connection is never fully established, no application-layer protocol (such as SMB on port 445 or 139) ever gets negotiated, so none of that traffic exists to capture. The scan proves a port is open without ever "talking" to the service running on it.

**Overall conclusion**
This capture confirms, at the packet level, that the Windows target has ports 135 (RPC), 139 (NetBIOS-SSN), and 445 (SMB) open — the same finding as the Nmap scan itself, but now verified independently from raw traffic. The absence of any application-layer protocol data is not a gap in the capture; it's the direct, expected result of how a half-open SYN scan behaves. To capture actual SMB protocol data, a full connection would need to be established — for example, using `smbclient -L 192.168.56.102` from Kali — which would complete the handshake and allow SMB negotiation to occur.

---

## Skills Demonstrated

- Live packet capture and `.pcapng` file management in Wireshark
- Protocol-level filtering (`tcp`, `arp`, `tcp.stream eq N`)
- Recognizing a SYN scan's packet signature (fixed source port, sequential destination ports, SYN/RST-ACK/SYN-ACK pattern)
- Following and interpreting an isolated TCP stream
- Independently validating scan-tool output (Nmap) using raw packet evidence
- Understanding the relationship between TCP handshake state and application-layer protocol visibility

---

## Lessons Learned

- ARP always precedes IP-based communication on a local network — seeing it first in the capture is expected, not noise to filter out.
- A SYN scan leaves a very distinct, recognizable fingerprint in traffic: one fixed source port hitting many sequential destination ports, with binary SYN-ACK/RST-ACK responses. This pattern is worth recognizing on sight when reviewing traffic for reconnaissance activity.
- Traffic analysis and active scanning validate each other — Nmap's port list and Wireshark's raw packets told the same story about ports 135, 139, and 445, but the packet capture proved it independently.
- A half-open (SYN) scan intentionally never completes a TCP handshake, which means no application-layer protocol (SMB, HTTP, etc.) will ever appear in that traffic — this is a deliberate design choice in Nmap to stay fast and less intrusive, not a limitation of Wireshark or the capture.
- Isolating a single stream (`tcp.stream eq N`) is far more useful for explaining a specific finding than scrolling through a raw packet list — it turns thousands of packets into a clean three-line story for a single port.

---

## Disclaimer

This project was performed entirely within an isolated, self-owned virtual lab environment for educational purposes. No traffic was captured or analyzed from systems or networks without explicit authorization.
