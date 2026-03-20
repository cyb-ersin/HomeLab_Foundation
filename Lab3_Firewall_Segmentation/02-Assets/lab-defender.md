# 🛡️ lab-defender

![Role](https://img.shields.io/badge/Role-Defender-green)
![OS](https://img.shields.io/badge/OS-Ubuntu%20Linux-orange)
![Platform](https://img.shields.io/badge/Platform-VirtualBox-lightgrey)
![IP](https://img.shields.io/badge/IP-172.20.10.2-blue)

---

## System Info

| Parameter | Value |
|---|---|
| Hostname | `lab-defender` |
| IP Address | `172.20.10.2` |
| OS | Ubuntu Linux (VirtualBox VM) |
| Host Machine | MacBook Pro |
| VirtualBox Mode | Bridged Adapter — en0: WLAN |
| User | `vboxuser` |

---

## Network Interfaces

| Interface | IP | Type |
|---|---|---|
| `enp0s3` | `172.20.10.2/28` | Bridged — hotspot network |
| `dummy0` | `10.10.10.1/24` | Virtual — trusted zone simulation |
| `lo` | `127.0.0.1` | Loopback |

---

## Listening Services

```
PORT   ADDRESS         SERVICE
22     0.0.0.0:22      OpenSSH
53     127.0.0.53:53   systemd-resolved (localhost only)
631    127.0.0.1:631   CUPS print service (localhost only)
```

> Only port 22 is exposed on all interfaces. Ports 53 and 631 are localhost-only — not reachable from outside.

---

## Firewall Configuration

### Status
```
Status:  active
Logging: on (medium)
Default: deny (incoming) · allow (outgoing) · disabled (routed)
```

### Rules
```
[ 1] 22/tcp    ALLOW IN    172.20.10.4      ← Kali only
[ 2] Anywhere  ALLOW IN    10.10.10.0/24   ← Trusted zone
```

### Rate Limiting
```
ufw limit proto tcp from any to any port 22
→ Auto-block after 6 connections in 30 seconds
```

---

## Key Commands Used

| Command | Purpose |
|---|---|
| `sudo ufw enable` | Activate firewall |
| `sudo ufw status verbose` | Show rules and default policy |
| `sudo ufw status numbered` | Show rules with index numbers |
| `sudo ufw allow proto tcp from 172.20.10.4 to any port 22` | Whitelist Kali for SSH |
| `sudo ufw allow from 10.10.10.0/24 to any` | Allow trusted zone |
| `sudo ufw limit proto tcp from any to any port 22` | Enable rate limiting |
| `sudo ufw delete 1` | Delete rule by number |
| `sudo ufw logging medium` | Set log verbosity |
| `sudo iptables -L -n -v` | Inspect underlying iptables rules |
| `ss -tlnp` | List listening ports |
| `sudo tail -f /var/log/ufw.log` | Monitor firewall logs in real time |
| `sudo journalctl _SYSTEMD_UNIT=ssh.service` | Read SSH auth logs |
| `sudo ip link add dummy0 type dummy` | Create virtual interface |
| `sudo ip addr add 10.10.10.1/24 dev dummy0` | Assign trusted zone IP |
| `sudo ip link set dummy0 up` | Bring virtual interface up |
