---
description: >-
  This page is dedicated to any and all Exchange Administration related content
  I can share.
---

# Exchange Administration

## EXCHANGE ONLINE / MICROSOFT 365 ADMINISTRATION

### Connect to Exchange Online

Use the following to import an Exchange Online PSSession:

```text
PS C:\> $credential = Get-Credential # opens prompt to collect credentials
PS C:\> $session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://outlook.office365.com/powershell-liveid/ -Credential $credential -Authentication Basic -AllowRedirection
PS C:\> Import-PSSession $session
```

### Disable Unused Features

To bulk disable IMAP, POP3, and OWA, you can just select all mailboxes and disable the features \(you **cannot** bulk-disable "OWA on Devices" of "SMTP Client Authentication" mailbox features using the web portal\). 

Therefore, it's best to use PowerShell ensure you disable anything unused \(MAPI and ActiveSync are left enabled with this command\):

```text
Get-CASMailbox | Set-CASMailbox -PopEnabled:$False -ImapEnabled:$False -OWAEnabled:$False -SmtpClientAuthenticationDisabled:$True -OWAForDevicesEnabled:$False
```

Turning off SMTP Authentication globally \(instead of per-mailbox\) can be achieved by modifying the Transport configuration:

```text
Set-TransportConfig -SmtpClientAuthenticationDisabled $True
```

Turning off IMAP and POP3 by default for new mailboxes can be done with a Client Access Service Plan modification \(**ensure you run the command above to disable anything that has previously been enabled!**\):

```text
Get-CASMailboxPlan | Set-CASMailboxPlan -PopEnabled:$False -ImapEnabled:$False
```

### Find Mailboxes with Inbox Rules

```text
$users = Get-Mailbox; ForEach ($user in $users) { Get-InboxRule -Mailbox $user.name | Select MailboxOwnerId,Name,Description | fl }
```

### List Connection Filter Object Properties

Listing the connection filter IP blocklist:

```text
Get-HostedConnectionFilterPolicy -Identity Default | Select-Object -ExpandProperty IPBlockList
```

## MAILBOX ADMINISTRATION

### Message Queue Totals

Get a total count of messages in your queue\(s\):

`Get-ExchangeServer | Where {$.IsHubTransportServer -eq "true"} | Get-Queue | Where {$_.Deliverytype -eq "MapiDelivery"} | Select-Object Identity, NextHopDomain, Status, MessageCount`

###  Per-Mailbox Outbox Items

Checking the outbox of your users' mailbox is also a good indicator to see if messages are stuck going outbound:

`Get-Mailbox -resultsize unlimited | Get-MailboxFolderStatistics -folderscope outbox | Sort-object foldersize -Descending | Select-Object Identity,Name,Foldertype,itemsinfolder,@{Name="FolderSize MB";expression={$_.folderSize.toMB()}}`

### Check for Mailbox-Level Send/Receive Restrictions

`Get-Mailbox -ResultSize unlimited | Format-Table Name,MaxSendSize,MaxReceiveSize`

### Export Mailbox to .PST:

Use the following to export a mailbox to a .PST file:

`New-MailboxExportRequest -Mailbox user -FilePath "\\fileserver\mailboxgraveyard\user.pst"`

Track the progress of your export request with: `Get-MailboxExportRequest`

### List Disconnected Mailboxes:

Get a list of the currently connected databases \(`Get-MailboxDatabase`\) and use the following command to see if there are any disconnected mailboxes:

`Get-MailboxStatistics -Database "<DB NAME>" | Where-Object {$_.DisconnectDate -Notlike $NULL} | FL DisplayName, DisconnectDate, MailboxGuid`

Once you have a mailbox you wish to remove, use the following command to permantenly remove it:

`Remove-Mailbox -Database "<DB NAME>" -StoreMailboxIdentity <MAILBOXGUID>`

### List Mailboxes by Size

Here is a useful command to get user information for a given mailbox while sorting the output by the total size:

`Get-MailboxDatabase | Get-MailboxStatistics | Select DisplayName, LastLoggedOnUserAccount, ItemCount, TotalItemSize, LastLogonTime, LastLogoffTime | Sort-Object TotalItemSize -Descending | Export-CSV C:\temp\mailboxes.csv`

## MESSAGE TRACKING

### Message Tracking Export to .CSV

Use the following to get a CSV output of the Message Tracking logs based on a few basic parameters. This is a good way to track individual emails that fail to flow properly:

`Get-MessageTrackingLog -Recipients:user@domain.com -Start "3/1/2018 1:00:00 AM" -End "3/14/2018 11:00:00 PM" | Select {$_.Recipients}, {$_.RecipientStatus}, * | Export-Csv C:\temp\user-emails.csv`

## OWA / ACTIVESYNC ADMINISTRATION

### List ActiveSync Devices

List all ActiveSync devices and the corresponding mailbox/user:

`Get-ActiveSyncDevice | Select-Object DeviceModel,FriendlyName,DeviceOS,UserDisplayName,Identity | Sort-Object DeviceModel | Export-CSV -Path C:\temp\activesyncusers.csv`

You can get a list of devices by username with the following:

`Get-ActiveSyncDeviceStatistics -Mailbox:"username" | Select-Object Guid,Identity,DeviceModel,LastSuccessSync | Sort-Object LastSuccessSync`

If you want to delete one, you can simply: `Remove-ActiveSyncDevice <GUID>`

You can do this by running a command that will check for any device that have not sync'd in over 365 days. Generate this list with the following:

`$OldDevices = Get-ActiveSyncDevice -ResultSize unlimited | Get-ActiveSyncDeviceStatistics | where {$_.LastSyncAttemptTime -lt (Get-Date).AddDays(-365)}`

And to remove them completely, you can use this:

`$OldDevices | foreach-object { Remove-ActiveSyncDevice([string]$_.Guid) -confirm:$false }`

You can also take a look at [a script I wrote to automate all of this](https://github.com/burmat/burmatscripts/blob/master/powershell/ActiveSyncCleanup.ps1). It will remove any ActiveSync devices that have not attempted to sync in 1 year, and it will also remove orphaned mobile devices \(e.g. the mailbox doesn't exist or the user account is disabled\). I run this quarterly to catch anything that might fall through the cracks.

_\(Find my most recent copy of `ActiveSyncCleanup.ps1` on_ [_my GitHub_](https://github.com/burmat/burmatscripts/blob/master/powershell/ActiveSyncCleanup.ps1)_\)_

### List Enabled OWA Accounts

I recommend that every once in a while, you check to see if OWA is enabled for users it doesn't have to be enabled for:

`Get-CASMailbox | Where {$_.OWAEnabled} | ft DisplayName,OwaEnabled`

You can disable it on a per-user basis:

`Set-CASMailbox -Identity appledeveloper -OWAEnabled $false`

Or you can disable it against all mailboxes with:

`$mbxs = Get-Mailbox; foreach ($mbx in $mbxs) { Set-CASMailbox -Identity $($mbx.Alias) -OWAEnabled $false }`

