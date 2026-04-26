\# Methodology



\## Attack Simulation: Atomic Red Team



Atomic Red Team (ART), maintained by Red Canary, is an open-source library

of small scripted tests that simulate specific MITRE ATT\&CK techniques.

Each test is defined in YAML and specifies the commands required to execute

one atomic attack action on a target system. For this project ART is the

source of known-malicious activity: specific ART tests are executed on a

controlled Ubuntu 24.04 VM, the resulting logs are captured, and those

logs are used as ground-truth positive examples when evaluating Sigma

detection rules.



\## Logging



The victim VM runs auditd with the Neo23x0 community ruleset for

security-relevant syscall monitoring, plus standard syslog auth.log

capture for authentication events.



\## Detection



Detection logic is written in Sigma format for portability and to follow

industry-standard conventions. Rules are evaluated via a custom Python

harness against labeled log data.



\## Evaluation



Each rule is scored against a labeled dataset: malicious events from

ART executions and benign events sampled from a 30-minute baseline

capture of normal user activity. Reported metrics are precision

(TP / (TP + FP)), recall (TP / (TP + FN)), and their harmonic mean F1.



\## Auditd Ruleset Adaptation



\### Starting Point

\- Source: Neo23x0 community ruleset (github.com/Neo23x0/auditd)

\- Rules commented out for Ubuntu 24.04 compatibility: 44

&#x20; (RHEL/CentOS paths, optional services like Filebeat, OpenLDAP,

&#x20; Kerberos, Docker, CrowdStrike, dnf/yum)

\- Loaded rule count after Ubuntu adaptation: 163



\### Project-Specific Additions

\- File: /etc/audit/rules.d/99-project-additions.rules

\- Rationale: project changes kept separate from upstream ruleset for

&#x20; traceability and reversibility

\- 3 coverage gaps identified by systematic audit against the 10 target

&#x20; ATT\&CK techniques:



&#x20; Gap 1 — T1003.008 (Credential Access: /etc/passwd and /etc/shadow):

&#x20;   Neo23x0 watches /etc/shadow with -p wa only. ATT\&CK T1003.008

&#x20;   simulates an attacker reading these files; reads are not caught

&#x20;   by 'wa'. Added -p r watches with keys cred\_read\_shadow,

&#x20;   cred\_read\_passwd, cred\_read\_gshadow.



&#x20; Gap 2 — T1098.004 (SSH authorized\_keys modification):

&#x20;   Neo23x0 has no watch on per-user authorized\_keys. auditd does

&#x20;   not support glob patterns; explicit per-user paths required.

&#x20;   Added watches on /root/.ssh/authorized\_keys and

&#x20;   /home/surya/.ssh/authorized\_keys with key ssh\_authkeys.



&#x20; Gap 3 — T1070.002 (Successful log truncation):

&#x20;   Neo23x0 covers delete and failed-truncate but not successful

&#x20;   truncate of /var/log/\* files. Added directory watch on /var/log

&#x20;   with key log\_tampering.



\- Final loaded rule count: 169 (163 + 6 additions)



\### Verification

All 6 added watches functionally verified via ausearch:

\- cred\_read\_shadow fires on `sudo cat /etc/shadow`

\- ssh\_authkeys fires on append to authorized\_keys

\- log\_tampering fires on file creation, truncation, and deletion

&#x20; inside /var/log



\### Known Tradeoffs

\- /etc/shadow read-watch generates substantial event volume from

&#x20; legitimate readers (PAM, sudo, sshd, systemd-logind). Filtering

&#x20; of legitimate readers performed in the Sigma detection rule, not

&#x20; at the auditd layer. Documented in docs/TUNING\_LOG.md.

\- chmod syscall rule from Neo23x0 has -F auid>=1000 filter, meaning

&#x20; SUID-set actions performed as true root (auid=0) will not be

&#x20; logged. Acceptable for this project because ART tests run via

&#x20; sudo from the surya user, which preserves auid=1000 through sudo.

