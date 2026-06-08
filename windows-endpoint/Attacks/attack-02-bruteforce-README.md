# 🔐 Attack 02 – Brute-Force Attack (Windows Endpoint)

## Attack Overview

| Field | Details |
|---|---|
| **Attack Name** | RDP/SMB Brute-Force Attack |
| **Target** | Windows 10 Endpoint (WIN-ENDPOINT) |
| **Attacker** | Kali Linux (192.168.x.x) |
| **MITRE Tactic** | Credential Access |
| **MITRE Technique** | T1110 – Brute Force |
| **Severity** | High |
| **Date Performed** | YYYY-MM-DD |

## Objective

To simulate a brute-force credential attack against the Windows endpoint targeting RDP/SMB services, testing Wazuh's ability to detect repeated failed authentication attempts and alert on credential access threats.

## Lab Environment

| Machine | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | `192.168.x.x` | Kali 2025.2 |
| WIN-ENDPOINT | Target/Victim | `192.168.x.x` | Windows 10 |
| Wazuh Server | SIEM/Monitor | `192.168.x.x` | Ubuntu Server 26.04 |

## Attack Diagram

```
┌──────────────┐     Brute-Force (RDP/SMB)     ┌──────────────────┐
│  Kali Linux  │ ──────────────────────────▶   │  WIN-ENDPOINT    │
│  (Attacker)  │   Hydra / CrackMapExec        │  (Victim)        │
└──────────────┘                                └────────┬─────────┘
                                                         │ Auth Logs
                                                         ▼
                                                ┌──────────────────┐
                                                │  Wazuh Server    │
                                                │  (SIEM)          │
                                                └──────────────────┘
```

## Kali Commands Used

```bash
# Hydra RDP Brute-Force
hydra -l administrator -P /usr/share/wordlists/rockyou.txt rdp://<TARGET_IP>

# Hydra SMB Brute-Force
hydra -l administrator -P /usr/share/wordlists/rockyou.txt smb://<TARGET_IP>

# CrackMapExec SMB
crackmapexec smb <TARGET_IP> -u administrator -p /usr/share/wordlists/rockyou.txt

# Medusa RDP
medusa -h <TARGET_IP> -u administrator -P /usr/share/wordlists/rockyou.txt -M rdp

# Custom wordlist generation
crunch 6 8 abcdefghijklmnopqrstuvwxyz0123456789 -o custom_wordlist.txt
```

## Screenshots

| Screenshot | Description |
|---|---|
| `screenshots/win-attack02-hydra-command.png` | Hydra brute-force execution |
| `screenshots/win-attack02-failed-logins.png` | Windows failed login events |
| `screenshots/win-attack02-wazuh-alert.png` | Wazuh brute-force alert |
| `screenshots/win-attack02-event-4625.png` | Event ID 4625 details |

## Endpoint Evidence

**Windows Security Event Log:**
```
Event ID: 4625 (Failed Logon) – Multiple instances
Event ID: 4624 (Successful Logon) – If brute-force succeeded
Event ID: 4776 (Credential Validation)
```

## Wazuh Alerts

| Alert ID | Rule ID | Description | Level | Timestamp |
|---|---|---|---|---|
| <!-- ID --> | 5710 | Multiple Windows logon failures | 10 | <!-- Time --> |
| <!-- ID --> | 5712 | Windows logon failure (brute force) | 10 | <!-- Time --> |

```json
{
  "rule": {
    "id": "5712",
    "description": "Windows brute force attempt detected",
    "level": 10
  },
  "agent": {
    "name": "WIN-ENDPOINT"
  },
  "data": {
    "win.eventdata.targetUserName": "administrator",
    "win.eventdata.ipAddress": "<KALI_IP>"
  }
}
```

## Relevant Event IDs

| Event ID | Source | Description | Relevance |
|---|---|---|---|
| 4625 | Windows Security | Failed Logon | Each brute-force attempt generates this event |
| 4624 | Windows Security | Successful Logon | Indicates successful credential compromise |
| 4776 | Windows Security | Credential Validation | NTLM authentication attempts |
| 4771 | Windows Security | Kerberos Pre-Auth Failed | Kerberos-based failures |
| 1 | Sysmon | Process Creation | Attacker tools spawning processes |

## Sysmon Events

```xml
<Event>
  <System>
    <EventID>3</EventID>
  </System>
  <EventData>
    <Data Name="SourceIp"><!-- Kali IP --></Data>
    <Data Name="DestinationPort">3389</Data>
    <Data Name="Protocol">tcp</Data>
  </EventData>
</Event>
```

## Log Analysis

### Key Findings

1. **Finding 1:** <!-- Number of failed login attempts observed -->
2. **Finding 2:** <!-- Time window of the attack -->
3. **Finding 3:** <!-- Whether any credentials were successfully compromised -->

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| **Tactic** | Credential Access |
| **Technique ID** | T1110 |
| **Technique Name** | Brute Force |
| **Sub-Technique** | T1110.001 – Password Guessing |
| **Procedure** | Adversary used Hydra from Kali Linux to perform dictionary-based brute-force attack against RDP/SMB services on the Windows endpoint |
| **Detection Opportunity** | Monitor for excessive Event ID 4625 (failed logon) from a single source IP within a short time window |

## Detection Logic

```xml
<rule id="100002" level="10" frequency="8" timeframe="120">
  <if_matched_sid>60122</if_matched_sid>
  <description>Possible RDP brute-force attack detected - multiple failed logons</description>
  <mitre>
    <id>T1110</id>
  </mitre>
</rule>
```

## Investigation Process

1. **Alert Triage:** Wazuh generated high-severity alert for multiple failed logon attempts
2. **Source Analysis:** Identified repeated failed logons from source IP `<KALI_IP>`
3. **Account Targeting:** Determined target account(s) being brute-forced
4. **Timeline Analysis:** Mapped attack duration and attempt frequency
5. **Success Check:** Verified if any successful logon (4624) followed the failed attempts
6. **Verdict:** Confirmed brute-force credential access attempt

## Findings

| # | Finding | Severity | Evidence |
|---|---|---|---|
| 1 | <!-- Finding --> | <!-- High --> | <!-- Event 4625 count --> |
| 2 | <!-- Finding --> | <!-- Critical/High --> | <!-- Success/Failure --> |

## Lessons Learned

- <!-- Account lockout policy effectiveness -->
- <!-- Detection threshold tuning requirements -->
- <!-- Response time analysis -->
- <!-- Recommendations for hardening RDP/SMB -->

---

> 📌 **Status:** 📝 Template — Update with actual findings after performing the attack simulation.
