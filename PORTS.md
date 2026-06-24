# 🔌 Common Ports & Services

A quick reference for the ports you'll see most on TryHackMe / HTB, what runs on them, and which cheat sheet to reach for next. Seeing a port open is a hint — it tells you *which tool to pull out*.

> 💡 Don't trust the default — always confirm the real service with `nmap -sV`. SSH on 2222, HTTP on 8080, etc. are common.

---

## Top targets (memorise these)

| Port | Proto | Service | Go enumerate with |
| --- | --- | --- | --- |
| **21** | TCP | FTP | `ftp`, anon login, [nmap](./nmap) `ftp-anon` |
| **22** | TCP | SSH | [hydra](./hydra) (brute), key reuse, [pivoting](./pivoting) |
| **23** | TCP | Telnet | cleartext — sniff with [wireshark](./wireshark) |
| **25** | TCP | SMTP | user enum (`VRFY`), `smtp-user-enum` |
| **53** | TCP/UDP | DNS | [dns](./dns) — zone transfer, subdomain brute |
| **80** | TCP | HTTP | [gobuster](./gobuster)/[ffuf](./ffuf), [nikto](./nikto), [burpsuite](./burpsuite) |
| **88** | TCP | Kerberos | [impacket](./impacket) — AS-REP/Kerberoast |
| **110/143** | TCP | POP3 / IMAP | mail creds |
| **111** | TCP | RPCbind | NFS enum (`showmount -e`) |
| **135/139/445** | TCP | MSRPC / NetBIOS / **SMB** | [smb](./smb), [netexec](./netexec) |
| **161** | UDP | SNMP | `snmpwalk`, community strings |
| **389/636** | TCP | LDAP / LDAPS | [netexec](./netexec) `--bloodhound`, `ldapsearch` |
| **443** | TCP | HTTPS | as 80 + cert info (`openssl s_client`) |
| **1433** | TCP | MSSQL | [impacket](./impacket) `mssqlclient.py` |
| **2049** | TCP | NFS | mount shares |
| **3306** | TCP | MySQL | creds, [sqlmap](./sqlmap) |
| **3389** | TCP | RDP | [hydra](./hydra), `xfreerdp` |
| **5432** | TCP | PostgreSQL | creds |
| **5985/5986** | TCP | WinRM | [evil-winrm](./evil-winrm) |
| **8080** | TCP | HTTP-alt / proxy | web app, Tomcat, [burpsuite](./burpsuite) |

---

## "What do I do with this port?" mapping

| You see… | Reach for |
| --- | --- |
| 80 / 443 / 8080 | Content discovery → [gobuster](./gobuster) / [ffuf](./ffuf); vuln scan → [nikto](./nikto); WordPress → [wpscan](./wpscan); manual → [burpsuite](./burpsuite) |
| 445 / 139 / 135 | [smb](./smb) enum, then [netexec](./netexec) for creds/exec |
| 53 | [dns](./dns) — try AXFR first |
| 88 / 389 (Domain) | AD attacks → [impacket](./impacket), [netexec](./netexec) |
| 5985 / 5986 | Shell via [evil-winrm](./evil-winrm) |
| 22 | [hydra](./hydra) brute or [pivoting](./pivoting) tunnel |
| 3306 / 1433 / 5432 | Database — creds, then [sqlmap](./sqlmap) |

---

## Reverse-shell / C2 ports to recognise (blue team)

Outbound traffic to these from an *internal* host is suspicious — see [DETECTION.md](./DETECTION.md#3-reverse-shell--c2-beaconing).

| Port | Often |
| --- | --- |
| **4444** | Metasploit default (`LPORT`) |
| **1337 / 31337** | "leet" — manual reverse shells |
| **9001 / 9002** | Common nc/Empire listeners |
| **8443 / 53 / 443** | Blending C2 into "normal" ports |
