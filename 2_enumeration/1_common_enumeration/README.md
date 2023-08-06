# Common Enumeration

Welcome to the common enumeration section! blah blah

## Beginning Enumeration

You are now ready to begin enumerating the environment.

1. Open PowerShell as Administrator
    1. Start menu -> Type `powershell` -> Right-click on "Windows PowerShell" and select "Run as administrator"
        1. Do _not_ double-click `PowerShell 7` on the desktop, as we do not want PS7

1. Run `nltest` to gather domain information
    1. `nltest /domain_trusts /all_trusts`
    1. `nltest /dclist:wheel`
   
    Expected output:
    ```
    PS C:\Users\Administrator\Desktop> nltest /domain_trusts /all_trusts
    List of domain trusts:
        0: WHEEL wheel.co (NT 5) (Forest Tree Root) (Primary Domain) (Native)
    The command completed successfully
    ```
    
    ```
    PS C:\Users\Administrator\Desktop> nltest /dclist:wheel
    Get list of DCs in domain 'wheel' from '\\TARVALON'.
    You don't have access to DsBind to wheel (\\TARVALON) (Trying NetServerEnum).
    List of DCs in Domain wheel
        \\TARVALON (PDC)
    The command completed successfully
    ```
   
   - You now know that the domain name is `wheel` and the DC's name is `TARVALON`.

## Enumerate via SharpHound

1. Disable Windows Defender:
    1. `Set-MpPreference -DisableRealtimeMonitoring 1`

1. Run SharpHound
   1. `cd c:\Users\Public\Desktop\LAB_FILES\assets\`
   1. `expand-archive SharpHound-v1.1.1.zip`
   1. `cd .\SharpHound-v1.1.1`
   1. `./sharphound.exe -d wheel`  **if you are already in domain user context, you don't need to use the clear text pw, but up to you**
   1. This generates the zip file and the bin you need for graphing.

1. Exfil over RDP
1. File Deletion

## BONUS: RYAN REMOVE ME

1. Run PowerView:
   1. `cd C:\Users\Public\Desktop\LAB_FILES\assets`
   1. `Expand-Archive PowerSploit.zip`
   1. `cd PowerSploit\PowerSploit\Recon`
   1. `. .\PowerView.ps1`   
   1. `get-cachedrdpconnection >> c:\perflogs\1.txt`  **doesn't work need config information**
   1. `invoke-userhunter >> c:\perflogs\2.txt` **works when ran with domain users, need to add more users etc.**