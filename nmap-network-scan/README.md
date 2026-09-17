# Nmap Network Scan Lab

A hands-on network reconnaissance and enumeration lab built in a virtualized environment, using Nmap to identify live hosts, open ports, running services, and service versions on a target machine.

---

## Objective

The goal of this project was to perform reconnaissance and enumeration against a target virtual machine using Nmap, in order to:

- Identify whether the target host was live and reachable
- Discover open ports and the attack surface they represent
- Identify running services and their versions
- Practice interpreting scan output the way a SOC analyst or penetration tester would during the early stages of an assessment

---

## Environment

| Component | Details |
|---|---|
| Attacker Machine | Kali Linux (VirtualBox VM) |
| Target Machine | Windows 11 (VirtualBox VM) |
| Network Mode | Host-Only Adapter / NAT Network |
| Hypervisor | Oracle VirtualBox |

Both VMs were configured on the same virtual network and connectivity was verified with `ping` prior to scanning.

---

## Tools Used

- **Nmap** — network discovery and security auditing
- **Terminal** (Kali Linux)

---

## Methodology & Commands

### 1. Host Discovery
Confirms the target is alive before running deeper scans.
```bash
nmap -sn <target IP>
```

### 2. Full Port Scan
Identifies all open ports across the target using a SYN (stealth) scan.
```bash
nmap -sS -p- <target IP>
```

### 3. Service & Version Detection
Determines what software/services are bound to the open ports.
```bash
nmap -sV <target IP>
```

### 4. Aggressive Scan
Combines OS detection, version detection, script scanning, and traceroute for a fuller picture. Used with caution — this scan is noisy and easily detected by an IDS/IPS.
```bash
nmap -A <target IP>
```

---

## Findings

| Port | State | Service | Version |
|---|---|---|---|
| 135 | open | TCP | Microsoft Windows RPC |
| 139 | open | TCP | Microsoft Windows netbios-ssn |
| 445 | open | TCP | microsoft-ds? |
Running: Microsoft Windows 11
OS CPE: cpe:/o:microsoft:windows_11
OS details: Microsoft Windows 11 24H2
Network Distance: 1 hop
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

### Screenshots

<img width="1278" height="1306" alt="4" src="https://github.com/user-attachments/assets/efe9a980-3e88-496b-8287-be10ef2720ed" />
<img width="1278" height="1306" alt="3" src="https://github.com/user-attachments/assets/cefc5be9-5f8d-4dda-bccd-e01b1d65a2a5" />
<img width="1278" height="1306" alt="2" src="https://github.com/user-attachments/assets/9c5300e6-fa90-424d-b883-78b318d48b76" />
<img width="1278" height="1306" alt="1" src="https://github.com/user-attachments/assets/b9548052-1018-47b5-bfbb-d7776e25f72c" />

---

## Analysis

- Port 135 — Microsoft RPC (Remote Procedure Call) Handles remote procedure calls, which Windows uses for inter-process communication and remote management. RPC is frequently targeted for enumeration because it can leak information about the system (hostname, running services, sometimes user accounts) and has historically been the entry point for major exploits such as MS03-026 (RPC/DCOM), which was used by the Blaster worm.

- Port 139 — NetBIOS Session Service Part of the legacy NetBIOS-over-TCP/IP suite, used for older file/printer sharing and name resolution. Its presence indicates NetBIOS is enabled on the target, which can be leveraged for network enumeration (device names, workgroup/domain info) and was historically exploited via null session attacks that allowed unauthenticated access to user/group data.

- Port 445 — Microsoft-DS (SMB) The modern SMB port, used for file sharing, printer sharing, and inter-process communication on Windows networks. This is one of the most significant ports from a security standpoint — it was the vector for EternalBlue (MS17-010), which powered WannaCry and NotPetya. Any exposed SMB service should be checked for version and patch level, since unpatched SMB remains one of the most exploited services in real-world breaches.

- Overall Analysis The combination of ports 135, 139, and 445 open indicates this Windows target has File and Printer Sharing enabled, exposing SMB and legacy NetBIOS services. This service trio is one of the most historically exploited combinations in Windows environments, matching the footprint associated with worms like Blaster and WannaCry. In a real-world assessment, the next step would be running targeted SMB enumeration scripts, such as:
```bash
- nmap --script smb-enum-shares,smb-os-discovery <target IP>
```
---

## Lessons Learned

- Scan results depend heavily on environment configuration, not just command syntax. My first -sS -p- scan came back with all ports filtered and no response — the issue wasn't Nmap, it was that Windows Defender Firewall was blocking inbound traffic by default. Disabling it on the target VM revealed the real port states.
  
- Privilege level matters for scan accuracy. SYN scans (-sS) require raw socket access, so running Nmap with sudo is necessary for the results to reflect what -sS is actually designed to do.
- A "filtered, no response" result is diagnostic information in itself. Rather than being a failure, it pointed directly at a firewall/network configuration issue, which mirrors real-world scenarios where filtered ports often mean a firewall is actively dropping traffic rather than the host being unreachable.
  
- Different scan types serve different purposes in a real workflow: -sn confirms a host is alive, -sS -p- maps the full port range, -sV identifies service versions, and -A pulls it all together (with added noise) — understanding when to use each is more valuable than memorizing the flags.
  
- Open ports tell a story about the target's role and risk. Seeing 135, 139, and 445 open together immediately signals a Windows host with File and Printer Sharing enabled — the same footprint associated with major historical exploits (Blaster, WannaCry). Recognizing these patterns is a core skill in translating raw scan output into actionable security analysis.
  
- Documentation is what turns a scan into a project. The scan itself takes seconds; explaining what the ports mean, why they matter, and what the logical next step would be (e.g., SMB enumeration) is what demonstrates actual security analysis skill to anyone reviewing the work.

---

## Disclaimer

This project was performed entirely within an isolated, self-owned virtual lab environment for educational purposes. No scans were performed against systems without explicit authorization.
