# ART Validation: T1082 — System Information Discovery

## Test Details
- **ART Technique:** T1082
- **Test ID:** Manual (uname, id, whoami, hostname)
- **Executed On:** ubuntu-webserver (192.168.248.139)
- **Executed By:** u-webserver (auid=1000)
- **Date Validated:** 2026-06-21

## Commands Executed

    uname -a && id && whoami && hostname

## Result
- **Auditd event:** Captured for each binary YES
- **Wazuh live alert rule 101014:** Fired at level 5 YES
- **MITRE mapping:** T1082 correctly tagged YES
- **Command and user captured:** hostname by auid=1000 YES

## Alert Sample

    Rule: 101014 (level 5) -> Tripwire [T1082]: System information discovery — hostname executed by 1000
    exe=/usr/bin/hostname key=tripwire_sysinfo
    AUID=u-webserver UID=u-webserver

## Auditd Rules Deployed

    -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/uname -F auid>=1000 -F auid!=-1 -k tripwire_sysinfo
    -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/id -F auid>=1000 -F auid!=-1 -k tripwire_sysinfo
    -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/whoami -F auid>=1000 -F auid!=-1 -k tripwire_sysinfo
    -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/hostname -F auid>=1000 -F auid!=-1 -k tripwire_sysinfo

## Notes
- auid>=1000 filter ensures only interactive users trigger the rule
- auid!=-1 filter excludes kernel/system processes
- Rule fires individually for each command executed
- Clustering of multiple commands in a short window indicates attacker pattern
