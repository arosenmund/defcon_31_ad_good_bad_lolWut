# Detecting Enumeration

Detecting enumeration in a Windows Domain environment is vital for security, as it can reveal sensitive information about users, groups, and network shares. This information can be exploited by attackers to facilitate unauthorized access or escalate privileges, leading to potential breaches and compromising the integrity of the entire network. Monitoring for enumeration helps in early detection and mitigation of these threats.

To detect Active Directory Enumeration, we'll be looking for anomalous activity on an enpoint using the following methods:
- Registry Changes (Sysmon Event ID 13)
- Process Creation (Sysmon Event ID 1)
- PowerShell Script Block Logging (Windows Event ID 4104)

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
- 

| TODO: Explanation -- Image (MsMpEng.exe), TargetObject, and Details (DWORD set to 1)

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
$event1 | where {$_.message -like "*SharpHound*" | select -ExpandProperty message}
```

| TODO: While this is a faily basic method to look for specific processes being execution, obfuscation of the filename can make this more difficult. However, understanding the behaviors of how these programs operate 

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

| Analysis: Note on indicators being present despite the script name change.

## Additional Notes

| TODO: RDP Detection

| TODO: File Deletions


# Defensive Measures

| TODO: least privilege and hardened infrastructure
