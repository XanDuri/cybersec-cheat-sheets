# 🔍 Nmap Cheat Sheet

**Nmap** (Network Mapper) is the go-to tool for host discovery, port scanning, and service/OS detection. It's usually the first thing you run against a target.

```bash
# General syntax
nmap [scan type] [options] [target]
```

> 💡 Many scan types (`-sS`, `-O`, `-sU`) require **root** privileges. Prefix with `sudo`.

---

## Target Specification

| Flag | Example | Description |
| --- | --- | --- |
| | `nmap 10.10.10.1` | Scan a single IP |
| | `nmap 10.10.10.1 10.10.10.2` | Scan multiple IPs |
| | `nmap 10.10.10.1-50` | Scan a range |
| | `nmap scanme.nmap.org` | Scan a domain |
| | `nmap 10.10.10.0/24` | Scan a whole subnet (CIDR) |
| `-iL` | `nmap -iL targets.txt` | Read targets from a file |
| `--exclude` | `nmap 10.10.10.0/24 --exclude 10.10.10.1` | Skip listed hosts |

## Host Discovery

| Flag | Example | Description |
| --- | --- | --- |
| `-sn` | `nmap -sn 10.10.10.0/24` | Ping sweep — find live hosts, no port scan |
| `-Pn` | `nmap -Pn 10.10.10.1` | Skip discovery — treat host as up (use when it "blocks ping") |
| `-sL` | `nmap -sL 10.10.10.0/24` | List targets only, don't scan |
| `-PR` | `nmap -PR 10.10.10.0/24` | ARP discovery on the local network |
| `-n` | `nmap -n 10.10.10.1` | Never do DNS resolution (faster) |

## Scan Techniques

| Flag | Example | Description |
| --- | --- | --- |
| `-sS` | `sudo nmap -sS 10.10.10.1` | TCP SYN "stealth" scan (default w/ root) |
| `-sT` | `nmap -sT 10.10.10.1` | TCP connect scan (default without root) |
| `-sU` | `sudo nmap -sU 10.10.10.1` | UDP scan (slow but finds UDP services) |
| `-sA` | `sudo nmap -sA 10.10.10.1` | TCP ACK scan — map firewall rules |

## Port Specification

| Flag | Example | Description |
| --- | --- | --- |
| `-p` | `nmap -p 80 10.10.10.1` | Scan a single port |
| `-p` | `nmap -p 1-100 10.10.10.1` | Scan a port range |
| `-p` | `nmap -p 22,80,443 10.10.10.1` | Scan specific ports |
| `-p-` | `nmap -p- 10.10.10.1` | Scan **all** 65535 ports |
| `-F` | `nmap -F 10.10.10.1` | Fast scan (top 100 ports) |
| `--top-ports` | `nmap --top-ports 1000 10.10.10.1` | Scan the top N most common ports |

## Service & Version / OS Detection

| Flag | Example | Description |
| --- | --- | --- |
| `-sV` | `nmap -sV 10.10.10.1` | Detect service versions |
| `-sV --version-intensity` | `nmap -sV --version-intensity 9 10.10.10.1` | Set accuracy (0–9, higher = more probes) |
| `-O` | `sudo nmap -O 10.10.10.1` | OS detection via TCP/IP fingerprinting |
| `-A` | `nmap -A 10.10.10.1` | Aggressive: `-sV` + `-O` + `-sC` + traceroute |

## NSE Scripts

| Flag | Example | Description |
| --- | --- | --- |
| `-sC` | `nmap -sC 10.10.10.1` | Run the default set of safe scripts |
| `--script` | `nmap --script http-title 10.10.10.1` | Run a single named script |
| `--script` | `nmap --script "http*" 10.10.10.1` | Run scripts matching a wildcard |
| `--script vuln` | `nmap --script vuln 10.10.10.1` | Check for known vulnerabilities |
| `--script-args` | `nmap --script smb-enum-shares --script-args smbusername=guest 10.10.10.1` | Pass arguments to a script |

**Handy NSE one-liners:**

| Command | Description |
| --- | --- |
| `nmap --script smb-enum-shares,smb-os-discovery 10.10.10.1` | Enumerate SMB shares & OS |
| `nmap --script dns-brute example.com` | Brute-force subdomains via DNS |
| `nmap -p 80 --script http-enum 10.10.10.1` | Enumerate common web paths |
| `nmap -p 21 --script ftp-anon 10.10.10.1` | Check for anonymous FTP login |

## Timing & Performance

| Flag | Example | Description |
| --- | --- | --- |
| `-T0` … `-T5` | `nmap -T4 10.10.10.1` | Speed: 0 = paranoid (IDS evasion), 3 = default, 5 = insane |
| `--min-rate` | `nmap --min-rate 1000 10.10.10.1` | Send at least N packets/sec |
| `--max-retries` | `nmap --max-retries 2 10.10.10.1` | Limit probe retransmissions |

## Firewall / IDS Evasion

| Flag | Example | Description |
| --- | --- | --- |
| `-f` | `nmap -f 10.10.10.1` | Fragment packets to slip past filters |
| `-D` | `nmap -D RND:10 10.10.10.1` | Use decoy IPs to hide your source |
| `-g` | `nmap -g 53 10.10.10.1` | Spoof a source port (e.g. 53/DNS) |
| `--data-length` | `nmap --data-length 200 10.10.10.1` | Append random data to packets |

## Output

| Flag | Example | Description |
| --- | --- | --- |
| `-oN` | `nmap -oN scan.txt 10.10.10.1` | Normal (human-readable) output to file |
| `-oX` | `nmap -oX scan.xml 10.10.10.1` | XML output |
| `-oG` | `nmap -oG scan.grep 10.10.10.1` | Grepable output |
| `-oA` | `nmap -oA scan 10.10.10.1` | Save in all three formats at once |
| `-v` / `-vv` | `nmap -vv 10.10.10.1` | Increase verbosity |
| `--open` | `nmap --open 10.10.10.1` | Show only open ports |
| `--reason` | `nmap --reason 10.10.10.1` | Explain why each port is in its state |

---

## 🔎 Spotting anomalies in results

Nmap isn't only offensive — blue teams use it to baseline a network and catch drift (rogue hosts, surprise open ports, downgraded services).

| Goal | Command | What it tells you |
| --- | --- | --- |
| Baseline a network | `nmap -sV -oX baseline.xml 10.10.10.0/24` | Snapshot of every host/port/version to compare against later |
| **Diff two scans** | `ndiff baseline.xml today.xml` | New hosts, newly-open ports, version changes since the baseline — a classic IOC of compromise |
| Find odd open ports | `nmap -p- --open 10.10.10.1` | A workstation exposing `4444`, `1337`, `8080`… often = backdoor / reverse-shell listener |
| Known vulns | `nmap --script vuln 10.10.10.1` | Surfaces CVEs an attacker could already be using |
| Spot weak/outdated services | `nmap -sV 10.10.10.1` | Old SSH/SMB/Apache versions = likely entry points |

```bash
# Detect drift: re-scan daily and diff against the baseline
nmap -sV -oX today.xml 10.10.10.0/24
ndiff baseline.xml today.xml      # red lines = something changed
```

> 🔗 To see what *your* scan looks like to a defender (and how it's detected), see [DETECTION.md → Port scanning](../DETECTION.md).

---

## 🧭 Typical workflow

```bash
# 1. Find live hosts
sudo nmap -sn 10.10.10.0/24

# 2. Quick full-port sweep on a target
nmap -p- --min-rate 1000 -T4 10.10.10.1 -oN ports.txt

# 3. Deep scan on the ports you found
nmap -sV -sC -p 22,80,443 10.10.10.1 -oN deep.txt
```

> 🔗 **Next steps:** found `445`? → [smb](../smb). Found `80/443`? → [gobuster](../gobuster) / [ffuf](../ffuf). Need the right wordlist? → [seclists](../seclists).
