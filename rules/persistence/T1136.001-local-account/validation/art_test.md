# ART Validation: T1136.001 — Local Account Creation

## Test Details
- **ART Technique:** T1136.001
- **Test ID:** Manual (useradd)
- **Executed On:** ubuntu-webserver (192.168.248.139)
- **Executed By:** u-webserver (auid=1000) via sudo
- **Date Validated:** 2026-06-19

## Commands Executed

    sudo useradd tripwire-test
    sudo useradd tripwire-test2
    sudo useradd tripwire-test3
    sudo useradd tripwire-test4

## Result
- **Syslog event:** Captured via journald → Wazuh agent YES
- **Wazuh built-in rule 5902:** Fired at level 8 YES
- **Wazuh Tripwire rule 101002:** Fired at level 10 YES
- **MITRE mapping:** T1136.001 correctly tagged YES

## Alert Sample

    Rule: 101002 (level 10) -> Tripwire [T1136.001]: New local user account created
    User: tripwire-test4
    Jun 19 23:34:46 u-webserver useradd[48510]: new user: name=tripwire-test4, UID=1005

## Notes
- Detection relies on Wazuh built-in syslog/journald parsing (rule 5902)
- No auditd dependency — journald captures useradd natively
- Clean detection with no false positives observed
