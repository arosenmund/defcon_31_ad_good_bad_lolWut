# Active Directory Enumeration

TAs typically enumerate a victim's AD environment in order to learn more about the environment. Enumeration can provide details concerning the network while also identifying vulnerabilities within the environment. Essentially, enumeration can help you understand where and what to attack. What systems are out there? What elevated accounts exist? Etc. etc.

## [Common Active Directory Enumeration Techniques](./1_common_enumeration/README.md)

Ransomware and other low-level actors often use well-known, easy-to-detect tools such as AdFind, AdRecon, PowerView, and BloodHound to perform enumeration. In the common enumeration section, you will learn how to use PowerView and SharpHound, the most common collector used for BloodHound. The term "Bring Your Own Tools" (BYOT) refers to when TAs bring such tools into a victim environment. BYOT methods include downloading via file sharing sites (see [https://for528.com/lots](https://for528.com/lots) for commonly used sites), copying files in via RDP, and more. In our case, we will be emuating a TA copying files into the network via an RDP session.

## [Detecting Active Directory Enumeration](./2_detection_enumeration/README.md)

Similar to credential gathering attacks, registry changes and process creation monitoring can be extremely useful for detecting AD enumeration. Probably no surprise to anyone, WEL events can also be very useful. This time around you will be learning to use WEL Event ID 4104 (PowerShell Script Block Logging) to identify AD enumeration tooling and methods.

## [Advanced Enumeration & Evading Detections](./3_advanced_enumeration/README.md)

This will follow the same cycle as before. The tools used for enumeration, such as SharpHound, and commonly adrecon, are well known and easily detectible. Even with obsfuscation, the number of modules included in tools like rubeus make it near impossible to compile without it being detected.  Less is more.  In this specific case, the dotnet interface within powershell allows for the use of a library called ADSIsearcher.  This library is included on all windows machines regardless of whether the AD tools are installed!  Further, they do not show up in process logging.  

These can and should be used in a custom runspace, or unmanaged powershell, within a compromised legitamate process.

This will evade the following detections:
- Common tool signatures
- Excessive LDAP queries to the domain controller
- Writing output to disk

