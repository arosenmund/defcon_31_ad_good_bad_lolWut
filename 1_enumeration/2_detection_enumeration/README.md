# Detecting Enumeration

Detecting enumeration in a Windows Domain environment is vital for security, as it can reveal sensitive information about users, groups, and network shares. This information can be exploited by attackers to facilitate unauthorized access or escalate privileges, leading to potential breaches and compromising the integrity of the entire network. Monitoring for enumeration helps in early detection and mitigation of these threats.

| Environment Note: Sysmon has been installed and configured in this environment to provide additional logging.

We will be using the Client endpoint as our "Defender Endpoint." Start by logging into the Client (TWORIVERS).

## Registry Value - Disabing Windows Defender

1. We will be using the Client endpoint as our "Defender Endpoint." Start by logging into the Client (tarvalon) if you aren't already logged in.

2. Start by creating a variable for Sysmon Event ID 13 (Registry value set)
```powershell
$event13 = get-winevent -logname Microsoft-Windows-Sysmon/Operational | where {$_.id -eq '13'}
```
| Note: This will only include the logs from when you create this variable. If you re-run commands and want to see the updated logs, re-run this command.

3. Filter for the target object of "DisableRealtimeMonitoring" being set (or changed)
```powershell
$event13 | where {$_.message -like "*DisableRealtimeMonitoring*"} 
```

4. Let's expand the message information to get more details
```powershell
$event13 | where {$_.message -like "*DisableRealtimeMonitoring*"} | select -ExpandProperty message
```

| TODO: Explanation -- Image (MsMpEng.exe), TargetObject, and Details (DWORD set to 1)

## Sysmon - Process Execution

1. Start by creating a variable for Sysmon Event ID 1. 
```powershell
$event1 = get-winevent -logname Microsoft-Windows-Sysmon/Operational | where {$_.id -like '1'}
```

2. Now we can filter for any processes where "nltest" was detected
```powershell
$event1 | where {$_.message -match "nltest"} | select -ExpandProperty message
```

3. Filter this down to have a cleaner output
```powershell
$event1 | where {$_.message -match "nltest"} | select -ExpandProperty message | findstr CommandLine
```

4. Repeat for PowerView and SharpHound
```powershell
$event1 | where {$_.message -like "*PowerView*" | select -ExpandProperty message}

$event1 | where {$_.message -like "*SharpHound*" | select -ExpandProperty message}
```

| TODO: While this is a faily basic method to look for specific processes being execution, obfuscation of the filename can make this more difficult. However, understanding the behaviors of how these programs operate 

## Process Execution - Using Elastic

| TODO

## Additional Notes

| TODO: RDP Detection

| TODO: File Deletions


# Defensive Measures

| TODO: least privilege and hardened infrastructure
