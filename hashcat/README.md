# ⚙️ Hashcat Cheat Sheet

**Hashcat** is the fastest password cracker, using your GPU to try huge numbers of candidates per second. You specify the hash type with `-m` and the attack mode with `-a`.

```bash
# General syntax
hashcat -m [hash-mode] -a [attack-mode] [hashfile] [wordlist/mask]
```

> 💡 Not sure of the mode? `hashcat --example-hashes` lists every `-m` value with a sample hash.

---

## Attack Modes (`-a`)

| Mode | Example | Description |
| --- | --- | --- |
| `-a 0` | `hashcat -m 0 -a 0 hash.txt rockyou.txt` | Straight (dictionary) |
| `-a 1` | `hashcat -m 0 -a 1 hash.txt left.txt right.txt` | Combinator (join two lists) |
| `-a 3` | `hashcat -m 0 -a 3 hash.txt ?a?a?a?a` | Brute-force (mask) |
| `-a 6` | `hashcat -m 0 -a 6 hash.txt rockyou.txt ?d?d` | Hybrid: wordlist + mask |
| `-a 7` | `hashcat -m 0 -a 7 hash.txt ?d?d rockyou.txt` | Hybrid: mask + wordlist |

## Common Hash Modes (`-m`)

| Mode | Hash type |
| --- | --- |
| `0` | MD5 |
| `100` | SHA1 |
| `1400` | SHA256 |
| `1000` | NTLM (Windows) |
| `1800` | sha512crypt (Linux $6$) |
| `3200` | bcrypt ($2*$) |
| `5600` | NetNTLMv2 |
| `22000` | WPA-PBKDF2 (Wi-Fi) |

## Mask Charsets (for `-a 3`)

| Token | Matches |
| --- | --- |
| `?l` | Lowercase a–z |
| `?u` | Uppercase A–Z |
| `?d` | Digits 0–9 |
| `?s` | Special characters |
| `?a` | All of the above |
| `?h` / `?H` | Hex 0–9a–f / 0–9A–F |

## Useful Flags

| Flag | Example | Description |
| --- | --- | --- |
| `--show` | `hashcat -m 0 hash.txt --show` | Show already-cracked hashes |
| `-O` | `hashcat -m 0 -a 0 -O hash.txt rockyou.txt` | Optimised kernels (faster, length-limited) |
| `-w` | `-w 3` | Workload profile (1=low … 4=nightmare) |
| `-r` | `-r rules/best64.rule` | Apply a rules file |
| `--force` | `--force` | Ignore warnings (e.g. in a VM) |
| `-o` | `-o cracked.txt` | Write results to a file |

---

## 🧭 Common commands

```bash
# Dictionary attack on MD5
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# NTLM with a rules file
hashcat -m 1000 -a 0 ntlm.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Brute-force an 8-char all-charset password
hashcat -m 0 -a 3 hash.txt ?a?a?a?a?a?a?a?a

# Hybrid: wordlist + two trailing digits
hashcat -m 0 -a 6 hash.txt rockyou.txt ?d?d

# Show what you've cracked
hashcat -m 0 hash.txt --show
```
