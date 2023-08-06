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

| TODO: Note on typical execution via CLI

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

5. Where else can we find strange activity? We can look for common LOL Binaries like comsvcs.dll
```powershell
$event1 | where {$_.message -match 'comsvcs.dll'} | select -ExpandProperty message | findstr CommandLine
```

| TODO: Explanation for comsvcs.dll
| Reference: https://lolbas-project.github.io/#/dump

## Registry Changes

Another thing we can look for is any changes to the registry related to WDigest. 

1. I think you've got these steps down by now!
```powershell
# Set the variable
$event13 = get-winevent -logname Microsoft-Windows-Sysmon/Operational | where {$_.id -eq '13'}

# Filter for WDigest
$event13 | where {$_.message -match 'WDigest'}

# Check the details
$event13 | where {$_.message -match 'WDigest'} | select -ExpandProperty message
```

## File Detections

1. Look for instances of .kribi files
```
cmd /c where /r c:\perflogs *.kirbi 
```

| TODO: Note on file system detection

# Defensive Measures

| TODO: MFA and audit/monitor accounts