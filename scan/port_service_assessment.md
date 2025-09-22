# Port & Service Risk Assessment (Host: 192.168.56.1)

## A — Services Overview
- **135/tcp — msrpc**  
  Windows RPC Endpoint Mapper (DCOM, service control, remote management).  
- **139/tcp — netbios-ssn**  
  Legacy SMB over NetBIOS (file/printer sharing, session services).  
- **445/tcp — microsoft-ds**  
  Modern SMB (v1/v2/v3) for file shares, domain services, IPC.  
- **4343/tcp — ssl/unicall?**  
  Nonstandard TLS service, likely vendor/custom app.  
- **4449/tcp — ssl/privatewire?**  
  Nonstandard TLS service, likely vendor/custom app.  

---

## B — Security Risks
- **135 (RPC):** High exploit history (RCE, info disclosure). Exposed endpoints → attack surface.  
- **139 (NetBIOS):** Legacy, weak security, leaks names/shares, supports null sessions.  
- **445 (SMB):** Critical target (ransomware, worms, NTLM relay). Dangerous if exposed.  
- **4343/4449 (Unknown TLS):** Unclear ownership, TLS misconfig possible, vendor bugs likely.  

---

## C — Investigation Steps
1. **Nmap scan for detail:**  
   ```bash
   nmap -sS -sV --version-intensity 9 --script=banner,ssl-cert    -p 135,139,445,4343,4449 192.168.56.1 -oN moreinfo.txt
   ```
2. **Check Windows host (Admin PowerShell):**
   ```powershell
   Get-SmbServerConfiguration | Format-List
   Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
   Get-SmbShare | Format-Table Name,Path
   Get-NetFirewallRule -Direction Inbound | Where { $_.DisplayName -match 'SMB|RPC' }
   ```
3. **Identify unknown ports:**  
   ```powershell
   netstat -ano | findstr ":4343"
   netstat -ano | findstr ":4449"
   tasklist /fi "PID eq <pid>"
   ```
4. **Inspect TLS certs:**  
   ```bash
   openssl s_client -connect 192.168.56.1:4343 -servername 192.168.56.1 < /dev/null
   openssl s_client -connect 192.168.56.1:4449 -servername 192.168.56.1 < /dev/null
   ```

---

## D — Remediation & Hardening
- **Block external exposure** (edge firewall).  
- **Disable SMBv1** (`SMB1Protocol`).  
- **Restrict access** to 135/139/445 via firewall rules (only mgmt subnet/VPN).  
- **Patch system & apps** (Windows Update + vendor patches for 4343/4449).  
- **Investigate unknown services** (stop/uninstall if unnecessary).  
- **Harden TLS** (valid certs, TLS 1.2/1.3 only).  
- **Enforce strong auth & monitoring** (no guest/anon SMB, log access).  
- **Network segmentation** (mgmt VLAN, VPN access).  

---

## E — Risk Matrix
| Port | Service | Risk | Notes |
|------|----------|------|-------|
| 445  | SMB      | 🔴 High | Wormable exploits, ransomware target |
| 135  | RPC      | 🟠 Med–High | Management but exploitable |
| 139  | NetBIOS  | 🟠 Medium | Legacy, weak security |
| 4343 | Unknown TLS | 🟠 Medium/High | Treat as high until verified |
| 4449 | Unknown TLS | 🟠 Medium/High | Treat as high until verified |

---

## F — Immediate Checklist
- Run targeted Nmap:  
  ```bash
  nmap -sV --script=ssl-cert,banner -p 135,139,445,4343,4449 192.168.56.1 -oN investigate.txt
  ```
- On host: check `netstat` → match PID → identify process.  
- Confirm SMB1 disabled, review shares, patch system.  
- Block 135/139/445 externally.  
- Isolate host if unknown services (4343/4449) are suspicious.  
