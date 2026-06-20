# RUNBOOK: T1562.001 — Impair Defenses: Disable or Modify Tools
**Rule IDs:** 101010 (agent stop) / 5402 overwrite (sudo disable)
**Level:** 14 (Critical) | **Tactic:** Defense Evasion
**Platform:** Linux | **Log Source:** Wazuh internal + Syslog/Journald

---

## Alert Summary
A security monitoring tool was stopped, disabled, or impaired. This is a
critical indicator of an attacker attempting to blind the security team
before or during an intrusion. Disabling auditd removes syscall-level
visibility; stopping the Wazuh agent cuts off log forwarding entirely.

## Detection Layers

### Layer 1: Wazuh Agent Stopped (Rule 101010)
Fires when the Wazuh manager detects that the ubuntu-webserver agent has
sent a stop signal (ossec: Agent stopped). This is the most direct detection
— the agent itself reports it is shutting down before going offline.

### Layer 2: Sudo Security Tool Disable (Rule 5402 overwrite)
Fires when sudo is used to run systemctl stop/disable/mask against a
security service. Implemented via overwrite="yes" on built-in rule 5402
because Wazuh child rules in custom files cannot chain from built-in
parent rules due to file load order constraints (confirmed Wazuh 4.12 behavior).

## What Triggered It
- Layer 1: ossec internal message "Agent stopped: ubuntu-webserver"
- Layer 2: Sudo log showing COMMAND=.*systemctl stop/disable auditd|wazuh

## False Positive Scenarios
- Authorized maintenance stopping services for upgrades
- Automated deployment scripts restarting security tools
- Security team testing active response capabilities

## Triage Steps
1. Immediately check if the agent is still offline: agent_control -l
2. Check what stopped the agent: grep "systemctl stop wazuh" /var/log/auth.log
3. Check if auditd is still running: systemctl status auditd
4. Review all activity from the user who stopped the service
5. Check for other T1562.001 indicators: new firewall rules, disabled services
6. If agent is offline, check the endpoint directly via out-of-band access

## Escalation Criteria
Escalate immediately — this is always critical:
- Any unplanned security tool disable is a Tier 1 incident
- Correlate with preceding lateral movement, privilege escalation, or data access
- If agent remains offline for more than 5 minutes without explanation: page on-call

## References
- https://attack.mitre.org/techniques/T1562/001/
- Wazuh rule 506: Agent stopped
- Wazuh rule file load order issue: github.com/wazuh/wazuh/issues/20146
