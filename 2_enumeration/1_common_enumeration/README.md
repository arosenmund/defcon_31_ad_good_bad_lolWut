# Common Enumeration

Welcome to the common enumeration section! In this workshop, you will be acting as the threat actor (TA). To begin, you will be connecting to the "Windows Attack Server" system within the range. This system serves as the external system from which the TA begins their attack.
- Yes, this system is technically on the same network as the rest of the servers. You're pretending as though it's external to the network.

You will be connecting to the victim network via RDP. Though you will simply be RDP'ing to the initial system, `TWORIVERS`, we are emulating a successful brute force or credential stuffing attack. We are not worried about the method by which the TA obtained the password for the `TWORIVERS\Administrator` account. Rather, we're focusing on what occurs once the initial access has been obtained.

## Connecting

1. Access the "Windows Attack Server" in the range.
    1. Under the "Connections" list, click the "Windows Attack Desktop" link
    
Once the session loads in your browser, you will be at the desktop of the Windows Attack Desktop machine.

1. Launch Remote Desktop Connection to begin the attack:
   `mstsc`
   - Or simply choose "Start" -> "Remote Desktop Connection"
1. Click "Show Options" at the bottom-left of the window so that you can fill out the destination machine & user details
1. Enter `172.31.24.111` for the "Computer" value
    - This is the IP address of the `TWORIVERS` host. Though this is an internal IP, you are pretending as though this is how the TA is connecting into the victim environment from the outside.
1. Enter `Administrator` for the "User name" value
1. Click "Connect" at the bottom-right to initiate the RDP session
1. When prompted, enter the Administrator user's password:
    - `Summerishere@2023!`

*CONGRATS!* You are now connected into the victim environment.

Again, we are pretending as though the TA, you in this case, was able to connect to RDP because they knew the Administrator account's password. This could have been obtained via brute forcing, password spraying, credential stuffing, or other means. For now, we're in!

_Let's enumerate!_

### BONUS: Weak passwords

That Administrator account password is weak, right? You might be surprised how often this type of password is found in large-scale environments.

For a list of insecure passwords that you and your users should avoid at all costs, see:
http://weakpasswords.net/

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