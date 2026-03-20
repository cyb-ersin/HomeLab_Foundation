# Lab Topology

> **Lab:** 03 — Firewall & Segmentation  
> **Date:** 2026-03-20

---

## Physical Setup

```
+-----------------------------------------------------+
|              iPhone Hotspot (AP)                    |
|              172.20.10.0/28                         |
|              Gateway: 172.20.10.1                   |
+----------+-----------------+-----------+------------+
           |                 |           |
    +------+------+   +------+----+   +--+-----------+
    | lab-defender|   |  MacBook  |   |  Kali Linux  |
    | 172.20.10.2 |   |172.20.10.3|   | 172.20.10.4  |
    | Ubuntu VM   |   |           |   | ThinkPad X250|
    | (VirtualBox)|   |  Host     |   | (baremetal)  |
    |             |   |           |   |              |
    |  DEFENDER   |   |   MGMT    |   |  ATTACKER    |
    +-------------+   +-----------+   +--------------+
```

---

## Logical Zone Design

```
+-----------------------------------------------------+
|  UNTRUSTED ZONE  172.20.10.0/28                     |
|                                                     |
|   Kali    172.20.10.4  ---+                         |
|   MacBook 172.20.10.3  ---+---> ufw FIREWALL        |
|                           |                         |
+---------------------------+-------------------------+
                            |
               +------------+-----------+
               |       ufw RULES        |
               |  default: deny         |
               |  [1] Kali -> :22  OK   |
               |  [2] MacBook -> :22 X  |
               +------------+-----------+
                            |
+---------------------------+-------------------------+
|  TRUSTED ZONE  10.10.10.0/24                        |
|                                                     |
|   dummy0  10.10.10.1  (simulated internal zone)     |
|   Policy: ALLOW ALL                                 |
|                                                     |
+-----------------------------------------------------+
```

---

## Device Inventory

| Hostname | IP | OS | Role | Interface |
|---|---|---|---|---|
| lab-defender | 172.20.10.2 | Ubuntu (VirtualBox) | Defender / Firewall host | enp0s3 (Bridged) |
| MacBook-Pro-von-Ersin | 172.20.10.3 | macOS | Management / Documentation | en0 (WLAN) |
| kali | 172.20.10.4 | Kali Linux 6.x | Attacker | wlan0 |
| dummy0 | 10.10.10.1 | — | Trusted zone simulation | dummy0 (virtual) |

---

## Network Details

| Parameter | Value |
|---|---|
| Subnet | 172.20.10.0/28 |
| Subnet Mask | 255.255.255.240 |
| Usable Hosts | 14 |
| Broadcast | 172.20.10.15 |
| Gateway (AP) | 172.20.10.1 |

> **/28 subnet** — iPhone hotspot uses a /28 by default, providing only 14 usable addresses. Sufficient for a 3-device lab environment.

---

## VirtualBox Network Configuration

| Setting | Value |
|---|---|
| Adapter | Adapter 1 |
| Attached to | Bridged Adapter (Netzwerkbrücke) |
| Interface | en0: WLAN |
| Effect | Ubuntu VM receives its own IP from iPhone DHCP — visible to all devices on the same hotspot |

> **Why Bridged?** NAT mode would give the VM a private address behind the MacBook — invisible to Kali. Bridged mode places the VM directly on the hotspot network, enabling real cross-device traffic for firewall testing.
