# Detecting Credential Grabbing

To detect credential abuse, we'll be looking for Mimikatz and credential stealing activity using the following methods:
- Token Adjustment (Windows Event ID 4703)
- Process Creation (Sysmon Event ID 1)
- Registry Changes (Sysmon Event ID 13)
- File detections (Direct file system query)

## Token Adjustment

| TODO: Test after logging is enabled
| Note: Additional logging is required for Event ID 4703.

1. We'll create a new variable for Event ID 4703 (Token Adjustment). And if we just list out all these events, it really shows how noisy this event is.
```powershell
## Look for Token Adjustement
$event4703 = get-winevent -logname Security | where {$_.id -eq '4703'}
# Look at all the events:
$event4703
```

2. What we are really interested in, is this event 4703 specifically where SeDebugPrivilege is used, so let's filter on that. 
```powershell
# Filter down a bit:
$event4703 | where {$_.message -match 'SeDebugPrivilege'}
```

3. Let's expand the message, filter on the process name, and see what's going on. 
```powershell
# Expand the message:
$event4703 | where {$_.message -match 'SeDebugPrivilege'} | select -ExpandProperty message | findstr "Process"
```

4. Unfortunately, it looks like OneDrive and Firefox are both using this debug privilege to accomplish something, so let's filter those out. Now we have a much cleaner list to sift through. 
```powershell
# Filter a bit more:
$event4703 | where {$_.message -match 'SeDebugPrivilege' -and $_.message -notmatch 'firefox.exe' -and $_.message -notmatch "OneDriveSetup.exe"} | select -ExpandProperty message
```

| Note: The thing that should stand out here is PowerShell. 

5. Let's change our filter to focus in on the PowerShell. 
```powershell
$event4703 | where {$_.message -match 'SeDebugPrivilege' -and $_.message -match 'powershell.exe'} | select -ExpandProperty message
```

This is in fact the Mimikatz activity. Unfortunately there's no clear indication of that because it was executed using PowerShell, but this is the process you'd need to use when narrowing in on anomalous behavior. And it doesn't make this any easier when vendors like Microsoft and Mozilla are using their applications in weird ways. 

## Process Creation

Typical execution of Mimikatz is done via a single-line command, for example:
```powershell
.\mimikatz.exe "log log.txt" "privilege::debug" "sekurlsa::logonpasswords" exit
``` 

We can start by looking for process creation events with the "sekurlsa" module being used.

1. Set the variable
```powershell
$event1 = get-winevent -logname Microsoft-Windows-Sysmon/Operational | where {$_.id -eq '1'}
```

2. Look for logs containing the string "sekurlsa"
```powershell
$event1 | where {$_.message -match 'sekurlsa'}
```

3. Ok, no results... let's try and see if we have any direct execution of mimikatz
```powershell
$event1 | where {$_.message -match 'mimikatz'}
```

4. Bingo! Let's expand the message and get a clean listing
```powershell
$event1 | where {$_.message -match 'mimikatz'} | select -ExpandProperty message

$event1 | where {$_.message -match 'mimikatz'} | select -ExpandProperty message | findstr CommandLine
```

5. Where else can we find strange activity? We can look for common LOLBins like comsvcs.dll
```powershell
$event1 | where {$_.message -match 'comsvcs.dll'} | select -ExpandProperty message | findstr CommandLine
```

**Analysis Notes**
- What is comsvcs.dll? 
    - This is a dynamic-link library file associated with the Microsoft Distributed Transaction Coordinator (MSDTC). MSDTC is a Windows service providing transaction infrastructure for distributed systems. In other words, it helps manage transactions that span multiple resource managers, such as databases, message queues, and file systems, especially when these resources are spread across multiple computers. In particular, the comsvcs.dll file is responsible for enabling various services related to distributed transactions, including object pooling, transactional object security, and just-in-time activation.
    - As with any piece of system software, while comsvcs.dll is not malicious itself, it can potentially be misused by an attacker. 
    - A great reference is the LOLBAS Project where you can research how various binaries, scripts, and libraries can be used in malicious ways:
        - https://lolbas-project.github.io/#/dump

## Registry Changes

Another thing we can look for is any changes to the registry related to WDigest. 

Some notes on WDigest:
- WDigest is an authentication protocol used by Microsoft's Windows operating system. Introduced in Windows XP, the protocol is designed to be used with HTTP and Simple Authentication Security Layer (SASL) exchanges.
- The main feature of WDigest is that it allows for credentials to be remembered by the system, so users are not repeatedly prompted to input their credentials. However, this is also a potential security concern, as the credentials are stored in memory, making them susceptible to being dumped by malicious software and used in what's known as "pass-the-hash" attacks.
- Because of these security risks, Microsoft introduced changes in Windows 8.1 and Windows Server 2012 R2 to not store passwords in memory by default, although this can be changed with a registry setting. In addition, in the Windows 10 operating system, WDigest is disabled by default.

1. I think you've got these steps down by now!
```powershell
# Set the variable
$event13 = get-winevent -logname Microsoft-Windows-Sysmon/Operational | where {$_.id -eq '13'}

# Filter for WDigest
$event13 | where {$_.message -match 'WDigest'}

# Check the details
$event13 | where {$_.message -match 'WDigest'} | select -ExpandProperty message
```

Using this, we can see the malcious activity where WDigest is being reenabled. After this change is made, credentials can be recovered by an adversary in clear-text for cached login-sessions. 

---

# Additional Notes

For the workshop, this is the end this section's instructions. Below is additional information for your enjoyment! 

Using Mimikatz, an adversary can export the cached tickets to be used in a PTT (Pass The Ticket) attack or to attempt cracking the hashes.
For example:
```powershell
.\mimikatz.exe "log log.txt" "privilege::debug" "sekurlsa::tickets /export" exit
```

When this is done, Mimikatz will store the tickets in a ".kirbi" format. The use of defensive tools or automation can be used to alert on this indicator, however this is a complicated and/or resource intestive task to accomplish at scale. 

For ad-hoc analysis, you can perform a direct query on the filesystem using the following technique:
```powershell
cmd /c where /r c:\perflogs *.kirbi 
```

Detecting a specific file system or file type across thousands of endpoints in an enterprise environment can be complex due to several reasons:
- Variety of Systems: An enterprise environment usually consists of different types of operating systems, each with their own file systems. For example, Windows uses NTFS or FAT32, Linux uses Ext4, XFS or Btrfs, and MacOS uses HFS+ or APFS. Each of these file systems has its own unique structure and metadata which adds to the complexity.
- Large Scale: Enterprises often have thousands of computers, servers, and other devices. Each of these devices can have a multitude of files, leading to billions of files to scan in total. This is a resource-intensive task in terms of both computing power and network bandwidth.
- Distributed Nature: Not all the devices are always connected to the network, and the devices can be geographically dispersed. This further complicates the process of file system detection as you would need to scan systems that are intermittently connected or are in different network segments or VPNs.
- Permission Issues: Different files and directories may have different access permissions. As a result, not all files may be accessible for scanning, especially if the scanning process does not have the necessary permissions.
- Security Considerations: In enterprise environments, there are usually strict security policies and regulations. This might limit the methods and tools you can use to scan the file systems. Additionally, any scanning activity needs to be done in a way that does not compromise the security of the system.
- Performance Impact: Scanning a file system can be a heavy operation. It needs to be done in a way that minimizes the impact on the system's performance, particularly during peak usage hours.
- File Type Identification: Identifying a specific file type is not just about looking at the file extension. It often involves reading the file headers or using other methods to determine the file type, which can be complicated and time-consuming.
- Changes Over Time: The state of a file system is not static. Files get created, modified, and deleted all the time. This dynamic nature adds another level of complexity to the detection process.

# Defensive Measures

Serveral methods can be effective as defensive measures against credential stealing and abuse in an Active Directory environment:

Multi-Factor Authentication (MFA): 
- Use MFA whenever possible. It makes it much harder for an attacker to use stolen credentials because they would need not just the password but also the second factor.

Least Privilege Access: 
- Only provide users with the minimum level of access necessary for them to complete their work. By limiting the scope of access for each user, you reduce the risk associated with a compromised account.

Regularly Audit and Review Accounts and Access: 
- Monitor for unusual access patterns or account behavior, which might indicate that an account has been compromised. Also, remove or reduce access for accounts that are inactive or no longer needed.

Protect Domain Admins: 
- Domain admin accounts have extensive access and are prime targets for attackers. Protect these accounts by using them only when necessary, monitoring them closely, and avoiding their use for routine tasks.

Implement Secure Administrative Workstations (SAWs): 
- These are dedicated machines used solely for sensitive tasks. They have strict security policies and are isolated from internet to prevent malware infections.