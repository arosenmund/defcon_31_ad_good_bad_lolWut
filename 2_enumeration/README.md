# Active Directory Enumeration

Ryan to include notes here. I got this, go away.

## [Common Active Directory Enumeration Techniques](./1_common_enumeration/README.md)

Ryan to include notes here. I got this, go away.

## [Detecting Active Directory Enumeration](./2_detection_enumeration/README.md)

Ryan to include notes here. I got this, go away.

## [Advanced Enumeration & Evading Detections](./3_advanced_enumeration/README.md)

This will follow the same cycle as before. The tools used for enumeration, such as sharhound, and commonly adrecon, are well known and easily detectible. Even with obsfuscation, the number of modules included in tools like rubeus make it near impossible to compile without it being detected.  Less is more.  In this specific case, the dotnet interface within powershell allows for the use of a library called ADSIsearcher.  This library is included on all windows machines regardless of whether the AD tools are installed!  Further, they do not show up in process logging.  

These can and should be used in a custom runspace, or unmanaged powershell, within a compromised legitamate process.

This will evade the following detections:
- Common tool signatures
- Excessive LDAP queries to the domain controller
- Writing output to disk

