# ART Validation: T1548.003 — Sudo and Sudo Caching

## Test Details
- **ART Technique:** T1548.003
- **Test ID:** Manual (sudo execution)
- **Executed On:** ubuntu-webserver (192.168.248.139)
- **Executed By:** u-webserver (auid=1000)
- **Date Validated:** 2026-06-21

## Commands Executed

    sudo id
    sudo whoami
    sudo -l

## Result
- **Syslog event:** Captured via journald -> Wazuh agent YES
- **Wazuh rule 5402:** Fired at level 3 YES
- **MITRE mapping:** T1548.003 correctly tagged YES
- **Command captured in alert:** COMMAND=/usr/bin/id YES

## Alert Sample

    Rule: 5402 (level 3) -> Successful sudo to ROOT executed.
    User: root
    Jun 21 13:xx:xx u-webserver sudo[xxxx]: u-webserver : TTY=/dev/pts/0 ;
    PWD=/home/u-webserver ; USER=root ; COMMAND=/usr/bin/id

## Detection Gap — sudo -l
sudo -l does NOT generate a syslog entry on Ubuntu 22.04 because it only
lists permissions without executing a command. Auditd execve watching for
/usr/bin/sudo also does not reliably catch it due to the PAM authentication
pathway used by sudo on Ubuntu 22.04.

Workaround: Monitor for sudo -l via pam_tty_audit if required in production.

## Notes
- Detection relies on Wazuh built-in syslog/journald parsing (rule 5402/5403)
- No custom auditd rule needed for sudo-to-root execution
- T1548.003 MITRE mapping already present in built-in Wazuh rules
- Tripwire rule 101011 restores this detection alongside T1562.001 overwrite
