# 🔵 Sysmon & Windows Event Logs Cheat Sheet

**Sysmon** (System Monitor) is a Sysinternals tool that logs rich endpoint telemetry to the Windows Event Log — the foundation of most Windows threat hunting. This sheet covers Sysmon Event IDs, the must-know **Security** Event IDs, and how to query them.

> 💡 Sysmon logs land in **`Microsoft-Windows-Sysmon/Operational`**. A good config (e.g. **SwiftOnSecurity's**) decides what gets logged — out of the box it's noisy.

---

## Setup

| Command | Description |
| --- | --- |
| `sysmon -accepteula -i config.xml` | Install with a config |
| `sysmon -c config.xml` | Update the running config |
| `sysmon -u` | Uninstall |

## Key Sysmon Event IDs

| ID | Event | Why it matters |
| --- | --- | --- |
| **1** | Process creation | The bread & butter — what ran, command line, parent, hashes |
| **3** | Network connection | Outbound connections → spot C2 / reverse shells |
| **7** | Image loaded | DLL side-loading / injection |
| **8** | CreateRemoteThread | Classic process injection |
| **10** | Process access | Credential theft (e.g. LSASS access by a weird process) |
| **11** | File created | Dropped payloads, ransomware |
| **12–14** | Registry events | Persistence, config changes |
| **13** | Registry value set | AutoRun keys etc. |
| **22** | DNS query | Catch DNS tunnelling / suspicious domains |

## Key Windows Security Event IDs

| ID | Meaning |
| --- | --- |
| **4624** | Successful logon (check **Logon Type**: 3=network, 10=RDP) |
| **4625** | **Failed** logon → brute-force when many appear |
| **4634** | Logoff |
| **4672** | Special privileges assigned (admin logon) |
| **4688** | Process creation (native, if audited) |
| **4720** | User account **created** |
| **4732** | Member added to a (admin) group |
| **4768 / 4769** | Kerberos TGT / service ticket requested (roasting!) |
| **7045** | New service installed (psexec, persistence) |

## Querying with PowerShell

| Command | Description |
| --- | --- |
| `Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 50` | Recent Sysmon events |
| `Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625}` | All failed logons |
| `Get-WinEvent -FilterHashtable @{LogName='...Sysmon/Operational';Id=1}` | All process-creation events |
| `... \| Where-Object {$_.Message -match "powershell"}` | Filter by content |
| `wevtutil qe Security /q:"*[System[(EventID=4624)]]" /f:text` | Query via wevtutil |

## Offline EVTX hunting

| Tool | Command | Description |
| --- | --- | --- |
| **Chainsaw** | `chainsaw hunt evtx/ -s sigma/ --mapping map.yml` | Fast Sigma-based hunting over EVTX |
| **Hayabusa** | `hayabusa csv-timeline -d evtx/ -o out.csv` | Timeline + built-in detections |
| **EvtxECmd** | `EvtxECmd.exe -d evtx/ --csv out/` | Parse EVTX to CSV (Eric Zimmerman) |

---

## 🧭 Typical hunting workflow

```powershell
# 1. Brute-force? Count failed logons by source
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625} |
  Group-Object {$_.Properties[19].Value} | Sort-Object Count -Descending

# 2. Suspicious child processes (e.g. Office spawning a shell)
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational';Id=1} |
  Where-Object {$_.Message -match "ParentImage.*WINWORD" -and $_.Message -match "cmd|powershell"}

# 3. Outbound to an odd port (possible C2)
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational';Id=3} |
  Where-Object {$_.Message -match "DestinationPort: 4444"}
```

> 🔗 Aggregate these at scale in a SIEM → [splunk](../splunk). Cross-tool anomaly recipes → [DETECTION.md](../DETECTION.md).
