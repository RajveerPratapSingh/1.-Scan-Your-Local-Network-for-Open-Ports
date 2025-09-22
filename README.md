# Scan Your Local Network for Open Ports

**Purpose**

This document presents a concise, professional guide for performing a lawful local network scan with Nmap on Windows 11. It describes the commands used, summarizes the findings, and lists practical remediation steps for common exposed services.

---

## Prerequisites
- Administrative access to the target host or written permission to scan the network.
- Nmap installed on Windows (example binary: `nmap-7.98-setup.exe`).
- PowerShell or Command Prompt access on the scanning host.

---

## Environment
From `ipconfig` on the scanning host:

```
For example
IPv4 Address : 192.168.50.1
Subnet Mask  : 255.255.255.0
```

Network range: `192.168.50.0/24`.

---

## Commands executed
1. Subnet scan (SYN scan) to discover active hosts and open ports:

```
nmap -sS 192.168.50.0/24
```

2. Targeted service and version scan for a specific host:

```
nmap -sS -sV -p 135,139,445,4343,4449 192.168.50.1
```

Generated artifacts:
- [`scan.html`](scan/scan.html) — subnet scan results.
- [`service_scan.html`](scan/service_scan.html) — service/version scan results for 192.168.50.1.
- [`Service assessment`](scan/port_service_assessment.md) — analyst notes and mitigation suggestions.

---

## Tools used
- Nmap (port scanning)
- ChatGPT (analysis and guidance)
- PowerShell (ipconfig)


