# Blue Team Cybersecurity Portfolio

**Marissa Reed** | Active Secret Clearance | CompTIA Security+  
Washington, DC Metro | marissa.reed112@gmail.com

---

## Labs Completed

| # | Lab | Tools Used | Status |
|---|-----|-----------|--------|
| 01 | Nessus Vulnerability Scan — SMB Signing | Nessus Essentials, Windows | ✅ Complete |
| 02 | Wireshark PCAP Analysis | Wireshark | 🔄 Coming Soon |
| 03 | Windows Event Log Analysis | Event Viewer | 🔄 Coming Soon |
| 04 | LetsDefend Incident Response | LetsDefend.io | 🔄 Coming Soon |

---

# Lab 01 — Nessus Essentials Vulnerability Scan

## Environment
- **Scanner:** Nessus Essentials (Windows)
- **Target:** Local network host
- **Scan type:** Basic Network Scan
- **Date:** June 2026
- **Analyst:** Marissa Reed

---

## Findings Summary

| Severity | Count |
|----------|-------|
| 🔴 Critical | 0 |
| 🟠 High | 0 |
| 🟡 Medium | 1 |
| 🟢 Low / Info | 24 |

---

## Finding 01 — SMB Signing Not Required

| Field | Detail |
|-------|--------|
| **Severity** | Medium |
| **CVSS v3.0 Base Score** | 5.3 |
| **CVSS v3.0 Vector** | AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:N |
| **CVSS v3.0 Temporal Score** | 4.6 |
| **CVSS v2.0 Base Score** | 5.0 |
| **CVSS v2.0 Temporal Score** | 3.7 |

### What it means
SMB (Server Message Block) is the protocol Windows uses to share files, printers, and other resources across a network. When SMB signing is not required, the server accepts unsigned SMB traffic. This allows an unauthenticated remote attacker to perform a **man-in-the-middle (MITM) attack** — intercepting and potentially modifying SMB communication between clients and the server without either party knowing.

### Attack scenario
An attacker on the same network intercepts SMB traffic between a workstation and file server. Without signing, the attacker can relay credentials, impersonate the server, or inject malicious data into the communication — all without needing a password.

### CVSS score breakdown
| Metric | Value | Meaning |
|--------|-------|---------|
| Attack Vector (AV) | Network | Exploitable remotely over the network |
| Attack Complexity (AC) | Low | No special conditions required |
| Privileges Required (PR) | None | No authentication needed |
| User Interaction (UI) | None | No user action required |
| Confidentiality (C) | None | No data disclosure |
| Integrity (I) | Low | Limited data modification possible |
| Availability (A) | None | No impact on availability |

### Remediation
**Windows fix:**
1. Open Group Policy Editor (`gpedit.msc`)
2. Navigate to: `Computer Configuration → Windows Settings → Security Settings → Local Policies → Security Options`
3. Find **"Microsoft network server: Digitally sign communications (always)"**
4. Set it to **Enabled**
5. Apply and restart the server

**Samba fix:**
Add the following to your `smb.conf` file:
```
[global]
server signing = mandatory
```
Then restart the Samba service.

### Verification
After applying the fix, re-run the Nessus scan against the same target. The "SMB Signing Not Required" finding should no longer appear, confirming the remediation was successful.

---

## What I Learned

- How to install and configure Nessus Essentials on Windows
- How to run a Basic Network Scan and interpret the results
- What CVSS v3.0 scores mean and how to break down each metric
- What SMB signing is and why it matters for network security
- How to remediate a real vulnerability using Group Policy on Windows
- How man-in-the-middle attacks work at a protocol level

---

## Tools Used

| Tool | Purpose | Cost |
|------|---------|------|
| Nessus Essentials | Vulnerability scanning | Free |
| VirtualBox | Virtual machine hosting | Free |
| Windows | Scan host | — |
| NIST NVD (nvd.nist.gov) | CVE research and context | Free |

---

## References
- [Nessus Plugin — SMB Signing Disabled](https://www.tenable.com/plugins/nessus/57608)
- [Microsoft Docs — SMB Signing](https://docs.microsoft.com/en-us/windows-server/storage/file-server/smb-signing-overview)
- [NIST NVD — CVSS v3.0 Scoring](https://nvd.nist.gov/vuln-metrics/cvss)
- [CISA — Known Exploited Vulnerabilities](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

---

*This lab is part of an ongoing cybersecurity skills development portfolio. New labs added regularly.*
