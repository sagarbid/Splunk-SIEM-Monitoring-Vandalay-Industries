# 📊 Splunk SIEM Monitoring — Vandalay Industries

> **Enterprise SIEM project: building custom Splunk dashboards, alerts, and threat correlation to detect DDoS, brute-force, and vulnerability scan activity.**

![Splunk](https://img.shields.io/badge/Tool-Splunk%20Enterprise-black?style=flat-square&logo=splunk)
![SIEM](https://img.shields.io/badge/Type-SIEM%20%2F%20SOC-blue?style=flat-square)
![Nessus](https://img.shields.io/badge/Tool-Nessus-green?style=flat-square)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=flat-square)

---

## 📌 Project Overview

This project simulates the role of a **SOC analyst responsible for SIEM administration** at a fictional enterprise — Vandalay Industries. Completed as part of the **Monash University Cybersecurity Bootcamp (2024)**, the project focuses on building production-grade Splunk monitoring infrastructure: custom dashboards, threshold-based alerts, and multi-source log correlation to detect and investigate real threats.

This directly mirrors the day-to-day responsibilities of a Tier 1/2 SOC analyst in an enterprise environment.

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

## 🚨 Attack Scenarios Detected

### 1. DDoS Attack
Detected a volumetric DDoS attack against the Vandalay web infrastructure by identifying abnormal request spikes from distributed source IPs.

```spl
index=apache_logs | timechart span=1m count by clientip 
| where count > 1000
```

### 2. Brute-Force Authentication Attack
Identified credential stuffing against Windows Active Directory by correlating failed login events (EventCode 4625) across multiple endpoints.

```spl
index=windows EventCode=4625 
| stats count by src_ip, user, ComputerName 
| where count > 20 
| sort -count
```

### 3. Vulnerability Scan Activity
Correlated Nessus scan output with Apache access logs to identify hosts being actively probed and cross-reference with known vulnerabilities.

```spl
index=nessus severity=Critical OR severity=High 
| join host [search index=apache_logs] 
| table host, vulnerability, last_seen, http_status
```

---

## 📊 Dashboards Built

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

## 🔔 Alerts Configured

| Alert Name | Trigger Condition | Severity |
|-----------|-------------------|----------|
| DDoS Threshold | >1000 req/min from single IP | Critical |
| Brute Force Detected | >20 failed logins in 5 min | High |
| Privilege Escalation | EventCode 4672 outside business hours | High |
| Vuln Scan Activity | Nessus critical finding on internet-facing host | Medium |

---

## 📋 Key Outcomes

- Reduced mean time to detect (MTTD) simulated attacks to under 3 minutes using threshold alerts
- Built a reusable SIEM dashboard template applicable to any enterprise environment
- Demonstrated correlation between vulnerability exposure and active exploitation attempts
- Produced a threat intelligence report summarising attack patterns, IoCs, and remediation priorities

---

## 👤 Author

**Sagar Bidari**
CompTIA Security+ CE | Monash University Cybersecurity Bootcamp Graduate
🌐 [bidarisagar.com](https://bidarisagar.com) | 💼 [LinkedIn](https://linkedin.com/in/sagarbidari)
