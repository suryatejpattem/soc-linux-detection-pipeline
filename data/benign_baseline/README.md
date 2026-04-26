# Benign Baseline Capture

## When
Baseline start: Sun Apr 26 03:54:48 PM EDT 2026
Baseline end:   Sun Apr 26 04:27:50 PM EDT 2026
Duration: ~33 minutes

## VM State
- Snapshot: victim-configured-with-project-rules
- Auditd rules loaded at capture time: 169
  (163 Neo23x0 + 6 from /etc/audit/rules.d/99-project-additions.rules)

## Files
- auth.log: PAM, sshd, sudo, login events from /var/log/auth.log
  Line count: 368
- audit.log: kernel syscall events from /var/log/audit/audit.log
  Line count: 21,436

## Activities Performed
- System inspection commands (whoami, id, uname, ps, top, df, free, etc.)
- Light sudo usage (sudo whoami, sudo ls /root, sudo cat /etc/hostname, sudo uptime)
- Editor session in nano (created and deleted ~/scratch.txt)
- Package activity: apt update, apt install tree, tree, apt remove tree
- Network activity: ping 8.8.8.8, curl to example.com
- Multiple terminal sessions opened and closed
- [Add or remove: SSH login(s) from Windows host]

## Activities Deliberately Excluded
The following overlap with target ATT&CK techniques and were excluded
from the baseline to avoid contaminating benign labels:
- User creation/deletion (T1136.001)
- Cron file modification (T1053.003)
- authorized_keys modification (T1098.004)
- Direct manual reads of /etc/shadow (T1003.008)
- Failed authentication attempts (T1110.001)
- sudo -l enumeration (T1548.003)

PAM/sshd/sudo internal reads of /etc/shadow are present in audit.log
and represent legitimate system behavior the project must not flag
as malicious.

## Known Contamination
The audit.log contains a small number of events from Stage 3 watch
verification testing performed shortly before the baseline window:
- Test read of /etc/shadow (key=cred_read_shadow)
- Test append to /home/surya/.ssh/authorized_keys (key=ssh_authkeys)
- Test creation, truncation, and deletion of /var/log/stage3_test.log
  (key=log_tampering)

These will be identified and excluded when building the labeled benign
dataset (Stage 12), either by timestamp filtering against the baseline
start time of 03:54:48 PM EDT or by direct identification of the test
artifacts.

## Rotated Logs Excluded
The /var/log/audit/ directory contained rotated logs (audit.log.1
through audit.log.4) totaling ~32 MB of pre-baseline data. These were
NOT included in this capture because they predate the baseline window
and contain unlabeled mixed activity from system boot, prior testing,
and idle operation. Including them would have inflated false positive
counts during evaluation without providing meaningful benign signal.
