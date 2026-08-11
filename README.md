# cyber-home-lab-with-splunk
Self‑directed home‑lab project – AD‑DC01 + Splunk SIEM + attack simulation &amp; SOC dashboard

# 🛡️ Cybersecurity Home Lab – SIEM Implementation & Attack Detection

**Self‑directed 8‑week project** – Build an isolated lab (AD‑DC01 + Splunk + Kali), ingest Windows Event Logs, run a multi‑stage attack chain, and visualise the results in a SOC‑style Splunk dashboard.

---

## Table of Contents

1. [Project Description](#project-description)
2. [Why This Project Is Necessary](#why-this-project-is-necessary)
3. [About Splunk](#about-splunk)
4. [Lab Architecture](#lab-architecture)
5. [How the Lab Works & Attack Simulation](#how-the-lab-works--attack-simulation)
6. [SOC Dashboard](#soc-dashboard)
7. [Outcomes & Next Steps](#outcomes--next-steps)
8. [Repository Structure](#repository-structure)
9. [How to Reproduce / Run the Lab](#how-to-reproduce--run-the-lab)
10. [License](#license)
11. [Contact](#contact)

---

## Project Description

The goal of this project is to **design, build, and validate** an end‑to‑end Security Information and Event Management (SIEM) detection pipeline that mirrors a small enterprise network.

* **Duration:** 8 weeks (June – August 2026)
* **Outcome:** Fully functional lab, Splunk‑based SOC dashboard, technical report, and a demonstrable skill set for SOC/VAPT internships or entry‑level security roles.

![AD‑DC01 VM](images/ad-dc01.png)
*Figure 1 – AD‑DC01 Windows Server 2019 Core domain controller (lab.local).*

---

## Why This Project Is Necessary

| Reason | Explanation |
|--------|-------------|
| **Employer Demand** | >80 % of enterprise SOCs list Splunk proficiency as a requirement; hands‑on AD administration is equally critical. |
| **Gap in Academia** | Coursework covers theory but rarely includes live log‑pipeline construction, audit‑policy tuning, or attack validation. |
| **Proof of Skill** | A validated home lab provides concrete, measurable evidence (event counts, detection queries, dashboards) that can be shown on a résumé or in an interview. |
| **Risk‑Free Experimentation** | Isolated VMnet1 network lets you run real attack techniques (nmap, crackmapexec, PowerShell) without affecting production systems. |

---

## About Splunk

* **Industry‑leading SIEM** – used for log collection, indexing, real‑time search, alerting, and visualisation.
* **Components in this lab**
  * **Splunk Enterprise Indexer** – Ubuntu 22.04 LTS VM (`splunk vm.png`)
  * **Splunk Universal Forwarder** – installed on AD‑DC01 to forward Windows Event Logs over TCP 9997.
* **Key capabilities leveraged**
  * Real‑time vs. historical search modes
  * SPL (`stats`, `timechart`, `eval`/`case`) for event correlation
  * Dashboard Studio & Classic Dashboard for evidentiary and SOC‑style views

![Splunk VM](images/splunk-vm.png)
*Figure 2 – Splunk Enterprise indexer running on Ubuntu.*

---

## Lab Architecture

The lab is built on a **VMware host‑only network (VMnet1, 192.168.136.0/24)** to guarantee complete isolation.

| Component | OS / Role | Static IP | Purpose |
|-----------|-----------|-----------|---------|
| **Host** | Windows 11 Pro (16 GB RAM) – VMware Workstation Pro | – | Hypervisor, provides networking & storage |
| **AD‑DC01** | Windows Server 2019 Server Core – Domain Controller for `lab.local` | 192.168.136.128 | Central authentication, source of Windows Event Logs |
| **Splunk Indexer** | Ubuntu 22.04 LTS – Splunk Enterprise | 192.168.136.129 | Indexes, stores, and searches logs |
| **Kali Linux** | Offensive Security Kali – Attacker platform | 192.168.136.130 | Executes reconnaissance, credential brute‑force, privilege escalation, lateral movement |

![Lab diagram (AD‑DC01, Splunk VM, Kali)](images/ad-dc01.png)
*Figure 3 – High‑level view of the isolated lab network (all three VMs on VMnet1).*

---

## How the Lab Works & Attack Simulation

### 1. Log Pipeline (Defensive Side)

1. **Enable Advanced Auditing** on AD‑DC01 (process creation, security group management, user account management, registry‑based command‑line logging).
2. **Install & Configure Splunk Universal Forwarder** on AD‑DC01 → forwards **Security, System, Application** logs to the Splunk indexer on port 9997.
3. **Verify Listener** on the indexer: `splunk enable listen 9997`.
4. **Confirm Ingestion** – Splunk query `index=* host=AD-DC01` returns **>2,900 indexed events**.

### 2. Attack Simulation (Offensive Side) – executed from Kali

| Stage | Command (run on Kali) | Expected Event(s) in Splunk |
|-------|----------------------|-----------------------------|
| **Reconnaissance** | `nmap -sV -sC 192.168.136.128` | Increased connection‑attempt noise in Security log (failed SMB/LDAP/RPC). |
| **Initial Access** | `crackmapexec smb 192.168.136.128 -u testuser -p /tmp/passlist.txt` | Fail‑fail‑success logon pattern: Event ID 4625 → 4625 → 4624 (source = Kali). |
| **Privilege Escalation** | `Add-ADGroupMember -Identity "Domain Admins" -Members testuser` (run on AD‑DC01 via PowerShell Remoting) | Event ID 4728 (Member = testuser, Group = Domain Admins). |
| **Lateral Movement** | `crackmapexec smb 192.168.136.128 -u testuser -p 'Summer2026!' -x "whoami"` | Event ID 4688 with `New_Process_Name=cmd.exe` and `Creator_Process_Name=WmiPrvSE.exe`. |

*All events are captured, indexed, and made available for correlation and dashboard visualisation.*

![Kali attack terminal](images/kali.png)
*Figure 4 – Kali terminal showing the credential brute‑force output (fail‑fail‑success).*

![Splunk detection snippet](images/splunk-dash.png)
*Figure 5 – Splunk search/table displaying the five key events (4625, 4625, 4624, 4728, 4688) with timestamps.*

---

## SOC Dashboard

A **presentation‑layer SOC dashboard** was built in Splunk Dashboard Studio (dark theme, grid layout) to give instant situational awareness.

* **Four KPI tiles** – Total Security Events, Failed Logons, Privilege Escalations, Lateral Movement Events.
* **Attack‑Progression Timeline** – Multi‑series timechart overlaying the four event types, clearly showing the chronological flow of the intrusion.
* **Top‑Attacker table** – Breaks down failed/successful authentications by source IP (highlights Kali’s IP).
* **Event‑Summary table** – Deduplicated, stage‑by‑stage narrative (event type, first timestamp, count).

![SOC Threat Detection Dashboard](images/main-dash.png)
*Figure 6 – SOC‑style dashboard showing KPIs, timeline, top attacker, and event summary.*

![Additional dashboard view](images/dash1.png)
*Figure 7 – Evidentiary dashboard (Classic) with failed‑logon timechart and source/account breakdown.*

---

## Outcomes & Next Steps

### ✅ Achieved Outcomes

* **End‑to‑end detection pipeline** – >2,900 Windows events ingested, all attack stages visible and correlated in Splunk.
* **Portfolio‑ready artefacts** – Technical report (Word/PDF), 7‑slide PowerPoint deck, VM snapshots, Splunk dashboard export (JSON), and this README.
* **Skill demonstration** – Windows/Linux system administration, SIEM log‑pipeline engineering, advanced Windows audit policy configuration, offensive‑tool validation (Nmap, crackmapexec, PowerShell), SPL query authoring, and SOC‑dashboard design.

### 🔜 Future Scope / Next Steps

| Idea | Description |
|------|-------------|
| **Insider‑threat simulation** | Use the host machine (or a second internal VM) with stolen credentials to model credential reuse from a trusted source. |
| **Data‑exfiltration simulation** | Copy a large file from AD‑DC01 to an external share (e.g., SMB to Kali) and detect via file‑copy/event‑ID 5140/5145. |
| **Automated response** | Trigger a Splunk alert → Webhook → PowerShell script that disables the compromised account or isolates the offending VM via NSX/VLAN. |
| **Threat‑intel enrichment** | Inject open‑source IOCs (e.g., Abuse.ch URLhaus) into Splunk to enrich events with reputation scores. |
| **Advanced dashboarding** | Add geolocation map of attack sources, risk‑scoring panels, and trend‑analysis visualisations. |

---

## Repository Structure
