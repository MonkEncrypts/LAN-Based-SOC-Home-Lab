# 🖥️ Windows Endpoint – Attack Simulations

This folder contains documented attack simulations performed against the **Windows 10** endpoint monitored by **Wazuh SIEM** with **Sysmon** telemetry.

## Endpoint Details

| Property | Value |
|---|---|
| **Operating System** | Windows 10 |
| **Hostname** | WIN-ENDPOINT |
| **RAM** | 3 GB |
| **CPU** | 4 vCPUs |
| **Storage** | 50 GB |
| **Network** | Bridged Adapter |
| **Monitoring Agent** | Wazuh Agent |
| **Telemetry** | Sysmon (SwiftOnSecurity Config) |

## Attack Simulations

| # | Scenario | MITRE Tactic | Technique ID | Status |
|---|---|---|---|---|
| 1 | [Nmap Reconnaissance Scan](Attacks/attack-01-nmap-scan-README.md) | Reconnaissance | T1046 | 📝 Template |
| 2 | [Brute-Force Attack](Attacks/attack-02-bruteforce-README.md) | Credential Access | T1110 | 📝 Template |
| 3 | [PowerShell Execution](Attacks/attack-03-powershell-execution-README.md) | Execution | T1059.001 | 📝 Template |
| 4 | [Persistence Mechanism](Attacks/attack-04-persistence-README.md) | Persistence | T1053 | 📝 Template |
| 5 | [Privilege Escalation](Attacks/attack-05-privilege-escalation-README.md) | Privilege Escalation | T1548 | 📝 Template |

> 📌 **Note:** Update the status column as you complete each simulation (📝 Template → ✅ Complete).

## Key Windows Event IDs

| Event ID | Source | Description |
|---|---|---|
| 4624 | Security | Successful Logon |
| 4625 | Security | Failed Logon |
| 4688 | Security | Process Creation |
| 4720 | Security | User Account Created |
| 4732 | Security | Member Added to Security Group |
| 7045 | System | Service Installed |
| 1 | Sysmon | Process Creation |
| 3 | Sysmon | Network Connection |
| 11 | Sysmon | File Created |
| 12/13 | Sysmon | Registry Event |
| 22 | Sysmon | DNS Query |
