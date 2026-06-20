# ART Validation: T1543.003 — Create Systemd Service for Persistence

## Test Details
- **ART Technique:** T1543.003
- **Test ID:** Manual (tee service file)
- **Executed On:** ubuntu-webserver (192.168.248.139)
- **Executed By:** u-webserver (auid=1000) via sudo
- **Date Validated:** 2026-06-20

## Commands Executed

    sudo tee /etc/systemd/system/tripwire-test.service << EOF
    [Unit]
    Description=Tripwire T1543.003 Test Service
    [Service]
    ExecStart=/bin/bash -c echo tripwire-test
    [Install]
    WantedBy=multi-user.target
    EOF
    sudo systemctl daemon-reload
    sudo rm /etc/systemd/system/tripwire-test.service

## Result
- **Auditd event:** Captured in /var/log/audit/audit.log YES
- **Wazuh live alert rule 101009:** Fired at level 11 YES
- **MITRE mapping:** T1543.003 correctly tagged YES
- **File path captured:** /etc/systemd/system/tripwire-test.service YES

## Alert Samples

    Rule: 101009 (level 11) -> Tripwire [T1543.003]: Systemd service file created or modified by tee
    PATH: name=/etc/systemd/system/tripwire-test.service nametype=CREATE

    Rule: 101009 (level 11) -> Tripwire [T1543.003]: Systemd service file created or modified by rm
    PATH: name=/etc/systemd/system/tripwire-test.service nametype=DELETE

## Notes
- Both CREATE and DELETE operations trigger the rule
- The audit.command field correctly identifies the creating process (tee)
- negate filter excludes known package managers: systemd|snapd|apt|dpkg|snap|update
- Auditd rules added to /etc/audit/rules.d/tripwire.rules on ubuntu-webserver
