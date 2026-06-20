# RUNBOOK: T1136.001 — Create Account: Local Account
**Rule ID:** 101002 | **Level:** 10 (High) | **Tactic:** Persistence
**Platform:** Linux | **Log Source:** Syslog/Journald → Wazuh

---

## Alert Summary
A new local user account was created on the system. While routine in some contexts,
unauthorized account creation is a primary persistence technique — attackers create
accounts to ensure continued access to compromised systems, independent of the
initial intrusion vector.

## What Triggered It
Wazuh rule 5902 (built-in) detected a `useradd` syslog entry indicating a new user
was added to the system. Tripwire rule 101002 chains from 5902 to add MITRE ATT&CK
tagging and increase alert visibility.

## False Positive Scenarios
- System administrators creating legitimate service accounts
- Software packages creating system users during installation (e.g. `www-data`, `mysql`)
- Automated provisioning pipelines

## Triage Steps
1. Identify the new username from the `User` field in the alert
2. Check who ran the command: look for preceding sudo/auth log entries
3. Verify if the account creation was authorized — check change tickets or admin logs
4. Check if the new account has been used: `last <username>`, `grep <username> /var/log/auth.log`
5. Check if the account was added to privileged groups: `groups <username>`
6. Look for off-hours timing — creation between 00:00–06:00 is higher risk

## Escalation Criteria
Escalate immediately if:
- Account was created outside business hours with no change ticket
- Account was immediately added to sudo or admin groups
- Account name mimics a system account (e.g. `sshd2`, `root2`)
- Creation followed other suspicious activity (lateral movement, privilege escalation)

## References
- https://attack.mitre.org/techniques/T1136/001/
- Validated with: sudo useradd tripwire-test (ART manual simulation)
