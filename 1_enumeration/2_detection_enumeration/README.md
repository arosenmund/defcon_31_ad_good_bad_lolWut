# Detecting Enumeration

Environment Note: 
- Sysmon has been installed and configured in this environment to provide additional logging

## Disabling Windows Defender

| TODO

## Process Execution

1. Start by creating a variable Sysmon Event ID 1. 
```powershell
$event1 = get-winevent -logname Microsoft-Windows-Sysmon/Operational | where {$_.id -like '1'}
```
| Note: This will only include the logs from when you create this variable. If you re-run commands and want to see the updated logs, re-run this command.

2. Now we can filter for any processes where "nltest" was detected
```powershell
$event1 | where {$_.message -match "nltest" | select -ExpandProperty message}
```

3. Filter this down to have a cleaner output
```powershell
$event1 | where {$_.message -match "nltest" | select -ExpandProperty message} | findstr CommandLine
```

4. Repeat for PowerView and SharpHound
```powershell
$event1 | where {$_.message -match "PowerView" | select -ExpandProperty message}

$event1 | where {$_.message -match "SharpHound" | select -ExpandProperty message}
```

| TODO: While this is a faily basic method to look for specific processes being execution, obfuscation of the filename can make this more difficult. However, understanding the behaviors of how these programs operate 

## Process Execution - Using Elastic

| TODO

## Additional Notes

| TODO: RDP Detection

| TODO: File Deletions


# Defensive Measures

| TODO: least privilege and hardened infrastructure
