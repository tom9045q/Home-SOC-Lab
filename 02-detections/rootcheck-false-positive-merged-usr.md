# Rootcheck "Trojaned Binary" False Positive

Date: 2026-07-01
Host: d01
Alert: Wazuh rootcheck, rule ID 510
Status: Resolved - False Positive

## What happened

Wazuh's rootcheck flagged 26 core system binaries (ls, cat, chmod, chown, passwd, 
uname, and others) as "Trojaned version of file detected," all within about 10 
seconds of each other. I investigated and confirmed this was a false positive caused 
by an outdated rootcheck signature misreading Ubuntu's merged /usr filesystem layout.

Rule ID: 510
Rule level: 7
Trigger window: 2026-07-01, 16:55:07 to 16:55:17
26 alerts total, matching generic OSSEC signatures like bash|^/bin/sh|file\.h|proc\.h|/dev/...

<img width="1919" height="1079" alt="2026-07-01_d01_rootcheck_wazuh-discover-overview_ruleid510 png" src="https://github.com/user-attachments/assets/74dcf0ae-2267-469b-bb61-8866f2dfa328" />


## How I checked it

First, compared installed binaries against their original install checksums:

sudo debsums -c

All mismatches were inside /var/ossec/ (Wazuh's own config and Python cache files, 
which are expected to change). None of the 26 flagged binaries showed up as mismatched.

Second, verified the actual packages the flagged binaries belong to:

dpkg -V coreutils
dpkg -V passwd
dpkg -V login

No output for any of them, meaning no discrepancies found between what apt installed 
and what's on disk now.

<img width="964" height="1079" alt="2026-07-01_d01_dpkg-verify_coreutils-passwd-login png" src="https://github.com/user-attachments/assets/7ed382a5-8293-4fe4-b5d6-2a2d473d56c0" />


Third, checked why it might have fired in the first place:

ls -la /bin | head -5

Result:
lrwxrwxrwx 1 root root 7 Apr 20 08:46 /bin -> usr/bin

/bin is a symlink to usr/bin, the standard layout on Ubuntu 17.04 and newer. Wazuh's 
rootcheck signatures predate this layout and are known to misfire against it.

<img width="961" height="1079" alt="2026-07-01_d01_bin-symlink-check png" src="https://github.com/user-attachments/assets/972568f6-24f1-4496-a8fe-157cd5c67fe4" />


## Conclusion

Two separate integrity checks came back clean on every flagged binary. Twenty-six 
core utilities getting flagged at once, in a single short burst, doesn't match how a 
real compromise would look, and it does match a known rootcheck issue on merged-usr 
systems.

## Next steps

- Tune or suppress rule 510 for this host, or update the signature paths for merged-usr
- Maybe cross-check with rkhunter for a second opinion
- Watch for this firing again on the next rootcheck cycle
