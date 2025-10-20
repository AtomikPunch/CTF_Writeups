---
title: "Crown Jewels"
description: "Investigating a domain controller compromise where an attacker dumped the NTDS.dit database using volume shadow copy abuse."
date: "2025-10-20"
difficulty: "Medium"
tags:
  - Digital Forensics
  - Windows Event Logs
  - NTDS.dit
  - Volume Shadow Copy
  - Active Directory
  - Incident Response
  - CTF
platform: "HackTheBox"
tools:
  - Event Viewer
  - MFTECmd (Eric Zimmerman)
  - Windows Event Log Analysis
references:
  - https://attack.mitre.org/datasources/DS0019/
  - https://ericzimmerman.github.io/
slug: crown-jewels-challenge
---

# Crown Jewels CTF Challenge Writeup

## Challenge Synopsis

Forela's domain controller is currently under active attack. The Domain Administrator account is believed to be compromised, and there is strong suspicion that the threat actor has successfully dumped the NTDS.dit database on the Domain Controller. 

We recently received a critical alert indicating that `vssadmin` was executed on the DC. Since this activity does not align with any routine scheduled operations, we have substantial reason to believe that the attacker has abused this LOLBIN (Living Off the Land Binary) utility to obtain the Domain environment's crown jewel - the NTDS.dit database.

Our mission is to perform rapid analysis on the provided artifacts for quick triage and, if possible, identify and neutralize the attacker's access as early as possible.

## Provided Artifacts

We were provided with four forensic files for analysis:

### Windows Event Log Files (EVTX)
1. **SECURITY.evtx** - Security event logs
2. **SYSTEM.evtx** - System event logs
3. **MICROSOFT-WINDOWS-NTFS.evtx** - NTFS file system event logs

### File System Artifact
4. **$MFT** - Master File Table (to be analyzed later in the investigation)

## Investigation Process

### Step 1: Identifying Volume Shadow Copy Service Activation

Our first objective was to determine when the Volume Shadow Copy Service (VSS) began running on the compromised system.

#### Event ID 7036 Analysis

To accomplish this, we searched for Event ID **7036**, which tracks when services start or stop. This event ID is particularly useful for detecting potentially malicious service tampering.

**Reference**: [MITRE ATT&CK - Service Modification Detection](https://attack.mitre.org/datasources/DS0019/#:~:text=Event%20ID%207040%20%2D%20Detects%20modifications,stop%2C%20potentially%20indicating%20malicious%20tampering)

#### Findings in SYSTEM.evtx

Upon examination of the `SYSTEM.evtx` file, we discovered logs indicating that the Volume Shadow Copy Service transitioned to a **running** state. This event provided us with a crucial timestamp that established when the shadow copy was created.

![running st](Capture/Crownjewels/C1.PNG)

With this timestamp identified, we now had a temporal anchor point to correlate other suspicious activities across the remaining event logs.

### Step 2: Identifying Compromised Account and Group Modifications

With the VSS activation timestamp established, we proceeded to analyze the `SECURITY.evtx` file to identify:
- Which user account the attacker compromised or utilized
- Which security group(s) the attacker added themselves to for privilege escalation

By filtering security events around the established timestamp, we were able to identify the specific account used by the threat actor and the group membership modifications that granted them elevated privileges within the domain.

![running st](Capture/Crownjewels/C2.PNG)
![running st](Capture/Crownjewels/C3.PNG)

### Step 3: Analyzing NTFS Event Logs

Next, we examined the `MICROSOFT-WINDOWS-NTFS.evtx` event log file. This log contained numerous undefined events, which required careful analysis.

#### Temporal Correlation

Using the Volume Shadow Copy timestamp as our reference point, we narrowed our investigation scope to fewer than 10 relevant events occurring within the critical time window.

#### Shadow Copy Snapshot Discovery

Within this refined dataset, we identified a specific log entry that clearly showed evidence of the shadow copy snapshot creation. This confirmed the attacker's use of VSS to create a snapshot of the system volume, likely for extracting the NTDS.dit database.

![running st](Capture/Crownjewels/C4.PNG)

### Step 4: Master File Table (MFT) Analysis

The final phase of our investigation involved analyzing the **$MFT** (Master File Table) file to locate the newly created NTDS.dit dump on disk.

#### Tool Used: MFTECmd

We utilized **MFTECmd**, a powerful tool from Eric Zimmerman's forensic suite, to parse the MFT and generate a comprehensive timeline of file system activities related to the NTDS dump.

**Command**: MFTECmd was used to parse the MFT file and extract timeline information.

#### Key Findings

Through the MFT analysis, we discovered:
- An **unusual file path** where the NTDS.dit database was copied
- The **creation timestamp** of the dumped database file
- Evidence of related files in the same suspicious location

![running st](Capture/Crownjewels/C5.PNG)

### Step 5: Identifying the SYSTEM Hive

Finally, using the suspicious file path identified in the MFT analysis, we located an additional critical file named **"SYSTEM"**.

This SYSTEM hive file is significant because attackers typically need both the NTDS.dit database and the SYSTEM registry hive to successfully extract password hashes offline. The SYSTEM hive contains the Boot Key (SYSKEY) required to decrypt the password hashes stored in NTDS.dit.

## Attack Chain Summary

1. **Initial Access**: Attacker compromised Domain Administrator account
2. **Privilege Abuse**: Modified group memberships for persistence and elevated access
3. **Defense Evasion**: Leveraged legitimate Windows utility (vssadmin) - LOLBIN technique
4. **Credential Dumping**: Created Volume Shadow Copy to access locked NTDS.dit file
5. **Data Exfiltration**: Copied both NTDS.dit and SYSTEM hive to unusual location for offline hash extraction

## Indicators of Compromise

- Unusual `vssadmin.exe` execution outside scheduled maintenance windows
- Volume Shadow Copy Service activation at unexpected times
- Unauthorized group membership modifications
- NTDS.dit and SYSTEM files in non-standard locations
- File system activity correlating with VSS snapshot creation
