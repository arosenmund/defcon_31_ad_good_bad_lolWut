# Common Enumeration

Welcome to the common enumeration section!

BLAH BLAH RYAN FILL ME OUT!

## Beginning Enumeration

Now that you have a domain account, you can begin domain enumeration.

At this point you should still be RDP'd to TWORIVERS from Lighteater (your attack machine). On TWORIVERS, do the following:

1. Use RunAs to run a PowerShell prompt as `WHEEL\Administrator`:
    1. Start menu -> type `run` -> click "Run (App)" -> Enter:
    
    ```powershell
    runas /user:wheel\Administrator powershell.exe
    ```
    
    - You will be prompted to `Enter the password for wheel\Administrator:`, which is:
        
        ```
        12qwaszx!@QWASZX
        ```
    
    A new PowerShell prompt with the title `Administrator: powershell.exe (running as wheel\Administrator` will open. Good!

    Now that we are running PowerShell as a domain admin, let's enumerate AD!

1. Run [Nltest](https://for528.com/nltest) to gather domain information
    
    ```
    nltest /domain_trusts /all_trusts
    nltest /dclist:wheel
    ```
   
    Expected output:
    
    ```powershell
    nltest /domain_trusts /all_trusts
    
    List of domain trusts:
        0: WHEEL wheel.co (NT 5) (Forest Tree Root) (Primary Domain) (Native)
    The command completed successfully
    PS C:\Windows\system32> nltest /dclist:wheel
    Get list of DCs in domain 'wheel' from '\\TARVALON'.
        tarvalon.wheel.co [PDC]  [DS] Site: Default-First-Site-Name
    The command completed successfully
    ```
    - NOTE: If you see the error `You don't have access to DsBind to wheel (\\TARVALON) (Trying NetServerEnum).`, you are not running PowerShell as the `WHEEL\Administrator` user. Please ensure to run step #1 above.
   
   **You now know that the domain name is `WHEEL` and the Primary Domain Controller (PDC) name is `TARVALON`.**

    Just to be safe, as this may be redundant, let's disable Windows Defender:

1. Disable Windows Defender:
    
    ```
    Set-MpPreference -DisableRealtimeMonitoring 1
    ```

## Enumerate via PowerView

!!!! RYAN FINISH THESE NOTES !!!!
!!!! RYAN FINISH THESE NOTES !!!!

The PowerView framework has been around for many years and BLAH BLAH BLAH

!!!! RYAN FINISH THESE NOTES !!!!
!!!! RYAN FINISH THESE NOTES !!!!

1. Import the PowerView module:
    
    ```powershell
    cd C:\Users\Public\Desktop\LAB_FILES\assets
    expand-archive PowerSploit.zip
    cd PowerSploit\PowerSploit\Recon
    . .\PowerView.ps1
    ```

1. Run the following PowerView modules to enumerate AD:
    
    ```powershell
    Get-Domain > c:\perflogs\1.txt
    Get-DomainComputer >> c:\perflogs\1.txt
    Get-DomainUser >> c:\perflogs\1.txt
    Get-DomainObject >> c:\perflogs\1.txt
    ```

1. Open the `1.txt` file in Notepad via:
    
    ```
    notepad c:\perflogs\1.txt
    ```

    **Do not close this PowerShell window.** We will be using it further.

!!!! RYAN FINISH THESE NOTES !!!!
!!!! RYAN FINISH THESE NOTES !!!!

## Enumerate via SharpHound

!!!! RYAN FINISH THESE NOTES !!!!
!!!! RYAN FINISH THESE NOTES !!!!

RYAN PUT SOME NOTES HERE RE: what SharpHound is, why it's used, derp derp

Using the same PowerShell window:

1. Prepare SharpHound for running:

    ```powershell
    cd c:\Users\Public\Desktop\LAB_FILES\assets\
    expand-archive SharpHound-v1.1.1.zip
    cd .\SharpHound-v1.1.1
    ```

1. Run SharpHound to enumerate and collect AD data:
    
    ```powershell
    ./sharphound.exe -d wheel
    ```
    
    - NOTE: This command may take several minutes to complete

    While the command is running, you will see output _similar_ to the following:
    
    ```powershell
    PS C:\Users\Public\Desktop\LAB_FILES\assets\SharpHound-v1.1.1> ./sharphound.exe -d wheel
    
    2023-08-06T19:14:50.1094020+00:00|INFORMATION|This version of SharpHound is compatible with the 4.3.1 Release of BloodHound
    2023-08-06T19:14:50.2863041+00:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
    2023-08-06T19:14:50.3159447+00:00|INFORMATION|Initializing SharpHound at 7:14 PM on 8/6/2023
    2023-08-06T19:14:50.4568173+00:00|INFORMATION|[CommonLib LDAPUtils]Found usable Domain Controller for wheel.co : tarvalon.wheel.co
    2023-08-06T19:14:50.5818340+00:00|INFORMATION|Flags: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
    2023-08-06T19:14:50.7583533+00:00|INFORMATION|Beginning LDAP search for wheel.co
    2023-08-06T19:14:50.7902226+00:00|INFORMATION|Producer has finished, closing LDAP channel
    2023-08-06T19:14:50.7902226+00:00|INFORMATION|LDAP channel closed, waiting for consumers
    ```
    
    When the command finishes, you will see additional output similar to:
    
    ```powershell
    2023-08-06T19:15:21.3732609+00:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 37 MB RAM
    2023-08-06T19:15:36.6673144+00:00|INFORMATION|Consumers finished, closing output channel
    2023-08-06T19:15:36.7306512+00:00|INFORMATION|Output channel closed, waiting for output task to complete
    Closing writers
    2023-08-06T19:15:36.9107179+00:00|INFORMATION|Status: 101 objects finished (+101 2.195652)/s -- Using 45 MB RAM
    2023-08-06T19:15:36.9107179+00:00|INFORMATION|Enumeration finished in 00:00:46.1657331
    2023-08-06T19:15:36.9855680+00:00|INFORMATION|Saving cache with stats: 59 ID to type mappings.
     59 name to SID mappings.
     1 machine sid mappings.
     2 sid to domain mappings.
     0 global catalog mappings.
    2023-08-06T19:15:37.0013392+00:00|INFORMATION|SharpHound Enumeration Completed at 7:15 PM on 8/6/2023! Happy Graphing!
    ```
    
1. Review SharpHound output:
    
    ```powershell
    expand-archive ./*_Bloodhound.zip
    ```
    
        - The output file will have a current timestamp. Thus, the above command will account for whatever timestamp was selected for your file.
    
    1. `cd *_blood` > then hit Tab > then hit enter
        
        - When you hit tab after typing the above, PS will auto-select the proper output directory
        - NOTE: If the above does not work, simply use `ls` to find your output folder. It will be named somethin similar to `cd .\20230812183030_BloodHound\`.

1. List the directory's contents via `ls *`
    
    - Your output may look similar to:
    
    ```
    Mode                 LastWriteTime         Length Name
    ----                 -------------         ------ ----
    -a----          8/6/2023   7:15 PM           6448 20230806191536_computers.json
    -a----          8/6/2023   7:15 PM          27601 20230806191536_containers.json
    -a----          8/6/2023   7:15 PM           2968 20230806191536_domains.json
    -a----          8/6/2023   7:15 PM           3752 20230806191536_gpos.json
    -a----          8/6/2023   7:15 PM          76223 20230806191536_groups.json
    -a----          8/6/2023   7:15 PM           1542 20230806191536_ous.json
    -a----          8/6/2023   7:15 PM          22986 20230806191536_users.json
    ```
    - Your output will have timestamps specific to when you ran SharpHound.
    
    **Begin review of these files in Notepad while Ryan shows you what you can do in Bloodhound up on the projector.**

!!!! RYAN FINISH THESE NOTES !!!!
!!!! RYAN FINISH THESE NOTES !!!!

1. Exfil over RDP
1. File Deletion

!!!! RYAN FINISH THESE NOTES !!!!
!!!! RYAN FINISH THESE NOTES !!!!