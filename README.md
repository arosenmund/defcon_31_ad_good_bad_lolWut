# Active Directory Attacks: The Good, The Bad, and the LoLWUT?

## Introduction to Active Director Post Exploitation

Welcome to our Defcon Workshop! This is going to be an intense 4 hours, but our commitment to you is that everyone will get something out of this. Whether red team, blue team, or just here for the lolz.

This is an _extremely_ hands on workshop. However, the lab is hosted over the internet by Pluralsight **FOR FREE** (thanks Pluralsight!). Each of you will get an individual lab environment, but it does take some setup from you.

While we introduce ourselves, **please proceed to the [Setup](./0_setup/README.md) page to get your lab environment started.**

A little about us:
- [Aaron Rosenmund](https://aaronrosenmund.com/about/)
- [Brandon DeVault](https://www.devaultsecurity.com/about/)
- [Ryan Chapman](https://incidentresponse.training/)

## The Flow

In this workshop, you will act as the threat actor (TA) breaking into an environment via Remote Desktop Protocol (RDP). We are not focusing on the method by which the TA (that's _you!_) accessed the environmnet. Rather, our focus is on the attacks that will be carried out against Active Directory (AD). After all, this is a workshop on AD attacks!

All of the information you need to follow along with the workshop, is included in the following sections:

### [Active Directory Credential Grabbing](./1_ad_credential_grabbing/)

The TA will obtain access to the network via a local administrator account. As such, the TA (again, that's _you!_) does not have access to a domain account. In order to enumerate the AD environment, the TA will need to obtain access to a domain account. That is exactly what you will be doing in this section.

### [Active Directory Enumeration](./2_enumeration/)

Once the TA obtains access to a domain account, they will begin AD enumeration. We will begin enumeration using common methodologies, which of course are easily detectable. After showing you how to detect these methods, we will move to a more advanced methodology.

### [Active Directory Kerberoasting](./3_kerberoasting/)

One of the most common attacks against AD is an attack on Kerberos known as Kerberoasting. In this section, you will learn how to perform a Kerberoasting attack using common tools. You will then learn how to perform a much more stealth version of this attack.

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
    - Using Impacket
    - PowerShell Scripts
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
