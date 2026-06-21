# RUNBOOK: T1082 — System Information Discovery
**Rule IDs:** 101013 (base) / 101014 (alert)
**Level:** 5 (Low-Medium) | **Tactic:** Discovery
**Platform:** Linux | **Log Source:** Auditd -> Wazuh

---

## Alert Summary
A system information discovery command (uname, id, whoami, hostname) was executed
by an interactive user. While these commands are common in normal use, in the context
of an intrusion they indicate an attacker enumerating the compromised system to
understand its environment, privileges, and OS version.

## What Triggered It
Auditd caught an execve syscall for a watched binary (uname/id/whoami/hostname)
from a user with auid >= 1000 (interactive user, not a system process).
The event was tagged with key tripwire_sysinfo.

## False Positive Scenarios
- Shell prompt configurations (PS1) that run id or whoami on every login
- System monitoring scripts running uname -r to check kernel version
- Health check scripts run by automation tools
- Developers running these commands routinely

## Triage Steps
1. Check audit.auid — which user ran the command?
2. Check the session context — was this during an active SSH session?
3. Check timing — was this right after a login or SSH connection?
4. Look for clustering — did multiple discovery commands run in sequence?
   (uname + id + whoami + hostname in rapid succession = high confidence)
5. Correlate with other alerts from the same session/user:
   - T1046 (port scan) before this = reconnaissance pattern
   - T1136.001 (account creation) after this = persistence pattern

## Escalation Criteria
Escalate if:
- Commands run in sequence within 60 seconds (attacker enumeration pattern)
- Commands follow a successful brute force (T1110.001) from same source
- Commands run by an unexpected or newly created account
- Commands preceded by lateral movement indicators

## References
- https://attack.mitre.org/techniques/T1082/
- Validated with: uname -a && id && whoami && hostname
