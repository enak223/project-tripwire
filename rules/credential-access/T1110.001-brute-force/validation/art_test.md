# ART Validation: T1110.001 — SSH Brute Force Password Guessing

## Test Details
- **ART Technique:** T1110.001
- **Test ID:** Manual (Hydra SSH brute force)
- **Executed On:** ubuntu-webserver (192.168.248.139) — target
- **Executed From:** Kali Linux (192.168.248.130) — attacker
- **Date Validated:** 2026-06-20

## Command Executed

    hydra -l fakeuser -P /usr/share/wordlists/rockyou.txt ssh://192.168.248.139 -t 4 -f

## Pre-requisites Discovered During Testing
- UFW on ubuntu-webserver was blocking port 22/tcp from IPv4 (only IPv6 rule existed)
- Fix: sudo ufw allow 22/tcp on ubuntu-webserver
- OpenSSH 9.x penalty system blocked Kali IP after repeated failures — fix: sudo systemctl restart ssh
- rockyou.txt was compressed on Kali — fix: sudo gunzip /usr/share/wordlists/rockyou.txt.gz
- Ubuntu 22.04 splits SSH into sshd and sshd-session processes
- Custom decoder required to map sshd-session to Wazuh sshd decoder chain

## Result
- **Journald:** Failed auth events captured YES
- **Wazuh rule 5710:** Fired for each individual failed attempt YES
- **Wazuh rule 5712:** Fired at brute force threshold (8 attempts/120s) YES
- **Wazuh Tripwire rule 101006:** Fired at level 14 YES
- **MITRE mapping:** T1110.001 correctly tagged YES

## Alert Sample

    Rule: 5710 (level 5) -> sshd: Attempt to login using a non-existent user
    Src IP: 192.168.248.130

    Rule: 5712 (level 10) -> sshd: brute force trying to get access to the system. Non existent user.
    Src IP: 192.168.248.130

    Rule: 101006 (level 14) -> Tripwire [T1110.001]: SSH brute force — non-existent user from 192.168.248.130
    Src IP: 192.168.248.130

## Engineering Notes
- sshd-session program_name matches ^sshd pattern in Wazuh decoder (confirmed via logtest)
- Custom tripwire_decoders.xml not strictly required but documents the Ubuntu 22.04 SSH split
- Detection is fully functional end-to-end without any decoder modification
