# 🕷️ Nikto Cheat Sheet

**Nikto** is a fast, noisy web server vulnerability scanner. It checks for thousands of known dangerous files, outdated software, and misconfigurations. Great for a quick first pass on a web target.

```bash
# General syntax
nikto -h <target>
```

> ⚠️ Nikto is **loud** — it makes thousands of requests and will trip any IDS. Fine for labs, not for stealth.

---

## Core Options

| Flag | Example | Description |
| --- | --- | --- |
| `-h` | `nikto -h 10.10.10.1` | Target host or URL |
| `-p` | `nikto -h 10.10.10.1 -p 8080` | Port (default 80) |
| `-ssl` | `nikto -h 10.10.10.1 -ssl` | Force HTTPS |
| `-h` (URL) | `nikto -h https://10.10.10.1:8443` | Full URL incl. scheme & port |
| `-T` | `nikto -h 10.10.10.1 -T 1234` | Tune which test types to run |
| `-Tuning x` | `nikto -h 10.10.10.1 -Tuning x` | Reverse — run all *except* x |
| `-useragent` | `nikto -h 10.10.10.1 -useragent "Mozilla/5.0"` | Spoof the User-Agent |
| `-id` | `nikto -h 10.10.10.1 -id admin:pass` | Basic-auth credentials |
| `-Display V` | `nikto -h 10.10.10.1 -Display V` | Verbose output |
| `-o` | `nikto -h 10.10.10.1 -o out.html -Format htm` | Save report (txt/htm/csv/xml) |

## Tuning categories (`-T`)

| Value | Tests |
| --- | --- |
| `1` | Interesting files / seen in logs |
| `2` | Misconfiguration / default files |
| `3` | Information disclosure |
| `4` | Injection (XSS/script/HTML) |
| `5` | Remote file retrieval |
| `6` | Denial of service |
| `8` | Command execution / remote shell |
| `9` | SQL injection |
| `x` | Reverse — everything *except* the listed |

---

## 🧭 Typical workflow

```bash
# 1. Quick scan of a discovered web port
nikto -h http://10.10.10.1

# 2. Focused scan for injection + command exec, save a report
nikto -h http://10.10.10.1 -Tuning 489 -o nikto.html -Format htm

# 3. Through a proxy (capture/inspect in Burp)
nikto -h http://10.10.10.1 -useragent "Mozilla/5.0" -Display V
```

> 🔗 Use alongside [gobuster](../gobuster)/[ffuf](../ffuf) for content discovery, [wpscan](../wpscan) for WordPress, and [sqlmap](../sqlmap) on any injectable params Nikto flags.
