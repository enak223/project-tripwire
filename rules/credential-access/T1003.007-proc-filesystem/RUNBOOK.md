# RUNBOOK: T1003.007 — OS Credential Dumping via /proc Filesystem
**Rule ID:** 101001 | **Level:** 12 (Critical) | **Tactic:** Credential Access
**Platform:** Linux | **Log Source:** Auditd → Wazuh

---

## Alert Summary
A process read a file under `/proc/` that is monitored for credential harvesting activity.
The `/proc/<pid>/environ` file exposes the complete environment of a running process,
which frequently contains passwords, API keys, database credentials, and session tokens
passed via environment variables.

## What Triggered It
Auditd caught a syscall (openat/readlink) against a monitored `/proc` path,
tagged with audit key `tripwire_proc_cred`. The rule fires when the key matches
in the Wazuh auditd event stream.

## Known Limitations (Homelab Context)
- Wazuh 4.12 has a confirmed bug where `audit.exe` is truncated at hyphen characters
  (e.g. `/usr/libexec/ptyxis-agent` decodes as `/usr/libexec/ptyxis`). Fixed in 4.14.
- ubuntu-webserver runs a GNOME desktop which generates background `/proc` reads
  from processes like `ptyxis-agent` and `gsd-housekeeping`. The auditd rule is
  scoped to `/proc/1/environ` specifically to minimize noise.
- Rule is logtest-confirmed working. Live FP tuning ongoing as desktop processes
  are baselined.

## False Positive Scenarios
- `sudo` reading `/proc/self/loginuid` or `/proc/self/fd/*` during privilege escalation
- `auditctl` reading `/proc/<pid>/status` during rule reload
- Security monitoring agents inspecting process state
- Any process reading `/proc/self/*` (its own process info)

## Triage Steps
1. Check `audit.command` field — is the process name expected/known?
2. Check `audit.auid` — which user initiated this?
3. Check the exact path in the alert — `/proc/1/environ` is highest severity;
   `/proc/self/status` is likely benign
4. Check for preceding suspicious activity from the same user/session
5. Cross-reference with T1136.001 or T1053.003 alerts in the same timeframe

## Escalation Criteria
Escalate immediately if:
- The reading process is not a known system or monitoring tool
- The target path is `/proc/1/environ` or another high-value process
- Alert follows recent account creation, privilege escalation, or lateral movement
- Multiple credential access alerts in a short window

## References
- https://attack.mitre.org/techniques/T1003/007/
- https://github.com/enak223/project-ghostnet (GhostNet ART validation)
- Wazuh bug: https://github.com/wazuh/wazuh/issues/18477
