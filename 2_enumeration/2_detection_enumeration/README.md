# Detecting Enumeration

Detecting enumeration in a Windows Domain environment is vital for security, as it can reveal sensitive information about users, groups, and network shares. This information can be exploited by attackers to facilitate unauthorized access or escalate privileges, leading to potential breaches and compromising the integrity of the entire network. Monitoring for enumeration helps in early detection and mitigation of these threats.

To detect Active Directory Enumeration, we'll be looking for anomalous activity on an enpoint using the following methods:
- Registry Changes (Sysmon Event ID 13)
- Process Creation (Sysmon Event ID 1)
- PowerShell Script Block Logging (Windows Event ID 4104)


## Process Creation (nltest and SharpHound execution)

1. Start by creating a variable for Sysmon Event ID 1 (Process Creation). 
```powershell
$event1 = get-winevent -logname Microsoft-Windows-Sysmon/Operational | where {$_.id -eq '1'}
```

2. Now we can filter for any processes where "nltest" was detected
```powershell
$event1 | where {$_.message -match "nltest"} | select -ExpandProperty message
```

3. Filter this down to have a cleaner output
```powershell
$event1 | where {$_.message -match "nltest"} | select -ExpandProperty message | findstr CommandLine
```

4. (Optional) Repeat for SharpHound
```powershell
$event1 | where {$_.message -like "*SharpHound*"} | select -ExpandProperty Message
```

| Analysis: While this is a faily basic method to look for specific processes being execution, obfuscation of the filename can make this more difficult. However, we can use this to understand the behaviors of how these programs operate. 

## Script Block Logging (PowerView being loaded)

1. Once again, let's start by creating a variable! This time for Windows Event ID 4104 (PowerShell Script Block Logging)
```powershell
$event4104 = get-winevent -logname Microsoft-Windows-Powershell/Operational | where {$_.id -eq '4104'}
```

2. We could directly look for the script PowerView.ps1 being executed
```powershell
$event4104 | where {$_.message -like "*PowerView.ps1*"}
```

3. But what if the name of the script file has been changed? Let's filter for any instances of the phrase "PowerView" and see what we get.
```powershell
$event4104 | where {$_.message -like "PowerView*"} | findstr PowerView
```

4. And finally, let's filter out any of the "Path" results
```powershell
$event4104 | where {$_.message -like "PowerView*"} | findstr PowerView | findstr /V Path
```

| Analysis: One of the simplest things an adversary may change is the filename. Adversaries may also make basic modifications like adding comments into the code to change the hash value. These simple obfuscation techniques can be defeated by looking for other indicators. 

---

For the workshop, this is the end this section's instructions. Below is additional information for your enjoyment! 

---

# Additional Notes

**What about RDP?**
Detecting malicious activity within RDP traffic can be challenging for several reasons:
- Encryption: RDP traffic is encrypted, which means that it's not easy to inspect the contents of the data being sent and received. Without being able to see what's inside the packets, it can be difficult for network monitoring tools to identify whether they contain any malicious activity.
- Legitimate Use: RDP is a commonly used protocol for legitimate purposes. This means that simply seeing RDP traffic on a network isn't necessarily a cause for alarm. Distinguishing between legitimate and malicious RDP traffic can be challenging.
- Behavioral Similarities: The activities performed by a malicious actor using RDP may not look that different from those performed by a legitimate user. For example, both might involve logging in, opening applications, viewing files, and so forth. Identifying a set of behaviors that are uniquely associated with malicious use of RDP can be difficult.
- Lack of Logs: RDP sessions might not always generate logs, or the logs that are generated might not contain enough detail to identify malicious activity. This can make it hard to track what happened during a session.
- Credential Use: If a malicious actor is able to obtain legitimate credentials (for example, through phishing, brute force attacks, or data breaches), they may be able to log into an RDP session just as a legitimate user would. This can make it difficult to detect the intrusion.
- Persistence: Attackers can configure RDP sessions to reconnect automatically, making their activity look more like normal, ongoing connectivity rather than an intrusion.

# Defensive Measures

Several methods can be effective as defensive measures against enumeration techniques in an Active Directory environment:

Least Privilege: 
- By adhering to the principle of least privilege, each user is given the minimum levels of access – or permissions – they need to perform their job functions. This limits the ability of unauthorized users, malicious insiders, or compromised accounts to access, manipulate, or enumerate sensitive data within the AD. If an account is compromised, the damage potential is limited because the account does not have unnecessary permissions.

Hardened Infrastructure: 
- Hardening your infrastructure involves implementing security measures to protect your environment against threats. This could include tactics like segmenting the network to limit lateral movement, installing up-to-date antivirus and intrusion detection/prevention systems, keeping systems and software patched and up-to-date, and disabling or removing unnecessary services or software.
