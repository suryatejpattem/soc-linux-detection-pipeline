# Methodology

## Attack Simulation: Atomic Red Team

Atomic Red Team (ART), maintained by Red Canary, is an open-source library of small scripted tests that simulate specific MITRE ATT&CK techniques.

Each test is defined in YAML and specifies the commands required to execute one atomic attack action on a target system. For this project, ART is used as the source of known-malicious activity. Specific ART tests are executed on a controlled Ubuntu 24.04 VM, the resulting logs are captured, and those logs are used as ground-truth positive examples when evaluating Sigma detection rules.

---

## Logging

The victim VM runs:
- `auditd` with the Neo23x0 community ruleset for syscall-level security monitoring
- Standard system logging (`auth.log`) for authentication-related events

---

## Detection

Detection logic is written in Sigma format to ensure portability and alignment with industry-standard practices.

Rules are evaluated using a custom Python-based evaluation harness against labeled log data.

---

## Evaluation

Each rule is evaluated using a labeled dataset consisting of:
- Malicious events generated from ART executions
- Benign events collected from a 30-minute baseline of normal user activity

Metrics computed:
- Precision = TP / (TP + FP)
- Recall = TP / (TP + FN)
- F1 Score = Harmonic mean of precision and recall

---

## Auditd Ruleset Adaptation

### Starting Point

- Source: Neo23x0 community ruleset (https://github.com/Neo23x0/auditd)
- Rules commented out for Ubuntu 24.04 compatibility: 44
  - Due to RHEL/CentOS-specific paths and optional services (Filebeat, OpenLDAP, Kerberos, Docker, CrowdStrike, dnf/yum)
- Loaded rule count after adaptation: 163

---

### Project-Specific Additions

- File: `/etc/audit/rules.d/99-project-additions.rules`
- Rationale: Keep project-specific modifications separate from upstream ruleset for traceability and reversibility

#### Gap 1 — T1003.008 (Credential Access: `/etc/passwd`, `/etc/shadow`)

- Neo23x0 monitors `/etc/shadow` with `-p wa` (write/attribute only)
- Attack technique involves reading these files
- Reads are not captured by `wa`

**Fix:**
- Added read (`-p r`) watches:
  - `/etc/shadow` → key: `cred_read_shadow`
  - `/etc/passwd` → key: `cred_read_passwd`
  - `/etc/gshadow` → key: `cred_read_gshadow`

---

#### Gap 2 — T1098.004 (SSH `authorized_keys` modification)

- No existing coverage in Neo23x0 ruleset
- `auditd` does not support glob patterns

**Fix:**
- Added explicit file watches:
  - `/root/.ssh/authorized_keys`
  - `/home/surya/.ssh/authorized_keys`
- Key: `ssh_authkeys`

---

#### Gap 3 — T1070.002 (Log Tampering via Truncation)

- Neo23x0 covers deletion and failed truncation
- Does not cover successful truncation

**Fix:**
- Added directory watch:
  - `/var/log`
- Key: `log_tampering`

---

### Final Rule Count

- Base rules: 163
- Project additions: 6
- Total: 169 rules loaded

---

### Verification

All added rules verified using `ausearch`:

- `cred_read_shadow` triggers on:
  - `sudo cat /etc/shadow`

- `ssh_authkeys` triggers on:
  - Append operations to `authorized_keys`

- `log_tampering` triggers on:
  - File creation
  - File truncation
  - File deletion within `/var/log`

---

### Known Tradeoffs

- **High Noise from `/etc/shadow` reads**
  - Legitimate processes (PAM, sudo, sshd, systemd-logind) generate frequent reads
  - Filtering is handled at the Sigma rule level, not within auditd
  - Documented in `docs/TUNING_LOG.md`

- **SUID detection limitation**
  - Neo23x0 `chmod` rule uses filter: `-F auid>=1000`
  - Actions performed as true root (`auid=0`) are not logged
  - Acceptable for this project since ART tests are executed via `sudo` from user `surya`, preserving `auid=1000`
