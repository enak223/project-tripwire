# Bonus Case Study: Real-World T1136.001 Discovery

## Overview

During a routine audit of the homelab environment, an undocumented privileged account
was discovered on the ubuntu-webserver (192.168.248.139) detection target — an account
with no memory of being created, no log trail, and active anti-forensics in place.
This is a real-world T1136.001 scenario, not a simulated one.

---

## Discovery

While checking system accounts, the following entry appeared in `/etc/passwd`:
butter:x:1002:0::/root:/bin/bash

Red flags immediately present:
- **GID 0** — root group membership
- **Home directory `/root`** — not a standard user home path
- **No recollection** of ever creating this account

Additional suspicious accounts found in the same audit:
art:x:1001:1001::/home/art:/bin/sh

evil_user:x:997:100:evil_account:/home/evil_user:/bin/bash

---

## Forensic Triage

### Step 1 — Log Analysis
Searched all available auth.log files including rotated and gzipped archives:

```bash
sudo grep butter /var/log/auth.log
sudo grep butter /var/log/auth.log.1
sudo zgrep butter /var/log/auth.log.2.gz
sudo grep useradd /var/log/auth.log.1
sudo zgrep useradd /var/log/auth.log.2.gz
```

**Result:** No useradd events for `butter` in any log file. Creation event had been
consumed by log rotation, indicating the account was created weeks or months prior.

### Step 2 — Password Age Check
```bash
sudo chage -l butter
```

**Result:**
Last password change: Jun 14, 2026

Password expires: never

Account expires: never

Password was set on June 14, 2026 — correlating with active GhostNet session activity
on the homelab that day.

### Step 3 — SSH Access Check
```bash
sudo ls -la /root/.ssh/
sudo cat /root/.ssh/authorized_keys
```

**Result:** `authorized_keys` was empty (0 bytes). No SSH backdoor established.

### Step 4 — Shadow File Check
```bash
sudo grep -E "butter|evil_user|art" /etc/shadow | cut -d: -f1,2
```

**Result:**
art:!          <- no password (never activated)

evil_user:!    <- no password (never activated)

butter:yy
y...  <- ACTIVE PASSWORD HASH

Critical finding: `butter` had a real password set. `art` and `evil_user` did not.
None of the operator's known passwords matched `butter`.

### Step 5 — Anti-Forensics Discovery
```bash
sudo ls -la /root/
```

**Findings:**
- `root/.bash_history` symlinked to `/dev/null` — all root commands silently discarded
- `/root/.bashrc` contained injected line at EOF: `set +o history`

This disables bash history for any interactive root shell, providing a second layer
of anti-forensics coverage.

---

## Indicators of Compromise (IOCs)

| Indicator | Value |
|-----------|-------|
| Account name | `butter` |
| UID | 1002 |
| GID | 0 (root group) |
| Home directory | `/root` |
| Shell | `/bin/bash` |
| Password | Active hash (unknown password) |
| `.bash_history` | Symlinked to `/dev/null` |
| `.bashrc` injection | `set +o history` at EOF |

---

## MITRE ATT&CK Mapping

| Technique | ID | Detail |
|-----------|----|--------|
| Create Account: Local Account | T1136.001 | `butter` account with root group membership |
| Indicator Removal: Clear Command History | T1070.003 | `.bash_history -> /dev/null` + `set +o history` in `.bashrc` |

---

## Response Actions Taken

```bash
# Lock all three suspicious accounts
sudo usermod -L butter
sudo usermod -L evil_user
sudo usermod -L art

# Remove .bashrc injection
sudo sed -i '/set +o history/d' /root/.bashrc

# Restore bash history file
sudo rm /root/.bash_history
sudo touch /root/.bash_history
sudo chmod 600 /root/.bash_history
sudo chown root:root /root/.bash_history
```

---

## Detection Gap Analysis

**Why Tripwire didn't catch this:**

The T1136.001 Wazuh rule (Rule ID 101002) fires on `useradd` events captured via
journald. The `butter` account was created before the Wazuh agent was deployed on
ubuntu-webserver, and before the log rotation window. Local auth.log retains only
~3 rotations of history.

**What this proves:**

Detection rules are only as good as your monitoring coverage window. This is exactly
why SIEM log forwarding with extended retention (90+ days) matters.

---

## Interview Narrative

During a homelab audit I discovered an undocumented privileged account named `butter`
with GID 0 and home directory set to `/root` — essentially a root-group backdoor with
an active password I couldn't identify. Forensic triage across all log files found no
creation event due to log rotation. Anti-forensics were in place: root's `.bash_history`
was symlinked to `/dev/null`, and `set +o history` had been injected into `/root/.bashrc`.
I locked all suspicious accounts, removed the injection, and restored bash history.
This is a real T1136.001 + T1070.003 scenario — not simulated — and it demonstrated
exactly why centralized log forwarding with long retention matters.

---

*Discovered: June 21, 2026 | ubuntu-webserver (192.168.248.139) | Project Tripwire homelab*
