# 🧠 Volatility Cheat Sheet

**Volatility** is the standard framework for **memory forensics** — analysing a RAM dump to find running processes, network connections, injected code, and credentials. Core of any DFIR / blue-team room.

```bash
# Volatility 3 (current)
vol -f memory.dmp <plugin> [args]

# Volatility 2 (legacy, needs a profile)
volatility -f memory.dmp --profile=Win7SP1x64 <plugin>
```

> 💡 **v3 vs v2:** Volatility **3** auto-detects the OS (no profile needed) and uses `windows.<plugin>`. Volatility **2** needs `imageinfo` to pick a `--profile` first. This sheet shows v3 names with v2 equivalents.

---

## First steps

| Goal | Vol 3 | Vol 2 |
| --- | --- | --- |
| Identify the image / OS | *(automatic)* | `volatility -f mem.dmp imageinfo` |
| List processes | `windows.pslist` | `pslist` |
| Process tree (parent/child) | `windows.pstree` | `pstree` |
| Hidden/unlinked processes | `windows.psscan` | `psscan` |

## Processes & DLLs

| Plugin (v3) | Description |
| --- | --- |
| `windows.pslist` | Active processes (from the linked list) |
| `windows.psscan` | Carve processes from memory — finds **hidden/terminated** ones |
| `windows.pstree` | Parent-child tree (spot `winword.exe → cmd.exe`) |
| `windows.cmdline` | Command line each process was started with |
| `windows.dlllist` | DLLs loaded per process |
| `windows.malfind` | **Find injected code / shellcode** (RWX regions) |
| `windows.ldrmodules` | Detect unlinked (hidden) DLLs |

## Network & files

| Plugin (v3) | Description |
| --- | --- |
| `windows.netscan` | Network connections & listening ports (find C2) |
| `windows.netstat` | Active connections |
| `windows.filescan` | Carve file objects from memory |
| `windows.dumpfiles --virtaddr <addr>` | Extract a file from memory |
| `windows.handles` | Open handles for a process |

## Credentials & registry

| Plugin (v3) | Description |
| --- | --- |
| `windows.hashdump` | Dump local NTLM hashes (→ crack with [hashcat](../hashcat)) |
| `windows.lsadump` | LSA secrets |
| `windows.cachedump` | Cached domain creds |
| `windows.registry.hivelist` | Registry hives in memory |
| `windows.registry.printkey --key "..."` | Read a registry key |

## Extracting evidence

| Plugin (v3) | Description |
| --- | --- |
| `windows.dumpfiles --pid <pid>` | Dump files for a process |
| `windows.memmap --pid <pid> --dump` | Dump a process's memory |
| `windows.procdump --pid <pid>` | Dump a process executable (→ scan on [VirusTotal](../RESOURCES.md)) |

---

## 🧭 Typical workflow

```bash
# 1. What was running?
vol -f mem.dmp windows.pstree

# 2. Anything suspicious spawned (e.g. office → shell, odd parent)?
vol -f mem.dmp windows.cmdline

# 3. Injected code / malware in memory?
vol -f mem.dmp windows.malfind

# 4. Where was it calling out to?
vol -f mem.dmp windows.netscan

# 5. Pull creds, then dump the suspect binary for analysis
vol -f mem.dmp windows.hashdump
vol -f mem.dmp windows.procdump --pid 1337 --dump
```

> 🔗 Crack dumped hashes with [hashcat](../hashcat)/[john](../john). Detonate/scan extracted binaries on the online tools in [RESOURCES.md](../RESOURCES.md). Correlate with endpoint logs in [sysmon](../sysmon).
