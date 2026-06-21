# RUNBOOK: T1082 — System Information Discovery
**Rule IDs:** 101013 (base) / 101014 (alert)
**Level:** 5 (Low-Medium) | **Tactic:** Discovery
**Platform:** Linux | **Log Source:** Auditd -> Wazuh

---

## Alert Summary
A system information discovery command (uname, id, whoami, hostname) was executed
by an interactive user. While common in normal use, in an intrusion context these
indicate an attacker enumerating the compromised system to understand its
environment, privileges, and OS version.

## What Triggered It
Auditd caught an execve syscall for a watched binary from a user with
auid >= 1000 (interactive user). Tagged with key tripwire_sysinfo.

## False Positive Scenarios
- Shell prompt configurations running id or whoami on login
- System monitoring scripts checking kernel version via uname
- Developers running these commands routinely

## Triage Steps
1. Check audit.auid — which user ran the command?
2. Check timing — right after a login or SSH connection?
3. Look for clustering — multiple discovery commands in rapid succession?
4. Correlate: T1046 before + T1082 + T1136.001 after = full attack pattern

## Escalation Criteria
Escalate if:
- Commands run in sequence within 60 seconds
- Commands follow a successful brute force (T1110.001)
- Commands run by an unexpected or newly created account

## References
- https://attack.mitre.org/techniques/T1082/
- Validated with: uname -a && id && whoami && hostname
