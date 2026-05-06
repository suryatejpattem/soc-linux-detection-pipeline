# T1136.001 — Create Local Account — Execution Record

## MITRE ATT&CK Technique
T1136.001 — Create Local Account

## Objective
Simulate realistic unauthorized local account creation on Ubuntu 24.04 using Atomic Red Team methodology, capture telemetry, and validate detection opportunities across auth.log and auditd.

## Original ART Command
useradd -M -N -r -s /bin/bash -c evil_account evil_user

## Modified Project Command
sudo useradd -M -N -r -s /bin/bash -c system_backup backupsvc

## Why Modified
The default ART username/comment were intentionally replaced with more realistic enterprise-style naming to simulate attacker attempts to blend into legitimate administrative or service-account naming conventions.

## Command Breakdown
- -M → No home directory
- -N → No user-specific group
- -r → System account
- -s /bin/bash → Interactive shell
- -c system_backup → Legitimate-looking comment field

## Cleanup Command
sudo userdel backupsvc

## Primary Detection Goal
Detect unauthorized local account creation through:
- sudo execution of useradd
- auditd process_creation telemetry
- ADD_USER audit events
- /etc/passwd and /etc/shadow modification

## Primary Telemetry Sources

### auth.log
- sudo command execution
- new user creation messages

### auditd
- exe="/usr/sbin/useradd"
- type=ADD_USER
- key="process_creation"
- key="etcpasswd"
- key="cred_read_shadow"
- key="etcgroup"

## Key Validation Finding
Initial `ausearch -x useradd -ts recent` queries failed due to narrow time scoping, not telemetry absence. Broader executable-path and raw audit log searches confirmed full audit coverage.

## Benign Consideration
Local account creation may be legitimate for:
- System administration
- Service accounts
- Application users
- Maintenance

## Detection Engineering Tradeoff
Username or comment alone should not be trusted for detection. Behavioral indicators are stronger:
- useradd execution
- system account creation
- interactive shell assignment
- account database modification

## Initial Sigma Concept
Sigma v1:
- Detect useradd execution

Sigma v2:
- Detect useradd with suspicious flags:
  - -M
  - -N
  - -r
  - /bin/bash

## Captured Artifacts
- auth_live.log
- audit_live.log
- auth_filtered.log
- audit_useradd_full.log
- audit_add_user.log
- audit_rules_snapshot.txt
- start_time.txt
- end_time.txt

## Project Value
This test establishes a validated baseline for Linux account-creation detection and demonstrates the importance of query validation, telemetry layering, and realistic adversary simulation.
