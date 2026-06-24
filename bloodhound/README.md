# 🐶 BloodHound Cheat Sheet

**BloodHound** maps Active Directory as a graph and reveals **attack paths** — "who can ultimately become Domain Admin?". You **collect** data with an ingestor, then **import** the JSON and run queries in the GUI.

> 💡 Two flavours: **legacy BloodHound** (Neo4j + `SharpHound`) and **BloodHound CE** (Community Edition, Docker-based). Queries/concepts are the same; this sheet covers both collectors.

---

## 1. Collect the data (ingestors)

| Collector | Example | Notes |
| --- | --- | --- |
| **SharpHound.exe** | `SharpHound.exe -c All` | Run on a domain-joined Windows host |
| **SharpHound.ps1** | `Invoke-BloodHound -CollectionMethod All` | PowerShell version |
| **bloodhound-python** | `bloodhound-python -d corp.local -u user -p pass -ns 10.10.10.1 -c All` | Remote, from Linux — no foothold on Windows needed |
| via [netexec](../netexec) | `nxc ldap 10.10.10.1 -u user -p pass --bloodhound -c all` | One-liner collection |

| Collection method (`-c`) | Gathers |
| --- | --- |
| `All` | Everything (default go-to) |
| `Session` | Logged-on users (find where creds are useful) |
| `LoggedOn` | Active sessions (needs admin) |
| `ACL` | Permissions/ACLs (the juicy privesc edges) |
| `DCOnly` | Only queries the DC (stealthier, no host touches) |

```bash
# Remote collection from your Kali box (most common on TryHackMe/HTB)
bloodhound-python -d corp.local -u bob -p 'Password1' -ns 10.10.10.1 -c All --zip
```

## 2. Import & explore

| Step | How |
| --- | --- |
| Start backend | legacy: `neo4j console` then run the BloodHound app · CE: `docker compose up` |
| Import | Drag the collected `.json`/`.zip` onto the GUI |
| Set targets | Right-click a node → **Mark as Owned** (what you control) / **Mark as High Value** |
| Find a path | Use the **path** search (from "Owned" → "Domain Admins") |

## 3. Pre-built queries (Analysis tab)

| Query | Reveals |
| --- | --- |
| Find all Domain Admins | The crown jewels |
| Shortest Paths to Domain Admins | Your route to DA |
| Shortest Paths from Owned Principals | Where *your* access leads |
| Kerberoastable users | Accounts with SPNs → [impacket](../impacket) GetUserSPNs |
| AS-REP Roastable users | No-preauth users → GetNPUsers |
| Find Computers with Unconstrained Delegation | Powerful delegation abuse |

## Common edges (what they mean)

| Edge | Abuse |
| --- | --- |
| `MemberOf` | Group membership inherits rights |
| `AdminTo` | Local admin on a machine |
| `GenericAll` / `GenericWrite` | Full/partial control over an object (reset passwords, etc.) |
| `WriteDacl` / `Owns` | Can grant yourself rights over the object |
| `ForceChangePassword` | Reset a user's password without knowing the old one |
| `CanRDP` / `CanPSRemote` | Interactive access ([evil-winrm](../evil-winrm)) |

---

## 🧭 Typical workflow

```text
1. Get any valid domain creds (spray with ../netexec, roast with ../kerbrute)
2. Collect:  bloodhound-python -c All  (or nxc --bloodhound)
3. Import the zip → Mark your user as Owned
4. Run "Shortest Paths from Owned Principals"
5. Walk each edge with the right tool (../impacket, ../evil-winrm, ../netexec)
```

> 🔗 Find users to target with [kerbrute](../kerbrute), grab ticket hashes with [impacket](../impacket), crack them with [hashcat](../hashcat).
