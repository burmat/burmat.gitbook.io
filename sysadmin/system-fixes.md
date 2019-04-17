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

## 





