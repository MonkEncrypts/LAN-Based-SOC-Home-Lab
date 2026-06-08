# 🔍 Attack 01 – Nmap Reconnaissance Scan (Windows Endpoint)

## Attack Overview

| Field | Details |
|---|---|
| **Attack Name** | Nmap Network Reconnaissance Scan |
| **Target** | Windows 10 Endpoint (WIN-ENDPOINT) |
| **Attacker** | Kali Linux (192.168.x.x) |
| **MITRE Tactic** | Reconnaissance / Discovery |
| **MITRE Technique** | T1046 – Network Service Scanning |
| **Severity** | Informational / Low |
| **Date Performed** | YYYY-MM-DD |

## Objective

To perform network reconnaissance against the Windows endpoint using Nmap to enumerate open ports, running services, and OS information — simulating the initial phase of an adversary's kill chain.

## Lab Environment

| Machine | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | `192.168.x.x` | Kali 2025.2 |
| WIN-ENDPOINT | Target/Victim | `192.168.x.x` | Windows 10 |
| Wazuh Server | SIEM/Monitor | `192.168.x.x` | Ubuntu Server 26.04 |

## Attack Diagram

```
┌──────────────┐        Nmap Scan         ┌──────────────────┐
│  Kali Linux  │ ──────────────────────▶  │  WIN-ENDPOINT    │
│  (Attacker)  │    Ports/Services/OS     │  (Victim)        │
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
| `screenshots/win-attack01-nmap-command.png` | Nmap command execution on Kali |
| `screenshots/win-attack01-nmap-results.png` | Nmap scan results |
| `screenshots/win-attack01-wazuh-alert.png` | Wazuh alert triggered |
| `screenshots/win-attack01-wazuh-dashboard.png` | Wazuh dashboard view |

## Endpoint Evidence

<!-- Document what was observed on the Windows endpoint -->

**Windows Security Event Log:**
```
Event ID: 5156 (Windows Filtering Platform - Connection Allowed)
Event ID: 5157 (Windows Filtering Platform - Connection Blocked)
```

**Sysmon Events:**
```
Event ID 3 – Network Connection Detected
- Source IP: <KALI_IP>
- Destination Port: Multiple
- Protocol: TCP
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
    "name": "WIN-ENDPOINT"
  },
  "data": {
    "srcip": "",
    "dstport": ""
  }
}
```

## Relevant Event IDs

| Event ID | Source | Description | Relevance |
|---|---|---|---|
| 5156 | Windows Security | WFP Connection Allowed | Inbound connections from attacker IP |
| 5157 | Windows Security | WFP Connection Blocked | Blocked connection attempts |
| 3 | Sysmon | Network Connection | Tracks inbound connections with process context |
| 22 | Sysmon | DNS Query | Any DNS resolution triggered by scan |

## Sysmon Events

```xml
<!-- Paste relevant Sysmon XML event here -->
<Event>
  <System>
    <EventID>3</EventID>
    <TimeCreated SystemTime="" />
  </System>
  <EventData>
    <Data Name="SourceIp"><!-- Kali IP --></Data>
    <Data Name="DestinationPort"><!-- Port --></Data>
    <Data Name="Protocol">tcp</Data>
  </EventData>
</Event>
```

## Log Analysis

### Key Findings

1. **Finding 1:** <!-- Describe what was observed in the logs -->
2. **Finding 2:** <!-- Describe any patterns identified -->
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
| **Procedure** | Adversary used Nmap from Kali Linux to perform SYN scan, service version detection, and OS fingerprinting against the Windows endpoint |
| **Detection Opportunity** | Monitor for rapid sequential connection attempts to multiple ports from a single source IP |

## Detection Logic

```xml
<!-- Wazuh Custom Rule (if created) -->
<rule id="100001" level="8">
  <if_sid>5156</if_sid>
  <description>Possible port scan detected - multiple connection attempts from single source</description>
  <mitre>
    <id>T1046</id>
  </mitre>
</rule>
```

## Investigation Process

1. **Alert Triage:** Received alert for multiple inbound connections from single source IP
2. **Source Identification:** Identified source IP as `<KALI_IP>` via Wazuh alert data
3. **Pattern Analysis:** Observed sequential port scanning pattern across multiple ports
4. **Log Correlation:** Cross-referenced Windows Security logs with Sysmon network events
5. **Scope Assessment:** Determined scan targeted full port range / common ports
6. **Verdict:** Confirmed reconnaissance activity — network service scanning

## Findings

| # | Finding | Severity | Evidence |
|---|---|---|---|
| 1 | <!-- Finding --> | <!-- Low/Med/High --> | <!-- Log reference --> |
| 2 | <!-- Finding --> | <!-- Low/Med/High --> | <!-- Log reference --> |
| 3 | <!-- Finding --> | <!-- Low/Med/High --> | <!-- Log reference --> |

## Lessons Learned

- <!-- What detection gaps were identified? -->
- <!-- How could detection be improved? -->
- <!-- What additional data sources would help? -->
- <!-- Were there any false positives to tune? -->
- <!-- What SOC procedures should be followed for this type of alert? -->

---

> 📌 **Status:** 📝 Template — Update with actual findings after performing the attack simulation.
