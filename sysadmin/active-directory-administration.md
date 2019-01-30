---
description: This page is dedicated to any and all Active Directory administration
---

# Active Directory Administration

## USER / HOST ENUMERATION

### Get Username and Email Addresses

`Get-ADUser -Filter * -Properties DisplayName, sAMAccountName | Select DisplayName, sAMAccountName` 

### Get Active Win10 Machine Patch Level \(Last Logon in 60 Days\)

`$LastLogon = (Get-Date).Adddays( -(60) ); $Workstations = Get-ADComputer -Filter { LastLogonTimeStamp -gt $LastLogon -and OperatingSystem -like 'Windows 10'} -Properties *; $Workstations = $Workstations | Select-Object -Property DNSHostname,OperatingSystem,OperatingSystemVersion,IPv4Address,LastLogonDate,DistinguishedName,SID; Export-Results -Output $Workstations -Path "C:\Users\burmat\Desktop\Workstations.csv"`

### Get Hosts Last Logon

Iterate all computer objects in a given domain and get the date/time for the last time they were logged into:

```text
Import-Module ActiveDirectory

function Get-ADHostsLastLogon() {

    $hnames = Get-ADComputer -Filter 'ObjectClass -eq "Computer"' | Select -Expand Name

    foreach ($hname in $hnames) {
        $dcs = Get-ADDomainController -Filter {Name -like "*"}
        $time = 0
        foreach($dc in $dcs) { 
            $computer = Get-ADComputer $hname | Get-ADObject -Properties lastLogon 
            if($computer.LastLogon -gt $time) {
                $time = $computer.LastLogon
            }
        }
        
        $dt = [DateTime]::FromFileTime($time).ToString('g')
        # 12/31/1600 will result if $time = 0 (never logged on before)
        Write-Host $dt", " $hname
    }
    Write-Host "Done."
}

Get-ADHostsLastLogon
```

_\(Find my most recent copy on_ [_my GitHub_](https://github.com/burmat/burmatscripts/blob/master/powershell/Get-ADHostsLastLogon.ps1)_\)_

### Get Users Last Logon

To iterate all user objects in AD and get their last logon time, use:

```text
Import-Module ActiveDirectory

function Get-ADUserLastLogon([string]$userName) {

    $dcs = Get-ADDomainController -Filter {Name -like "*"}
    $time = 0
    foreach($dc in $dcs) { 
        $hostname = $dc.HostName
        $user = Get-ADUser $userName | Get-ADObject -Properties lastLogon 
        if($user.LastLogon -gt $time) {
            $time = $user.LastLogon
        }
    }
    
    $dt = [DateTime]::FromFileTime($time)
    Write-Host $username "last logged on at:" $dt 
}

$unames = Get-ADUser -Filter 'ObjectClass -eq "User"' | Select -Expand SamAccountName
foreach ($uname in $unames) { Get-ADUserLastLogon($uname); } 
```

_\(Find my most recent copy on_ [_my GitHub_](https://github.com/burmat/burmatscripts/blob/master/powershell/Get-ADUserLastLogon.ps1)_\)_

### Get Stale Hosts

Use the following to generate a list of hosts that have not been logged into for the past 30 days:

```text
Import-Module ActiveDirectory

function Get-StaleComputers() {
    $time = (Get-Date).Adddays(-30)
    Get-ADComputer -Filter { LastLogonTimeStamp -lt $time } -Properties LastLogonTimeStamp | Select-Object Name,@{Name="Stamp"; Expression={[DateTime]::FromFileTime($_.lastLogonTimestamp)}} # | Export-CSV C:\temp\unused_machines.csv -notypeinformation
    Write-Host done.
}

Get-StaleComputers
```

_\(Find my most recent copy on_ [_my GitHub_](https://github.com/burmat/burmatscripts/blob/master/powershell/Get-ADStaleHosts.ps1)_\)_

## DOMAIN MAINTENANCE

### Move Object to Retire OU

I like to use the scripts above \([Get Hosts Last Logon](https://burmat.gitbook.io/security/~/drafts/-LNR5JMBrAPfNXI0R06-/primary/sysadmin/active-directory-administration#get-hosts-last-logon) and [Get Users Last Logon](https://burmat.gitbook.io/security/~/edit/drafts/-LNR5JMBrAPfNXI0R06-/sysadmin/active-directory-administration#get-users-last-logon)\) to automatically move objects into the "Retire" OU using the following command\(s\):

```text
# to move a user:
 Get-ADUser $uname | Move-ADObject -TargetPath 'OU=Retire,DC=burmat,DC=co' 
 
# to move a computer:
 Get-ADComputer $hname | Move-ADObject -TargetPath 'OU=Retire,DC=burmat,DC=co' 
```

It's now trivial to [disable all objects in the given OU](https://burmat.gitbook.io/security/~/edit/drafts/-LNR5JMBrAPfNXI0R06-/sysadmin/active-directory-administration#disable-everything-in-ou).

### Disable Everything in OU

Every few weeks, I run the following \(as Domain Admin\) to ensure the OU I use for my "Recycle Bin" is filled with only disabled accounts:

`Get-ADUser -Filter * -SearchBase 'OU=Retire,DC=burmat,DC=co' | Disable-ADAccount` 

### Set Domain User Password

`$uname = 'burmat'; $pass = "Password123!'; $securepass = ConvertTo-SecureString $pass -AsPlainText -Force; Set-DomainUserPassword -Identity $uname -AccountPassword $securepass;`

## FILE SYSTEM ADMINISTRATION

### Getting Directory Sizes

I use the following command to generate a list of user profile's on a file server. It is useful to keep track of users that are exceeding our expectations when it comes to consuming space on a global server:

`Get-ChildItem | Where-Object { $.PSIsContainer } | ForEach-Object { $.Name + ": " + "{0:N2}" -f ((Get-ChildItem $_ -Recurse | Measure-Object Length -Sum -ErrorAction SilentlyContinue).Sum / 1MB) + " MB" }`

### Tail a File

Similar to `tail -f filename`, you can use `Get-Content` to watch a file for changes:

 `Get-Content -Path "\\server\logs\prod.server.log" -Wait` 

## MISC CLEANUP / MANAGEMENT

### Clear Cached \(mscachev2\) Credentials

A domain-joined endpoint that is taken from the domain might still have cached \(mscachev2\) domain logins residing on it. This is why I always wipe the system or use the following to remove any cached credentials:

Run `regedit` and give your current local account Write access to the "SECURITY" node. After restarting `regedit`, navigate to:  `HKEY_LOCAL_MACHINE\Security\Cache`

Cached credentials are stored in the binary values of `NL$1` through `NL$10`.  Zeroing out these values will clear the cached entries. Delete them if you want to remove them and disable this feature completely. 

