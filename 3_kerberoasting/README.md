# Kerberoasting

Kerberoasting is an attack that takes advantage of the fact that many organizations do not use Managed Service Accounts in AD. Rather, they assign a Service Principal Name (SPN), often referred to as a "spin", to a general AD account. This effectively makes a general AD user account a service account. However, it leaves the account vulnerable to the kerberoasting attack.

Any domain account can request and receive a Kerberos ticket for an account that is used as a service account from the ticket granting sevice (TGS). The trick here is that these tickets are encrypted using the NTLM hash of the service account. A kerberoasting attack takes place when a domain user requests and receives such a ticket in memory, dumps the ticket to disk, and then cracks the NTLM hash to then obtain the cleartext NTLM password.

You might ask: "Why not have 'super secure' (LONG!) passwords for service accounts? Isn't that a norm?" Yes and NO! Sadly.

## [Common Kerberoasting Tools](./1_common_kerberoasting/README.md)

We begin our adventures into kerberoasting with, you guessed it, some well known tools that automate most of the attack. We first begin with Rubeus, which literally automates the task from A-Z. We will then show you a way to perform the attack with good 'ol Mimikatz, with a little bit of love from PowerShell.

## [Detecting Kerberaosting](./2_detection_kerberoasting/README.md)

This time in our detection phase, we will rely on data contained within an Elasticsearch instance. We have pre-built some dashboards to aid in this analysis. We will be using similar methods of analyzing things like process execution, yet will be doing so within the log aggregation environment.

## [Advanced Kerberoasting & Evading Detections](./3_advanced_kerberoasting/README.md)

As we have in the past two modules, we will again turn defender back on, and execute kerberaosting with a dotnet interface in a powershell one liner.  When ran in a custom runspace, this is near impossible to detect.  The hash is written to the console, not to disk, you have full control over any naming and the users targeted.
