# Common Enumeration

1. Login to the Client (tarvalon).
2. Open a powershell terminal.
3. Get the domain infomration of the computer:
`nslookup client01`
1. Run NLTEST
   1. `nltest /dclist:wheel`
   2. `nltest /domain_trusts`
   3. `nltest /domain_trusts /all_trusts`
2. Run powerview.
   1. Open a powershell terminal.
   2. Cd C:\Users\Public\LAB_FILES\