# 🔍 Evidence — Port Scan

![Type](https://img.shields.io/badge/Type-Network%20Reconnaissance-orange)
![Tool](https://img.shields.io/badge/Tool-nmap-lightgrey)
![Source](https://img.shields.io/badge/Source-172.20.10.4-red)
![Target](https://img.shields.io/badge/Target-172.20.10.2-green)

> **Date:** 2026-03-20 · **Time:** ~14:49 UTC

---

## 📸 Screenshots

> Add screenshots here:
> - `nmap-filtered.png` — nmap output before allow rule (all ports filtered)
> - `nmap-open.png` — nmap output after allow rule (port 22 open)
> - `ufw-block-portscan.png` — ufw.log showing BLOCK entries during scan

---

## 📋 nmap Output — Before Rule

```
Starting Nmap scan against 172.20.10.2
All 1000 scanned ports on 172.20.10.2 are in filtered state
```

**Interpretation:** ufw default DROP policy — all packets silently discarded. Host appears offline to external scanners.

---

## 📋 nmap Output — After Rule

```
PORT   STATE  SERVICE  VERSION
22/tcp open   ssh      OpenSSH 9.x
```

**Interpretation:** Single `allow 22/tcp` rule immediately changed port 22 from `filtered` to `open`. Firewall controls surface exposure directly.

---

## 📋 ufw Log — Port Scan Signature

Raw log entries captured during nmap SYN scan:

```
2026-03-20T14:49:24.202833+00:00 lab-defender kernel: [UFW BLOCK] IN=enp0s3 OUT= SRC=172.20.10.4 DST=172.20.10.2 PROTO=TCP SPT=56395 DPT=135 SYN
2026-03-20T14:49:24.202838+00:00 lab-defender kernel: [UFW AUDIT] IN=enp0s3 OUT= SRC=172.20.10.4 DST=172.20.10.2 PROTO=TCP SPT=56395 DPT=995 SYN
2026-03-20T14:49:24.202986+00:00 lab-defender kernel: [UFW BLOCK] IN=enp0s3 OUT= SRC=172.20.10.4 DST=172.20.10.2 PROTO=TCP SPT=56395 DPT=995 SYN
2026-03-20T14:49:24.202990+00:00 lab-defender kernel: [UFW AUDIT] IN=enp0s3 OUT= SRC=172.20.10.4 DST=172.20.10.2 PROTO=TCP SPT=56395 DPT=23  SYN
2026-03-20T14:49:24.202992+00:00 lab-defender kernel: [UFW BLOCK] IN=enp0s3 OUT= SRC=172.20.10.4 DST=172.20.10.2 PROTO=TCP SPT=56395 DPT=23  SYN
```

---

## 🔎 Log Field Analysis

| Field | Value | Meaning |
|---|---|---|
| `[UFW BLOCK]` | — | Packet dropped — no rule matched |
| `[UFW AUDIT]` | — | Packet seen — logged before decision |
| `IN=enp0s3` | enp0s3 | Arrived on external interface |
| `SRC` | `172.20.10.4` | Attacker IP (Kali) |
| `DST` | `172.20.10.2` | Defender IP (lab-defender) |
| `PROTO` | TCP | Protocol |
| `SPT` | `56395` | Attacker source port (fixed during scan) |
| `DPT` | `135, 995, 23...` | Target port — changes each packet |
| `SYN` | — | First packet of TCP handshake |

---

## 🚨 Detection Indicators

| Indicator | Description |
|---|---|
| Same `SRC` | Single source — not distributed |
| Rapidly changing `DPT` | Sequential port sweep |
| Fixed `SPT` | nmap reuses source port per scan batch |
| All `SYN` flags | Half-open scan — no full handshake |
| Timestamp burst | Hundreds of packets within milliseconds |

**Verdict:** Automated port scan — nmap SYN sweep signature confirmed.
