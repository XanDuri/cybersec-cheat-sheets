# 🟠 Burp Suite Cheat Sheet

**Burp Suite** is the go-to web proxy for intercepting, inspecting, and tampering with HTTP(S). The free **Community** edition covers most TryHackMe/early-pentest needs; **Intruder** is rate-limited there.

> 💡 **Setup:** point your browser proxy at `127.0.0.1:8080`, then install Burp's CA cert (browse to `http://burp` → *CA Certificate*) so HTTPS doesn't error. The **FoxyProxy** extension makes toggling easy.

---

## The core tools

| Tool | What it's for |
| --- | --- |
| **Proxy** | Intercept & modify requests/responses in flight |
| **Repeater** | Re-send a single request over and over, tweaking it (`Ctrl+R` to send a request here) |
| **Intruder** | Automated request fuzzing/brute-force (positions + payloads) |
| **Decoder** | Encode/decode (URL, Base64, hex, HTML) |
| **Comparer** | Diff two responses byte-by-byte |
| **Target → Site map** | Tree of everything you've browsed |
| **Sequencer** | Analyse randomness of tokens (session IDs) |

## Proxy / Intercept

| Action | How | Description |
| --- | --- | --- |
| Toggle intercept | Proxy → Intercept is on/off | Hold or release requests |
| Forward | `Ctrl+F` | Send the held request on |
| Drop | — | Discard the held request |
| Send to Repeater | right-click → `Ctrl+R` | Iterate on one request |
| Send to Intruder | right-click → `Ctrl+I` | Set up automated attack |
| HTTP history | Proxy → HTTP history | Everything that passed through (search/filter here) |

## Intruder attack types

| Type | Use case |
| --- | --- |
| **Sniper** | One payload set, one position at a time — fuzz a single field |
| **Battering ram** | Same payload into *all* positions at once |
| **Pitchfork** | Parallel sets (e.g. user[1]+pass[1], user[2]+pass[2]) |
| **Cluster bomb** | Every combination — classic user × password brute-force |

> 🔎 Sort Intruder results by **Status** and **Length** — the odd-one-out response is usually the hit (valid login, found file…).

## Match & filter tips

| Goal | Where | Tip |
| --- | --- | --- |
| Find a string in responses | filter bar / Search | e.g. hunt for `flag{`, `error`, `welcome` |
| Spot the valid login | Intruder columns | a different response **Length** = success |
| Bypass client-side checks | intercept the request | edit the value the JS "validated" |
| Tamper with hidden fields | Repeater | change `price`, `role=user`→`admin`, `isAdmin=false`→`true` |

---

## 🧭 Typical workflow

```text
1. Browse the app through Burp → builds the Site map automatically
2. Find an interesting request → send to Repeater (Ctrl+R)
3. Probe it: change params, methods, headers → watch the response
4. Found an injectable/brute-forceable field? → send to Intruder (Ctrl+I)
5. Set payload positions §like-this§, load a wordlist, run, sort by Length
```

> 🔗 Burp pairs well with [sqlmap](../sqlmap) (`-r request.txt` from a saved Burp request) and wordlists from [seclists](../seclists).
