# Helper Commands

## BATCH / CMD

### Uninstall Windows Updates:

Run the following \(update the `list` to contain valid KB numbers\) to uninstall updates on a local system:

```text
@ECHO OFF
SET list=3133977 3137061 3138612 3138901 3139923

for %%a in (%list%) do (
	%windir%\syswow64\wusa.exe /uninstall /kb:%%a /quiet /norestart /log​
)
```

### Get Remote Host Windows Updates

Run the following \(update the `hosts` list with valid hostnames\) to get .CSV output of updates on remote systems:

```text
@echo off
SET dFormat=_%date:~-4,4%%date:~-10,2%%date:~-7,2%
SET sCMDparams=qfe get Caption, Description, HotfixID, InstallDate, InstalledBy /format:csv
SET hosts=HOSTNAME1 HOSTNAME2

ECHO %dFormat%
ECHO .
FOR %%f in (%hosts%) DO (
      ECHO Processing %%f
      ECHO   Output file: %%f_%dFormat%.csv
      wmic /NODE:%%f /output:%%f_%dFormat%.csv %sCMDparams%
  )
ECHO .
ECHO Remember to delete the first blank line
ECHO .
ECHO .
ECHO Finished.
ECHO .
```

### Send a Message to Remote Computer

```text
msg susie /SERVER:WKST-ABC.burmat.co "Please call 123-456-7890 x369"
```

Or you can launch it to all users on a given computer if you don't know the username.  Enable verbose mode \(`/V`\) and wait for a callback \(`/W`\) to get a callback when the user closes it:

`msg * /V /W /SERVER:WKST-123 This is a test message`

### Restart Remote Machine \(With Warning\)

This script is better kept out in a public place and called with the a remote command utility to loop through a list of hostnames. You can just replace the `%1` to hardcode a host if you want though.

```text
@echo off
REM shutdown -m \\%1 /a
echo Start Checking %1

ping -n 1 -l 100 %1
IF %ERRORLEVEL%==1 GOTO BAD
shutdown -r -f -m \\%1 -t 300 -d up:125:1 -c "Please SAVE Your work. In 5 minutes, this machine will reboot for a policy updates (Msg time: %time%)"
GOTO GOOD

:BAD
ECHO %1,Failed,%date%,%time% >> BadPCs.txt
GOTO EXIT

:GOOD
ECHO %1,Good,%date%,%time% >> GoodPCs.txt
GOTO EXIT

:EXIT
echo Stop Checking %1
```

## POWERSHELL

### Remotely Log User Out

```text
PS C:\WINDOWS\system32> Invoke-Command -ComputerName 'SERVER01' -ScriptBlock { quser }
 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
 admin                 console             1  Active      none   12/24/2018 10:35 AM

PS C:\WINDOWS\system32> Invoke-Command -ComputerName 'SERVER01' -ScriptBlock { logoff 1 }
```



