# Detecting Credential Grabbing

Detecting credential abuse in an Active Directory environment is vital to safeguard organizational assets, protect user identities, and prevent unauthorized access that can lead to data breaches and system compromises.

To detect credential abuse, we'll be looking for Mimikatz and credential stealing activity using the following methods:
- Process Creation (Sysmon Event ID 1)
- Registry Changes (Sysmon Event ID 13)
- File detections (Direct file system query)

| Environment Note: Sysmon has been installed and configured in this environment to provide additional logging.

**We will be using the Client endpoint as our "Defender Endpoint." Start by logging into the Client (TWORIVERS).**

## Registry Value Set (Disabing Windows Defender)

1. Start by creating a variable for Sysmon Event ID 13 (Registry value set)
```powershell
$event13 = get-winevent -logname Microsoft-Windows-Sysmon/Operational | where {$_.id -eq '13'}
```
| *Note:* This will only include the logs from when you create this variable. If you re-run commands and want to see the updated logs, re-run this command.

| *Note 2:* You may get an error talking about a parameter reference - this is ok!

2. Filter for the target object of "DisableRealtimeMonitoring" being set (or changed)
```powershell
$event13 | where {$_.message -match "DisableRealtimeMonitoring"} 
```

3. Let's expand the message information to get more details
```powershell
$event13 | where {$_.message -like "*DisableRealtimeMonitoring*"} | select -ExpandProperty message
```

**Analysis Notes:**
- What is MsMpEng.exe?
    - This is a core process of Windows Defender, responsible for scanning files for malware when they are accessed, scheduling and performing system scans, updating malware definitions, and other related tasks.
    - In this instance, we see this listed as the "Image," or executable, being used to modify the registry.
- The "TargetObject" field in the Registry Key being modified.
- The "Details" field shows the DWORD being set to a value of "1"
- Based on all of this, we can determine what actually took place here! Research/Googling may be required!


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

I think you've got these steps down by now!
```powershell
# Set the variable (Not necessary if you did this up above)
$event13 = get-winevent -logname Microsoft-Windows-Sysmon/Operational | where {$_.id -eq '13'}

# Filter for WDigest
$event13 | where {$_.message -match 'WDigest'}

# Check the details
$event13 | where {$_.message -match 'WDigest'} | select -ExpandProperty message
```

Using this, we can see the malcious activity where WDigest is being reenabled. After this change is made, credentials can be recovered by an adversary in clear-text for cached login-sessions. 

---

For the workshop, this is the end this section's instructions. Below is additional information for your enjoyment! 

---

# Additional Notes

## Token Adjustment

Auditing for Token Adjustments can be enabled, which will create a new security log: Event ID 4703. It's worth noting that this event can be extremely noisy and difficult to discover malicious traffic. However, this is an additional method if Sysmon or other data sources are unavailable.

| Note: Additional logging is required for Event ID 4703. This is not currently enabled in the lab environment.

Here is an example workflow:

```powershell
# Create a new variable for Event ID 4703 (Token Adjustment). And if we just list out all these events, it really shows how noisy this event is.
$event4703 = get-winevent -logname Security | where {$_.id -eq '4703'}
# Look at all the events:
$event4703

# What we are really interested in, is this event 4703 specifically where SeDebugPrivilege is used, so let's filter on that. 
$event4703 | where {$_.message -match 'SeDebugPrivilege'}

# Let's expand the message, filter on the process name, and see what's going on. 
$event4703 | where {$_.message -match 'SeDebugPrivilege'} | select -ExpandProperty message | findstr "Process"

# Unfortunately, it looks like OneDrive and Firefox are both using this debug privilege to accomplish something, so let's filter those out. Now we have a much cleaner list to sift through. 
$event4703 | where {$_.message -match 'SeDebugPrivilege' -and $_.message -notmatch 'firefox.exe' -and $_.message -notmatch "OneDriveSetup.exe"} | select -ExpandProperty message

# Note: The thing that should stand out here is PowerShell. 

# Let's change our filter to focus in on the PowerShell. 
$event4703 | where {$_.message -match 'SeDebugPrivilege' -and $_.message -match 'powershell.exe'} | select -ExpandProperty message
```

This workflow can identify Mimikatz activity. Unfortunately there's no clear indication of that because it was executed using PowerShell, but this is the process you'd need to use when narrowing in on anomalous behavior. And it doesn't make this any easier when vendors like Microsoft and Mozilla are using their applications in weird ways. 

---

# Defensive Measures

Several methods can be effective as defensive measures against credential stealing and abuse in an Active Directory environment:

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