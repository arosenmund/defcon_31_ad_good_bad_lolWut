## Outline

Introduction (15 mins)
 - Welcome and Introductions
    - Overview of the workshop
    - Explain the focus: Active Directory Tradecraft
    - Discuss the techniques covered: Credential Grabbing, Enumeration, and Kerberoasting
 - Explain the structure: Attack -> Detect/Defend -> Exploit Development
 - Set ground rules and expectations (active participation, respectful environment, encouraging questions and discussion!)
 
Environment Setup (30 mins)
 - GitHub Lab Instructions
 - Setup free accounts - Pluralsight Lab Environment
 - Troubleshooting

Active Directory Credential Grabbing (1 hr)
  - Ryan - Credential Grabbing (20 mins)
    - Mimikatz
  - Brandon - Detecting Credential Theft (20 mins)
    - Unusual processes and services
    - Exploring event logs
    - Defensive measures: MFA and audit/monitor accounts
  - Aaron - Advanced Credential Grabbing (20 mins)
    - Process impersonation
    - Process Herpaderping(Yep its a thing)
    - Potential Detections

Active Directory Enumeration (1 hr)
  - Ryan - Enumerating AD (20 mins)
    - Traditional enumeration via nltest
    - Enumeration with PowerShell - PowerView
    - Enumeration with BloodHound
  - Brandon - Detecting AD Enumeration (20 mins)
    - Increased LDAP queries
    - Unusual PowerShell commands
    - Suspicious network patterns
    - Detecting enumeration activity: log monitoring and IDS
    - Defensive Measures: least privilege and hardened infrastructure
  - Aaron - Advancing enumeration techniques (20 mins)
    - ADSISEARCHER type accelerator and POWERSHELL_ISE
    - LDAP queries via COM by using ADO's ADSI provider
    - Potential detections

Kerberoasting (1 hr)
  - Ryan - Techniques for Kerberoasting (20 mins)
    - Using Rubeus
    - Using Mimikatz
  - Brandon - Detecting Kerberoasting (20 mins)
    - Unusual Kerberos service ticket requests
    - Suspicious PowerShell activity
    - Unexpected encryption downgrade attempts
    - Defensive measures: least privilege access, monitor service accounts, and strong encryption for service account tickets
  - Aaron - Advanced Kerberoasting Techniques (20 minutes)
    - ADSISEARCHER and From Scratch Ticket Requests
    - Certified Pre-Owned Certificate Creation for ADCS
    - Potential Detections

Conclusion (15 mins)
  - Recap of the scenario
  - Additional resources and next steps for learning
  - Final Q&A and open discussion
