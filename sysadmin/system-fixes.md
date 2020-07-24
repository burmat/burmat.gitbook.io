---
description: Any and all fixes that might help another person.
---

# System Fixes

## DCOM ERROR EVENT 10016

#### Have you found the following DistributedCOM \(DCOM\) event log on your system? 

_The application-specific permission settings do not grant Local Activation permission for the COM Server application with CLSID {D63B10C5-BB46-4990-A94F-E40B9D520160} and APPID {9CA88EE3-ACB7-47C8-AFC4-AB702511C276} to the user NT AUTHORITY\SYSTEM SID \(S-1-5-18\) from address LocalHost \(Using LRPC\) running in the application container Unavailable SID \(Unavailable\). This security permission can be modified using the Component Services administrative tool._

{% hint style="info" %}
Keep in mind that the user account might be different. For example, if you are using a specially created service account, `COMPUTER\mssqlsvc` for example.
{% endhint %}

This problem has happened a few times to me in the past, and loading `DCOMCNFG` to alter the offending APPID is easy enough. If you have this particular offending APPID for **RunTimeBroker \(**_**{9CA88EE3-ACB7-47C8-AFC4-AB702511C276}**_**\)**, it might be a little harder. If you get to the permissions and they are all disabled, it is because the incorrect owner has been established to this service. To fix this problem, follow these three steps:

#### Registry Permissions:

For the following two registry keys:

* `HKEY_CLASSES_ROOT\AppID{9CA88EE3-ACB7-47c8-AFC4-AB702511C276}`
* `HKEY_CLASSES_ROOT\CLSID{D63B10C5-BB46-4990-A94F-E40B9D520160}`

1. _**Right-Click**_ the key &gt; _**Permissions**_ &gt; _**Advanced**_
2. Change the owner to `<COMPUTERNAME>\Administrators`
3. Under _**Permission Entries**_, add `<COMPUTER>\Administrators` and give _**Full Control**_

#### DCOM Permissions

1. Run _**`DCOMCNFG`**_ &gt; _**Component Services**_ &gt; _**Computers**_ &gt; _**My Computer**_ &gt; _**DCOM Config**_
2. Find the _**RunTimeBroker** service_ &gt; _**Right-Click**_ &gt; _**Properties**_ &gt; _**Security**_

_**Launch and Activation Permissions**_ might prompt you to clear up any problems, but afterwards, you should be able to add the offending user account properly.

## WSUS "Synchronizations" Crashing with "Reset Node"

The following can sometimes be seen when you observe the "Synchronizations" screen within WSUS and it crashes with the "Reset Node" option:

```text
System.Net.WebException -- The operation has timed out

Source
System.Web.Services

Stack Trace:
   at System.Web.Services.Protocols.WebClientProtocol.GetWebResponse(WebRequest request)
   at Microsoft.UpdateServices.Internal.DatabaseAccess.ApiRemotingCompressionProxy.GetWebResponse(WebRequest webRequest)
   at System.Web.Services.Protocols.SoapHttpClientProtocol.Invoke(String methodName, Object[] parameters)
   at Microsoft.UpdateServices.Internal.ApiRemoting.ExecuteSPSearchUpdates(String updateScopeXml, String preferredCulture, Int32 publicationState)
   at Microsoft.UpdateServices.Internal.DatabaseAccess.AdminDataAccessProxy.ExecuteSPSearchUpdates(String updateScopeXml, String preferredCulture, ExtendedPublicationState publicationState)
   at Microsoft.UpdateServices.Internal.BaseApi.Update.SearchUpdates(UpdateScope searchScope, ExtendedPublicationState publicationState, UpdateServer updateServer)
   at Microsoft.UpdateServices.UI.AdminApiAccess.UpdateManager.GetUpdates(ExtendedUpdateScope filter)
   at Microsoft.UpdateServices.UI.AdminApiAccess.WsusSynchronizationInfo.InitializeDerivedProperties()
   at Microsoft.UpdateServices.UI.SnapIn.Pages.SyncResultsListPage.GetSyncInfoRow(WsusSynchronizationInfo syncInfo)
   at Microsoft.UpdateServices.UI.SnapIn.Pages.SyncResultsListPage.GetListRows()
```

This also has the side effect of crashing the IIS AppPool for WSUS \(WsusPool\) as well.  Initial searches indicated that increasing the "Queue Length" property to 25k and the "Private Memory Limit" to 0 \(unlimited\) would help fix the problem, but I eventually found the correct fix from these posts:

* [https://social.technet.microsoft.com/Forums/en-US/b1515b21-efcc-4b6f-9171-754c63d02716/when-i-click-synchronous-the-wsus-will-stopped?forum=winserverwsus](https://social.technet.microsoft.com/Forums/en-US/b1515b21-efcc-4b6f-9171-754c63d02716/when-i-click-synchronous-the-wsus-will-stopped?forum=winserverwsus)
* [https://docs.microsoft.com/en-us/archive/blogs/sus/clearing-the-synchronization-history-in-the-wsus-console](https://docs.microsoft.com/en-us/archive/blogs/sus/clearing-the-synchronization-history-in-the-wsus-console)

In short, there is a record in the database for a synchronization event that is causing the WSUS screen to crash, and deleting the historical records is a good way to flush out the problem.

{% hint style="info" %}
If you are not using MSSQL Express for WSUS updates, you have to connect over a named pipe to access the database. This means you'll need SQL Management Studio on the local system because it won't be accessible remotely.  If you are using MSSQL Express as the database solution, you will have to use a different connection string and it may be accessible remotely if you have configured it as such. 
{% endhint %}

For me, I had to connect over the named piped instance using the connection string: `\\.pipe\MICROSOFT##WID\tsql\query`

I ran the following query to look at the current records I was about to delete:

```text
SELECT * 
FROM tbEventInstance 
WHERE 
    EventNamespaceID = '2' AND 
    EVENTID IN('381', '382', '384', '386', '387', '389');
```

I did not notice anything obvious that could be causing the crash, so I took a backup of the server and ran the following to delete all records:

```text
DELETE FROM tbEventInstance 
WHERE 
    EventNamespaceID = '2' AND 
    EVENTID IN('381', '382', '384', '386', '387', '389');
```

I restarted the IIS AppPool for WSUS to be sure I flushed any problems in memory and "Reset Node" in WSUS. That was enough to keep the application from crashing, but firing off a manual sync and ensuring that it still does not crash seems to indicate the problem is gone.

