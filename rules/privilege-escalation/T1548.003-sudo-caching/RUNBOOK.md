# RUNBOOK: T1548.003 — Abuse Elevation: Sudo and Sudo Caching
**Rule IDs:** 5402 (level 3) / 5403 (level 4) / 101011 (level 3)
**Tactic:** Privilege Escalation / Defense Evasion
**Platform:** Linux | **Log Source:** Syslog/Journald -> Wazuh

---

## Alert Summary
A user executed a command as root via sudo. While routine in many environments,
unauthorized sudo usage is a primary privilege escalation path on Linux systems.
Attackers who gain access as a low-privilege user will immediately attempt to
enumerate and abuse sudo permissions.

## Detection Coverage

### Rule 5402 — Successful sudo to ROOT (level 3)
Fires when sudo successfully executes a command as root. Logs the full
COMMAND= field showing exactly what was run. Already maps T1548.003.

### Rule 5403 — First time user executed sudo (level 4)
Fires the first time a specific user runs sudo. Higher severity because
it may indicate a new account or attacker trying sudo for the first time.

### Rule 101011 — Tripwire sudo restore (level 3)
Restores standard sudo-to-root detection after rule 5402 was partially
overwritten to catch T1562.001 (security tool disable via sudo).

## Key Detection Gap — sudo -l Enumeration
sudo -l (list available sudo permissions) does NOT generate a syslog entry
on Ubuntu 22.04 because it doesn't execute a command. Auditd execve watching
for /usr/bin/sudo also doesn't reliably catch it due to PAM/auth pathway.
This is a known detection gap documented for transparency.

## False Positive Scenarios
- Legitimate administrators running sudo for authorized tasks
- Automated scripts using sudo for package management or configuration
- CI/CD pipelines requiring elevated privileges

## Triage Steps
1. Identify the user and command: check srcuser and command fields
2. Is the COMMAND expected for this user at this time?
3. Check if the user was recently created (correlate with T1136.001 alerts)
4. Check if sudo followed a brute force (correlate with T1110.001 alerts)
5. Review all commands run via sudo in the session: grep srcuser alerts.log
6. Check for privilege persistence: crontabs, SSH keys, new accounts added

## Escalation Criteria
Escalate if:
- Sudo used by a newly created or unexpected account
- Sudo execution immediately follows SSH brute force success
- Sensitive commands: passwd, useradd, crontab, systemctl, chmod 777
- Sudo used outside business hours with no change ticket

## References
- https://attack.mitre.org/techniques/T1548/003/
- Wazuh built-in rules: 5400, 5402, 5403
- Validated with: sudo id && sudo whoami on ubuntu-webserver
