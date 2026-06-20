# ART Validation: T1053.003 — Cron Job Persistence

## Test Details
- **ART Technique:** T1053.003
- **Test ID:** Manual (crontab modification)
- **Executed On:** ubuntu-webserver (192.168.248.139)
- **Executed By:** u-webserver (auid=1000)
- **Date Validated:** 2026-06-19

## Commands Executed

    (crontab -l 2>/dev/null; echo "* * * * * echo tripwire-test") | crontab -
    (crontab -l 2>/dev/null; echo "* * * * * echo tripwire-test2") | crontab -
    (crontab -l 2>/dev/null; echo "* * * * * echo tripwire-test3") | crontab -
    (crontab -l 2>/dev/null; echo "* * * * * echo tripwire-test4") | crontab -

## Result
- **Syslog event:** Captured via journald -> Wazuh agent YES
- **Wazuh built-in rule 2832:** Fired at level 5 YES
- **Wazuh Tripwire rule 101005:** Fired at level 10 YES
- **MITRE mapping:** T1053.003 correctly tagged YES

## Alert Sample

    Rule: 101005 (level 10) -> Tripwire [T1053.003]: Crontab entry modified
    Jun 19 23:34:46 u-webserver crontab[48516]: (u-webserver) REPLACE (u-webserver)

## Notes
- Detection relies on Wazuh built-in syslog/journald parsing (rule 2832)
- No auditd dependency — journald captures crontab activity natively
- Clean detection with no false positives observed
- Auditd rules for /etc/cron.d also in place as secondary coverage
