# 🔍 Attack 01 – Nmap Reconnaissance Scan (Ubuntu Endpoint)

## Attack Overview

| Field | Details |
|---|---|
| **Attack Name** | Nmap Network Reconnaissance Scan |
| **Target** | Ubuntu Desktop 26.04 (UBUNTU-ENDPOINT) |
| **Attacker** | Kali Linux (192.168.x.x) |
| **MITRE Tactic** | Reconnaissance / Discovery |
| **MITRE Technique** | T1046 – Network Service Scanning |
| **Severity** | Informational / Low |
| **Date Performed** | YYYY-MM-DD |

## Objective

To perform network reconnaissance against the Ubuntu endpoint using Nmap to enumerate open ports, running services, and OS information — simulating the initial phase of an adversary's kill chain.

## Lab Environment

| Machine | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | `192.168.x.x` | Kali 2025.2 |
| UBUNTU-ENDPOINT | Target/Victim | `192.168.x.x` | Ubuntu Desktop 26.04 |
| Wazuh Server | SIEM/Monitor | `192.168.x.x` | Ubuntu Server 26.04 |

## Attack Diagram

```
┌──────────────┐        Nmap Scan         ┌──────────────────┐
  Kali Linux   │ ──────────────────────▶  │ UBUNTU-ENDPOINT  │
  (Attacker)   │    Ports/Services/OS     │  (Victim)        │
└──────────────┘                          └────────┬─────────┘
                                                   │ Logs
                                                   ▼
                                          ┌──────────────────┐
                                          │  Wazuh Server    │
                                          │  (SIEM)          │
                                          └──────────────────┘
```

## Kali Commands Used

```bash
# Basic TCP SYN Scan
nmap -sS <TARGET_IP>

# Service Version Detection
nmap -sV <TARGET_IP>

# OS Detection
nmap -O <TARGET_IP>

# Aggressive Scan (Service, OS, Scripts, Traceroute)
nmap -A <TARGET_IP>

# Full Port Scan
nmap -p- <TARGET_IP>

# Scan with Script Engine
nmap --script=vuln <TARGET_IP>
```

## Screenshots

> 📸 Add screenshots to `screenshots/` and reference them below:

| Screenshot | Description |
|---|---|
| `screenshots/ubuntu-attack01-nmap-command.png` | Nmap command execution on Kali |
| `screenshots/ubuntu-attack01-nmap-results.png` | Nmap scan results |
| `screenshots/ubuntu-attack01-wazuh-alert.png` | Wazuh alert triggered |
| `screenshots/ubuntu-attack01-wazuh-dashboard.png` | Wazuh dashboard view |

## Endpoint Evidence

<!-- Document what was observed on the Ubuntu endpoint -->

**Linux System Logs (`/var/log/syslog` or `/var/log/auth.log`):**
```
Jun  6 12:00:01 ubuntu-endpoint sshd[1234]: Connection from <KALI_IP> port 54321 on local port 22
Jun  6 12:00:01 ubuntu-endpoint sshd[1234]: Did not receive identification string from <KALI_IP> port 54321
```

## Wazuh Alerts

| Alert ID | Rule ID | Description | Level | Timestamp |
|---|---|---|---|---|
| <!-- ID --> | <!-- Rule --> | <!-- Description --> | <!-- Level --> | <!-- Time --> |

```json
// Paste relevant Wazuh alert JSON here
{
  "rule": {
    "id": "",
    "description": "",
    "level": 0
  },
  "agent": {
    "name": "UBUNTU-ENDPOINT"
  },
  "data": {
    "srcip": "",
    "dstport": ""
  }
}
```

## Relevant Linux Logs

| Log File | Description | Relevance |
|---|---|---|
| `/var/log/auth.log` | Authentication log | Captures SSH connection attempts and scanning touches |
| `/var/log/syslog` | System log | General log showing service events and connections |

## Log Analysis

### Key Findings

1. **Finding 1:** <!-- Describe what was observed in the logs -->
2. **Finding 2:** <!-- Describe any patterns identified (e.g. rapid succession of connection drops) -->
3. **Finding 3:** <!-- Describe correlation between events -->

### Log Correlation

<!-- Describe how different log sources correlated to confirm the scan activity -->

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| **Tactic** | Reconnaissance / Discovery |
| **Technique ID** | T1046 |
| **Technique Name** | Network Service Scanning |
| **Sub-Technique** | N/A |
| **Procedure** | Adversary used Nmap from Kali Linux to perform SYN scan, service version detection, and OS fingerprinting against the Ubuntu endpoint |
| **Detection Opportunity** | Monitor for rapid sequential connection attempts to multiple ports from a single source IP |

## Detection Logic

```xml
<!-- Wazuh Custom Rule (if created) -->
<rule id="100006" level="8">
  <if_sid>31100</if_sid>
  <description>Possible port scan detected - connections from single source without auth</description>
  <mitre>
    <id>T1046</id>
  </mitre>
</rule>
```

## Investigation Process

1. **Alert Triage:** Received alert for multiple inbound connections from single source IP
2. **Source Identification:** Identified source IP as `<KALI_IP>` via Wazuh alert data
3. **Pattern Analysis:** Observed connection attempts to multiple ports
4. **Log Correlation:** Checked `/var/log/auth.log` for connection drops
5. **Scope Assessment:** Determined scan target services
6. **Verdict:** Confirmed reconnaissance activity — network service scanning

## Findings

| # | Finding | Severity | Evidence |
|---|---|---|---|
| 1 | <!-- Finding --> | <!-- Low/Med/High --> | <!-- Log reference --> |
| 2 | <!-- Finding --> | <!-- Low/Med/High --> | <!-- Log reference --> |

## Lessons Learned

- <!-- What detection gaps were identified? -->
- <!-- How could detection be improved? -->
- <!-- What additional data sources would help? -->
- <!-- Were there any false positives to tune? -->
- <!-- What SOC procedures should be followed for this type of alert? -->

---

> 📌 **Status:** 📝 Template — Update with actual findings after performing the attack simulation.
