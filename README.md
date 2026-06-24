# 🛡️ Cybersecurity & Pentesting Cheat Sheets

A collection of clean, practical cheat sheets for the security tools you actually use — built while working through TryHackMe rooms and home labs. Each tool lives in its own folder with a dedicated `README.md` so you can jump straight to what you need.

Every command comes with an **example** and a **plain-English description** of what it does, so this works as a quick reference whether you're mid-engagement or just learning.

> ⚠️ **For authorized testing and learning only.** Use these tools exclusively on systems you own or have explicit written permission to test. Unauthorized access to computer systems is illegal.

---

## 📂 Contents

### 🔍 Recon & Scanning
| Tool | What it's for |
| --- | --- |
| [**nmap**](./nmap) | Network mapping, port scanning, service & OS detection |
| [**dns**](./dns) | DNS recon — dig, zone transfers, subdomain enumeration |
| [**seclists**](./seclists) | Wordlists for brute-forcing & fuzzing (the fuel for the tools below) |

### 📡 Network & Packet Analysis
| Tool | What it's for |
| --- | --- |
| [**tcpdump**](./tcpdump) | CLI packet capture and filtering |
| [**wireshark**](./wireshark) | GUI packet analysis & display filters |
| [**netcat**](./netcat) | The "Swiss army knife" — connections, listeners, file transfer, shells |

### 🚨 IDS / IPS
| Tool | What it's for |
| --- | --- |
| [**snort**](./snort) | Intrusion detection: sniffing, logging, rule-based alerting |

### 🌐 Web Hacking
| Tool | What it's for |
| --- | --- |
| [**gobuster**](./gobuster) | Directory, DNS & vhost brute-forcing |
| [**ffuf**](./ffuf) | Fast web fuzzer |
| [**burpsuite**](./burpsuite) | Intercepting proxy — Repeater, Intruder, tampering |
| [**nikto**](./nikto) | Web server vulnerability scanner |
| [**wpscan**](./wpscan) | WordPress enumeration & brute-forcing |
| [**sqlmap**](./sqlmap) | Automated SQL injection & database takeover |

### 🔑 Password Attacks
| Tool | What it's for |
| --- | --- |
| [**hydra**](./hydra) | Online brute-forcing of network logins |
| [**john**](./john) | John the Ripper — offline hash cracking |
| [**hashcat**](./hashcat) | GPU-accelerated hash cracking |

### 🗂️ Enumeration & Exploitation
| Tool | What it's for |
| --- | --- |
| [**smb**](./smb) | SMB/Windows enumeration (enum4linux, smbclient, rpcclient) |
| [**metasploit**](./metasploit) | Exploitation framework + msfvenom + searchsploit |
| [**shells**](./shells) | Reverse **and** bind shells (netcat, bash, socat) + shell stabilisation |

### 🪟 Active Directory / Windows
| Tool | What it's for |
| --- | --- |
| [**netexec**](./netexec) | NetExec / CrackMapExec — AD & SMB swiss-army knife |
| [**impacket**](./impacket) | Kerberoasting, secretsdump, psexec/wmiexec & more |
| [**bloodhound**](./bloodhound) | Graph AD attack paths to Domain Admin |
| [**kerbrute**](./kerbrute) | Kerberos username enumeration & password spraying |
| [**evil-winrm**](./evil-winrm) | Interactive WinRM shell (incl. Pass-the-Hash) |

### ⬆️ Privilege Escalation & Pivoting
| Tool | What it's for |
| --- | --- |
| [**privesc**](./privesc) | LinPEAS/WinPEAS, manual checks, GTFOBins & LOLBAS |
| [**pivoting**](./pivoting) | SSH tunnels, chisel, proxychains |

### 🧩 Crypto / Forensics
| Tool | What it's for |
| --- | --- |
| [**crypto**](./crypto) | Encoding, hashing, CyberChef, steganography & file forensics |
| [**volatility**](./volatility) | Memory forensics — analyse RAM dumps |

### 🔵 Blue Team / SOC
| Tool | What it's for |
| --- | --- |
| [**splunk**](./splunk) | SIEM searching with SPL |
| [**sysmon**](./sysmon) | Sysmon & Windows Event Log threat hunting |

### 📑 Reference pages
| Page | What it's for |
| --- | --- |
| [**DETECTION.md**](./DETECTION.md) | 🔎 Detection playbook — one anomaly, many tools + MITRE ATT&CK mapping |
| [**RESOURCES.md**](./RESOURCES.md) | 🌍 Online tools — CyberChef, VirusTotal, Shodan, CrackStation & more |
| [**PORTS.md**](./PORTS.md) | Common ports → service → which tool to use next |
| [**GLOSSARY.md**](./GLOSSARY.md) | Acronyms, TCP flags, Event IDs, hashcat modes, HTTP codes |

---

## 🧭 Methodology — how a box flows

Most engagements (and TryHackMe rooms) follow the same loop. Use this as a map and click through to the tool you need at each step.

```text
            ┌─────────────────────────────────────────────────────────┐
            │  1. RECON      2. ENUM       3. EXPLOIT                  │
            │  what's there  poke services  get a foothold             │
            └─────────────────────────────────────────────────────────┘
                                                       │
            ┌──────────────────────────────────────────▼──────────────┐
            │  4. PRIVESC    5. LOOT        6. PIVOT / LATERAL         │
            │  become root   grab creds     reach the next host        │
            └─────────────────────────────────────────────────────────┘
```

| Step | Goal | Reach for |
| --- | --- | --- |
| **1. Recon** | Find hosts, ports, services | [nmap](./nmap), [dns](./dns) → cross-ref [PORTS.md](./PORTS.md) |
| **2. Enumeration** | Dig into each open service | [smb](./smb), [gobuster](./gobuster)/[ffuf](./ffuf), [nikto](./nikto), [wpscan](./wpscan), [burpsuite](./burpsuite) |
| **3. Exploitation** | Get initial access | [metasploit](./metasploit), [sqlmap](./sqlmap), [hydra](./hydra), [shells](./shells) |
| **4. Privilege escalation** | Low-priv → root/SYSTEM | [privesc](./privesc) |
| **5. Loot & crack** | Harvest creds/hashes | [john](./john), [hashcat](./hashcat), [crypto](./crypto) |
| **6. Pivot / lateral** | Reach internal hosts | [pivoting](./pivoting), [netexec](./netexec), [impacket](./impacket), [bloodhound](./bloodhound), [kerbrute](./kerbrute), [evil-winrm](./evil-winrm) |
| **🔵 Defend** | Detect & investigate the above | [snort](./snort), [splunk](./splunk), [sysmon](./sysmon), [volatility](./volatility), [DETECTION.md](./DETECTION.md) |

> 🌍 Plenty of steps have a no-install web equivalent — see [RESOURCES.md](./RESOURCES.md) (CyberChef, VirusTotal, Shodan, CrackStation…).

---

## 🚀 How to use

Click into any folder for that tool's cheat sheet. Each follows the same layout:

- Commands grouped into logical sections (e.g. *Host Discovery*, *Output*)
- A `Flag / Command` → `Example` → `Description` table for fast scanning
- Code blocks for multi-step workflows and one-liners with pipes

---

## 🤝 Contributing

Found a mistake or have a command to add? Open an issue or PR. Keep entries consistent with the existing table format.

## 📄 License

[MIT](./LICENSE) — free to use, share, and adapt.
