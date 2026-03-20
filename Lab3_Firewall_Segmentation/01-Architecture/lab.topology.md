# 🗺️ Lab Topology

> **Lab:** 03 — Firewall & Segmentation  
> **Date:** 2026-03-20  
> **Network:** iPhone Hotspot · `172.20.10.0/28`

---

## 🌐 Physical Network Diagram

```mermaid
graph TD
    AP["📱 iPhone Hotspot (AP)
    172.20.10.0/28
    GW: 172.20.10.1"]

    DEF["🛡️ lab-defender
    172.20.10.2
    Ubuntu VM · VirtualBox
    ── DEFENDER ──"]

    MGMT["💻 MacBook Pro
    172.20.10.3
    macOS
    ── MANAGEMENT ──"]

    ATK["⚔️ Kali Linux
    172.20.10.4
    ThinkPad X250 · baremetal
    ── ATTACKER ──"]

    AP --> DEF
    AP --> MGMT
    AP --> ATK

    style AP fill:#1e3a5f,color:#ffffff,stroke:#4a90d9
    style DEF fill:#1a4731,color:#ffffff,stroke:#2ecc71
    style MGMT fill:#2d2d2d,color:#ffffff,stroke:#888888
    style ATK fill:#4a1010,color:#ffffff,stroke:#e74c3c
```

---

## 🔥 Firewall Zone Architecture

```mermaid
graph LR
    subgraph UNTRUSTED["🔴 UNTRUSTED ZONE · 172.20.10.0/28"]
        ATK["⚔️ Kali
        172.20.10.4"]
        MGMT["💻 MacBook
        172.20.10.3"]
    end

    subgraph FW["🔥 ufw FIREWALL · lab-defender"]
        R1["✅ Rule 1
        Kali → :22/tcp
        ALLOW"]
        R2["❌ Rule 2
        MacBook → :22/tcp
        BLOCK"]
        DEF_POLICY["⚠️ Default Policy
        deny incoming
        allow outgoing"]
    end

    subgraph TRUSTED["🟢 TRUSTED ZONE · 10.10.10.0/24"]
        DMZ["🖥️ dummy0
        10.10.10.1
        Simulated Internal Zone
        ALLOW ALL"]
    end

    ATK -->|"SSH :22"| R1
    MGMT -->|"SSH :22"| R2
    R1 --> DMZ
    DEF_POLICY -.->|"blocks all others"| R2

    style UNTRUSTED fill:#3d1515,color:#ffffff,stroke:#e74c3c
    style FW fill:#1e2a3d,color:#ffffff,stroke:#4a90d9
    style TRUSTED fill:#1a3d2b,color:#ffffff,stroke:#2ecc71
    style R1 fill:#1a4731,color:#ffffff,stroke:#2ecc71
    style R2 fill:#4a1010,color:#ffffff,stroke:#e74c3c
    style DEF_POLICY fill:#2d2000,color:#ffffff,stroke:#f39c12
    style DMZ fill:#1a3d2b,color:#ffffff,stroke:#2ecc71
```

---

## 📋 Device Inventory

| # | Hostname | IP Address | OS | Role |
|---|---|---|---|---|
| 🛡️ | `lab-defender` | `172.20.10.2` | Ubuntu (VirtualBox) | Defender / Firewall host |
| 💻 | `MacBook-Pro-von-Ersin` | `172.20.10.3` | macOS | Management / Documentation |
| ⚔️ | `kali` | `172.20.10.4` | Kali Linux 6.x | Attacker |
| 🔵 | `dummy0` | `10.10.10.1` | — | Trusted zone (virtual) |

---

## 📡 Network Parameters

| Parameter | Value |
|---|---|
| Subnet | `172.20.10.0/28` |
| Subnet Mask | `255.255.255.240` |
| Usable Hosts | 14 |
| Broadcast | `172.20.10.15` |
| Gateway | `172.20.10.1` |

> **Why /28?** iPhone hotspot assigns a `/28` by default — only 14 usable addresses. Sufficient for this 3-device lab.

---

## ⚙️ VirtualBox Bridged Mode

```mermaid
graph LR
    subgraph NAT["❌ NAT Mode (not used)"]
        VM_NAT["Ubuntu VM
        10.0.2.x
        (hidden)"]
        MAC_NAT["MacBook
        172.20.10.3"]
        VM_NAT -.->|invisible to Kali| MAC_NAT
    end

    subgraph BRIDGE["✅ Bridged Mode (used)"]
        VM_BR["Ubuntu VM
        172.20.10.2
        (visible)"]
        MAC_BR["MacBook
        172.20.10.3"]
        KALI_BR["Kali
        172.20.10.4"]
        VM_BR <-->|direct traffic| MAC_BR
        VM_BR <-->|direct traffic| KALI_BR
    end

    style NAT fill:#3d1515,color:#ffffff,stroke:#e74c3c
    style BRIDGE fill:#1a3d2b,color:#ffffff,stroke:#2ecc71
```

> Bridged Adapter places the VM directly on the hotspot network. The VM gets its own DHCP lease — fully reachable by Kali for real firewall testing.
