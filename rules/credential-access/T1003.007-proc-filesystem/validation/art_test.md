# ART Validation: T1003.007 — /proc Filesystem Credential Access

## Test Details
- **ART Technique:** T1003.007
- **Test ID:** Manual (cat /proc/1/environ)
- **Executed On:** ubuntu-webserver (192.168.248.139)
- **Executed By:** u-webserver (auid=1000) via sudo
- **Date Validated:** 2026-06-19

## Command Executed

    sudo cat /proc/1/environ > /dev/null

## Expected Behavior
Auditd catches the openat() syscall against /proc/1/environ and tags it
with key tripwire_proc_cred. Wazuh ingests the auditd log and matches rule 101001.

## Result
- **Auditd:** Event captured in /var/log/audit/audit.log YES
- **Wazuh logtest:** Rule 101001 fires at level 12 YES
- **Wazuh live alert:** Confirmed in alerts.log YES
- **MITRE mapping:** T1003.007 correctly tagged YES

## Wazuh Logtest Output (Phase 3)

    id: 101001
    level: 12
    description: Tripwire [T1003.007]: /proc filesystem read — possible credential harvesting
    mitre.id: T1003.007
    mitre.tactic: Credential Access
    mitre.technique: Proc Filesystem
    Alert to be generated.

## Tuning Notes
- Wazuh 4.12 has a confirmed bug: audit.exe truncated at hyphen characters
  e.g. /usr/libexec/ptyxis-agent decoded as /usr/libexec/ptyxis
  Bug reference: https://github.com/wazuh/wazuh/issues/18477 (fixed in 4.14)
- ubuntu-webserver runs GNOME desktop generating background /proc reads
- Workaround: use audit.command field (comm= value) for process identification
- Auditd rule scoped to /proc/1/environ to reduce noise from desktop processes
- Wazuh CDB list updated: echo tripwire_proc_cred:write >> /var/ossec/etc/lists/audit-keys
