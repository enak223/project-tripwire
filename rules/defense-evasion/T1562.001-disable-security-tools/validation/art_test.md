# ART Validation: T1562.001 — Disable or Modify Security Tools

## Test Details
- **ART Technique:** T1562.001
- **Test ID:** Manual (systemctl stop wazuh-agent)
- **Executed On:** ubuntu-webserver (192.168.248.139)
- **Executed By:** u-webserver (auid=1000) via sudo
- **Date Validated:** 2026-06-20

## Commands Executed

    sudo systemctl stop wazuh-agent
    sudo systemctl start wazuh-agent  # cleanup

## Result
- **Wazuh rule 506:** Fired — Agent stopped YES
- **Wazuh Tripwire rule 101010:** Fired at level 14 YES
- **MITRE mapping:** T1562.001 correctly tagged YES
- **Agent disconnection confirmed:** agent_control -l shows Disconnected YES

## Alert Sample

    Rule: 101010 (level 14) -> Tripwire [T1562.001]: Wazuh agent on ubuntu-webserver stopped
    ossec: Agent stopped: ubuntu-webserver->any

## Engineering Notes

### Wazuh Child Rule File Load Order Bug
Custom rules in /var/ossec/etc/rules/ cannot chain from built-in rules
in /var/ossec/ruleset/rules/ as child rules (via if_sid) because Wazuh
processes child rules only for rules in the same file or loaded before.

Confirmed on Wazuh 4.12.0: github.com/wazuh/wazuh/issues/20146
Workaround used: Chain from rule 506 (internal Wazuh message, not syslog)
which works because 506 is in a different rule processing path.

### Layer 2 Detection (sudo disable)
Rule 5402 overwrite="yes" implemented to catch sudo-based security tool
disabling. The overwrite is necessary because the built-in 5402/5403 rules
intercept the event before custom child rules can evaluate it.
