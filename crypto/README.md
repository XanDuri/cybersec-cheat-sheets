# 🔐 Crypto, Encoding & Stego Cheat Sheet

CTF-style "decode this" challenges and quick crypto tasks. Covers **CyberChef**, **base64/hex/xxd**, hashing, **openssl**, and basic **steganography/forensics** (steghide, exiftool, binwalk).

> 🧠 **Encoding ≠ encryption ≠ hashing.** Encoding (Base64/hex) is reversible with no key. Hashing is one-way. Encryption needs a key. Identify which one you're looking at first.

---

## Identify it first

| Clue | Likely |
| --- | --- |
| Ends in `=` / `==`, A–Z a–z 0–9 +/ | **Base64** |
| Only `0-9 a-f` | **Hex** |
| `$2y$`, `$6$`, `$1$` | bcrypt / sha512crypt / md5crypt → crack with [john](../john) |
| 32 hex chars | **MD5** · 40 = SHA-1 · 64 = SHA-256 |
| Shifted letters | **ROT13 / Caesar** |
| Tool | `hashid <hash>` or `hash-identifier` to fingerprint |

## Encoding / decoding (CLI)

| Command | Description |
| --- | --- |
| `echo -n 'text' \| base64` | Encode Base64 |
| `echo 'dGV4dA==' \| base64 -d` | Decode Base64 |
| `echo -n 'text' \| xxd` | Hex dump |
| `echo '74657874' \| xxd -r -p` | Hex → text |
| `echo 'text' \| tr 'A-Za-z' 'N-ZA-Mn-za-m'` | ROT13 |
| `echo -n 'text' \| md5sum` | MD5 hash |
| `echo -n 'text' \| sha256sum` | SHA-256 hash |

## CyberChef

The browser-based "Cyber Swiss-Army knife" — chain operations in a **Recipe**. Indispensable for layered encodings.

| Feature | Use |
| --- | --- |
| **Magic** operation | Auto-detect & suggest decodings — try this first |
| Recipe chaining | Stack From Base64 → From Hex → Gunzip etc. |
| Common ops | From Base64, From Hex, ROT13, XOR Brute Force, Magic, Entropy |

## openssl

| Command | Description |
| --- | --- |
| `openssl enc -aes-256-cbc -d -in file.enc -out file` | Decrypt AES |
| `openssl rsautl -decrypt -inkey priv.pem -in cipher` | RSA decrypt with a private key |
| `openssl x509 -in cert.pem -text -noout` | Inspect a certificate |
| `openssl s_client -connect host:443` | Grab/inspect a TLS cert live |
| `openssl passwd -6 'password'` | Generate a `/etc/shadow`-style hash |

## Steganography & file forensics

| Command | Description |
| --- | --- |
| `file mystery` | Identify a file's real type (ignore the extension!) |
| `exiftool image.jpg` | Read metadata (flags hide in comments/GPS) |
| `strings -n 8 file` | Pull readable strings from a binary |
| `binwalk file` | Find embedded files/archives |
| `binwalk -e file` | Extract embedded files |
| `steghide extract -sf image.jpg` | Pull hidden data (asks for passphrase) |
| `zsteg image.png` | LSB stego in PNG/BMP |
| `stegseek file.jpg rockyou.txt` | Brute-force a steghide passphrase |

---

## 🧭 Typical workflow

```bash
# 1. What is this file, really?
file mystery && exiftool mystery && strings mystery | head

# 2. Embedded data?
binwalk -e mystery

# 3. Layered text encoding? → paste into CyberChef → "Magic"
echo 'NBSWY3DP' | base64 -d            # (or let CyberChef figure it out)

# 4. A hash? Identify, then crack
hashid '$6$xyz$...'                      # → see ../john / ../hashcat
```

> 🔗 Crack identified hashes with [john](../john)/[hashcat](../hashcat).
