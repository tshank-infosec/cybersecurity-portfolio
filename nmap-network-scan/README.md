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

> Replace this section with your actual scan results.

| Port | State | Service | Version |
|---|---|---|---|
| 22 | open | SSH | OpenSSH x.x |
| 80 | open | HTTP | Apache httpd x.x |
| ... | ... | ... | ... |

**OS Fingerprint:** _(fill in from `-A` scan output)_

### Screenshots

> Add terminal screenshots for each scan stage.

- `screenshots/01-host-discovery.png`
- `screenshots/02-port-scan.png`
- `screenshots/03-service-version.png`
- `screenshots/04-aggressive-scan.png`

---

## Analysis

- **Open SSH (22)** indicates remote administrative access is enabled on the target — a common target for brute-force and credential-based attacks if not hardened.
- **Open HTTP (80)** indicates a web server is running, expanding the attack surface to web application vulnerabilities.
- **Version banners** (e.g., Apache 2.4.29) can be cross-referenced against CVE databases (NVD, Exploit-DB) to check for known vulnerabilities affecting that specific version.
- Together, this information forms the initial attack surface map an attacker or pentester would use to plan further enumeration or exploitation.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Active Scanning: Scanning IP Blocks | T1595.001 | Used to discover live hosts on the network |
| Active Scanning: Vulnerability Scanning | T1595.002 | Version detection used to identify potentially vulnerable services |
| Network Service Discovery | T1046 | Enumeration of open ports and services on the target |
| Gather Victim Host Information: Software | T1592.002 | Identifying software and versions running on the target |

---

## Lessons Learned

- Different Nmap scan types (`-sn`, `-sS`, `-sV`, `-A`) each serve a distinct purpose in the reconnaissance workflow — from a quiet host check to a full aggressive sweep.
- Service version detection is a critical bridge between reconnaissance and vulnerability research.
- Aggressive scans (`-A`) provide the most detail but generate significant network noise, making them unsuitable for stealthy real-world engagements without careful scoping.
- Building and documenting a lab like this reinforces the full recon → enumeration → analysis workflow used in real SOC and penetration testing work.

---

## Disclaimer

This project was performed entirely within an isolated, self-owned virtual lab environment for educational purposes. No scans were performed against systems without explicit authorization.
