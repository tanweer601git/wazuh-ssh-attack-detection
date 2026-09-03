# Wazuh SSH Attack Detection & Investigation
SOC lab demonstrating SSH attack detection, investigation, MITRE ATT&amp;CK mapping, and firewall containment using Wazuh SIEM.


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


## 🔎 Incident Investigation

### Detection

The simulated SSH authentication activity was detected by Wazuh on the monitored Ubuntu endpoint.

Wazuh generated an alert with:

| Field | Value |
| Agent | Ubuntu-SOC-Lab |
| Agent ID | 001 |
| Endpoint IP | 192.168.237.148 |
| Source IP | 192.168.237.141 |
| Source User | wronguser |
| Service | SSH / sshd |
| Rule ID | 5710 |
| Rule Level | 5 |
| Detection | Attempt to login using a non-existent user |

---

### Alert Analysis

The Wazuh alert identified repeated authentication attempts against the SSH service using the non-existent username `wronguser`.

The alert metadata showed:

- **Source IP:** `192.168.237.141`
- **Target:** `192.168.237.148`
- **Username:** `wronguser`
- **Protocol:** SSH
- **Destination Port:** `22`
- **Rule ID:** `5710`
- **Rule Level:** `5`
- **Rule Groups:** `sshd`, `authentication_failed`, `invalid_login`

The activity was correlated with the endpoint's SSH authentication logs.

---

### MITRE ATT&CK Mapping

Wazuh mapped the detected activity to the following MITRE ATT&CK techniques:

| Technique | ID | Relevance |

| Password Guessing | T1110.001 | Authentication attempts against SSH |
| SSH | T1021.004 | SSH service used for remote access |

The alert metadata also associated the activity with **Credential Access** and **Lateral Movement** tactics.

---

### Endpoint Log Validation


The detection was independently validated using the Ubuntu authentication logs.

```bash
sudo grep "Failed password" /var/log/auth.log | tail -n 20

---

### 🛡️ Containment

After confirming the suspicious SSH authentication activity, network-level containment was applied on the monitored Ubuntu endpoint using UFW.

The identified source IP address was blocked from accessing the SSH service:

```bash
sudo ufw deny from 192.168.237.141 to any port 22 proto tcp
sudo ufw status numbered
---

## 📊 Findings Summary

The investigation confirmed a controlled SSH authentication attack against the monitored Ubuntu endpoint.

| Finding | Details |

| Attack Type | SSH authentication / brute-force activity |
| Source IP | `192.168.237.141` |
| Target IP | `192.168.237.148` |
| Target Service | SSH |
| Destination Port | `22/TCP` |
| Username Attempted | `wronguser` |
| Wazuh Rule ID | `5710` |
| Wazuh Alert Level | `5` |
| Detection | Attempt to log in using a non-existent user |
| MITRE ATT&CK | `T1110.001 - Password Guessing` |
| Containment | UFW firewall rule |
| Containment Status | Active |

### Key Findings

- Multiple failed SSH authentication attempts were observed from the same source IP.
- The attempted username `wronguser` did not exist on the monitored endpoint.
- Wazuh successfully detected and generated an alert for the suspicious authentication activity.
- Endpoint authentication logs independently confirmed the failed SSH attempts.
- The activity was mapped to **MITRE ATT&CK T1110.001 (Password Guessing)**.
- The identified source IP was contained using a UFW firewall rule.
- UFW confirmed that traffic from `192.168.237.141` to SSH port `22` was denied.
- The SSH service remained operational after containment.
<img width="1580" height="615" alt="apktool" src="https://github.com/CharanCSE02/Reverse-Engineering-APk-file/blob/main/Screenshot%202025-11-25%20164924.png" />

### SOC Analyst Conclusion

The investigation demonstrates a complete SOC incident-response workflow from initial detection through investigation, log validation, threat classification, containment, and verification.

---

## ⚠️ Risk Analysis

The observed activity represents a **low-to-medium severity SSH authentication threat** within the controlled lab environment.

### Potential Impact

If successful, repeated SSH authentication attempts could allow an attacker to obtain unauthorized access to the endpoint. Depending on the privileges of the compromised account, this could lead to:

- Unauthorized system access
- Credential compromise
- Privilege escalation
- Persistence on the endpoint
- Lateral movement to other systems
- Data access or modification

### Risk Factors

| Risk Factor | Assessment |
| Attack Vector | SSH remote authentication |
| Authentication Activity | Multiple failed attempts |
| Source | Single internal source IP |
| Target | Ubuntu SSH service |
| Successful Login | Not observed |
| Privilege Obtained | None |
| Detection | Wazuh SIEM |
| Containment | UFW source-IP blocking |
| Overall Risk | Low-Medium |

### Risk Assessment

The immediate risk was reduced because the authentication attempts were unsuccessful and the attacking source IP was identified and blocked at the firewall level.

However, repeated authentication attempts against exposed SSH services should be investigated because continued activity could indicate password guessing, credential attacks, or preparation for unauthorized access.

---

## 🛡️ Recommendations

Based on the investigation, the following security controls are recommended:

- **Restrict SSH access** to trusted administrative networks or IP addresses where possible.
- **Use SSH key-based authentication** instead of password-based authentication.
- **Disable direct root login** over SSH.
- **Implement rate limiting or Fail2Ban** to reduce repeated authentication attempts.
- **Use strong, unique passwords** for accounts that require password authentication.
- **Monitor SSH authentication logs** continuously through the SIEM.
- **Create alerting thresholds** for repeated failed authentication attempts from the same source IP.
- **Review and remove unused accounts** to reduce the attack surface.
- **Keep OpenSSH and the Ubuntu operating system updated** with security patches.
- **Maintain firewall rules** that block known malicious or unauthorized source addresses.
- **Investigate repeated authentication attempts** even when no successful login occurs.
- **Correlate authentication activity with other endpoint and network telemetry** to identify possible lateral movement or follow-on activity.

### SOC Response Improvement

A production SOC could further improve this workflow by automating source-IP enrichment, threat-intelligence checks, alert correlation, and firewall containment while maintaining appropriate change-control and monitoring procedures.

---

## 📚 Lessons Learned

This investigation provided practical experience with a complete SOC incident-response workflow.

Key lessons learned:

- **SIEM Detection:** Wazuh can identify suspicious SSH authentication activity and generate actionable security alerts.
- **Alert Investigation:** Alert metadata can be used to identify the source IP, target endpoint, attempted username, service, and detection rule.
- **Log Correlation:** SIEM alerts should be validated against endpoint authentication logs to confirm the underlying activity.
- **Threat Classification:** SSH authentication attacks can be mapped to relevant MITRE ATT&CK techniques such as Password Guessing (T1110.001).
- **Incident Containment:** Firewall controls can be used to restrict communication from an identified malicious or unauthorized source.
- **Verification:** Security controls should be tested after implementation to confirm that the intended traffic is blocked.
- **Defense in Depth:** Detection, investigation, logging, containment, and verification should work together as part of an effective SOC process.
- **Continuous Monitoring:** Repeated authentication failures should be monitored because unsuccessful attempts can be an early indicator of credential-based attacks.

### Skills Demonstrated

- Wazuh SIEM monitoring
- SSH attack detection
- Security alert investigation
- Linux authentication log analysis
- Source IP identification
- MITRE ATT&CK mapping
- Incident response
- Firewall-based containment
- Security control verification
---

## 🛠️ Tools Used

| Tool | Purpose |
| **Wazuh SIEM** | Security monitoring, alert generation, and investigation |
| **Ubuntu 24.04.4 LTS** | Monitored endpoint and attack simulation environment |
| **OpenSSH** | SSH service used during the authentication attack simulation |
| **Linux Authentication Logs** | Validation of failed SSH authentication attempts |
| **UFW (Uncomplicated Firewall)** | Source IP containment and traffic blocking |
| **MITRE ATT&CK** | Threat technique classification and mapping |
| **GitHub** | Project documentation, evidence, and version control |

---
## 🎯 Scope

This project was conducted entirely within a controlled lab environment.

The scope included:

- SSH authentication activity against the monitored Ubuntu endpoint.
- Wazuh detection and alert investigation.
- Analysis of Linux authentication logs.
- Identification of the suspicious source IP.
- MITRE ATT&CK technique mapping.
- UFW-based source IP containment.
- Verification of the firewall control.

No unauthorized external systems were targeted or tested.

---

## ⚠️ Disclaimer

This project was performed in an isolated and controlled lab environment for educational and cybersecurity training purposes only.

The SSH authentication activity was intentionally simulated against a system under the author's control.

No unauthorized systems, networks, accounts, or services were targeted.

The techniques and commands demonstrated in this repository should only be used on systems for which proper authorization has been obtained.

---

## 🏁 Conclusion

This project demonstrated a complete SOC investigation workflow for suspicious SSH authentication activity using Wazuh.

The investigation progressed from attack simulation and SIEM detection to alert analysis, source IP identification, endpoint log validation, MITRE ATT&CK mapping, firewall containment, and verification.

The exercise provided practical experience in security monitoring, incident investigation, threat classification, and network-level containment.

Overall, the lab demonstrates how a SOC analyst can use SIEM telemetry and endpoint evidence to investigate suspicious authentication activity and apply appropriate containment measures.


