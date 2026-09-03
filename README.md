# Brute-Force-Attack-Detection & Investigation-Using-Wazuh
SOC lab demonstrating SSH attack detection, investigation, MITRE ATT&amp;CK mapping, and firewall containment using Wazuh SIEM.

---

## 📌 Project Overview

This project simulates a controlled SSH authentication attack against an Ubuntu endpoint monitored by Wazuh.

The objective was to demonstrate a practical SOC workflow:

1. Generate suspicious SSH authentication activity
2. Collect and monitor the activity using Wazuh
3. Investigate generated security alerts
4. Identify the attacking source IP
5. Map the activity to MITRE ATT&CK
6. Validate the underlying Linux authentication logs
7. Contain the source using UFW firewall rules
8. Verify the effectiveness of the containment

This lab represents a simplified real-world SOC incident investigation workflow.

---

## 🏗️ Lab Environment

| Component | Details |
|---|---|
| SIEM | Wazuh |
| Wazuh Version | v4.14.7 |
| Endpoint OS | Ubuntu 24.04.4 LTS |
| Endpoint | Ubuntu-SOC-Lab |
| Wazuh Agent ID | 001 |
| Endpoint IP | 192.168.237.148 |
| SSH Service | OpenSSH |
| SSH Port | 22/TCP |
| Firewall | UFW |
| Log Source | systemd journal / authentication logs |

---

## 🎯 Objective

The primary objective of this lab was to investigate suspicious SSH authentication activity and demonstrate how a SOC analyst can:

- Detect authentication failures
- Identify invalid usernames
- Determine the source IP address
- Analyze Wazuh alert metadata
- Correlate SIEM alerts with endpoint logs
- Map activity to MITRE ATT&CK
- Apply network-level containment
- Verify that the containment control is active

---

## 🔴 Attack Simulation

A controlled SSH authentication attack was simulated from a separate Ubuntu system.

The following SSH connection was attempted against the monitored Ubuntu endpoint:

```bash
ssh wronguser@192.168.237.148
