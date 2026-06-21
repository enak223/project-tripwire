# RUNBOOK: T1046 — Network Service Discovery
**Rule ID:** 86601 (overwrite) | **Level:** 10 (High) | **Tactic:** Discovery
**Platform:** Linux | **Log Source:** Suricata -> Wazuh (GhostNet integration)

---

## Alert Summary
A network port scan was detected from an internal or external host. Attackers
perform network service discovery to identify open ports, running services,
and potential attack vectors before exploitation. Nmap is the most common tool.

## What Triggered It
Suricata (running on ubuntuai, monitoring ens37) detected packets matching
ET SCAN signatures — specifically the Nmap Scripting Engine HTTP user-agent
or characteristic SYN scan patterns from the source IP.

## Detection Architecture Notes
- Suricata runs on ubuntuai (192.168.248.20) monitoring ens37 (192.168.248.0/24)
- ET Open ruleset provides scan detection signatures
- GhostNet integration forwards Suricata EVE JSON to Wazuh
- Rule 86601 overwrite="yes" adds T1046 MITRE mapping
- Wazuh 4.12 file load order constraint requires overwrite approach

## False Positive Scenarios
- Authorized vulnerability scans by security team (schedule during maintenance)
- Network monitoring tools (Nagios, Zabbix) performing health checks
- IT asset discovery tools running legitimately
- Penetration testing engagements — verify against change calendar

## Triage Steps
1. Identify the scanning source IP: check src_ip field in alert
2. Determine if source is internal or external
3. Cross-reference with change calendar — is a scan authorized right now?
4. Check what ports were targeted: review Suricata flow logs for that src_ip
5. Check if exploitation attempts followed the scan: look for connection attempts
   to discovered open ports in the minutes after the scan
6. If unauthorized: block the source IP at the firewall

## Escalation Criteria
Escalate if:
- Scan comes from an unexpected internal host (lateral movement indicator)
- Scan immediately precedes exploitation attempts
- Scan targets critical infrastructure (domain controllers, databases)
- Multiple scan types observed (SYN, UDP, version detection) in sequence

## References
- https://attack.mitre.org/techniques/T1046/
- Validated with: nmap -sS -sV --open -T4 -p- 192.168.248.139
- Tool: Nmap 7.99 from Kali Linux (192.168.248.130)
- Suricata signature: ET SCAN Possible Nmap User-Agent Observed (SID 2024364)
