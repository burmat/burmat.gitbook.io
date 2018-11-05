---
description: One of the greatest responsibilities of a sysadmin - Reading logs.
---

# Log Parsing

## APACHE - HTTP Logs

### Get Visitor IP's by Count

Get IP address hits by count:

```text
cat access.log | cut -d " " -f 1 | sort | uniq -c | sort -urn
```

### Get Page Requests by IP

Get the pages hit by a given IP address:

```text
cat access.log | grep '172.11.1.123' | cut -d "\"" -f 2 | uniq -c
```

### IIS - HTTP Logs

Pretty easy with PowerShell - You can just specify the things you wish to search before the pipe and the noise you want to exclude after the pipe:

`Select-String -Pattern "pageName" *.log | Select-String -Pattern "10.1.1.12","10.1.1.123" -NotMatch`

## ZABBIX - Windows Event Logs

### Add an Item to a Host:

Get the Zabbix Agent up and running on an endpoint.  Once you have the agent installed on the host, add it into your Zabbix server. You can specify the type of log you want to have tracked, and can get granular with it. You want to add an item to your new host, similar to:

```text
eventlog[System,,"Warning|Error|Failure",,,,]
```

This is a good start to monitor errors, but I also like to parse events on domain controllers with the following item:

```text
eventlog[Security,,,,,,]
```

Keep in mind that the larger your domain is, the more events that will be hitting your DC constantly.  In this item, we have specifically said to read in all events.  Properly set the history storage for these items with this in mind. 

We can now use triggers to alarm for us based on specific event ID's. For example, the following expression for a trigger will tell use when a new user is created:

```text
{HOST:eventlog[Security,,,,,,].logeventid(4720)}>0
```

Here is one that will watch for account lockouts:

```text
{HOST:eventlog[Security,,,,,,].logeventid(4740)}>0
```

I also have one that will tell me when passwords are reset and other concerning domain activity.  You can get an idea of how powerful this can be, and using this setup has proven to be flexible and important in my daily routine.  

Now that the item and triggers are set up, all that is left is linking your triggers to actions and send alarms based on the triggers to get the jump on your users / adversaries.

## SELINUX - audit.log

### A More Readable  Form:

Prevent headaches - parse safely:

```text
cat /var/log/audit/audit.log | audit2why
```

### Denied:

Look for `denied` events:  `cat /var/log/audit/audit.log | grep denied`

And it makes it easy to generate allow policies: `cat /var/log/audit/audit.log | grep <service> | grep denied | audit2allow -M <service>`

{% hint style="info" %}
[Allowing Access: audit2allow](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security-enhanced_linux/sect-security-enhanced_linux-fixing_problems-allowing_access_audit2allow)
{% endhint %}



