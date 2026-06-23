# 📚 SecLists Cheat Sheet

**SecLists** is *the* collection of wordlists for security testing — usernames, passwords, URLs, subdomains, fuzzing payloads, and more. It's the fuel for almost every brute-force/enumeration tool ([gobuster](../gobuster), [ffuf](../ffuf), [hydra](../hydra), [hashcat](../hashcat)…).

```bash
# Install on Kali / Debian
sudo apt install seclists

# Or clone the latest
git clone https://github.com/danielmiessler/SecLists.git
```

> 💡 After `apt install`, everything lives under **`/usr/share/seclists/`**.

---

## 🌐 DNS — subdomains & vhosts

`/usr/share/seclists/Discovery/DNS/`

| File | Use |
| --- | --- |
| `subdomains-top1million-5000.txt` | Fast subdomain/vhost sweep |
| `subdomains-top1million-110000.txt` | Deeper subdomain/vhost sweep |
| `namelist.txt` | Generic DNS name brute-force |
| `bitquark-subdomains-top100000.txt` | Large alternative list |

## 🗂️ Web Content — directories & files

`/usr/share/seclists/Discovery/Web-Content/`

| File | Use |
| --- | --- |
| `common.txt` | Quick directory/file check |
| `directory-list-2.3-medium.txt` | The classic go-to dir brute-force list |
| `raft-medium-directories.txt` | Directories (RAFT, well-curated) |
| `raft-medium-files.txt` | Files (RAFT) |
| `api/api-endpoints.txt` | API endpoint discovery |

## 🔑 Passwords

`/usr/share/seclists/Passwords/`

| File | Use |
| --- | --- |
| `Common-Credentials/10-million-password-list-top-1000.txt` | Fast password spray |
| `Common-Credentials/10k-most-common.txt` | Slightly larger spray list |
| `Leaked-Databases/rockyou.txt` | The famous rockyou list |
| `Default-Credentials/default-passwords.txt` | Vendor/default creds |

> 💡 On Kali, `rockyou.txt` also lives at `/usr/share/wordlists/rockyou.txt` (you may need to `gunzip rockyou.txt.gz` first).

## 👤 Usernames

`/usr/share/seclists/Usernames/`

| File | Use |
| --- | --- |
| `top-usernames-shortlist.txt` | Quick username list for login attacks |
| `Names/names.txt` | Common first names |
| `xato-net-10-million-usernames.txt` | Huge username list |

## 💥 Fuzzing

`/usr/share/seclists/Fuzzing/`

| File | Use |
| --- | --- |
| `LFI/LFI-Jhaddix.txt` | Local File Inclusion payloads |
| `SQLi/Generic-SQLi.txt` | SQL injection payloads |
| `XSS/XSS-Jhaddix.txt` | Cross-site scripting payloads |
| `special-chars.txt` | Special-character fuzzing |

---

## 🧭 Putting it together

```bash
# Directory brute-force (gobuster + SecLists)
gobuster dir -u http://10.10.10.1 \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt

# Subdomain fuzzing (ffuf + SecLists)
ffuf -u http://10.10.10.1 -H "Host: FUZZ.example.com" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 1234

# SSH password spray (hydra + SecLists)
hydra -l admin \
  -P /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt \
  ssh://10.10.10.1
```
