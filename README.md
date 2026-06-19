# 🛡️ SOC Detection Lab — ELK Stack + Fleet + C2 Simulation

A self-hosted, full-lifecycle detection lab simulating a small organization's SOC environment — centralized log management, agent-based endpoint monitoring, alert-to-ticket workflows, and adversary simulation using a real C2 framework.

> **Disclaimer:** This lab is hosted entirely within my own private VPC and isolated subnet. All offensive activity (Mythic C2, attacker laptop) targets only systems I own within this environment. No external systems were accessed or tested.

---

## 🎯 Lab Objective

Build a realistic, end-to-end SOC environment that mirrors how real organizations detect and respond to threats:

1. **Centralize logs** from multiple endpoints into a SIEM
2. **Manage agents** across mixed operating systems from a single control plane
3. **Simulate real adversary behavior** using an actual command-and-control framework (not just scripted attacks)
4. **Generate and triage alerts** through a ticketing workflow — not just raw log searches

---

## 🖥️ Architecture

```
                         ┌─────────────────┐
                         │     Internet      │
                         └─────────┬─────────┘
                                   │
                  ┌────────────────┼────────────────┐
                  │                                  │
        ┌──────────────────┐              ┌──────────────────┐
        │ SOC Analyst Laptop │              │ Attacker Laptop  │
        │  (connects via      │              │   (Kali Linux)   │
        │  Elastic/Kibana)    │              └─────────┬────────┘
        └──────────────────┘                            │
                                              ┌──────────────────┐
                                              │   C2 Server      │
                                              │   (Mythic)        │
                                              └──────────────────┘

┌──────────────────────────── VULTR — VPC ─────────────────────────────┐
│  Private Network: 172.31.0.0/24                                      │
│  IP Range: 172.31.0.1 – 254 | Subnet Mask: 255.255.255.0             │
│                                                                        │
│        ┌───────────────────┐         ┌───────────────────┐           │
│        │  Elastic + Kibana  │ ◄─────► │  OS Ticket Server  │           │
│        │      (SIEM)        │ Alerts/  │   (Ticketing)      │           │
│        └─────────┬─────────┘ Tickets  └───────────────────┘           │
│                  │                                                     │
│         Manage Agents / Forward Logs                                  │
│                  │                                                     │
│    ┌─────────────┼─────────────┐                                     │
│    │             │             │                                      │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐                               │
│ │ Windows  │ │  Fleet   │ │ Ubuntu   │                               │
│ │ Server   │◄┤  Server  ├►│ Server   │                               │
│ │ (RDP)    │ │ (Managed)│ │ (SSH)    │                               │
│ └──────────┘ └──────────┘ └──────────┘                               │
└────────────────────────────────────────────────────────────────────┘
```

---

## 🧩 Components

| Component | Purpose | Notes |
|---|---|---|
| **Elastic + Kibana** | SIEM — log aggregation, search, and visualization | Central detection engine for the lab |
| **Fleet Server** | Centralized agent management | Deploys and manages Elastic Agents on all endpoints |
| **OS Ticket Server** | Alert-to-ticket workflow | Simulates real SOC triage and case management |
| **Windows Server** (RDP enabled) | Target endpoint | Forwards logs to Elastic via agent |
| **Ubuntu Server** (SSH enabled) | Target endpoint | Forwards logs to Elastic via agent |
| **Attacker Laptop** (Kali Linux) | Adversary simulation | Initiates attacks against the environment |
| **Mythic C2 Server** | Command & control framework | Real-world C2 simulation, not just scripted payloads |
| **SOC Analyst Laptop** | Defender access point | Connects to Elastic/Kibana via web GUI over the internet |

---

## ⚙️ Build Log

*(To be filled in as the lab is built — documenting setup steps, configuration decisions, and any issues encountered along the way.)*

### Phase 1 — Infrastructure Setup
- [ ] Provision VULTR VPC with private subnet (172.31.0.0/24)
- [ ] Deploy Windows Server, Ubuntu Server, Fleet Server, Elastic/Kibana, and OS Ticket Server instances
- [ ] Configure network security groups / firewall rules between components

### Phase 2 — SIEM & Agent Management
- [ ] Install and configure Elastic + Kibana
- [ ] Deploy Fleet Server and enroll Windows + Ubuntu endpoints
- [ ] Verify log forwarding from both endpoints into Elastic
- [ ] Build initial Kibana dashboards for visibility

### Phase 3 — Alerting & Ticketing
- [ ] Configure Elastic detection rules / alerts
- [ ] Integrate alerts with OS Ticket Server
- [ ] Test end-to-end: trigger event → alert fires → ticket created

### Phase 4 — Adversary Simulation
- [ ] Stand up Mythic C2 server
- [ ] Configure Kali attacker laptop
- [ ] Execute simulated attack chain against Windows/Ubuntu targets
- [ ] Validate detection: confirm Elastic captures and alerts on adversary activity

### Phase 5 — Detection & Response
- [ ] Investigate generated alerts in Kibana
- [ ] Trace attacker activity back through logs
- [ ] Document findings and close the loop in the ticketing system

---

## 🐛 Issues Encountered & Fixes

*(Document configuration problems here as they come up — this section is often the most valuable part of the writeup.)*

| Issue | Cause | Fix |
|---|---|---|
| *Example: Agent not reporting to Fleet* | *Firewall blocking enrollment port* | *Opened port 8220 between Fleet and endpoint* |

---

## 🧠 Key Takeaways

*(To be filled in after completing the build — focus on what you learned, not just what you did.)*

---

## 📋 Planned Enhancements

- [ ] Add Sysmon to Windows endpoint for deeper process-level visibility
- [ ] Build custom Kibana detection rules mapped to MITRE ATT&CK techniques
- [ ] Add a second attacker technique (e.g., lateral movement) to test detection coverage
- [ ] Automate ticket creation with severity-based routing
- [ ] Write a full incident response report simulating a real engagement

---

## 🔧 Tools & Technologies

`Elastic Stack (ELK)` `Kibana` `Fleet` `Mythic C2` `Kali Linux` `Windows Server` `Ubuntu Server` `OS Ticket` `VULTR VPC` `MITRE ATT&CK`

---

## 📫 Contact

**Angel Ruiz**
🔐 SOC Analyst (Tier 1) | Cybersecurity Analyst | CompTIA Security+ Certified
💼 [LinkedIn](https://www.linkedin.com/in/aruizcanel/) · 🖥️ [GitHub](https://github.com/ARUIZCANEL) · 📧 aruizcanel@yahoo.com
