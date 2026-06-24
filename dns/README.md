# 🌐 DNS Recon Cheat Sheet

DNS enumeration is early recon — map a domain's hosts, mail servers, and (if you're lucky) pull the whole zone. Tools here: **dig**, **nslookup**, **host**, **dnsenum**, **dnsrecon**, **fierce**.

```bash
# General shape
dig [@server] [name] [record-type] [options]
```

> 💡 Add discovered hostnames to your `/etc/hosts` so tools resolve the lab domain (e.g. `10.10.10.1  target.thm`).

---

## dig — the precise one

| Command | Example | Description |
| --- | --- | --- |
| Basic A record | `dig example.com` | Resolve a name to an IP |
| Short answer | `dig +short example.com` | Just the result, no noise |
| Pick a record | `dig example.com MX` | A, AAAA, MX, NS, TXT, SOA, CNAME, PTR |
| Use a DNS server | `dig @10.10.10.1 example.com` | Query a *specific* server (the target's own) |
| All records | `dig example.com ANY` | Ask for everything (often filtered now) |
| **Zone transfer** | `dig @10.10.10.1 example.com AXFR` | Dump the entire zone — huge win if misconfigured |
| Reverse lookup | `dig -x 10.10.10.1` | IP → hostname (PTR) |
| Trace | `dig +trace example.com` | Follow resolution from the root servers |

## nslookup & host (quick & cross-platform)

| Command | Example | Description |
| --- | --- | --- |
| `nslookup` | `nslookup example.com` | Quick A lookup (works on Windows too) |
| set type | `nslookup -type=MX example.com` | Query a record type |
| server | `nslookup example.com 10.10.10.1` | Use a specific resolver |
| `host` | `host example.com` | One-line lookup |
| `host -t` | `host -t ns example.com` | Specific record type |
| `host -l` | `host -l example.com 10.10.10.1` | Attempt a zone transfer |

## Subdomain & zone enumeration

| Command | Example | Description |
| --- | --- | --- |
| `dnsenum` | `dnsenum example.com` | Records + zone transfer + subdomain brute-force |
| `dnsrecon` | `dnsrecon -d example.com` | Standard enumeration |
| `dnsrecon -t axfr` | `dnsrecon -d example.com -t axfr` | Try zone transfer on every NS |
| `dnsrecon -D` | `dnsrecon -d example.com -D subs.txt -t brt` | Brute-force subdomains from a wordlist |
| `fierce` | `fierce --domain example.com` | Find non-contiguous IP space & subdomains |
| `gobuster dns` | `gobuster dns -d example.com -w subs.txt` | Fast subdomain brute (see [gobuster](../gobuster)) |

## whois

| Command | Description |
| --- | --- |
| `whois example.com` | Registrar, creation date, name servers, contacts |

---

## 🧭 Typical workflow

```bash
# 1. Find the name servers
dig +short example.com NS

# 2. Always try a zone transfer against each NS (instant map if it works)
dig @ns1.example.com example.com AXFR

# 3. If AXFR is refused, brute-force subdomains
dnsenum --enum example.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

> 🔗 Wordlists for `-D`/`-w` live in [seclists](../seclists). Found a web vhost? → [gobuster](../gobuster) / [ffuf](../ffuf).
> 🔗 Detecting DNS abuse (tunnelling) from the blue side: [DETECTION.md → DNS tunnelling](../DETECTION.md#5-dns-tunnelling).
