# 🛡️ MyDFIR 30-Day SOC Analyst Challenge

A fully documented, hands-on SOC environment built from scratch in the cloud over 30 days — following the [MyDFIR 30-Day SOC Analyst Challenge](https://www.youtube.com/playlist?list=PLG6KGSNK4PuBb0OjyDIdACZnb8AoNBeq6) by Steven at MyDFIR.

> **Disclaimer:** All offensive activity in this lab targets only systems I own within an isolated private cloud environment. No external systems were accessed or tested. This project exists purely for educational purposes.

---

## 🎯 Objective

Build a realistic, end-to-end SOC environment that mirrors how real organizations detect and respond to threats — from log ingestion and alert creation all the way through adversary simulation, detection, investigation, and ticketing.

---

## 🖥️ Lab Architecture

```
                        ┌─────────────────┐
                        │     Internet      │
                        └────────┬──────────┘
                                 │
             ┌───────────────────┼───────────────────┐
             │                                        │
  ┌──────────────────┐                    ┌──────────────────┐
  │  SOC Analyst      │                    │  Attacker Laptop  │
  │  Laptop           │                    │  (Kali Linux)     │
  │  (Kibana Web GUI) │                    └────────┬─────────┘
  └──────────────────┘                              │
                                         ┌──────────────────┐
                                         │   C2 Server       │
                                         │   (Mythic)        │
                                         └──────────────────┘

┌──────────────────────── VULTR VPC ──────────────────────────┐
│  Private Network: 172.31.0.0/24                              │
│  IP Range: 172.31.0.1 – 254 | Subnet: 255.255.255.0         │
│                                                               │
│   ┌─────────────────┐        ┌─────────────────┐            │
│   │ Elastic + Kibana │◄──────►│  osTicket Server │           │
│   │     (SIEM)       │Alerts/ │   (Ticketing)    │           │
│   └────────┬─────────┘Tickets └─────────────────┘           │
│            │                                                  │
│     Manage Agents / Forward Logs                             │
│            │                                                  │
│   ┌────────┴────────────────────────┐                        │
│   │                                  │                        │
│ ┌──────────┐  ┌──────────┐  ┌──────────┐                    │
│ │ Windows  │  │  Fleet   │  │ Ubuntu   │                    │
│ │ Server   │◄─┤  Server  ├─►│ Server   │                    │
│ │ (RDP)    │  │ (Managed)│  │ (SSH)    │                    │
│ └──────────┘  └──────────┘  └──────────┘                    │
└──────────────────────────────────────────────────────────────┘
```

---

## 🧩 Components

| Component | Purpose | Details |
|---|---|---|
| **Elastic + Kibana** | SIEM | Log ingestion, detection rules, dashboards, alerting |
| **Fleet Server** | Agent management | Centrally deploys and manages Elastic Agents across endpoints |
| **Windows Server** | Target endpoint | RDP enabled, Sysmon installed, Elastic Agent enrolled |
| **Ubuntu Server** | Target endpoint | SSH enabled, Elastic Agent enrolled |
| **osTicket** | Ticketing system | Integrated with Elastic via webhook for alert-to-ticket workflow |
| **Mythic C2** | Adversary simulation | Real C2 framework deployed via Docker on external network |
| **Kali Linux** | Attacker machine | Crowbar, Hydra, Nmap, payload delivery |
| **VULTR VPC** | Cloud hosting | Private subnet isolating all lab components |

---

## 📅 Challenge Breakdown

### Phase 1 — Planning & Architecture (Days 1–2)
- Designed the full logical network diagram mapping all components and their relationships
- Understood how each piece fits together: SIEM ↔ Fleet ↔ Agents ↔ C2 ↔ Ticketing
- Chose VULTR as the cloud hosting platform for the VPC environment

### Phase 2 — ELK Stack Setup (Days 3–7)
- Provisioned VULTR VPC with private subnet (172.31.0.0/24)
- Deployed and configured Elasticsearch and Kibana on a dedicated Ubuntu server
- Configured firewall rules to restrict access appropriately
- Learned how logs are parsed, indexed, and queried in Elasticsearch
- Explored Kibana's Discover, Dashboards, and Alerts interfaces

### Phase 3 — Windows Endpoint & Sysmon (Days 8–9)
- Deployed Windows Server with RDP enabled
- Installed Sysmon for deep endpoint visibility — process creation, network connections, file events
- Configured Sysmon to forward logs into Elasticsearch via Elastic Agent
- Installed Splunk Add-on equivalent (Elastic integration) for proper Sysmon log parsing

### Phase 4 — Fleet Server & Agent Management (Days 10–13)
- Deployed Fleet Server for centralized Elastic Agent management
- Enrolled Windows Server and Ubuntu Server as managed endpoints
- Troubleshot architecture mismatch (arm64 vs x86_64) and SSL certificate issues during agent installation
- Verified log forwarding from both endpoints into the ELK stack
- Exposed SSH on Ubuntu to the internet — observed real-world bot brute force attempts within minutes

### Phase 5 — Brute Force Detection & Dashboards (Days 14–17)
- Built Kibana detection rules for SSH brute force and RDP brute force activity
- Analyzed Windows RDP authentication logs — Event IDs for failed and successful logins
- Investigated real SSH brute force attempts from internet-facing bots
- Built live Kibana dashboards — 4 maps and 4 tables showing SSH/RDP failed and accepted activity
- Learned how tools like Shodan can find exposed RDP services in seconds

### Phase 6 — C2 Setup & Attack Planning (Days 18–21)
- Studied C2 frameworks — how attackers use them for persistent access, lateral movement, and exfiltration
- Built a full attack diagram mapping every phase of the kill chain before executing anything:
  - Initial Access → Persistence → Defense Evasion → Execution → C2 → Collection → Exfiltration
- Deployed Mythic C2 server on a dedicated external Ubuntu instance using Docker
- Configured Mythic on a separate network to simulate real external attacker infrastructure

### Phase 7 — Adversary Simulation (Days 22–23)
- Executed full attack chain from Kali Linux attacker machine:
  1. **Reconnaissance** — Nmap scan of target environment
  2. **Initial Access** — RDP brute force using Crowbar with custom wordlist
  3. **Defense Evasion** — Disabled Windows Defender via PowerShell
  4. **Execution** — Downloaded Mythic Apollo agent to target via PowerShell
  5. **C2 Establishment** — Active Meterpreter/Mythic callback confirmed in C2 dashboard
  6. **Post-exploitation** — Remote command execution (whoami, ipconfig, netstat)
  7. **Exfiltration** — Pulled password file from target through C2 channel

### Phase 8 — Detection & Investigation (Days 24–25)
- Switched to defender perspective — investigated the full attack in Kibana
- Built detection rules targeting key Sysmon event codes:
  - **Event ID 1** — Process creation (Apollo agent spawning cmd.exe)
  - **Event ID 3** — Network connections (C2 callbacks to Mythic server)
  - **Event ID 5001** — Windows Defender disabled (defense evasion indicator)
- Identified Mythic agent immediately — svchost.exe running from wrong directory path
- Traced full command execution chain through process GUID correlation
- Investigated real unrecognized international login detected in live RDP logs

### Phase 9 — osTicket Integration (Days 26–28)
- Deployed osTicket on dedicated server within VPC
- Integrated osTicket with Elastic via webhook — alerts automatically generate tickets
- Investigated full ticket lifecycle end to end:
  - Alert fires in Kibana → Ticket created in osTicket → Analyst investigates → Documents findings → Closes ticket
- Practiced writing investigation documentation inside tickets — timeline, findings, remediation

### Phase 10 — Final Configuration & Wrap Up (Days 29–30)
- Configured Elastic Defend for automated endpoint response capabilities
- Reviewed and tuned detection rules to reduce false positives
- Completed final investigation exercises across SSH brute force, RDP brute force, and C2 activity
- Documented complete lab teardown and lessons learned

---

## 🔍 Key Investigations Completed

| Investigation | Alert Type | Key Finding |
|---|---|---|
| SSH Brute Force | Failed auth threshold | Bots hitting Ubuntu SSH within minutes of port 22 exposure |
| RDP Brute Force | Failed auth threshold | Crowbar cracked RDP credentials using custom wordlist |
| Defense Evasion | Sysmon Event ID 5001 | Windows Defender disabled via PowerShell before payload |
| C2 Callback | Sysmon Event ID 3 | svchost.exe making outbound connection to Mythic server |
| Malicious Process | Sysmon Event ID 1 | Apollo agent spawning cmd.exe from non-standard path |
| Data Exfiltration | Network anomaly | Outbound file transfer through established C2 channel |
| Unauthorized Login | Auth anomaly | Unrecognized international IP login detected in RDP logs |

---

## 🧠 Key Takeaways

- **Configuration matters more than tools.** Fleet Server installation issues (wrong architecture, SSL certs, port rules) cost more time than the actual attack simulation — and taught more about how the stack works under the hood.
- **Real attacks are fast.** The entire kill chain from initial brute force to active C2 session took minutes. Detection is only useful if the tooling is already in place.
- **Process lineage is everything.** svchost.exe running from a non-standard path is an immediate red flag — legitimate Windows processes have predictable parent-child relationships.
- **The ticketing workflow matters as much as detection.** An alert nobody follows up on is as dangerous as one that never fired.
- **Expose a port, get attacked immediately.** Port 22 and 3389 facing the internet attract automated bots within minutes. This is not theoretical.
- **Doing both sides makes you better at each.** Understanding how the attack was built made detection faster. Understanding what defenders look for made the attack more realistic.

---

## 🐛 Issues Encountered & Fixes

| Issue | Cause | Fix |
|---|---|---|
| Fleet Server wouldn't install | Downloaded arm64 agent on x86_64 server | Ran `uname -m` first, downloaded correct architecture |
| Windows agent "service did not respond" | Self-signed cert timeout during install | Added `--insecure` flag to install command |
| Elastic indexes tab not loading | Bad `inputs.conf` referencing non-existent index | Removed custom config, created `endpoint` index first, re-applied config |
| Agent enrollment failing | Port 8220 blocked between Windows VM and Fleet | Opened port 8220 in VULTR firewall rules |
| Splunk not parsing Sysmon logs | Missing Sysmon Add-on | Installed Splunk Add-on for Sysmon from Splunkbase |
| Ghost service blocking reinstall | Failed install left broken Windows service | Used `sc.exe delete`, took ownership of locked folder, rebooted |

---

## 📋 Planned Enhancements

- [ ] Add MITRE ATT&CK technique mapping table for each attack phase
- [ ] Build automated Elastic Defend response playbook
- [ ] Add second attacker technique (lateral movement between endpoints)
- [ ] Create custom Sigma rules for key detection scenarios
- [ ] Write full incident report documenting the simulated engagement

---

## 🔧 Tools & Technologies

`Elastic Stack (ELK)` `Kibana` `Elasticsearch` `Fleet Server` `Elastic Agent` `Sysmon` `Mythic C2` `Apollo Agent` `Kali Linux` `Crowbar` `Hydra` `Nmap` `Wireshark` `osTicket` `Docker` `VULTR VPC` `Windows Server` `Ubuntu Server` `PowerShell` `Bash` `MITRE ATT&CK`

---

## 📚 Resources

- [MyDFIR 30-Day SOC Analyst Challenge Playlist](https://www.youtube.com/playlist?list=PLG6KGSNK4PuBb0OjyDIdACZnb8AoNBeq6)
- [Elastic Documentation](https://www.elastic.co/docs)
- [Mythic C2 Framework](https://github.com/its-a-feature/Mythic)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [Sysmon Configuration](https://github.com/SwiftOnSecurity/sysmon-config)

---

## 📫 Contact

**Angel Ruiz**
🔐 SOC Analyst (Tier 1) | CompTIA Security+ Certified | Correlation One — Information Security Analyst (Graduated with Honors)
💼 [LinkedIn](https://www.linkedin.com/in/aruizcanel/) · 🖥️ [GitHub](https://github.com/ARUIZCANEL) · 📧 aruizcanel@yahoo.com
