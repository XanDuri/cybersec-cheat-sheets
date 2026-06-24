# 🔵 Splunk SPL Cheat Sheet

**Splunk** is a leading SIEM. You query data with **SPL** (Search Processing Language) — a pipeline where each `|` passes results to the next command. Essential for SOC / blue-team work.

```text
# General shape
index=<index> sourcetype=<type> <filters> | command1 | command2 ...
```

> ⚠️ **On lab VMs:** if Splunk doesn't start automatically, launch it manually. And set the **time picker to "All time"** — otherwise historical events fall outside the default window and your search looks empty.

---

## Basic Searching

| Search | Description |
| --- | --- |
| `index=main` | Everything in an index |
| `index=main sourcetype=access_combined` | Filter by source type |
| `index=main status=404` | Filter by field value |
| `index=main "failed password"` | Free-text keyword search |
| `index=main host=server01` | Filter by host |

## Filtering Commands

| Command | Example | Description |
| --- | --- | --- |
| `search` | `... \| search status=500` | Filter mid-pipeline |
| `where` | `... \| where bytes > 10000` | Filter with expressions/comparisons |
| `dedup` | `... \| dedup user` | Remove duplicate values of a field |
| `head` | `... \| head 20` | Keep the first N results |
| `tail` | `... \| tail 20` | Keep the last N results |

## Reporting & Stats

| Command | Example | Description |
| --- | --- | --- |
| `stats` | `... \| stats count by src_ip` | Aggregate (count, sum, avg…) by field |
| `top` | `... \| top 10 url` | Most common values |
| `rare` | `... \| rare user` | Least common values |
| `table` | `... \| table _time, src_ip, action` | Show only chosen fields as a table |
| `sort` | `... \| sort -count` | Sort (`-` = descending) |
| `timechart` | `... \| timechart count by status` | Time-series chart |

## Field Manipulation

| Command | Example | Description |
| --- | --- | --- |
| `eval` | `... \| eval mb=bytes/1024/1024` | Create/modify a field |
| `rename` | `... \| rename src_ip AS source` | Rename a field |
| `fields` | `... \| fields src_ip, dest_ip` | Keep only listed fields |
| `rex` | `... \| rex field=_raw "user=(?<usr>\w+)"` | Extract a field with regex |
| `fillnull` | `... \| fillnull value="N/A"` | Replace nulls |

## Common Stats Functions

| Function | Example | Description |
| --- | --- | --- |
| `count` | `stats count` | Number of events |
| `dc` | `stats dc(src_ip)` | Distinct count |
| `sum` | `stats sum(bytes)` | Total |
| `avg` | `stats avg(response_time)` | Average |
| `values` | `stats values(user) by host` | List unique values |

---

## 🧭 SOC-style examples

```text
# Top source IPs hitting the web server
index=web sourcetype=access_combined | top 10 clientip

# Failed login attempts grouped by user
index=auth "failed password" | stats count by user | sort -count

# Spot brute-force: >10 failures from one IP
index=auth action=failure | stats count by src_ip | where count > 10

# Traffic over time, split by HTTP status
index=web | timechart count by status

# Extract usernames from raw logs, then count
index=auth | rex field=_raw "user=(?<username>\w+)" | stats count by username
```

---

## 🔎 Detecting anomalies (threat hunting)

```text
# Port scan: one src_ip touching many distinct dest_ports
index=firewall action=blocked
| stats dc(dest_port) AS ports values(dest_port) AS port_list by src_ip
| where ports > 20 | sort -ports

# Brute-force then success (likely compromise)
index=auth (action=failure OR action=success)
| stats count(eval(action="failure")) AS fails count(eval(action="success")) AS wins by user, src_ip
| where fails > 10 AND wins > 0

# Data exfiltration: top outbound talkers by bytes
index=firewall direction=outbound
| stats sum(bytes_out) AS total by src_ip | sort -total | head 10

# C2 beaconing: near-constant interval between callbacks to one dest
index=proxy | sort 0 _time | streamstats current=f last(_time) AS prev by src_ip, dest_ip
| eval gap = _time - prev | stats avg(gap) AS avg stdev(gap) AS jitter count by src_ip, dest_ip
| where count > 20 AND jitter < 5

# New / rare process (possible malware) — compare to a lookup of known-good
index=sysmon EventCode=1 | rare 20 Image

# Account created then added to admins shortly after
index=wineventlog (EventCode=4720 OR EventCode=4732)
| transaction TargetUserName maxspan=10m | search EventCode=4720 EventCode=4732
```

> 🔗 The same anomalies in nmap / tcpdump / Wireshark / Snort: see [DETECTION.md](../DETECTION.md).
