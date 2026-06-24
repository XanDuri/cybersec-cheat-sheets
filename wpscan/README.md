# 🔵 WPScan Cheat Sheet

**WPScan** is a dedicated WordPress security scanner — it enumerates plugins, themes, users, and known vulnerabilities. If you see a WordPress site, reach for this.

```bash
# General syntax
wpscan --url <target> [options]
```

> 💡 Vuln data needs a free **API token** from [wpvulndb / wpscan.com](https://wpscan.com/api) — pass it with `--api-token`. Without it you still get enumeration, just no CVE matching.

---

## Core Options

| Flag | Example | Description |
| --- | --- | --- |
| `--url` | `wpscan --url http://10.10.10.1` | Target site |
| `--enumerate` | `wpscan --url http://site --enumerate` | Enumerate (see modes below) |
| `--api-token` | `wpscan --url http://site --api-token TOKEN` | Enable vulnerability lookups |
| `--random-user-agent` | `wpscan --url http://site --rua` | Rotate User-Agent to dodge filters |
| `--disable-tls-checks` | `wpscan --url https://site --disable-tls-checks` | Ignore bad/self-signed certs |
| `-o` | `wpscan --url http://site -o scan.txt` | Save output |

## Enumeration modes (`--enumerate` / `-e`)

| Code | Example | Enumerates |
| --- | --- | --- |
| `vp` | `-e vp` | **Vulnerable** plugins |
| `ap` | `-e ap` | **All** plugins (thorough, slower) |
| `vt` | `-e vt` | Vulnerable themes |
| `u` | `-e u` | **Usernames** (feeds password attacks) |
| `u1-50` | `-e u1-50` | User IDs in a range |
| `cb` | `-e cb` | Config backups |
| combo | `-e vp,vt,u` | Several at once |

## Password attack (brute-force)

| Flag | Example | Description |
| --- | --- | --- |
| `--passwords` | `wpscan --url http://site --passwords rockyou.txt` | Wordlist to try |
| `--usernames` | `--usernames admin` | Single user or a list |
| `--password-attack` | `--password-attack xmlrpc` | Method: `xmlrpc` (fast) or `wp-login` |
| combined | `wpscan --url http://site -U users.txt -P rockyou.txt` | Spray a user list |

---

## 🧭 Typical workflow

```bash
# 1. Enumerate users + vulnerable plugins/themes
wpscan --url http://10.10.10.1 -e u,vp,vt --api-token TOKEN

# 2. Brute-force a found user via the fast xmlrpc endpoint
wpscan --url http://10.10.10.1 -U admin -P /usr/share/wordlists/rockyou.txt \
  --password-attack xmlrpc
```

> 🔗 Wordlists → [seclists](../seclists). Cracked nothing online? Pull hashes and crack offline with [john](../john)/[hashcat](../hashcat).
