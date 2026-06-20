# RUNBOOK: T1070.003 — Indicator Removal: Clear Command History
**Rule ID:** 101004 | **Level:** 9 (High) | **Tactic:** Defense Evasion
**Platform:** Linux | **Log Source:** Auditd -> Wazuh

---

## Alert Summary
The .bash_history file was written to or modified. Attackers clear or truncate
command history to remove forensic evidence of commands executed during an intrusion,
making incident response and timeline reconstruction more difficult.

## What Triggered It
Auditd detected a write syscall against /home/<user>/.bash_history or
/root/.bash_history, tagged with key tripwire_clear_history. The watch is
scoped specifically to history files to minimize false positive noise.

## Detection Architecture Notes
- Auditd watch: -w /home/u-webserver/.bash_history -p wa
- Base rule 101003 (level 0, noalert) catches the key match silently
- Alert rule 101004 fires only when audit.command is NOT a known-good process
- Wazuh CDB list updated: tripwire_clear_history:write
- Initial broad /home watch caused FP noise from gvfsd-metadata — narrowed to .bash_history only

## False Positive Scenarios
- Bash writing history on legitimate session close
- Text editors creating backup files in home directory
- cp/mv operations touching history files during backup

## Triage Steps
1. Check audit.command — what process wrote to .bash_history?
2. Check audit.auid — which user's history was modified?
3. Check timing — was this during an active session or after hours?
4. Check file size: wc -l ~/.bash_history — was it truncated to 0?
5. Check auth logs for the session around the same timestamp
6. Look for other defense evasion indicators in the same window

## Escalation Criteria
Escalate immediately if:
- History file was truncated to 0 bytes
- Modification was made by a non-shell process
- Event follows privilege escalation or lateral movement alerts
- Multiple users history files cleared in a short window

## References
- https://attack.mitre.org/techniques/T1070/003/
- Validated with: cat /dev/null > ~/.bash_history
