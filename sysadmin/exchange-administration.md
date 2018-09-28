---
description: >-
  This page is dedicated to any and all Exchange Administration related content
  I can share.
---

# Exchange Administration

## MAILBOX ADMINISTRATION

### Message Queue Totals

Get a total count of messages in your queue\(s\):

`Get-ExchangeServer | Where {$.IsHubTransportServer -eq "true"} | Get-Queue | Where {$.Deliverytype -eq "MapiDelivery"} | Select-Object Identity, NextHopDomain, Status, MessageCount`

###  Per-Mailbox Outbox Items

Checking the outbox of your users' mailbox is also a good indicator to see if messages are stuck going outbound:

`Get-Mailbox -resultsize unlimited | Get-MailboxFolderStatistics -folderscope outbox | Sort-object foldersize -Descending | Select-Object Identity,Name,Foldertype,itemsinfolder,@{Name="FolderSize MB";expression={$_.folderSize.toMB()}}`

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

`Get-MessageTrackingLog -Recipients:user@domain.com -Start "3/1/2018 1:00:00 AM" -End "3/14/2018 11:00:00 PM" | Select {$.Recipients}, {$.RecipientStatus}, * | Export-Csv C:\temp\user-emails.csv`

## OWA / ACTIVESYNC ADMINISTRATION

### List ActiveSync Devices

List all ActiveSync devices and the corresponding mailbox/user:

`Get-ActiveSyncDevice | Select-Object DeviceModel,FriendlyName,DeviceOS,UserDisplayName,Identity | Sort-Object DeviceModel | Export-CSV -Path C:\temp\activesyncusers.csv`

You can get a list of devices by username with the following:

`Get-ActiveSyncDeviceStatistics -Mailbox:"username" | Select-Object Identity,LastSuccessSync`

I like to check in every few weeks and make sure there are not old devices laying around in the system. You can do this by running a command that will check for any device that have not sync'd in over 365 days. Generate this list with the following:

`$OldDevices = Get-ActiveSyncDevice -ResultSize unlimited | Get-ActiveSyncDeviceStatistics | where {$_.LastSyncAttemptTime -lt (get-date).adddays(-365)}`

And to remove them completely, you can use this:

`$OldDevices | foreach-object { Remove-ActiveSyncDevice([string]$_.Guid) -confirm:$false }`

### List Enabled OWA Accounts

I recommend that every once in a while, you check to see if OWA is enabled for users it doesn't have to be enabled for:

`Get-CASMailbox | Where {$_.OWAEnabled} | ft DisplayName,OwaEnabled`

You can disable it on a per-user basis:

`Set-CASMailbox -Identity appledeveloper -OWAEnabled $false`

Or you can disable it against all mailboxes with:

`$mbxs = Get-Mailbox; foreach ($mbx in $mbxs) { Set-CASMailbox -Identity $($mbx.Alias) -OWAEnabled $false }`
