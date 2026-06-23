# 🌐 Gobuster Cheat Sheet

**Gobuster** brute-forces URIs (directories/files), DNS subdomains, and virtual hosts using a wordlist. Fast, written in Go, and a staple for web enumeration.

```bash
# General syntax
gobuster [mode] -u [target] -w [wordlist] [options]
```

> 💡 Common wordlists live in `/usr/share/wordlists/` and `/usr/share/seclists/`.

---

## Modes

| Mode | Example | Description |
| --- | --- | --- |
| `dir` | `gobuster dir -u http://10.10.10.1 -w list.txt` | Directory/file brute-force |
| `dns` | `gobuster dns -d example.com -w subs.txt` | DNS subdomain brute-force |
| `vhost` | `gobuster vhost -u http://10.10.10.1 -w subs.txt` | Virtual host discovery |

## Common Options

| Flag | Example | Description |
| --- | --- | --- |
| `-u` | `-u http://10.10.10.1` | Target URL |
| `-w` | `-w /usr/share/wordlists/dirb/common.txt` | Wordlist to use |
| `-x` | `-x php,txt,html` | Append file extensions |
| `-t` | `-t 50` | Number of concurrent threads |
| `-s` | `-s 200,301,302` | Only show these status codes |
| `-b` | `-b 404,403` | Blacklist (hide) these status codes |
| `-k` | `-k` | Skip TLS certificate verification |
| `-o` | `-o results.txt` | Write output to a file |
| `-r` | `-r` | Follow redirects |
| `-H` | `-H "Authorization: Bearer xyz"` | Add a custom header |

---

## 🧭 Common commands

```bash
# Directory brute-force with extensions
gobuster dir -u http://10.10.10.1 -w /usr/share/wordlists/dirb/common.txt -x php,txt,html

# Bigger wordlist, more threads, save output
gobuster dir -u http://10.10.10.1 \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -t 50 -o dirs.txt

# Subdomain enumeration
gobuster dns -d example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Virtual host discovery
gobuster vhost -u http://10.10.10.1 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```
