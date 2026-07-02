SSH Brute Force Attack — Incident Investigation
Date: July 1-2, 2026
Analyst: Thomas Marvin
Severity: Medium
Status: Resolved — Attack Unsuccessful

Environment
The target machine, d01, is a dedicated Ubuntu Server 24.04 physical host running a Wazuh 4.14.5 all-in-one deployment (manager, indexer, and dashboard). Wazuh monitors d01 via its built-in agent (ID 000), which captures authentication events, system logs, and security alerts locally. The simulated attacker machine was a Windows 11 host running WSL2 (Ubuntu), connected to the same 192.168.1.x home network as d01.

<img width="962" height="1079" alt="01-environment-baseline" src="https://github.com/user-attachments/assets/3c3e12ea-d650-4496-903a-336751857bd1" />

Attack Simulation
A simulated SSH brute force attack was executed using Hydra v9.6 from the attacker machine (192.168.1.67) targeting the lp01 user account on d01 (192.168.1.154) over port 22. The rockyou.txt wordlist (14.3 million entries) was used as the password list.
Command executed from attacker machine (WSL2):
bashhydra -l lp01 -P rockyou.txt ssh://192.168.1.154 -t 4 -V
Flag breakdown:

-l lp01 — target username
-P rockyou.txt — password wordlist
ssh://192.168.1.154 — target IP via SSH protocol
-t 4 — 4 parallel threads
-V — verbose output

<img width="1101" height="606" alt="06-hydra-installed" src="https://github.com/user-attachments/assets/c25e9051-3202-402e-a9c7-5f967fa0d8f0" />



Detection
Wazuh detected the attack within seconds. The total alert count spiked from 36 baseline alerts to 197, with 131 authentication failures recorded — compared to 0 authentication failures in the pre-attack baseline.

<img width="901" height="1027" alt="07-post-attack-alert-spike" src="https://github.com/user-attachments/assets/6a841869-dba6-417a-b0e2-55f28944be71" />


Key rules triggered:
Rule IDDescriptionLevel5760sshd: authentication failed52501syslog: User authentication failure52502syslog: User missed the password more than one time105758Maximum authentication attempts exceeded85557unix_chkpwd: Password check failed5
MITRE ATT&CK techniques identified:

T1110 — Brute Force
T1110.001 — Password Guessing
T1021 — Remote Services


<img width="1919" height="1079" alt="07-post-attack-dashboard" src="https://github.com/user-attachments/assets/f62d1d07-f54a-4a76-9b07-19b01adb8a7f" />

<img width="1919" height="1079" alt="08-events-raw-alerts" src="https://github.com/user-attachments/assets/34534f88-715d-4219-970c-47f4c448e8c9" />




Investigation
Expanding a raw alert in the Wazuh Events tab confirmed the following indicators:
FieldValueSource IP192.168.1.67 (attacker/WSL machine)Target userlp01Target port22 (SSH)Agentd01 (ID 000)Rule fired37 times (rule 5760 alone)Raw logsshd-session[10450]: Failed password for lp01 from 192.168.1.67 port 63626 ssh2

<img width="1919" height="1079" alt="09-alert-detail-evidence" src="https://github.com/user-attachments/assets/c4b6b4c6-df14-4918-9612-78ac296379b3" />


Key investigative findings:

All authentication failures occurred at identical timestamps (22:38:06.748), confirming automated tooling rather than a human manually entering passwords
131 failures vs. 13 successes — an abnormal failure ratio indicating a credential stuffing/brute force pattern
Single source IP (192.168.1.67) responsible for all failures — a distributed attack would show multiple source IPs
d01's SSH service enforced MaxAuthTries, automatically dropping Hydra's connections after repeated failures — visible in Hydra output as "Connection reset by peer"


Response
Immediate containment:
Block source IP 192.168.1.67 at the firewall level to prevent further attempts.
Escalation:
Escalate to SOC Tier 2 for deeper analysis — determine whether the attack is targeted or part of automated scanning, and check for any additional indicators of compromise beyond authentication logs.
Stakeholder notification:
Notify management per incident response policy.
Outcome:
Attack was unsuccessful. No credentials were compromised. d01's SSH rate limiting (MaxAuthTries) functioned as intended, automatically terminating connections after repeated failures.

Lessons Learned

SSH rate limiting (MaxAuthTries) and strong credential policy stopped the attack before any breach occurred — demonstrating defense-in-depth effectiveness against automated tooling
Wazuh's escalating rule levels (5 → 10) reflect increasing confidence of malicious intent — one failed login is noise, 131 simultaneous failures from one IP is a signal
Real-world SSH brute force success typically requires misconfigured rate limiting, weak/default credentials, or direct internet exposure — none of which applied here
Baseline alert review before the attack (71 alerts, all benign install-time activity) was essential context — without it, distinguishing normal from malicious would be harder


Tools Used
ToolPurposeWazuh 4.14.5SIEM — alert detection and investigationHydra v9.6SSH brute force simulationrockyou.txtPassword wordlist (14.3M entries)WSL2 (Ubuntu on Windows 11)Attacker environmentUbuntu Server 24.04Target host (d01)

Paste that into 03-attack-simulation/README.md, insert your screenshots where marked, and commit it.
Then come back and explain the Investigation section to me in your own words — that's still the condition of the deal.
