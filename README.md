# 📊 Splunk SIEM Monitoring — Vandalay Industries

> **SIEM design project: specifying Splunk dashboards, alerts, and threat correlation logic to detect DDoS, brute-force, and vulnerability scan activity.**

![Splunk](https://img.shields.io/badge/Tool-Splunk%20Enterprise-black?style=flat-square&logo=splunk)
![SIEM](https://img.shields.io/badge/Type-SIEM%20%2F%20SOC-blue?style=flat-square)
![Nessus](https://img.shields.io/badge/Tool-Nessus-green?style=flat-square)
![Status](https://img.shields.io/badge/Status-Documented%20Design-yellow?style=flat-square)

---

## 📌 Project Overview

This project is a **design exercise in SIEM administration**, done as part of the **Monash University Cybersecurity Bootcamp**: specifying how a SOC analyst would configure Splunk — dashboards, threshold-based alerts, and multi-source log correlation — to detect DDoS, brute-force, and vulnerability-scan activity for a fictional enterprise, Vandalay Industries.

> **Honesty note:** the SPL queries and alert definitions below are real, syntactically valid Splunk search and `savedsearches.conf` content (now in [`detection/`](detection/)) that I wrote and checked for correctness. They were **not run against a live Splunk instance with real log data** — there's no deployed Splunk server, no ingested logs, and no screenshots of the dashboards or alerts firing. Treat this as documented detection logic, not a completed, evidenced SOC deployment. See [What I'd Improve](#-what-id-improve) for what turning this into the real thing would take.

---

## 🎯 Objectives

- Configure Splunk to ingest and parse multiple log sources
- Build dashboards that provide real-time visibility into security events
- Create threshold-based alerts for attack pattern detection
- Correlate Nessus vulnerability scan data with web server logs
- Document findings and produce actionable threat intelligence reports

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| Splunk Enterprise | SIEM platform |
| Nessus | Vulnerability scanner |
| Apache Web Server Logs | Web traffic analysis |
| Windows Event Logs | Endpoint monitoring |
| SPL (Search Processing Language) | Splunk query language |

---

## 🚨 Attack Scenarios — Detection Logic

Each scenario below is a search designed to **surface** that attack pattern in Splunk, not a confirmed detection from a real incident. Full copies live in [`detection/splunk_searches.spl`](detection/splunk_searches.spl).

### 1. DDoS Attack
Designed to flag a volumetric DDoS against the Vandalay web infrastructure by catching abnormal request spikes from a single source IP.

```spl
index=apache_logs | timechart span=1m count by clientip 
| where count > 1000
```

### 2. Brute-Force Authentication Attack
Designed to flag credential-stuffing / brute-force attempts against Windows Active Directory by correlating failed login events (EventCode 4625) across endpoints.

```spl
index=windows EventCode=4625 
| stats count by src_ip, user, ComputerName 
| where count > 20 
| sort -count
```

### 3. Vulnerability Scan Activity
Designed to correlate Nessus scan output with Apache access logs, to cross-reference actively-probed hosts with known vulnerabilities.

```spl
index=nessus severity=Critical OR severity=High 
| join host [search index=apache_logs] 
| table host, vulnerability, last_seen, http_status
```

---

## 📊 Dashboards Designed

These panels are the **designed layout** for three dashboards — what each would show if deployed against live data. They have not been built in an actual Splunk instance, so there are no dashboard screenshots to link.

**Dashboard 1: Attack Overview**
- Live event count by severity
- Top source IPs by volume
- Geographic attack origin map
- Alert trigger timeline

**Dashboard 2: Authentication Monitor**
- Failed vs successful login ratio
- Brute-force attempt heatmap by hour
- Compromised account watchlist

**Dashboard 3: Vulnerability Posture**
- Critical/High vulnerability count by host
- Patch status correlation
- Attack surface trending over time

---

## 🔔 Alerts — Designed Logic

Translated into real `savedsearches.conf` stanzas in [`detection/savedsearches.conf`](detection/savedsearches.conf) — copy-paste-ready for a real Splunk deployment, but not yet scheduled against a live search head.

| Alert Name | Trigger Condition | Severity |
|-----------|-------------------|----------|
| DDoS Threshold | >1000 req/min from single IP | Critical |
| Brute Force Detected | >20 failed logins in 5 min | High |
| Privilege Escalation | EventCode 4672 outside business hours | High |
| Vuln Scan Activity | Nessus critical finding on internet-facing host | Medium |

> The "internet-facing host" condition needs an asset-classification lookup (CMDB export, asset tags) that doesn't exist in this project — `detection/savedsearches.conf` uses a placeholder field (`host_exposure`) and says so inline.

---

## 📋 Project Takeaways

- Translated three plain-English attack scenarios into working SPL and Splunk alert-config syntax (`savedsearches.conf`) — the detection logic itself is real and checked for syntactic correctness.
- Practiced the design side of SIEM administration: dashboard layout, alert thresholds, and multi-source correlation (web logs ↔ vulnerability scan data).
- Did **not** stand up a live Splunk instance or ingest real/simulated log data — so there's no measured MTTD, no dashboard screenshots, and no fired-alert evidence. See below for what that would take.

---

## 💡 Lessons Learned

- **Writing correct SPL is only half the job.** Translating the three described alerts into actual `savedsearches.conf` stanzas surfaced a gap the prose description glossed over: "internet-facing host" isn't a real Splunk field, it's a business concept that needs an asset-classification lookup behind it in a real deployment.
- **A dashboard description and a built dashboard are very different artifacts.** It's easy to list panels that would be useful; it's a different skill to actually configure them against ingested data and verify they render correctly.
- **"Complete" needs evidence, not just a finished write-up.** Revisiting this project after building out the other homelab repos (Nmap scanner, Hashcat cracking, RDP brute-force detection) made the gap between "documented" and "demonstrated" obvious — this one was still describing a deployment that never happened.

## 🔧 What I'd Improve

- **Stand up an actual Splunk instance** (free Splunk Free tier or a Docker container) and ingest sample Apache/Windows Event Log/Nessus data — even synthetic — so these searches can be run for real and the results captured.
- **Build the three dashboards for real** and add screenshots to a `screenshots/` folder, replacing the "designed layout" framing with actual evidence.
- **Schedule the alerts in `detection/savedsearches.conf`** against live data and capture a fired-alert screenshot or exported alert history.
- **Replace the `host_exposure` placeholder** with a real lookup (even a small CSV lookup table simulating a CMDB) so the vulnerability-scan alert is runnable as-is, not just documented.
- **Measure an actual MTTD** once alerts are live, instead of the unsupported "under 3 minutes" claim this README used to make.

---

## 👤 Author

**Sagar Bidari**
CompTIA Security+ CE | Monash University Cybersecurity Bootcamp Graduate
🌐 [bidarisagar.com](https://bidarisagar.com) | 💼 [LinkedIn](https://linkedin.com/in/sagarbidari)
