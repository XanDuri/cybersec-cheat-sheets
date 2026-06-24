# 🌍 Online Tools & Resources

Browser-based tools that pair with the CLI cheat sheets — no install needed. Bookmark these; they cover encoding, hash cracking, malware/file analysis, OSINT, email/phishing triage, and vuln research.

> ⚠️ **OPSEC warning:** anything you paste/upload to a public site (hashes, files, URLs, samples) **leaves your machine** and may be shared with vendors. Never submit client data, real malware you must keep confidential, or live engagement artefacts to public services. Use offline tools ([john](./john), [hashcat](./hashcat), [crypto](./crypto)) when in doubt.

---

## 🔐 Encoding, decoding & deobfuscation
| Tool | Use |
| --- | --- |
| [CyberChef](https://gchq.github.io/CyberChef/) | The "cyber swiss-army knife" — chain Base64/hex/XOR/gunzip etc. Try the **Magic** op (see [crypto](./crypto)) |
| [JavaScript Obfuscator](https://obfuscator.io/) | Obfuscate JS (understand what malware authors do) |
| [Obfuscator.io Deobfuscator](https://obf-io.deobfuscate.io/) | Reverse obfuscated JavaScript |
| [de4js](https://lelinhtinh.github.io/de4js/) | Another JS unpacker/deobfuscator |
| [CesarCipher / dCode](https://www.dcode.fr/en) | Classic ciphers, encodings, puzzles (great for CTFs) |

## #️⃣ Hashes & cracking
| Tool | Use |
| --- | --- |
| [CrackStation](https://crackstation.net/) | Lookup of common/leaked hashes (MD5/SHA1/NTLM) against a huge rainbow table |
| [hashes.com](https://hashes.com/en/decrypt/hash) | Submit hashes for community cracking + identify hash type |
| [Hashcat example hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) | Reference for **every `-m` mode** and its format (see [hashcat](./hashcat)) |
| [CyberChef → Analyse hash](https://gchq.github.io/CyberChef/) | Identify hash length/type |

## 🦠 Malware & file analysis
| Tool | Use |
| --- | --- |
| [VirusTotal](https://www.virustotal.com/) | Multi-engine scan of files/hashes/URLs/IPs + community intel |
| [MetaDefender Cloud](https://metadefender.opswat.com/) | Multi-AV scanning & sanitisation |
| [Hybrid Analysis](https://www.hybrid-analysis.com/) | Free automated malware sandbox |
| [Any.Run](https://app.any.run/) | Interactive malware sandbox (watch it run) |
| [URLScan.io](https://urlscan.io/) | Sandbox a URL — screenshots, requests, verdicts |
| [Triage (tria.ge)](https://tria.ge/) | Automated sandbox with detailed reports |

## 🔎 OSINT, reputation & recon
| Tool | Use |
| --- | --- |
| [Shodan](https://www.shodan.io/) | Search engine for internet-exposed devices/services |
| [Censys](https://search.censys.io/) | Like Shodan — hosts, certs, exposed services |
| [Cisco Talos Reputation](https://talosintelligence.com/reputation_center) | IP/domain reputation & email-sender lookup |
| [AbuseIPDB](https://www.abuseipdb.com/) | Is this IP a known abuser? |
| [WhereGoes (Redirect Checker)](https://wheregoes.com/) | Trace a URL's full redirect chain |
| [crt.sh](https://crt.sh/) | Certificate transparency → discover subdomains |
| [ViewDNS.info](https://viewdns.info/) | Bundle of DNS/whois/recon lookups |
| [Have I Been Pwned](https://haveibeenpwned.com/) | Check emails/passwords against breaches |

## 📧 Email & phishing triage
| Tool | Use |
| --- | --- |
| [Google Messageheader](https://toolbox.googleapps.com/apps/messageheader/) | Parse raw email headers & delivery hops |
| [MXToolbox](https://mxtoolbox.com/) | MX/SPF/DKIM/DMARC + blacklist checks |
| [dmarcian Domain Checker](https://dmarcian.com/domain-checker/) | Inspect a domain's DMARC/SPF/DKIM posture |
| [PhishTool](https://www.phishtool.com/) | Analyse phishing emails (blue-team rooms) |

## 🐞 Vulnerabilities & exploits
| Tool | Use |
| --- | --- |
| [Exploit-DB](https://www.exploit-db.com/) | Public exploits & PoCs (CLI: `searchsploit`, see [metasploit](./metasploit)) |
| [CVE Details](https://www.cvedetails.com/) | Browse CVEs by product/version |
| [NVD](https://nvd.nist.gov/) | Authoritative CVE database with CVSS scores |
| [GTFOBins](https://gtfobins.github.io/) | Unix binaries for privesc (see [privesc](./privesc)) |
| [LOLBAS](https://lolbas-project.github.io/) | Windows living-off-the-land binaries |
| [Revshells](https://www.revshells.com/) | Reverse-shell command generator (see [shells](./shells)) |
| [MITRE ATT&CK](https://attack.mitre.org/) | TTP knowledge base (see [DETECTION.md](./DETECTION.md)) |

## 🎓 Practice platforms
| Platform | Focus |
| --- | --- |
| [TryHackMe](https://tryhackme.com/) | Guided, beginner-friendly rooms |
| [Hack The Box](https://www.hackthebox.com/) | Boxes & labs (more open-ended) |
| [PortSwigger Web Security Academy](https://portswigger.net/web-security) | Free, excellent web-hacking labs (pairs with [burpsuite](./burpsuite)) |
| [OverTheWire](https://overthewire.org/wargames/) | Linux/CLI wargames |
| [PentesterLab](https://pentesterlab.com/) | Web exploitation exercises |

---

> 🔗 Prefer offline equivalents for sensitive data: encoding/stego → [crypto](./crypto), hash cracking → [john](./john) / [hashcat](./hashcat).
