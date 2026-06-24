# ⚡ ffuf Cheat Sheet

**ffuf** (Fuzz Faster U Fool) is a fast web fuzzer. You place the keyword `FUZZ` wherever you want to inject wordlist entries — URLs, headers, POST data, parameters.

```bash
# General syntax
ffuf -u [URL with FUZZ] -w [wordlist] [options]
```

> 💡 The `FUZZ` keyword is the injection point — it can go anywhere in the request.

---

## Core Options

| Flag | Example | Description |
| --- | --- | --- |
| `-u` | `-u http://10.10.10.1/FUZZ` | Target URL with the `FUZZ` marker |
| `-w` | `-w wordlist.txt` | Wordlist |
| `-e` | `-e .php,.html,.txt` | Append extensions to each entry |
| `-H` | `-H "Host: FUZZ.example.com"` | Custom header (used for vhost fuzzing) |
| `-X` | `-X POST` | HTTP method |
| `-d` | `-d "user=FUZZ&pass=x"` | POST body data |
| `-t` | `-t 100` | Number of threads |
| `-o` | `-o out.json` | Output file |
| `-c` | `-c` | Colourised output |

## Matchers & Filters

| Flag | Example | Description |
| --- | --- | --- |
| `-mc` | `-mc 200,301` | **Match** these status codes |
| `-ms` | `-ms 1234` | Match by response size |
| `-fc` | `-fc 404,403` | **Filter out** these status codes |
| `-fs` | `-fs 0` | Filter out responses of this size |
| `-fw` | `-fw 12` | Filter out by word count |
| `-fl` | `-fl 5` | Filter out by line count |

---

## 🧭 Common commands

```bash
# Directory fuzzing
ffuf -u http://10.10.10.1/FUZZ -w /usr/share/wordlists/dirb/common.txt

# With extensions, filtering out 404s
ffuf -u http://10.10.10.1/FUZZ -w list.txt -e .php,.html -fc 404

# Virtual host fuzzing (filter the default-size response)
ffuf -u http://10.10.10.1 -H "Host: FUZZ.example.com" -w subs.txt -fs 1234

# POST parameter / login fuzzing
ffuf -u http://10.10.10.1/login -X POST -d "username=admin&password=FUZZ" \
  -H "Content-Type: application/x-www-form-urlencoded" -w passwords.txt -fc 401

# GET parameter discovery
ffuf -u "http://10.10.10.1/page?FUZZ=value" -w params.txt -fs 0
```

> 💡 **Gotchas:**
> - Drowning in noise? Run a baseline first, see the size/word count of the "not found" page, then `-fs`/`-fw` it out — or let `-ac` (auto-calibration) figure it out.
> - For **vhost** fuzzing the server returns 200 for everything, so filter by **size** (`-fs`), not status code.
> - `-recursion -recursion-depth 2` walks into discovered directories automatically.
> - Wordlists live in [seclists](../seclists) (`Discovery/Web-Content/`).

> 🔗 Confirmed an injectable parameter? → [sqlmap](../sqlmap). Want to tamper requests by hand? → [burpsuite](../burpsuite).
