# 🦈 Wireshark Cheat Sheet

**Wireshark** is the standard GUI tool for capturing and analysing network traffic. The real power is in **display filters** — typed into the bar at the top to narrow down what you see.

> 💡 **Capture filters** (BPF syntax, set before capture) ≠ **display filters** (set after, much richer). This sheet focuses on display filters.

---

## Filtering by Address & Port

| Filter | Description |
| --- | --- |
| `ip.addr == 10.10.10.1` | Any packet to or from this IP |
| `ip.src == 10.10.10.1` | Source IP only |
| `ip.dst == 10.10.10.1` | Destination IP only |
| `ip.addr == 10.10.10.0/24` | Whole subnet |
| `tcp.port == 80` | TCP port 80, either direction |
| `udp.port == 53` | UDP port 53 |
| `eth.addr == 00:11:22:33:44:55` | Filter by MAC address |

## Filtering by Protocol

| Filter | Description |
| --- | --- |
| `http` | HTTP traffic only |
| `dns` | DNS traffic only |
| `tcp` / `udp` / `icmp` | By transport/protocol |
| `arp` | ARP traffic |
| `tls` | TLS/SSL traffic |
| `ftp` | FTP control traffic |

## Digging into HTTP

| Filter | Description |
| --- | --- |
| `http.request` | All HTTP requests |
| `http.response` | All HTTP responses |
| `http.request.method == "POST"` | POST requests (often logins) |
| `http.host == "example.com"` | Requests to a specific host |
| `http.response.code == 200` | Responses with a given status code |

## TCP Flags & Analysis

| Filter | Description |
| --- | --- |
| `tcp.flags.syn == 1` | SYN packets (handy for spotting scans) |
| `tcp.flags.reset == 1` | RST packets (refused/closed connections) |
| `tcp.analysis.retransmission` | Retransmitted packets (loss/latency) |
| `tcp.stream eq 0` | Isolate a single TCP conversation |

## Searching Packet Contents

| Filter | Description |
| --- | --- |
| `frame contains "password"` | Match a string anywhere in the packet |
| `http contains "login"` | Match a string within HTTP |
| `tcp matches "user.*"` | Regex match within TCP payload |

## Combining Filters

| Operator | Example | Description |
| --- | --- | --- |
| `and` / `&&` | `ip.addr == 10.10.10.1 && http` | Both conditions |
| `or` / `\|\|` | `tcp.port == 80 or tcp.port == 443` | Either condition |
| `not` / `!` | `not arp` | Exclude a condition |

---

## 🧭 Useful tricks

- **Follow a stream:** right-click a packet → **Follow → TCP/HTTP Stream** to reconstruct the whole conversation (great for reading cleartext creds).
- **Statistics → Conversations:** see who's talking to whom and how much.
- **Statistics → Protocol Hierarchy:** breakdown of protocols in the capture.
- **File → Export Objects → HTTP:** pull files/images transferred over HTTP.
- **Colourise:** right-click a field → **Apply as Column** to add it to the packet list view.
