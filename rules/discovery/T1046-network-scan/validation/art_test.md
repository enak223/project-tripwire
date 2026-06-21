# ART Validation: T1046 — Network Service Discovery

## Test Details
- **ART Technique:** T1046
- **Test ID:** Manual (Nmap port scan)
- **Executed From:** Kali Linux (192.168.248.130) — attacker
- **Target:** ubuntu-webserver (192.168.248.139)
- **Detected By:** Suricata on ubuntuai (192.168.248.20) monitoring ens37
- **Date Validated:** 2026-06-20

## Commands Executed

    nmap -sS -p 1-1000 192.168.248.139
    nmap -sS -sV --open -T4 -p- 192.168.248.139

## Result
- **Suricata ET SCAN signature fired:** YES — 21 alerts
- **Signature:** ET SCAN Possible Nmap User-Agent Observed (SID 2024364)
- **Wazuh rule 86601 overwrite:** Fired at level 10 YES
- **MITRE mapping:** T1046 correctly tagged YES
- **Source IP captured:** 192.168.248.130 YES

## Logtest Confirmation

    Phase 3:
    id: 86601
    level: 10
    description: Tripwire [T1046]: Network scan detected — ET SCAN Possible Nmap User-Agent Observed from 192.168.248.130
    mitre.id: T1046
    mitre.tactic: Discovery
    mitre.technique: Network Service Discovery

## Engineering Notes
- Basic SYN scan (-sS -p 1-1000) did NOT trigger ET SCAN signatures
- Service version scan (-sV) triggers Nmap HTTP user-agent signature via NSE
- Suricata must be monitoring the correct interface (ens37 = 192.168.248.0/24)
- UFW on ubuntu-webserver must allow port 22/tcp from Kali for SSH scan detection
- Wazuh 4.12 file load order constraint: overwrite="yes" required on built-in 86601
