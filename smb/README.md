# 🗂️ SMB Enumeration Cheat Sheet

SMB (ports **139/445**) is a goldmine on Windows networks. These tools enumerate shares, users, and policies — often without credentials via "null sessions."

Covers: **enum4linux**, **smbclient**, **rpcclient**, **smbmap**, **nmap SMB scripts**.

---

## enum4linux

All-in-one SMB enumeration wrapper.

| Flag | Example | Description |
| --- | --- | --- |
| `-a` | `enum4linux -a 10.10.10.1` | Do everything (full enumeration) |
| `-U` | `enum4linux -U 10.10.10.1` | List users |
| `-S` | `enum4linux -S 10.10.10.1` | List shares |
| `-G` | `enum4linux -G 10.10.10.1` | List groups |
| `-P` | `enum4linux -P 10.10.10.1` | Password policy |
| `-o` | `enum4linux -o 10.10.10.1` | OS information |

## smbclient

FTP-like client for browsing/accessing shares.

| Command | Description |
| --- | --- |
| `smbclient -L //10.10.10.1 -N` | List shares with a null (no-password) session |
| `smbclient -L //10.10.10.1 -U user` | List shares as a user |
| `smbclient //10.10.10.1/share -N` | Connect to a share anonymously |
| `smbclient //10.10.10.1/share -U user` | Connect as a user |

**Inside an smbclient session:**

| Command | Description |
| --- | --- |
| `ls` | List files |
| `get file.txt` | Download a file |
| `put file.txt` | Upload a file |
| `mget *` | Download multiple files |
| `cd dir` | Change directory |

## rpcclient

Query the target over MS-RPC (great after a null session).

| Command | Description |
| --- | --- |
| `rpcclient -U "" -N 10.10.10.1` | Connect with a null session |
| `enumdomusers` | List domain users (run inside rpcclient) |
| `enumdomgroups` | List domain groups |
| `querydominfo` | Domain / policy info |
| `lookupnames admin` | Resolve a name to a RID/SID |

## smbmap

Quickly show share permissions.

| Command | Description |
| --- | --- |
| `smbmap -H 10.10.10.1` | List shares + access (read/write) |
| `smbmap -H 10.10.10.1 -u user -p pass` | Authenticated listing |
| `smbmap -H 10.10.10.1 -R share` | Recursively list a share's contents |

## Nmap SMB Scripts

| Command | Description |
| --- | --- |
| `nmap -p 445 --script smb-os-discovery 10.10.10.1` | Identify OS via SMB |
| `nmap -p 445 --script smb-enum-shares 10.10.10.1` | Enumerate shares |
| `nmap -p 445 --script smb-enum-users 10.10.10.1` | Enumerate users |
| `nmap -p 445 --script smb-vuln-ms17-010 10.10.10.1` | Check for EternalBlue |

---

## 🧭 Typical workflow

```bash
# 1. Quick share + OS overview
nmap -p 139,445 --script smb-os-discovery,smb-enum-shares 10.10.10.1

# 2. Full enumeration
enum4linux -a 10.10.10.1 | tee enum.txt

# 3. List and browse shares anonymously
smbclient -L //10.10.10.1 -N
smbclient //10.10.10.1/Public -N
```
