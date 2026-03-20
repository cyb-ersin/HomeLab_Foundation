# 🏗️ Zone Design

![Zones](https://img.shields.io/badge/Zones-2-blue)
![Model](https://img.shields.io/badge/Model-Whitelist-green)
![Default](https://img.shields.io/badge/Default%20Policy-DENY-red)

> **Lab:** 03 — Firewall & Segmentation | **Date:** 2026-03-20

---

## 🎯 Design Principle

> **Default deny. Explicit allow.**

Every inbound connection is blocked unless a specific rule permits it. This is the foundational principle of host-based firewalling and zero-trust network design.

---

## 🔴 UNTRUSTED ZONE

| Parameter | Value |
|---|---|
| Subnet | `172.20.10.0/28` |
| Devices | Kali Linux · MacBook Pro · iPhone Hotspot |
| Trust Level | Low — external / adversarial |
| Default Policy | `deny (incoming)` |
| Allowed | Port `22/tcp` from `172.20.10.4` (Kali) only |

**Rationale:** Any device on this subnet is treated as potentially hostile. Only the explicitly whitelisted IP (Kali, used for controlled testing) can reach the defender on a single port.

---

## 🟢 TRUSTED ZONE

| Parameter | Value |
|---|---|
| Subnet | `10.10.10.0/24` |
| Interface | `dummy0` (simulated) |
| Trust Level | High — internal / administrative |
| Default Policy | `allow (incoming)` |
| Allowed | All ports, all protocols |

**Rationale:** Represents an internal administrative network — equivalent to a management VLAN in enterprise environments. Full access is granted because the zone itself is trusted by design.

---

## 📊 Zone Comparison

| Property | 🔴 UNTRUSTED | 🟢 TRUSTED |
|---|---|---|
| Subnet | `172.20.10.0/28` | `10.10.10.0/24` |
| Default | DENY | ALLOW |
| SSH Access | Kali only | All |
| Web Access | ❌ Blocked | ✅ Allowed |
| ICMP (ping) | ✅ Allowed (ufw default) | ✅ Allowed |
| Interface | `enp0s3` (physical) | `dummy0` (virtual) |

---

## 🔑 Why Two Zones?

In a real SOC environment, network segmentation serves three purposes:

**1. Blast radius reduction**
If an attacker compromises a device in the untrusted zone, they cannot freely move to trusted systems. The firewall forces them to escalate explicitly.

**2. Visibility**
Zone boundaries create natural chokepoints for logging. All traffic crossing a zone boundary is inspectable — this is where IDS sensors are placed.

**3. Policy clarity**
"Is this traffic allowed?" becomes a simple zone membership question: is the source trusted or untrusted? This maps directly to how SOC analysts triage alerts.

---

## 🏢 Real-World Equivalent

| This Lab | Enterprise Equivalent |
|---|---|
| UNTRUSTED zone | Internet-facing DMZ |
| TRUSTED zone | Internal management VLAN |
| dummy0 interface | Layer 3 switch / VLAN interface |
| ufw rules | Perimeter firewall ACL |
| Kali whitelist | VPN-only admin access |
