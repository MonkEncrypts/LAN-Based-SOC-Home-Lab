# 🐧 Ubuntu Endpoint – Attack Simulations

This folder contains documented attack simulations performed against the **Ubuntu Desktop 26.04** endpoint monitored by **Wazuh SIEM**.

## Endpoint Details

| Property | Value |
|---|---|
| **Operating System** | Ubuntu Desktop 26.04 |
| **Hostname** | UBUNTU-ENDPOINT |
| **RAM** | 3 GB |
| **CPU** | 3 vCPUs |
| **Storage** | 25 GB |
| **Network** | Bridged Adapter |
| **Monitoring Agent** | Wazuh Agent |
| **Key Log Sources** | syslog, auth.log, kern.log, audit.log |

## Attack Simulations

| # | Scenario | MITRE Tactic | Technique ID | Status |
|---|---|---|---|---|
| 1 | [Nmap Reconnaissance Scan](Attacks/attack-01-nmap-scan-README.md) | Reconnaissance | T1046 | 📝 Template |
| 2 | [SSH Brute-Force Attack](Attacks/attack-02-ssh-bruteforce-README.md) | Credential Access | T1110.001 | 📝 Template |
| 3 | [Privilege Escalation](Attacks/attack-03-privilege-escalation-README.md) | Privilege Escalation | T1548.003 | 📝 Template |
| 4 | [Persistence Mechanism](Attacks/attack-04-persistence-README.md) | Persistence | T1053.003 | 📝 Template |
| 5 | [Suspicious File Activity](Attacks/attack-05-suspicious-file-activity-README.md) | Collection | T1005 | 📝 Template |

> 📌 **Note:** Update the status column as you complete each simulation (📝 Template → ✅ Complete).

## Key Linux Log Sources

| Log File | Description |
|---|---|
| `/var/log/auth.log` | Authentication events (SSH, sudo, login) |
| `/var/log/syslog` | General system messages |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/audit/audit.log` | Audit framework events |
| `/var/log/wtmp` | Login records (use `last` command) |
| `/var/log/btmp` | Failed login records (use `lastb` command) |
| `/var/log/cron.log` | Cron job execution logs |
| `/var/ossec/logs/ossec.log` | Wazuh agent logs |
