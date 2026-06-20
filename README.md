# 🪤 Project Tripwire

### SIEM Detection Rule Library — Wazuh · Sigma · MITRE ATT&CK · Atomic Red Team

> *"Every rule is a trap. Every trap is a story. Tripwire makes sure attackers tell it."*

[![Status](https://img.shields.io/badge/Status-Active%20Development-brightgreen?style=flat-square)](https://github.com/enak223)
[![Stack](https://img.shields.io/badge/Stack-Wazuh%20%7C%20Sigma%20%7C%20Atomic%20Red%20Team%20%7C%20MITRE%20ATT%26CK-blue?style=flat-square)](https://github.com/enak223)
[![Rules](https://img.shields.io/badge/Rules-4%20Validated-orange?style=flat-square)](https://github.com/enak223/project-tripwire/tree/main/rules)
[![License](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)](https://github.com/enak223/project-tripwire/blob/main/LICENSE)

---

## 📌 Overview

**Project Tripwire** is a curated detection rule library built for Wazuh SIEM, with every rule written in both native Wazuh XML format and portable Sigma format. Each rule is mapped to a specific MITRE ATT&CK technique, tested against real attack simulations using Atomic Red Team, and documented with the analyst context needed to act on an alert — not just receive one.

It answers three questions for every detection:

- **What are we detecting?** — A specific adversary behavior mapped to a MITRE ATT&CK technique, with the Wazuh rule and Sigma rule that catches it.
- **Does it actually work?** — Every rule has a documented Atomic Red Team test case executed in the homelab against live endpoints, with the resulting Wazuh alert as proof.
- **What do you do when it fires?** — Each rule ships with an analyst runbook: what the alert means, false positive context, triage steps, and escalation guidance.

Not a rule dump. A detection engineering portfolio.

---

## ✅ Detection Coverage (v0.2)

| Technique | Name | Tactic | Level | Platform | Status |
|-----------|------|--------|-------|----------|--------|
| T1003.007 | OS Credential Dumping: /proc Filesystem | Credential Access | 12 | Linux | ✅ Validated |
| T1136.001 | Create Account: Local Account | Persistence | 10 | Linux | ✅ Validated |
| T1070.003 | Indicator Removal: Clear Command History | Defense Evasion | 9 | Linux | ✅ Validated |
| T1053.003 | Scheduled Task/Job: Cron | Execution / Persistence | 10 | Linux | ✅ Validated |

---

## 🏗️ Architecture
```
┌──────────────────────────────────────────────────────────────────────┐

│                       TRIPWIRE PIPELINE                              │

│                                                                      │

│  ┌──────────────┐      ┌──────────────┐      ┌──────────────────┐    │

│  │   SIMULATE   │────▶│    DETECT    │────▶│    VALIDATE      │     │

│  │              │      │              │      │                  │    │

│  │ Atomic Red   │      │ Wazuh Rules  │      │ Alert Confirmed  │    │

│  │ Team (ART)   │      │ Sigma Rules  │      │ Log Evidence     │    │

│  │ Kali Linux   │      │ Decoders     │      │ No False Neg.    │    │

│  └──────────────┘      └──────────────┘      └──────────────────┘    │

│                                                       │              │

│  ┌──────────────┐      ┌──────────────┐      ┌────────▼─────────┐    │

│  │   DOCUMENT   │◀────│     MAP      │◀────│    TUNE           │    │

│  │              │      │              │      │                  │    │

│  │ Analyst      │      │ MITRE ATT&CK │      │ Threshold Adj.   │    │

│  │ Runbook      │      │ Tactic/Tech  │      │ False Pos. Rate  │    │

│  │ Triage Guide │      │ Sub-technique│      │ Rule Confidence  │    │

│  └──────────────┘      └──────────────┘      └──────────────────┘    │

│                                                                      │

│         ┌─────────────────────────────────────────┐                  │

│         │         DETECTION COVERAGE MATRIX       │                  │

│         │   ATT&CK Navigator heatmap per phase    │                  │

│         │   Rule count by tactic · confidence     │                  │

│         └─────────────────────────────────────────┘                  │

└──────────────────────────────────────────────────────────────────────┘
```

**Rule Lifecycle:**
```
Atomic Red Team test selected (MITRE technique)

└──▶ Attack simulated on homelab endpoint (Kali / Windows 11 / Ubuntu)

└──▶ Wazuh ingests logs (Sysmon / Auditd / ossec agent)

└──▶ Custom Wazuh rule written → alert fires

└──▶ Sigma rule authored for portability

└──▶ MITRE ATT&CK technique mapped

└──▶ Analyst runbook documented

└──▶ Rule committed to library

```
---

## 🧰 Tech Stack

| Component | Tool | Role |
| --------- | ---- | ---- |
| **SIEM** | Wazuh 4.12 | Log ingestion, rule engine, alert correlation, active response |
| **Rule Standard** | Sigma | Vendor-neutral detection rule format for portability |
| **ATT&CK Framework** | MITRE ATT&CK v15 | Tactic/technique mapping for every rule |
| **ATT&CK Visualization** | ATT&CK Navigator | Coverage heatmap generation per tactic |
| **Attack Simulation** | Atomic Red Team | Validated test cases per technique (T-codes) |
| **Windows Telemetry** | Sysmon (SwiftOnSecurity cfg) | Process creation, network, registry, file event logging |
| **Linux Telemetry** | Auditd | Syscall-level event logging on Ubuntu endpoints |
| **Attacker Platform** | Kali Linux | ART execution, manual attack simulation |
| **Log Visualization** | Wazuh Dashboard | Real-time alert review and rule tuning |
| **Scripting** | Python 3.10+ | Coverage matrix generation, ATT&CK Navigator JSON export |
| **Host OS** | Ubuntu 22.04 | Wazuh manager host (192.168.248.20) |
| **Virtualization** | VMware Workstation | Homelab multi-VM environment |

---

## ✨ Features

### 🪤 Detection Rule Library

- Native **Wazuh XML rules** deployable directly to `/var/ossec/etc/rules/`
- Companion **Sigma rules** for every detection — portable to Splunk, Elastic, Microsoft Sentinel
- Rules organized by MITRE ATT&CK tactic: Initial Access → Execution → Persistence → Privilege Escalation → Defense Evasion → Credential Access → Discovery → Lateral Movement → Collection → Exfiltration → Command & Control → Impact
- Rule severity levels aligned to Wazuh's 1-15 scale with documented rationale
- Each rule targets a specific log source: Sysmon (Windows), Auditd (Linux), or Wazuh agent (both)

### 🧪 Atomic Red Team Validation

- Every rule in the library has at least one documented Atomic Red Team test case
- Tests executed live in the homelab against real endpoints — not mocked
- Validation record includes: ART test ID, command executed, expected log artifact, Wazuh rule fired, alert level, time-to-detect
- False negative testing: rules are tuned until detection is confirmed before committing

### 🗺️ MITRE ATT&CK Coverage Matrix

- ATT&CK Navigator JSON heatmap generated from the rule library — shows exactly which techniques are covered
- Coverage tracked by tactic phase, technique, and sub-technique
- Confidence scoring per rule: High / Medium / Low based on validation results and false positive rate
- Updated via `scripts/generate_navigator.py`

### 📋 Analyst Runbooks

- Every rule ships with a markdown runbook in `rules/<tactic>/<technique>/RUNBOOK.md`
- Runbook structure: Alert Summary → What Triggered It → False Positive Scenarios → Triage Steps → Escalation Criteria → References
- Written for a Tier 1 SOC analyst — actionable within 5 minutes of alert receipt

### 🔧 Wazuh Decoder Library

- Custom decoders for log formats not natively parsed by Wazuh (Sysmon XML, custom auditd fields)
- Decoder validation scripts confirm field extraction before rule deployment
- Decoders designed to be additive — no modification of Wazuh default decoders

---

## 📁 Project Structure
```
project-tripwire/

├── README.md

├── .gitignore

│

├── rules/

│   ├── credential-access/

│   │   └── T1003.007-proc-filesystem/    # ✅ Validated

│   │       ├── wazuh_rule.xml

│   │       ├── sigma_rule.yml

│   │       ├── RUNBOOK.md

│   │       └── validation/

│   │           ├── art_test.md

│   │           └── alert_sample.json

│   ├── persistence/

│   │   └── T1136.001-local-account/      # ✅ Validated

│   │       ├── wazuh_rule.xml

│   │       ├── sigma_rule.yml

│   │       ├── RUNBOOK.md

│   │       └── validation/

│   │           ├── art_test.md

│   │           └── alert_sample.json

│   ├── defense-evasion/

│   │   └── T1070.003-clear-history/      # ✅ Validated

│   │       ├── wazuh_rule.xml

│   │       ├── sigma_rule.yml

│   │       ├── RUNBOOK.md

│   │       └── validation/

│   │           ├── art_test.md

│   │           └── alert_sample.json

│   ├── execution/

│   │   └── T1053.003-cron-job/           # ✅ Validated

│   │       ├── wazuh_rule.xml

│   │       ├── sigma_rule.yml

│   │       ├── RUNBOOK.md

│   │       └── validation/

│   │           ├── art_test.md

│   │           └── alert_sample.json

│   ├── initial-access/

│   ├── privilege-escalation/

│   ├── lateral-movement/

│   ├── discovery/

│   ├── collection/

│   ├── exfiltration/

│   ├── command-and-control/

│   └── impact/

│

├── wazuh/

│   ├── decoders/

│   └── tripwire_rules.xml                # Master Wazuh rule file (all techniques)

│

├── auditd/

│   └── audit.rules                       # Validated auditd rules for ubuntu-webserver

│

├── sigma/

│   ├── compiled/

│   └── backends/

│

├── scripts/

│   ├── generate_navigator.py

│   ├── validate_rules.py

│   └── coverage_report.py

│

├── coverage/

│   └── tripwire_navigator.json

│

└── docs/

├── rule_writing_guide.md

├── sigma_conversion_guide.md

├── art_testing_guide.md

└── wazuh_rule_levels.md

---

## ⚙️ Setup & Installation
```

### Prerequisites

Wazuh 4.12 manager running (192.168.248.20 — ubuntuai VM)
Wazuh agents enrolled on Windows 11 (192.168.248.128) and Ubuntu Web Server (192.168.248.139)
Sysmon installed on Windows 11 with SwiftOnSecurity config
Auditd running on Ubuntu endpoints
Kali Linux (192.168.248.130) for Atomic Red Team execution
Python 3.10+
sigma-cli (for Sigma rule conversion)


### 1. Clone the Repository

```bash
git clone https://github.com/enak223/project-tripwire.git
cd project-tripwire
```

### 2. Register Tripwire Audit Keys in Wazuh CDB List

```bash
# Run on ubuntuai (192.168.248.20) — inside Wazuh Docker container
sudo docker exec single-node-wazuh.manager-1 bash -c "
echo tripwire_proc_cred:write >> /var/ossec/etc/lists/audit-keys
echo tripwire_account_create:write >> /var/ossec/etc/lists/audit-keys
echo tripwire_clear_history:write >> /var/ossec/etc/lists/audit-keys
echo tripwire_cron:write >> /var/ossec/etc/lists/audit-keys
"
```

### 3. Deploy Wazuh Detection Rules

```bash
# Run on ubuntuai (192.168.248.20)
sudo docker cp wazuh/tripwire_rules.xml single-node-wazuh.manager-1:/var/ossec/etc/rules/tripwire_rules.xml
sudo docker exec single-node-wazuh.manager-1 /var/ossec/bin/wazuh-control restart
```

### 4. Deploy Auditd Rules on Linux Endpoints

```bash
# Run on ubuntu-webserver (192.168.248.139)
sudo cp auditd/audit.rules /etc/audit/rules.d/tripwire.rules
sudo auditctl -D
sudo auditctl -R /etc/audit/rules.d/tripwire.rules
sudo service auditd restart
```

### 5. Validate with Atomic Red Team

```bash
# Run on ubuntu-webserver (192.168.248.139)

# T1003.007
sudo cat /proc/1/environ > /dev/null

# T1136.001
sudo useradd tripwire-test

# T1070.003
cat /dev/null > ~/.bash_history

# T1053.003
(crontab -l 2>/dev/null; echo "* * * * * echo tripwire") | crontab -
```

```bash
# Confirm alerts fired — run on ubuntuai (192.168.248.20)
sudo docker exec single-node-wazuh.manager-1 grep -A 2 "101001\|101002\|101004\|101005" /var/ossec/logs/alerts/alerts.log | tail -20
```

---

## 🔧 Engineering Notes

### Wazuh 4.12 Known Bug — audit.exe Truncation
Wazuh 4.12 has a confirmed decoder bug where the `audit.exe` field is truncated
at hyphen characters in binary names (e.g. `/usr/libexec/ptyxis-agent` is decoded
as `/usr/libexec/ptyxis`). This affects FP suppression rules that rely on `audit.exe`
matching. **Workaround:** use `audit.command` (the `comm=` field) instead, which
correctly preserves the full process short name. Fixed in Wazuh 4.14.
Bug reference: https://github.com/wazuh/wazuh/issues/18477

### Wazuh CDB List Requirement
Custom auditd keys must be registered in `/var/ossec/etc/lists/audit-keys` on the
Wazuh manager for proper event routing. Without this, auditd events with custom
keys may not match rules correctly. See Setup Step 2 above.

### False Positive Tuning — Desktop Environment Noise
ubuntu-webserver runs a full GNOME desktop, which generates background activity
from processes like `gvfsd-metadata`, `ptyxis-agent`, and `gsd-housekeeping`.
Detection rules are tuned to minimize noise:
- T1003.007: Auditd rule removed from broad /proc watch; rule relies on logtest-confirmed detection
- T1070.003: Auditd watch scoped to .bash_history files only (not all of /home)
- T1136.001 and T1053.003: Clean detections via syslog/journald with no FP noise

---

## 🏠 Homelab Environment
```
┌──────────────────────────────────────────────────────────────┐

│            TRIPWIRE HOMELAB — VMware Workstation             │

│                                                              │

│  VMnet: NAT / Host-Only (192.168.248.0/24)                   │

│                                                              │

│  ┌──────────────────────────────────────────────────────┐    │

│  │  VM 1: ubuntuai — Wazuh Manager + Rule Engine        │    │

│  │  IP: 192.168.248.20                                  │    │

│  │  Role: Wazuh 4.12, custom rules, decoders, coverage  │    │

│  │        scripts, ATT&CK Navigator JSON generation     │    │

│  └──────────────────────────────────────────────────────┘    │

│                                                              │

│  ┌──────────────────────────────────────────────────────┐    │

│  │  VM 2: ubuntu-webserver — Linux Detection Target     │    │

│  │  IP: 192.168.248.139                                 │    │

│  │  Role: Wazuh agent, Auditd telemetry, ART target     │    │

│  └──────────────────────────────────────────────────────┘    │

│                                                              │

│  ┌──────────────────────────────────────────────────────┐    │

│  │  VM 3: Kali Linux — Attack Simulation                │    │

│  │  IP: 192.168.248.130                                 │    │

│  │  Role: Atomic Red Team execution, manual TTPs        │    │

│  └──────────────────────────────────────────────────────┘    │

│                                                              │

│  ┌──────────────────────────────────────────────────────┐    │

│  │  VM 4: Windows 11 — Windows Detection Target         │    │

│  │  IP: 192.168.248.128                                 │    │

│  │  Role: Wazuh agent, Sysmon telemetry, ART target     │    │

│  └──────────────────────────────────────────────────────┘    │

└──────────────────────────────────────────────────────────────┘
```

---

## 🔐 Security Notes

- Rule IDs in the `101000-101999` range are reserved for Tripwire — no conflict with Wazuh defaults, GhostNet (100600-100699), or NullByte (100500).
- No credentials, API keys, or host-identifying data committed to the repository.
- Atomic Red Team tests are run only on owned homelab VMs — never against external or production systems.
- Sigma rules are marked `status: test` until validated against live telemetry.
- **Authorized use only.** All detection content must only be deployed on systems you own or have explicit written authorization to monitor.

---

## 🗺️ Roadmap

| Phase | Feature | Status |
| ----- | ------- | ------ |
| v0.1 | Repo scaffold + folder structure | ✅ Complete |
| v0.1 | Wazuh manager ready + agents enrolled (Win11 + Ubuntu) | ✅ Complete |
| v0.1 | Sysmon deployed on Windows 11 | ✅ Complete |
| v0.1 | Auditd configured on Ubuntu endpoints | ✅ Complete |
| v0.2 | 4 seed rules — T1003.007, T1136.001, T1070.003, T1053.003 | ✅ Complete |
| v0.2 | Sigma companion rules for all 4 seed techniques | ✅ Complete |
| v0.2 | Analyst runbooks for all 4 seed techniques | ✅ Complete |
| v0.2 | ART validation records for all 4 seed techniques | ✅ Complete |
| v0.2 | Wazuh CDB list updated with custom Tripwire keys | ✅ Complete |
| v0.3 | 10 rules across 5 tactics — all ART validated | 🔄 In Progress |
| v0.4 | ATT&CK Navigator heatmap (auto-generated) | 🔲 Planned |
| v0.5 | 25 rules across all 12 ATT&CK tactics | 🔲 Planned |
| v0.5 | Coverage matrix markdown report | 🔲 Planned |
| v1.0 | Sigma → Splunk SPL conversion pipeline | 🔲 Planned |
| v1.0 | Sigma → Elastic EQL conversion pipeline | 🔲 Planned |
| v1.1 | Tripwire + GhostNet unified alert dashboard | 🔲 Planned |
| v1.2 | CI pipeline — auto-validate rule XML on push | 🔲 Planned |

---

## 🔗 Related Projects

| Project | Description | Link |
| ------- | ----------- | ---- |
| **Project GhostNet** | Passive network baselining & anomaly detection — Zeek · Suricata · Wazuh | [github.com/enak223/project-ghostnet](https://github.com/enak223/project-ghostnet) |
| **Project NullByte** | Automated CVE detection & remediation reporting — Nmap · NVD API · AI | [github.com/enak223/project-nullbyte](https://github.com/enak223/project-nullbyte) |
| **Agentic AI for Security** | AI-powered SOC automation — n8n · Claude API · Wazuh | [github.com/enak223/agentic-ai-for-security](https://github.com/enak223/agentic-ai-for-security) |

---

## 👤 Author

**Eliezer Fuentes** — Cybersecurity Professional

Detection Engineering | Threat Hunting | SOC Automation | Vulnerability Management

[![GitHub](https://img.shields.io/badge/GitHub-enak223-181717?style=flat&logo=github)](https://github.com/enak223)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-eliezerfuentes-0A66C2?style=flat&logo=linkedin)](https://www.linkedin.com/in/eliezerfuentes/)

---

> *Set the trap. Know the trigger. Never miss.* >
