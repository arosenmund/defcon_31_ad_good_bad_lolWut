# Detecting Credential Grabbing

To detect credential abuse, we'll be looking for Mimikatz activity using three methods:
- Windows Event ID 4703 (Token Adjustment)
- File detections

## Windows Event Logs

| Note: Additional logging is required for Event ID 4703.

1. We'll create a new variable for Event ID 4703 (Token Adjustment). And if we just list out all these events, it really shows how noisy this event is.
```powershell
## Look for Token Adjustement
$event4703 = get-winevent -logname Security | where {$_.id -like '4703'}
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

## File Detections

1. Look for instances of .kribi files
```
cmd /c where /r c:\perflogs *.kirbi 
```

| TODO: Note on file system detection

## Process Execution Using Sysmon

1. Set the variable
```powershell
$event1 = get-winevent -logname Microsoft-Windows-Sysmon/Operational | where {$_.id -like '1'}
```

2. Look for logs containing the string "sekurlsa"
```powershell
$event1 | where {$_.message -match 'sekurlsa'}

# Expand the message field to see all the details
$event1 | where {$_.message -match 'sekurlsa'} | select -ExpandProperty message

# Filter the output to get a clean listing
$event1 | where {$_.message -match 'sekurlsa'} | select -ExpandProperty message | findstr CommandLine
```


# Defensive Measures

| TODO: MFA and audit/monitor accounts