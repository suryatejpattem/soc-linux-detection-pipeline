# Project Scope

## Objective
Build and evaluate 10 Sigma detection rules for Linux endpoint telemetry,
covering a representative spread of MITRE ATT&CK tactics. Each rule will be
tested against real attack procedures from Atomic Red Team and scored on a
labeled dataset.

## Target Techniques

| # | ATT&CK ID | Technique | Tactic | Log Source |
|---|-----------|-----------|--------|------------|
| 1 | T1110.001 | SSH Brute Force | Credential Access | auth.log |
| 2 | T1078.003 | Valid Accounts: Local Accounts | Initial Access / Persistence | auth.log |
| 3 | T1059.004 | Unix Shell (suspicious commands) | Execution | auditd EXECVE |
| 4 | T1136.001 | Create Local Account | Persistence | auth.log + auditd |
| 5 | T1548.001 | SUID/SGID abuse | Privilege Escalation | auditd |
| 6 | T1548.003 | Sudo abuse | Privilege Escalation | auth.log (sudo) |
| 7 | T1053.003 | Cron persistence | Persistence | auditd file-watch |
| 8 | T1098.004 | SSH Authorized Keys modification | Persistence | auditd file-watch |
| 9 | T1003.008 | /etc/passwd and /etc/shadow access | Credential Access | auditd file-watch |
| 10 | T1070.002 | Clear Linux System Logs | Defense Evasion | auditd |

## Out of Scope
- Multi-host lateral movement (requires 2+ VM setup)
- Container escape techniques
- Kernel-level rootkit detection
- Windows attacks (separate project)

## Success Criteria
- 10 Sigma rules written and committed
- Each rule tested against the corresponding Atomic Red Team test
- Precision, recall, and F1 score reported per rule on a labeled test set
- At least 3 rules iterated to v2 with documented before/after metrics
- Final report published with honest limitations

## Auditd Ruleset Adaptation

### Starting Point
- Neo23x0 community ruleset
- Original rule count: [number]
- Rules commented out for Ubuntu compatibility: 44 (RHEL paths, optional services)
- Final loaded rule count after Neo23x0 adaptation: 163

### Project-Specific Additions
- File: /etc/audit/rules.d/99-project-additions.rules
- Rationale: keep project changes separate from upstream ruleset for traceability
- Additions (and why each was needed):
  - [T1003.008 read-watches — what gap they fill]
  - [T1098.004 authorized_keys — what gap they fill]
  - [any others after verification]

### Known Tradeoffs
- /etc/shadow read-watch volume: [observed events/hour after capture]
- Filtering strategy: performed in Sigma detection content, not at auditd layer
- Rationale: [your reasoning here]
