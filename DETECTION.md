# 🔎 Detection Playbook — One Anomaly, Many Tools

A blue-team companion to the cheat sheets. Each section takes **one attack pattern** and shows how to spot it with the different tools in this repo (`tcpdump`, `wireshark`, `snort`, `splunk`, `nmap`). Great for SOC rooms on TryHackMe and for understanding *what your own offensive actions look like to a defender*.

> 🧠 **Mindset:** an anomaly is just *normal traffic, but wrong* — wrong volume, wrong timing, wrong direction, wrong port, or wrong content. Every recipe below keys on one of those.

---

## 📇 Quick index

| Anomaly | Signal to key on |
| --- | --- |
| [Port scanning](#1-port-scanning) | One source → many ports/hosts, lots of SYN/RST |
| [Brute-force login](#2-brute-force-login) | Many auth failures from one source, then maybe a success |
| [Reverse shell / C2](#3-reverse-shell--c2-beaconing) | Odd outbound port, regular "heartbeat" timing |
| [Data exfiltration](#4-data-exfiltration) | Large/abnormal **outbound** volume |
| [DNS tunnelling](#5-dns-tunnelling) | Long, frequent, high-entropy DNS queries |
| [ARP spoofing (MITM)](#6-arp-spoofing--mitm) | Two MACs claiming one IP |
| [Cleartext credentials](#7-cleartext-credentials) | `user`/`pass` strings in plaintext protocols |
| [Lateral movement](#8-lateral-movement-windowsad) | Unusual SMB/WinRM/Kerberos, new admin accounts |

---

## 1. Port scanning

**What it is:** an attacker (e.g. [nmap](./nmap)) probing many ports/hosts to map what's open.
**Tell-tale:** one source IP → many destination ports in a short window; a flood of SYN (and answering RST) packets.

**tcpdump**
```bash
# Watch SYNs pile up against different ports from one source
sudo tcpdump -nn 'tcp[tcpflags] == tcp-syn'
# Stealth scans (no/odd flags)
sudo tcpdump -nn 'tcp[tcpflags] == 0'                       # NULL scan
sudo tcpdump -nn 'tcp[tcpflags] == (tcp-fin|tcp-push|tcp-urg)'  # Xmas scan
```

**Wireshark**
```text
tcp.flags.syn==1 && tcp.flags.ack==0     # connection attempts
tcp.flags.reset==1                       # closed ports replying (scan footprint)
```

**Snort**
```text
alert tcp any any -> $HOME_NET any (msg:"Possible SYN port scan"; \
  flags:S; detection_filter:track by_src, count 20, seconds 5; sid:1000010; rev:1;)
```

**Splunk**
```text
index=firewall action=blocked
| stats dc(dest_port) AS ports by src_ip
| where ports > 20 | sort -ports
```

---

## 2. Brute-force login

**What it is:** [hydra](./hydra) / [crackmapexec/netexec](./netexec) hammering a login (SSH, RDP, SMB, HTTP, FTP…).
**Tell-tale:** many authentication *failures* from one source, fast, sometimes followed by a *success* (= breach).

**tcpdump** (volume of new connections to the service)
```bash
sudo tcpdump -nn 'tcp[tcpflags] == tcp-syn and dst port 22'
```

**Snort**
```text
alert tcp any any -> $HOME_NET 22 (msg:"SSH brute-force"; flow:to_server; flags:S; \
  detection_filter:track by_src, count 5, seconds 60; sid:1000012; rev:1;)
```

**Splunk** (the money query — failures *then* a success)
```text
index=auth (action=failure OR action=success)
| stats count(eval(action="failure")) AS fails count(eval(action="success")) AS wins by user, src_ip
| where fails > 10 AND wins > 0
```

Windows equivalent: failed logons = **EventCode 4625**, success = **4624** (see [GLOSSARY.md](./GLOSSARY.md)).

---

## 3. Reverse shell / C2 beaconing

**What it is:** a compromised host phones home (see [shells](./shells), [metasploit](./metasploit)).
**Tell-tale:** *outbound* connection to an unusual high port (4444, 1337, 9001…), and/or callbacks at a suspiciously **regular interval** (beaconing).

**tcpdump**
```bash
sudo tcpdump -nn 'dst port 4444 or dst port 1337 or dst port 9001'
```

**Snort**
```text
alert tcp $HOME_NET any -> any 4444 (msg:"Reverse shell to 4444"; \
  flow:to_server,established; sid:1000014; rev:1;)
```

**Splunk** (beacon = low jitter between callbacks)
```text
index=proxy | sort 0 _time
| streamstats current=f last(_time) AS prev by src_ip, dest_ip
| eval gap = _time - prev
| stats avg(gap) AS avg stdev(gap) AS jitter count by src_ip, dest_ip
| where count > 20 AND jitter < 5
```

**Wireshark:** `Statistics → Conversations` → look for a flow with many small, evenly-spaced packets.

---

## 4. Data exfiltration

**What it is:** stolen data leaving the network.
**Tell-tale:** abnormally large or unusual **outbound** transfer; upload >> download to a single external host.

**tcpdump**
```bash
sudo tcpdump -nn 'greater 1000 and dst net not 10.0.0.0/8'
```

**Wireshark:** `Statistics → Conversations` → sort by *Bytes A→B*; one internal host pushing a lot out is suspicious.

**Splunk**
```text
index=firewall direction=outbound
| stats sum(bytes_out) AS total by src_ip dest_ip
| sort -total | head 10
```

---

## 5. DNS tunnelling

**What it is:** smuggling data inside DNS queries to bypass firewalls (iodine, dnscat2).
**Tell-tale:** very long subdomains, high query volume to one domain, high-entropy / base32-looking labels.

**tcpdump**
```bash
sudo tcpdump -nn -A port 53      # eyeball the query names — long & random?
```

**Wireshark**
```text
dns.qry.name.len > 40
dns && frame.len > 200
```

**Splunk**
```text
index=dns | eval qlen=len(query)
| stats avg(qlen) AS avg max(qlen) AS max count by query_domain
| where max > 50 OR count > 1000 | sort -count
```

---

## 6. ARP spoofing / MITM

**What it is:** attacker poisons ARP caches to intercept LAN traffic.
**Tell-tale:** two different MAC addresses claiming the same IP; a flood of unsolicited ARP replies.

**Wireshark** (built-in detector)
```text
arp.duplicate-address-detected
arp.opcode == 2          # ARP replies — a burst of these is a red flag
```

**tcpdump**
```bash
sudo tcpdump -nn -e arp        # -e shows MACs; watch for one IP, many MACs
```

---

## 7. Cleartext credentials

**What it is:** passwords sent over unencrypted protocols (FTP/21, Telnet/23, HTTP/80).
**Tell-tale:** `USER`/`PASS`, `Authorization:`, or login form fields visible in payload.

**tcpdump**
```bash
sudo tcpdump -nn -A 'port 21 or port 23 or port 80' | grep -iE 'user|pass|login'
```

**Wireshark**
```text
http.request.method == "POST"      # then right-click → Follow → HTTP Stream
ftp.request.command == "PASS"
```

**Snort**
```text
alert tcp any any -> any any (msg:"Cleartext password"; content:"password"; nocase; sid:1000015; rev:1;)
```

---

## 8. Lateral movement (Windows/AD)

**What it is:** moving host-to-host with stolen creds/tickets ([netexec](./netexec), [impacket](./impacket), [evil-winrm](./evil-winrm)).
**Tell-tale:** logons from unusual hosts, Kerberos ticket abuse, new admin accounts, service creation.

**Splunk / Windows Event Logs**
```text
# Account created, then added to a privileged group soon after
index=wineventlog (EventCode=4720 OR EventCode=4732)
| transaction TargetUserName maxspan=10m
| search EventCode=4720 EventCode=4732

# Pass-the-hash style logon (NTLM, network logon type 3)
index=wineventlog EventCode=4624 Logon_Type=3 Authentication_Package=NTLM
| stats count by Account_Name, Source_Network_Address
```

**Wireshark:** `kerberos` (a burst of `AS-REQ` across many usernames = AS-REP roasting / kerbrute), `smb2`.

Key Event IDs (4624/4625/4672/4768/4769/7045) are listed in [GLOSSARY.md](./GLOSSARY.md).

---

## 🎯 Mapping to MITRE ATT&CK

[MITRE ATT&CK](https://attack.mitre.org/) is the industry-standard catalogue of adversary **tactics** (the *why* — the goal) and **techniques** (the *how*). Tagging your detections with technique IDs (e.g. `T1110`) is how SOC teams communicate and how detection rules (Sigma, Splunk ES, etc.) are organised.

| Anomaly above | Tactic | Technique (ID) |
| --- | --- | --- |
| [Port scanning](#1-port-scanning) | Reconnaissance / Discovery | Active Scanning (**T1595**), Network Service Discovery (**T1046**) |
| [Brute-force login](#2-brute-force-login) | Credential Access | Brute Force (**T1110**) — Password Spraying (T1110.003) |
| [Reverse shell / C2](#3-reverse-shell--c2-beaconing) | Command & Control | Application Layer Protocol (**T1071**), Non-Standard Port (**T1571**) |
| [Data exfiltration](#4-data-exfiltration) | Exfiltration | Exfiltration Over C2 Channel (**T1041**), Over Alternative Protocol (**T1048**) |
| [DNS tunnelling](#5-dns-tunnelling) | Command & Control | DNS (**T1071.004**), Protocol Tunneling (**T1572**) |
| [ARP spoofing / MITM](#6-arp-spoofing--mitm) | Credential Access / Collection | Adversary-in-the-Middle (**T1557**) |
| [Cleartext credentials](#7-cleartext-credentials) | Credential Access | Network Sniffing (**T1040**), Unsecured Credentials (**T1552**) |
| [Lateral movement](#8-lateral-movement-windowsad) | Lateral Movement / Cred Access | Pass-the-Hash (**T1550.002**), Kerberoasting (**T1558.003**), Remote Services (**T1021**) |

**The 14 ATT&CK tactics (attacker's goals, in rough order):**

```text
Recon → Resource Dev → Initial Access → Execution → Persistence →
Privilege Escalation → Defense Evasion → Credential Access → Discovery →
Lateral Movement → Collection → Command & Control → Exfiltration → Impact
```

> 💡 In a SOC room, the question "which ATT&CK technique is this?" comes up constantly. Map the *behaviour* (a burst of 4625s = T1110) rather than the tool — the same technique shows up regardless of whether the attacker used [hydra](./hydra), [netexec](./netexec), or [kerbrute](./kerbrute).

---

## 🧭 General hunting method

1. **Baseline first.** You can't spot "abnormal" without knowing "normal" (use [`nmap` + `ndiff`](./nmap#-spotting-anomalies-in-results) for the network, lookups/known-good lists for hosts).
2. **Pick a pivot:** source IP, user, port, or domain.
3. **Aggregate, don't eyeball:** `stats count by …`, `top`, `rare`, Wireshark *Conversations*.
4. **Look for the wrong-ness:** wrong volume / timing / direction / port / content.
5. **Confirm with packets:** drop to [tcpdump](./tcpdump)/[Wireshark](./wireshark) and *follow the stream*.

> ⚠️ Tune thresholds to your environment — every number above (counts, seconds, bytes) is a starting point, not gospel.
