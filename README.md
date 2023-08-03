# Active Directory Attacks: The Good, The Bad, and the LoLWUT?

## Introduction to Active Director Post Exploitation

Welcome to our Defcon Workshop!  This is going to be an intense 4 hours, but our commitment to you is that everyone will get something out of this. Whether red team, blue team, or just here for the lolz.  

This is an extremely hands on workshop.  However, the lab is hosted over the internet by Pluralsight **FOR FREE** (thanks pluralsight).  Each of you will get an individual lab environment, but it does take some setup from you.

While we introduce ourselves.

[Aaron Rosenmund](AaronRosenmund.com)
[Brandon DeVault](DevaultSecurity.com)
[Ryan Chapman](https://incidentresponse.training/)

Please proceed to the [Setup](./0_setup/README.md) page to get your lab environment started.












Outline:


Introduction (15 mins)
 - Welcome and Introductions
    - Overview of the workshop
    - Explain the focus: Active Directory Tradecraft
    - Discuss the techniques covered: Enumeration, Credential Grabbing, and Kerberoasting
 - Explain the structure: Attack -> Detect/Defend -> Exploit Development
 - Set ground rules and expectations (active participation, respectful environment, encouraging questions and discussion!)
Environment Setup (30 mins)
 - GitHub Lab Instructions
 - Setup free accounts - Pluralsight Lab Environment
 - Troubleshooting
Active Directory Enumeration (1 hr)
  - Ryan - Enumerating AD (20 mins)
    - Traditional Enumeration (ADUC) and LDAP queries
    - Enumeration with PowerShell - PowerView
    - Enumeration with BloodHound
  - Brandon - Detecting AD Enumeration (20 mins)
    - Increased LDAP queries
    - Unusual PowerShell commands
    - Suspicious network patterns
    - Detecting enumeration activity: log monitoring and IDS
    - Defensive Measures: least privilege and hardened infrastructure
  - Aaron - [Advancing enumeration techniques](./advanced_enumderation.md) (20 mins)
    - ADSISEARCHER type accelerator and POWERSHELL_ISE
    - LDAP queries via COM by using ADO's ADSI provider
    - Potential detections
Active Directory Credential Grabbing (1 hr)
  - Ryan - Credential Grabbing (20 mins)
    - Mimikatz
    - CrackMapExec
  - Brandon - Detecting Credential Theft (20 mins)
    - Unusual processes and services
    - Exploring event logs
    - Defensive measures: MFA and audit/monitor accounts
  - Aaron - [Advanced Credential Grabbing](./advanced_credentials.md) (20 mins)
    - Process impersonation
    - Process Herpaderping(Yep its a thing)
    - Potential Detections
Kerberoasting (1 hr)
  - Ryan - Techniques for Kerberoasting (20 mins)
    - Using Rubeus
    - Using Impacket
    - PowerShell Scripts
  - Brandon - Detecting Kerberoasting (20 mins)
    - Unusual Kerberos service ticket requests
    - Suspicious PowerShell activity
    - Unexpected encryption downgrade attempts
    - Defensive measures: least privilege access, monitor service accounts, and strong encryption for service account tickets
  - Aaron - [Advanced Kerberoasting Techniques](./advanced_kerberoast.md) (20 minutes)
    - ADSISEARCHER and From Scratch Ticket Requests
    - Certified Pre-Owned Certificate Creation for ADCS
    - Potential Detections
Conclusion (15 mins)
  - Recap of the scenario
  - Additional resources and next steps for learning
  - Final Q&A and open discussion
