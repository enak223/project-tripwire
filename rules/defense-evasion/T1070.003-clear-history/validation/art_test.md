# ART Validation: T1070.003 — Clear Command History

## Test Details
- **ART Technique:** T1070.003
- **Test ID:** Manual (truncate .bash_history)
- **Executed On:** ubuntu-webserver (192.168.248.139)
- **Executed By:** u-webserver (auid=1000)
- **Date Validated:** 2026-06-19

## Commands Executed

    history -c
    cat /dev/null > ~/.bash_history

## Result
- **Auditd event:** Captured in /var/log/audit/audit.log YES
- **ausearch:** key=tripwire_clear_history, comm=bash, path=/home/u-webserver/.bash_history YES
- **Wazuh live alert rule 101004:** Fired at level 9 YES
- **MITRE mapping:** T1070.003 correctly tagged YES

## Alert Sample

    Rule: 101004 (level 9) -> Tripwire [T1070.003]: .bash_history modified
    comm="bash" exe="/usr/bin/bash" key="tripwire_clear_history"
    PATH: name="/home/u-webserver/.bash_history"

## Tuning Notes
- Initial auditd rule watched all of /home — caused FP noise from gvfsd-metadata
- Narrowed to watch only .bash_history files specifically
- Wazuh CDB list required: echo tripwire_clear_history:write >> /var/ossec/etc/lists/audit-keys
- Wazuh 4.12 audit.exe bug workaround: use audit.command field for negate filtering
