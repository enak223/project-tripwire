# RUNBOOK: T1110.001 — Brute Force: Password Guessing (SSH)
**Rule IDs:** 101006 (non-existent user) / 101007 (valid user)
**Level:** 14 (Critical) | **Tactic:** Credential Access
**Platform:** Linux | **Log Source:** Syslog/Journald -> Wazuh

---

## Alert Summary
Multiple failed SSH authentication attempts from the same source IP were detected
within a short timeframe. This indicates an automated brute force attack using
tools like Hydra, Medusa, or Ncrack to guess SSH credentials. If successful,
the attacker gains shell access to the system.

## What Triggered It
Wazuh built-in rule 5712 (non-existent user) or 5720 (valid user) detected 8+
failed SSH logins from the same IP within 120 seconds. Tripwire rules 101006/101007
chain from these to add T1110.001 MITRE tagging and escalate severity to level 14.

## Detection Architecture Notes
- Ubuntu 22.04 OpenSSH 9.x splits into sshd (listener) and sshd-session (handler)
- Failed auth messages come from sshd-session, not sshd
- Custom decoder (wazuh/decoders/tripwire_decoders.xml) maps sshd-session to
  the sshd decoder chain so built-in SSH rules fire correctly
- UFW must allow port 22/tcp from attacker networks for detection to work

## False Positive Scenarios
- Legitimate users repeatedly mistyping passwords
- Automated scripts with stale/wrong SSH credentials
- CI/CD pipelines misconfigured to hit wrong SSH endpoint

## Triage Steps
1. Identify the attacking IP: check Src IP field in the alert
2. Check geolocation/ASN of the source IP — is it expected?
3. Check if any login SUCCEEDED after the brute force:
   grep "Accepted" /var/log/auth.log | grep <src_ip>
4. Check journalctl for successful logins: journalctl -u ssh | grep Accepted
5. If login succeeded, immediately check for new accounts, cron jobs, and SSH keys added
6. Block the IP: sudo ufw deny from <src_ip> to any port 22

## Escalation Criteria
Escalate immediately if:
- Any successful login followed the brute force from the same IP
- Brute force is targeting multiple accounts simultaneously
- Source IP belongs to a known threat actor or TOR exit node
- Attack volume exceeds 100 attempts per minute

## References
- https://attack.mitre.org/techniques/T1110/001/
- Validated with: Hydra v9.7 from Kali Linux (192.168.248.130)
- Tool: hydra -l fakeuser -P rockyou.txt ssh://192.168.248.139 -t 4 -f
