# SQL Server Administration

## TRANSFER SQL SERVER USER LOGINS

If you restore a database from one server on to a foreign server, you might need a way to bring over the SQL user accounts too. This occurs frequently when you are restoring an application to a test/backup server that does not have the same user accounts configured.

You can easily script out the current users by running the following script \(originally from Microsoft\):  [https://github.com/burmat/burmatscripts/blob/master/mssql/export\_logins.sql](https://github.com/burmat/burmatscripts/blob/master/mssql/export_logins.sql)

And executing the resulting stored procedure:

```text
EXEC sp_help_revlogin
```

Paste the output of this stored procedure into the destination server to bring the user account in with their original SID's and current passwords. You can run the following query on both servers to compare the SID's if you wish:

```text
select loginname, sid from master.sys.syslogins
```

{% hint style="info" %}
Resource: [https://support.microsoft.com/en-us/help/918992/how-to-transfer-logins-and-passwords-between-instances-of-sql-server](https://support.microsoft.com/en-us/help/918992/how-to-transfer-logins-and-passwords-between-instances-of-sql-server)
{% endhint %}

