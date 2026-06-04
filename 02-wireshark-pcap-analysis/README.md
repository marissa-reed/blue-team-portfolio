 Lab 02 — Wireshark PCAP Analysis: NetSupport RAT C2 Detection

**Analyst:** Marissa Reed | Active Secret Clearance | CompTIA Security+  
**Date:** June 2026  
**Tool:** Wireshark 4.x (Windows)  
**PCAP Source:** malware-traffic-analysis.net (training exercise)

---

## Overview

Analyzed a PCAP file from a real malware infection using Wireshark.
Identified active Command and Control (C2) beaconing traffic from a
NetSupport RAT — a Remote Access Trojan that gives attackers full
remote control of an infected machine. Confirmed malicious activity
using VirusTotal threat intelligence.

---

## Summary of Findings

| Field | Detail |
|-------|--------|
| **Malware Family** | NetSupport RAT |
| **C2 Server IP** | 45.131.214.85 |
| **C2 URL** | http://45.131.214.85/fakeurl.htm |
| **Protocol** | HTTP (unencrypted) |
| **User Agent** | NetSupport Manager/1.3 |
| **Activity** | C2 beaconing — CMD=POLL, CMD=ENCD |
| **VirusTotal Verdict** | 15/94 vendors — Malicious/Malware |
| **Threat Level** | High |

---

## Indicators of Compromise (IOCs)

| Type | Value | Verdict |
|------|-------|---------|
| IP Address | 45.131.214.85 | Malicious — confirmed C2 server |
| URL | http://45.131.214.85/fakeurl.htm | Malicious — known NetSupport RAT endpoint |
| User Agent | NetSupport Manager/1.3 | Malicious — RAT beacon identifier |
| Command | CMD=POLL | C2 check-in — attacker polling for commands |
| Command | CMD=ENCD | Encrypted C2 communication — active session |

---

## Technical Analysis

### Step 1 — Initial Discovery
Applied the `http` display filter in Wireshark and immediately identified
suspicious outbound HTTP traffic to **45.131.214.85**. The destination IP
stood out because legitimate traffic does not POST to a raw IP address
on port 80 — domains are used instead.

### Step 2 — TCP Stream Analysis
Right-clicked the suspicious packet → Follow → TCP Stream. The raw
conversation revealed:

```
POST http://45.131.214.85/fakeurl.htm HTTP/1.1
User-Agent: NetSupport Manager/1.3
Content-Type: application/x-www-form-urlencoded
Host: 45.131.214.85
Connection: Keep-Alive

CMD=POLL
INFO=1
ACK=1
```

**What this means line by line:**

| Line | Meaning |
|------|---------|
| `POST /fakeurl.htm` | Infected machine calling home to C2 server — `/fakeurl.htm` is a known hardcoded NetSupport RAT endpoint |
| `User-Agent: NetSupport Manager/1.3` | Malware identifying itself — NetSupport is a legitimate RMM tool weaponized as a RAT |
| `Connection: Keep-Alive` | Persistent connection maintained — attacker has ongoing access |
| `CMD=POLL` | Infected machine checking in: "any commands for me?" — classic C2 beacon |
| `CMD=ENCD` | Attacker responding with encrypted commands |
| `DATA=u.2h.r..4...` | Encrypted C2 traffic — attacker actively sending instructions |

### Step 3 — Threat Intelligence Confirmation
Submitted **45.131.214.85** to VirusTotal. Results confirmed malicious by
15 independent security vendors:

| Vendor | Verdict |
|--------|---------|
| BitDefender | Malware |
| ESET | Suspicious |
| Fortinet | Malware |
| G-Data | Malware |
| AlphaSOC | Malware |
| SOCRadar | Malware |
| Dr.Web | Malicious |
| VIPRE | Malware |
| + 7 more | Malicious/Suspicious |

15/94 vendors flagging as malicious is a high-confidence verdict —
this is definitively a malicious C2 server, not a false positive.

---

## Attack Narrative

Based on the PCAP analysis, the following attack sequence occurred:

1. **Initial infection** — a host on the network was compromised and
   NetSupport RAT was installed (delivery method not visible in this PCAP)

2. **C2 beaconing begins** — the infected machine starts sending regular
   HTTP POST requests to 45.131.214.85/fakeurl.htm with CMD=POLL,
   checking in with the attacker's C2 server

3. **Attacker responds** — the C2 server replies with CMD=ENCD containing
   encrypted commands — the attacker has established an active session
   and is sending instructions to the infected machine

4. **Persistent access** — Connection: Keep-Alive ensures the C2 channel
   stays open, giving the attacker continuous remote control

---

## Wireshark Filters Used

| Filter | Purpose | Result |
|--------|---------|--------|
| `http` | Identify all HTTP traffic | Found suspicious POSTs to 45.131.214.85 |
| `ip.addr == 45.131.214.85` | Isolate all traffic to/from C2 | Confirmed repeated beaconing pattern |
| `http.request.method == "POST"` | Find outbound data | Revealed CMD=POLL and CMD=ENCD commands |
| `dns` | Check for domain lookups | No DNS — attacker used raw IP to avoid DNS logging |

**Notable observation:** The attacker used a raw IP address instead of
a domain name. This is a deliberate technique to avoid DNS-based
detection — no domain lookup means no DNS log entry, making the
traffic harder to detect with DNS filtering tools.

---

## Remediation Recommendations

If this were a real incident, the recommended response would be:

1. **Immediate isolation** — disconnect the infected host from the
   network to stop C2 communication
2. **Block the C2 IP** — add 45.131.214.85 to firewall blocklist
   immediately across all perimeter devices
3. **Hunt for lateral movement** — search for other hosts communicating
   with 45.131.214.85 in firewall and proxy logs
4. **Full disk forensics** — image the infected host before remediation
   to preserve evidence
5. **Malware removal** — identify and remove NetSupport RAT persistence
   mechanisms (registry keys, scheduled tasks, startup entries)
6. **Root cause analysis** — determine how NetSupport RAT was initially
   delivered (phishing email, drive-by download, RDP brute force)

---

## What I Learned

- How to identify C2 beaconing by looking for repeated HTTP POSTs
  to a raw IP address
- How to use Follow → TCP Stream to read raw attacker communication
- What NetSupport RAT C2 traffic looks like (CMD=POLL, CMD=ENCD pattern)
- Why attackers use raw IPs instead of domains to evade DNS detection
- How to use VirusTotal to confirm malicious IPs with threat intel
- How to build an attack narrative from raw packet data
- The difference between encrypted and unencrypted C2 channels

---

## Tools Used

| Tool | Purpose | Cost |
|------|---------|------|
| Wireshark 4.x | PCAP analysis and packet inspection | Free |
| VirusTotal | Threat intelligence — IP reputation lookup | Free |
| malware-traffic-analysis.net | Real-world PCAP training samples | Free |

---

## References
- [VirusTotal Report — 45.131.214.85](https://www.virustotal.com/gui/ip-address/45.131.214.85)
- [NetSupport RAT — MITRE ATT&CK T1219](https://attack.mitre.org/techniques/T1219/)
- [CISA Alert — NetSupport RAT](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)

---

*Part of an ongoing blue team cybersecurity skills portfolio.*  
*Previous lab: [Lab 01 — Nessus Vulnerability Scan](../01-nessus-vuln-scan/README.md)*
