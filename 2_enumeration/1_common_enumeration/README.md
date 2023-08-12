# Common Enumeration

Welcome to the common enumeration section!

Now that you have a domain account, you can begin domain enumeration. Here in the common enumeration section, you will be using tooling common to ransomware actors. These tools are noisy! Not only do detections exist, but so do prevention methods. Even though these methods are well-known and recycled time and time again by such actors, the sad truth is that they are quite effective. Our hope here is to show you how these methods work so that you can help prevent and detect them in your networks.

## Beginning Enumeration

We will begin with a common enumeration tool included on most DCs called `nltest`. This tool provides basic information concerning domain trusts and DC names. 

At this point you should still be RDP'd to TWORIVERS from Lighteater (your attack machine). On TWORIVERS, do the following:

1. Use RunAs to run a PowerShell prompt as `WHEEL\Administrator`:
    
    - Start menu -> type `run` -> click "Run (App)" -> enter:
    
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
    
    ```powershell
    Set-MpPreference -DisableRealtimeMonitoring 1
    ```

## Enumerate via PowerView

[PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) is a part of the (now defunct, yet still used widely) [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) framework made by a group named the PowerShell Mafia. The framework has been around for many years and is a go-to for many TAs. While PowerSploit offers numerous attack tools such Invoke-Mimikatz and Invoke-Shellcode, we will be focusing on PowerView, which is a stand-alone, importbale PS module.

Before we can use PowerView, we must import the module. We will be importing a module from disk, as we are pretending as though the TA copied the file into the victim envrionment via RDP. Another method used commonly is importation of the script following a download via a PS "download cradle," which is a fun term used to reference a string of PS code that can download data from the internet. Again, we'll be loading the file directly. Let's do it!

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

**CONGRATS! You have enumerated AD via PowerView!** 

Your instructor will walk through some of the data in the `1.txt` file.

## Enumerate via SharpHound

[BloodHound](https://github.com/BloodHoundAD/BloodHound) is one of the most common enumeration tools you will see in attacks such as ransomware campaigns. The best description of BloodHound comes directly from the author's via their GitHub repo:

"BloodHound uses graph theory to reveal the hidden and often unintended relationships within an Active Directory or Azure environment. Attackers can use BloodHound to easily identify highly complex attack paths that would otherwise be impossible to quickly identify. Defenders can use BloodHound to identify and eliminate those same attack paths. Both blue and red teams can use BloodHound to easily gain a deeper understanding of privilege relationships in an Active Directory or Azure environment."

While BloodHound is the tool that does the relationship mapping, the tool requires a data set provided through a collector. A few collectors exist, but the most widely used collector is [SharpHound](https://github.com/BloodHoundAD/SharpHound), which is exactly what we'll be using.

Using the same elevated PowerShell window as you've been using:

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
    
    - `cd *_Blood` > then hit Tab > then hit enter
        
        - When you hit tab after typing the above, PS will auto-select the proper output directory. Make sure to type a capitabl "B" or it may not work.
        - NOTE: If the above does not work, simply use `ls` to find your output folder. It will be named somethin similar to `cd .\20230812183030_BloodHound\`.

1. List the directory's contents via `ls *`
    
    - Your output may look similar to:
    
    ```powershell
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

**Congrats! You have now enumerated the environment via BloodHound!**

Normally the TA would not expand the archive within the environment. Rather, they would simply copy the results archive out via RDP or another method, then import into BloodHound for review.

Your instructor will now show you some analysis of this data set in Bloodhound.
