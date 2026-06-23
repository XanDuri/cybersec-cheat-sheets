# 🔓 John the Ripper Cheat Sheet

**John the Ripper** (JtR) is a versatile offline password cracker. Give it a hash file, point it at a wordlist (or let it run its own modes), and it recovers plaintext passwords.

```bash
# General syntax
john [options] [hash file]
```

> 💡 Many file types (zip, rar, ssh keys, etc.) need a helper to extract a crackable hash first — see the `*2john` tools below.

---

## Core Cracking Options

| Flag | Example | Description |
| --- | --- | --- |
| | `john hash.txt` | Auto-detect format, run default modes |
| `--wordlist` | `john --wordlist=rockyou.txt hash.txt` | Dictionary attack |
| `--rules` | `john --wordlist=rockyou.txt --rules hash.txt` | Apply word-mangling rules |
| `--format` | `john --format=raw-md5 hash.txt` | Force a specific hash format |
| `--incremental` | `john --incremental hash.txt` | Brute-force (incremental) mode |
| `--show` | `john --show hash.txt` | Display already-cracked passwords |
| `--restore` | `john --restore` | Resume an interrupted session |

## Common Formats (`--format=`)

| Format | Hash type |
| --- | --- |
| `raw-md5` | Plain MD5 |
| `raw-sha1` | Plain SHA1 |
| `raw-sha256` | Plain SHA256 |
| `nt` | Windows NTLM |
| `sha512crypt` | Linux `/etc/shadow` ($6$) |
| `bcrypt` | bcrypt ($2a$/$2y$) |

## Hash Extraction Helpers (`*2john`)

| Tool | Example | Description |
| --- | --- | --- |
| `unshadow` | `unshadow /etc/passwd /etc/shadow > hashes.txt` | Combine passwd + shadow into a crackable file |
| `zip2john` | `zip2john secret.zip > zip.hash` | Extract a hash from a password-protected zip |
| `rar2john` | `rar2john secret.rar > rar.hash` | …from a RAR archive |
| `ssh2john` | `ssh2john id_rsa > rsa.hash` | …from an encrypted SSH private key |
| `office2john` | `office2john doc.docx > office.hash` | …from a protected Office document |

---

## 🧭 Typical workflow

```bash
# 1. Linux shadow file
unshadow /etc/passwd /etc/shadow > hashes.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --show hashes.txt

# 2. Crack a protected zip
zip2john secret.zip > zip.hash
john --wordlist=rockyou.txt zip.hash

# 3. Force format + apply rules
john --format=raw-md5 --wordlist=rockyou.txt --rules hash.txt
```
