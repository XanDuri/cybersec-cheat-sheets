# 📡 tcpdump Cheat Sheet

**tcpdump** is a command-line packet capture and analysis tool. Great for quick captures on servers with no GUI, and for saving `.pcap` files to open later in Wireshark.

```bash
# General syntax
sudo tcpdump [options] [filter expression]
```

> 💡 Capturing usually needs **root**. Use `-w` to save a pcap, then analyse it in [Wireshark](../wireshark).

---

## Core Flags

| Flag | Example | Description |
| --- | --- | --- |
| `-i` | `sudo tcpdump -i eth0` | Capture on a specific interface |
| `-i any` | `sudo tcpdump -i any` | Capture on all interfaces |
| `-D` | `tcpdump -D` | List available interfaces |
| `-n` | `sudo tcpdump -n` | Don't resolve hostnames |
| `-nn` | `sudo tcpdump -nn` | Don't resolve hostnames **or** ports |
| `-c` | `sudo tcpdump -c 10` | Capture only N packets then stop |
| `-w` | `sudo tcpdump -w cap.pcap` | Write packets to a file |
| `-r` | `tcpdump -r cap.pcap` | Read packets from a file |
| `-A` | `sudo tcpdump -A` | Print payload as ASCII |
| `-X` | `sudo tcpdump -X` | Print payload as hex + ASCII |
| `-e` | `sudo tcpdump -e` | Show link-layer (MAC) headers |
| `-v` / `-vv` | `sudo tcpdump -vv` | Increase verbosity |

## Filter Expressions

| Filter | Example | Description |
| --- | --- | --- |
| `host` | `tcpdump host 10.10.10.1` | Traffic to/from a host |
| `src` / `dst` | `tcpdump src 10.10.10.1` | Source-only / destination-only |
| `port` | `tcpdump port 80` | Traffic on a port |
| `portrange` | `tcpdump portrange 20-80` | Traffic on a port range |
| `net` | `tcpdump net 10.10.10.0/24` | Traffic on a subnet |
| `tcp` / `udp` / `icmp` | `tcpdump icmp` | Filter by protocol |

## Combining Filters

| Operator | Example | Description |
| --- | --- | --- |
| `and` | `tcpdump host 10.10.10.1 and port 80` | Both conditions |
| `or` | `tcpdump port 80 or port 443` | Either condition |
| `not` | `tcpdump not port 22` | Exclude a condition |

---

## 🎯 Filtering by TCP flags (scan detection)

You can match exact flag combinations with `tcp[tcpflags]`. This is the key to spotting scans on the wire.

| Filter | Example | Catches |
| --- | --- | --- |
| SYN only | `tcpdump 'tcp[tcpflags] == tcp-syn'` | Connection attempts / **SYN scan** |
| SYN+ACK | `tcpdump 'tcp[tcpflags] == (tcp-syn\|tcp-ack)'` | Open ports replying |
| RST | `tcpdump 'tcp[tcpflags] & tcp-rst != 0'` | Closed/refused ports (lots of RST = scan in progress) |
| NULL scan | `tcpdump 'tcp[tcpflags] == 0'` | Nmap `-sN` (no flags set) |
| FIN scan | `tcpdump 'tcp[tcpflags] == tcp-fin'` | Nmap `-sF` |
| Xmas scan | `tcpdump 'tcp[tcpflags] == (tcp-fin\|tcp-push\|tcp-urg)'` | Nmap `-sX` |

## 🔎 Detecting anomalies

```bash
# Port scan: one source firing SYNs at many ports (watch the dst port column climb)
sudo tcpdump -nn 'tcp[tcpflags] == tcp-syn'

# Possible reverse shell / C2: outbound to an odd high port
sudo tcpdump -nn 'dst port 4444 or dst port 1337'

# ICMP tunnelling / exfil: pings with unusually large payloads
sudo tcpdump -nn 'icmp and greater 100'

# Cleartext credentials crossing the wire (FTP/HTTP/Telnet)
sudo tcpdump -nn -A 'port 21 or port 80 or port 23' | grep -iE 'user|pass|login'

# DNS exfiltration: abnormally long / frequent queries
sudo tcpdump -nn -A port 53
```

> 🔗 Same anomalies, other tools (Wireshark / Snort / Splunk): see [DETECTION.md](../DETECTION.md).

---

## 🧭 Common one-liners

```bash
# Save HTTP traffic to/from a host as a pcap
sudo tcpdump -i eth0 -nn 'host 10.10.10.1 and port 80' -w http.pcap

# Watch DNS queries live in plain text
sudo tcpdump -i eth0 -nn -A port 53

# Capture 100 packets then stop, no resolution
sudo tcpdump -i eth0 -nn -c 100

# Read a saved capture and filter for a host
tcpdump -nn -r capture.pcap host 10.10.10.1
```

> 💡 **Gotchas:** quote filter expressions that contain `|` or spaces. `greater N` / `less N` filter by packet length. Without `-w` you only see headers — add `-A`/`-X` for payload.
