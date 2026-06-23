# 🛡️ Cybersecurity & Pentesting Cheat Sheets
![License](https://img.shields.io/github/license/XanDuri/cybersec-cheat-sheets)
![Last commit](https://img.shields.io/github/last-commit/XanDuri/cybersec-cheat-sheets)
![Stars](https://img.shields.io/github/stars/XanDuri/cybersec-cheat-sheets?style=social)

A collection of clean, practical cheat sheets for the security tools you actually use — built while working through TryHackMe rooms and home labs. Each tool lives in its own folder with a dedicated `README.md` so you can jump straight to what you need.

Every command comes with an **example** and a **plain-English description** of what it does, so this works as a quick reference whether you're mid-engagement or just learning.

> ⚠️ **For authorized testing and learning only.** Use these tools exclusively on systems you own or have explicit written permission to test. Unauthorized access to computer systems is illegal.

---

## 📂 Contents

### 🔍 Recon & Scanning
| Tool | What it's for |
| --- | --- |
| [**nmap**](./nmap) | Network mapping, port scanning, service & OS detection |
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

### 🔵 Blue Team / SOC
| Tool | What it's for |
| --- | --- |
| [**splunk**](./splunk) | SIEM searching with SPL |

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
