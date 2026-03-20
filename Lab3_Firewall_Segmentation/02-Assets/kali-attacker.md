# ⚔️ kali-attacker

![Role](https://img.shields.io/badge/Role-Attacker-red)
![OS](https://img.shields.io/badge/OS-Kali%20Linux-purple)
![Platform](https://img.shields.io/badge/Platform-Baremetal-lightgrey)
![IP](https://img.shields.io/badge/IP-172.20.10.4-red)

---

## System Info

| Parameter | Value |
|---|---|
| Hostname | `kali` |
| IP Address | `172.20.10.4` |
| OS | Kali Linux (baremetal) |
| Hardware | Lenovo ThinkPad X250 |
| Interface | `wlan0` |

---

## Network Interface

```
wlan0: 172.20.10.4/28
       brd 172.20.10.15
       scope global dynamic noprefixroute
```

---

## Attack Tools Used

| Tool | Version | Purpose |
|---|---|---|
| `nmap` | — | Port scanning and service detection |
| `hydra` | — | SSH brute force |
| `rockyou.txt` | 14M entries | Password wordlist |

---

## Attack Commands

### Phase 1 — Reconnaissance

```bash
# Initial scan — no rules active
nmap -sS -p 1-1000 172.20.10.2

# Fast scan with timeout
nmap -sV --open -Pn --max-retries 1 --host-timeout 30s 172.20.10.2

# After allow rule — service detection
nmap -sV --open -Pn 172.20.10.2
```

| Flag | Meaning |
|---|---|
| `-sS` | SYN scan — stealth, no full handshake |
| `-p 1-1000` | Scan first 1000 ports |
| `-sV` | Detect service version |
| `--open` | Show only open ports |
| `-Pn` | Skip ping, scan directly |
| `--max-retries 1` | One attempt per port |
| `--host-timeout 30s` | Abort after 30 seconds |

### Phase 2 — SSH Access Test

```bash
# Successful (Kali is whitelisted)
ssh vboxuser@172.20.10.2

# Blocked (MacBook is not whitelisted)
# ssh vboxuser@172.20.10.2  ← run from MacBook
```

### Phase 3 — Brute Force

```bash
# Prepare wordlist
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Run brute force
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.20.10.2 -t 4
```

| Flag | Meaning |
|---|---|
| `-l root` | Target username |
| `-P rockyou.txt` | Password wordlist |
| `ssh://` | Target protocol |
| `-t 4` | 4 parallel threads |

---

## Observed Behaviors

| Test | Expected | Actual |
|---|---|---|
| nmap (no rules) | filtered | ✅ All ports filtered |
| nmap (after allow 22) | port 22 open | ✅ OpenSSH detected |
| SSH from Kali | connected | ✅ Login prompt |
| SSH from MacBook | blocked | ✅ Connection refused |
| Hydra (no limit) | attempts pass | ✅ Failed password logs |
| Hydra (with limit) | rate limited | ✅ UFW BLOCK on DPT=22 |
