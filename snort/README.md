# 🚨 Snort Cheat Sheet

**Snort** is an open-source IDS/IPS (Intrusion Detection/Prevention System). It runs in three modes: **sniffer**, **packet logger**, and **NIDS** (rule-based detection).

```bash
# General syntax
sudo snort [mode/options] -i [interface]
```

> 💡 Test before you run: `sudo snort -T -c /etc/snort/snort.conf` validates your config and rules.

---

## Verify & Test

| Flag | Example | Description |
| --- | --- | --- |
| `-V` | `snort -V` | Show version / confirm install |
| `-T` | `sudo snort -T -c /etc/snort/snort.conf` | Test the configuration, don't run |

## Sniffer Mode

| Flag | Example | Description |
| --- | --- | --- |
| `-v` | `sudo snort -i eth0 -v` | Verbose — show TCP/IP headers |
| `-d` | `sudo snort -i eth0 -d` | Also show packet payload (data) |
| `-e` | `sudo snort -i eth0 -e` | Also show data-link (Ethernet) headers |
| `-X` | `sudo snort -i eth0 -X` | Show the full packet from the link layer up |
| `-vde` | `sudo snort -i eth0 -vde` | Combine: headers + data + link layer |

## Packet Logger Mode

| Flag | Example | Description |
| --- | --- | --- |
| `-l` | `sudo snort -i eth0 -l ./logs` | Log packets to a directory (binary) |
| `-K ASCII` | `sudo snort -i eth0 -K ASCII -l ./logs` | Log in human-readable ASCII |
| `-r` | `sudo snort -r capture.pcap` | Read & process a saved pcap |

## NIDS Mode (rule-based detection)

| Flag | Example | Description |
| --- | --- | --- |
| `-c` | `sudo snort -c /etc/snort/snort.conf -i eth0` | Run with a config/ruleset |
| `-A console` | `sudo snort -c snort.conf -A console -i eth0` | Print alerts to the console |
| `-A fast` | `sudo snort -c snort.conf -A fast -i eth0` | Fast one-line alert format |
| `-A full` | `sudo snort -c snort.conf -A full -i eth0` | Full alert detail |
| `-q` | `sudo snort -q -c snort.conf -A console -i eth0` | Quiet — suppress banner/stats |
| `-r` + `-c` | `sudo snort -c snort.conf -r cap.pcap -A console` | Replay a pcap through your rules |

## IPS (inline) Mode

| Flag | Example | Description |
| --- | --- | --- |
| `-Q --daq afpacket` | `sudo snort -Q --daq afpacket -i eth0:eth1 -c snort.conf` | Inline mode — can **drop** malicious traffic |

---

## ✍️ Rule Syntax

```text
action protocol src_ip src_port -> dst_ip dst_port (options)
```

| Part | Example | Meaning |
| --- | --- | --- |
| action | `alert` | What to do (`alert`, `log`, `drop`, `reject`) |
| protocol | `tcp` | Protocol (`tcp`, `udp`, `icmp`, `ip`) |
| direction | `->` | Flow direction (`->` or `<>` bidirectional) |
| options | `(msg:"..."; sid:1000001;)` | Detection options |

**Example rules:**

```text
# Alert on any traffic to port 80
alert tcp any any -> any 80 (msg:"HTTP traffic detected"; sid:1000001; rev:1;)

# Alert when a payload contains a string (case-insensitive)
alert tcp any any -> any any (msg:"Possible cleartext password"; content:"password"; nocase; sid:1000002; rev:1;)

# Alert on ICMP (ping)
alert icmp any any -> any any (msg:"ICMP ping detected"; sid:1000003; rev:1;)
```

## Common Rule Options

| Option | Example | Description |
| --- | --- | --- |
| `msg` | `msg:"Alert text";` | Message shown when the rule fires |
| `sid` | `sid:1000001;` | Unique rule ID (use >1,000,000 for local rules) |
| `rev` | `rev:1;` | Rule revision number |
| `content` | `content:"admin";` | Match a string/byte sequence in payload |
| `nocase` | `nocase;` | Make `content` case-insensitive |
| `flags` | `flags:S;` | Match TCP flags (S=SYN, A=ACK, F=FIN…) |
| `flow` | `flow:to_server,established;` | Match connection state/direction |

---

## 🧭 Typical workflow

```bash
# 1. Validate config + rules
sudo snort -T -c /etc/snort/snort.conf

# 2. Run live IDS, alerts to console
sudo snort -q -c /etc/snort/snort.conf -A console -i eth0

# 3. Investigate a capture against your rules
sudo snort -c /etc/snort/snort.conf -r suspicious.pcap -A console
```
