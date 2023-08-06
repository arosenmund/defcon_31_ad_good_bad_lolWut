# Advanced Credential Grabbing



## Custom LSASS Dump

> Turning defender off, can either be difficult, or not guarunteed to stay that way due to domain policies, or even other EDR implementations. So what to do?
> Even using AMSI bypasses can be concerning these days due to the frequency at which the signature are updated even for the bypasses. 
> More advanced actors are using custom tooling.  There are two considerations.  Executables can be executed in memory, but then the process is not directly tied to an executable. So you can inject into another process that is already running, which is really the way to go.
> Other options are to bring your own executable, or use signed executables already on the box to execute DLL files through sideloading to do your bidding.

Some examples of these techniques are what we will cover here and in the other adanced modules.

1. From the Lighteater (Attacker Box), RDP into the TWORIVERS (client) with the local administrator account.  TWORIVERS\Administrator:Summerishere@2023!
2. Open an Administrative powershell terminal.
3. Change directory `cd c:\Users\Public\Desktop\LAB_FILES\assets`
4. Unzip the custom-procdump folder.
`expand-archive .\custom-procdump.zip`
5. CD into the "custom-procdump" folder.
6. Execute the nimprocdump.exe program.
> This will find the process with lsass, and dump it into a file.  Important not to add this kind of info into a command line while you have commandline process auditing on. Which is why you build the options into the executable, and then additional information like PIDs are logged as arguments, which can later be used for detections.

**use nim custom .dll inject maybe, if I can make it work**

## Create Domain Save with 
https://www.microsoft.com/en-us/security/blog/2023/05/24/volt-typhoon-targets-us-critical-infrastructure-with-living-off-the-land-techniques/

If you then end up with domain creds, one next step can be to grab the has credentials from the entire domain of course! There are a number of ways that have been flagged to essentaill steal the ntds.dit file from the Domain Controller.

Notes: VSSADMIN gets caught because it's use is so commonly associated with the steps that happen before you are ransomed.  Other tools like ninja copy require extended interaction within a remote session of powershell and loading of an additional library, all super noisy.

> If you can't tell, I reall think using tools that already exist on the sytem, or are commonly used by admins helps stay under the radar, and it makes detections for automated tooling really difficult. One such way is to use the ntdsutil tool that is built into domain controllers to create backup media.  Further, using wmic is just really hard to catch.  This is being actively used in the volt typhoon campaigns.

1. Remote into the TWORIVERS (client ) 172.31.24.111 with the domain admin credentials: wheel\ralthor : JustAn0therG!ng3r
2. As this user, open a administrative command prompt. (Leave defender on.)
3. Run the following command to launch this remote process on the Domain Controller.

```batch
wmic /node: /user: /password: process call create ntdsutil \"ac i ntds\" ifm \"create full c:\Windows\temp\pew\" qq"
```
4. Retrieve the file from sysvol.
5. Use the DINTERNALS package to pull the hashes.
`get-addbaccount -all -dbPath c:\ntds.dit -bootkey $key`
6. Recommend dropping these into a file, and they can be used later for pass the hash attacks.


> From here you sould properly cover your tracks, but that is a different workshop.

## Total Not Mimikatz Ticket Generation

