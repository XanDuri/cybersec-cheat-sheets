# 📖 Glossary & Quick Reference

Terms, acronyms, and lookup tables that come up across the cheat sheets — handy when a TryHackMe room throws a new abbreviation at you.

---

## Acronyms

| Term | Meaning |
| --- | --- |
| **AD** | Active Directory — Microsoft's directory/identity service |
| **C2 / C&C** | Command & Control — attacker's server a compromised host calls back to |
| **CVE** | Common Vulnerabilities and Exposures — a public vuln ID (e.g. CVE-2021-44228) |
| **IDS / IPS** | Intrusion Detection / Prevention System (e.g. [snort](./snort)) |
| **IOC** | Indicator of Compromise — evidence of an attack (bad IP, hash, filename) |
| **LFI / RFI** | Local / Remote File Inclusion (web) |
| **LOLBAS / GTFOBins** | Living-off-the-land binaries (Win / Unix) abused for privesc — see [privesc](./privesc) |
| **MITM** | Man-in-the-Middle |
| **NSE** | Nmap Scripting Engine |
| **PoC** | Proof of Concept (exploit) |
| **PtH / PtT** | Pass-the-Hash / Pass-the-Ticket |
| **RCE** | Remote Code Execution |
| **SIEM** | Security Information & Event Management (e.g. [splunk](./splunk)) |
| **SOC** | Security Operations Centre (the blue team) |
| **SPN** | Service Principal Name — target of Kerberoasting |
| **SUID / SGID** | Set-User/Group-ID bit on Unix files (privesc vector) |
| **TTP** | Tactics, Techniques & Procedures |
| **XSS / SSRF / IDOR** | Web vulns: Cross-Site Scripting / Server-Side Request Forgery / Insecure Direct Object Reference |

---

## TCP flags (for scan detection)

| Flag | Letter | Meaning |
| --- | --- | --- |
| SYN | S | Start a connection |
| ACK | A | Acknowledge |
| FIN | F | Graceful close |
| RST | R | Abrupt reset (port closed/refused) |
| PSH | P | Push buffered data |
| URG | U | Urgent data |

A normal handshake = **SYN → SYN/ACK → ACK**. Lots of lone SYNs or odd combos = a scan. See [DETECTION.md](./DETECTION.md#1-port-scanning).

---

## Windows logon types (Event 4624/4625)

| Type | Meaning |
| --- | --- |
| 2 | Interactive (at the keyboard) |
| 3 | **Network** (SMB, used in PtH) |
| 4 | Batch (scheduled task) |
| 5 | Service |
| 10 | **RemoteInteractive** (RDP) |

---

## Key Windows Event IDs

| ID | Meaning |
| --- | --- |
| 4624 / 4625 | Logon success / failure |
| 4672 | Admin privileges assigned |
| 4720 | Account created |
| 4732 | Added to a group |
| 4768 / 4769 | Kerberos TGT / service ticket |
| 7045 | Service installed |
| Sysmon 1 / 3 | Process creation / network connection |

(Full Sysmon list → [sysmon](./sysmon).)

---

## Hashcat mode quick-reference

| Hash | `-m` mode |
| --- | --- |
| MD5 | 0 |
| SHA-256 | 1400 |
| NTLM | 1000 |
| NetNTLMv2 | 5600 |
| Kerberos AS-REP | 18200 |
| Kerberos TGS (roast) | 13100 |
| bcrypt | 3200 |
| sha512crypt (`$6$`) | 1800 |

(More in [hashcat](./hashcat).)

---

## HTTP status codes (web enum)

| Code | Meaning | In fuzzing… |
| --- | --- | --- |
| 200 | OK | Found it |
| 301/302 | Redirect | Often a real dir (trailing slash) |
| 401 | Unauthorized | Auth required — worth attacking |
| 403 | Forbidden | Exists but blocked — try bypasses |
| 404 | Not Found | Filter these out (`-fc 404`) |
| 500 | Server Error | You broke something (good for injection) |
