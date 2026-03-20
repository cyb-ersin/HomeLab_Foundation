# 📚 Lessons Learned

![Lab](https://img.shields.io/badge/Lab-03%20Firewall%20%26%20Segmentation-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Date](https://img.shields.io/badge/Date-2026--03--20-lightgrey)

---

## 🔑 Core Takeaways

### 1. Open port ≠ accessible port

`ss -tlnp` showed SSH listening on `0.0.0.0:22` — yet Kali could not connect. The firewall sits between the network and the service. A port can be fully open internally while completely invisible externally. These are two different realities that `ss` and `nmap` reveal independently.

### 2. Default deny is the foundation

With zero allow rules, nmap returned `filtered` on all 1000 ports. The host was invisible. Every exposed port is a conscious, deliberate decision — not a default. This is the correct mental model for designing any secure system.

### 3. ufw is iptables

`sudo iptables -L -n -v` revealed that every ufw rule translates directly to an iptables chain entry. ufw is a human-friendly abstraction — the kernel enforces the same netfilter rules underneath. Understanding both layers matters when troubleshooting or working in environments where ufw is not available.

### 4. Firewall logs tell two stories

`[UFW BLOCK]` and `[UFW AUDIT]` are not interchangeable. BLOCK means the packet was dropped. AUDIT means it was logged as it passed through a rule. Reading both together reveals what was attempted, what was allowed, and what was denied — the full picture.

### 5. Brute force is identifiable without IDS

No Snort, no Suricata, no SIEM. Two commands were sufficient to confirm an automated brute force attack:
- `grep "DPT=22"` on ufw.log → rotating SPT on fixed DPT
- `journalctl` → 4x `Failed password` per second

Pattern recognition in raw logs is a fundamental SOC skill. Tools automate it — but analysts must understand what the tools are looking for.

### 6. Rate limiting degrades, not stops

`ufw limit` forced hydra to slow down — SPT stopped rotating, new sessions were blocked. But a patient attacker can spread attempts over time and evade rate limits entirely. The next layer is `fail2ban` (persistent IP banning) or an IDS with automated response.

### 7. Zone segmentation is a mindset

The trusted/untrusted zone distinction is not just a firewall concept. It is the mental model that underlies every access control decision in security operations. Before adding any rule, the first question is: which zone does this traffic come from, and what is the trust level of that zone?

---

## 🔗 Connection to Next Lab

This lab established the defensive baseline. The gap that remains: **real-time detection.**

Rate limiting reacted to brute force — but only after it started. There was no alert, no automatic block based on behavioral analysis, no correlation between the port scan and the subsequent brute force attempt.

**Lab 04 — IDS & SIEM** will address this directly:
- Snort/Suricata for real-time signature-based detection
- Log aggregation and correlation
- Alert generation on port scan and brute force patterns
- The same attacks performed in this lab will trigger automated alerts

> The firewall is the wall. The IDS is the alarm system. Both are necessary.

---

## 📊 Skills Developed

| Skill | Context |
|---|---|
| Host-based firewall configuration | ufw enable, rules, rate limiting |
| Kernel-level rule inspection | iptables -L -n -v |
| Network zone design | Trusted / untrusted segmentation |
| Port scan detection | ufw log pattern analysis |
| Brute force detection | ufw log + journalctl correlation |
| Log filtering | grep, awk, tail -f |
| Virtual interface management | dummy0, ip link, ip addr |

---

## 🏢 SOC Relevance

| Lab Activity | Real-World SOC Equivalent |
|---|---|
| Reading ufw BLOCK logs | Analyzing firewall deny logs in SIEM |
| Identifying port scan signature | L1 alert triage — recon detection |
| Identifying brute force signature | L1 alert triage — T1110 detection |
| Correlating ufw + journalctl | Multi-source log correlation |
| Applying rate limiting | Tier-1 response — containment action |
| Zone-based rules | ACL review and policy enforcement |
