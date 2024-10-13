# Helper Commands

## Windows

### Uninstall Windows Updates:

Run the following (update the `list` to contain valid KB numbers) to uninstall updates on a local system:

{% code overflow="wrap" %}
```powershell
@ECHO OFF
SET list=3133977 3137061 3138612 3138901 3139923

for %%a in (%list%) do (
	%windir%\syswow64\wusa.exe /uninstall /kb:%%a /quiet /norestart /log​
)
```
{% endcode %}

### Run-As Session

{% code overflow="wrap" %}
```powershell
runas /user:burmat.local\nathan /netonly powershell
```
{% endcode %}

### Get Remote Host Windows Updates

Run the following (update the `hosts` list with valid hostnames) to get .CSV output of updates on remote systems:

{% code overflow="wrap" %}
```powershell
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
{% endcode %}

### Send a Message to Remote Computer

{% code overflow="wrap" %}
```powershell
msg susie /SERVER:WKST-ABC.burmat.co "Please call 123-456-7890 x369"
```
{% endcode %}

Or you can launch it to all users on a given computer if you don't know the username. Enable verbose mode (`/V`) and wait for a callback (`/W`) to get a callback when the user closes it: `msg * /V /W /SERVER:WKST-123 This is a test message`

### Restart Remote Machine (With Warning)

This script is better kept out in a public place and called with the a remote command utility to loop through a list of hostnames. You can just replace the `%1` to hardcode a host if you want though.

{% code overflow="wrap" %}
```powershell
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
{% endcode %}

### Windows Firewall

#### Enable Network Discovery

{% code overflow="wrap" %}
```powershell
netsh advfirewall firewall set rule "group=\"Network Discovery\"" new enable=Yes
```
{% endcode %}

#### Enable File and Printer Settings

{% code overflow="wrap" %}
```powershell
netsh advfirewall firewall set rule "group=\"File and Printer Sharing\"" new enable=Yes
```
{% endcode %}

#### Allow All Inbound

{% code overflow="wrap" %}
```powershell
netsh advfirewall firewall add rule name="Open the Gates" dir=in action=allow protocol=TCP localport=1-65535
```
{% endcode %}

#### Allow All Output

{% code overflow="wrap" %}
```powershell
netsh advfirewall firewall add rule name="Open the Gates" dir=out action=allow protocol=TCP localport=1-65535
```
{% endcode %}

## PowerShell

### Remotely Log User Out

{% code overflow="wrap" %}
```powershell
Invoke-Command -ComputerName 'SERVER01' -ScriptBlock { quser }
Invoke-Command -ComputerName 'SERVER01' -ScriptBlock { logoff 1 }
```
{% endcode %}

### Creating an SMB Share

{% code overflow="wrap" %}
```powershell
New-SmbShare -Name "burmat" -Path "C:\tools" | Grant-SmbShareAccess -Account Everyone -AccessRight Full -Force
```
{% endcode %}

### Sending an Email

Leveraging an M365 mail relay:

{% code overflow="wrap" %}
```powershell
Send-MailMessage -SmtpServer 'burmat-co.mail.protection.outlook.com' -To 'victim@company.com' -From 'informationsecurity@company.com' -Subject 'Test' -Body 'Mail relay test' -BodyAsHTML
```
{% endcode %}

### Retrieve Exchange Version

{% code overflow="wrap" %}
```powershell
((([ADSI]"LDAP://cn=Microsoft Exchange,cn=Services,cn=Configuration,DC=vcfcorp,DC=com")).Children).msExchProductID
```
{% endcode %}

### Users that have Reversible Encryption Enabled

{% code overflow="wrap" %}
```powershell
Get-ADUser -Filter * -Properties * | Select-Object name,AllowReversiblePasswordEncryption
```
{% endcode %}

### Base64 Encoding PowerShell

{% code overflow="wrap" %}
```powershell
$string = 'Add-LocalGroupMember -Group "Administrators" -Member "burmat.co\jsmith"';
$b64 = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($string));
Write-Output $string ' -- Converted Output:';
Write-Output $b64;
Write-Output $b64 | clip;
Write-Output "Copied to clipboard";
```
{% endcode %}

### Renaming Files to MD5 Value

{% code overflow="wrap" %}
```powershell
# recursive *.txt lookup to rename to md5 value, removing duplicates
Get-ChildItem "C:\temp" -Filter "*.txt" | ForEach-Object {
	$filepath = $_.FullName
	$hashed = Get-FileHash $filepath -Algorithm MD5
	$newpath = $(Split-Path $filepath) + "\" + $hashed.Hashed + ".txt"
	Copy-Item -Path $filepath -Destination $newpath -Force
	Remove-Item -Path $filepath -Force
}
```
{% endcode %}

### Getting PowerShell Versioning

{% code overflow="wrap" %}
```powershell
$PSVersionTable.PSVersion
[Environment]::Is64BitProcess # x86 or x64
```
{% endcode %}

### Listing Recycle Bin Contents

{% code overflow="wrap" %}
```powershell
(New-Object -ComObject Shell.Application).NameSpace(0x0a).Items() | Select-Object Name,Size,Path
```
{% endcode %}

### Creating a Bunch of Domain Users

{% code overflow="wrap" %}
```powershell
1..10000 | ForEach-Object { New-ADUser -Name employee$_ -AccountPassword (ConvertTo-SecureString S3cur3PASS123 -AsPlainText -Force) -UserPrincipalName employee$_@$env:userdnsdomain -ChangePasswordAtLogon $False -Enabled $True}
```
{% endcode %}

### Keep a Computer Awake

{% code overflow="wrap" %}
```powershell
while($true)
  $Pos = [System.Windows.Forms.Cursor]::Position
  [System.Windows.Forms.Cursor]::Position = New-Object System.Drawing.Point((($Pos.X) + 1) , $Pos.Y)
  Start-Sleep -Seconds 60
}
```
{% endcode %}

### Enumerate Child Processes

The following will show you the children executed under the parent process ID provided

{% code overflow="wrap" %}
```powershell
Get-WmiObject win32_process | where { $_.ParentProcessId -eq 2448 }
```
{% endcode %}

### DNS Reverse Lookup for Hostname List

{% code overflow="wrap" %}
```powershell
Get-Content .\hostnames.txt | ForEach-Object { nslookup $_ } | Select-String "Address"
```
{% endcode %}

### AES Encrypt & Decrypt Text

Encrypting

{% code overflow="wrap" %}
```powershell
using namespace System.Security.Cryptography
$t = 'text-that-should-be-encrypted';
$k = "burmat1234!";
$sha = New-Object SHA256Managed;
$aes = New-Object AesManaged;
$aes.Mode = [CipherMode]::CBC;
$aes.Padding = [PaddingMode]::Zeros;
$aes.BlockSize = 128;
$aes.KeySize = 256;
$aes.Key = $sha.ComputeHash([System.Text.Encoding]::UTF8.GetBytes($k));
$bytes = [System.Text.Encoding]::UTF8.GetBytes($t);

$crypt = $aes.CreateEncryptor();
$encbytes = $crypt.TransformFinalBlock($bytes,0,$bytes.Length);
$encbytes = $aes.IV + $encbytes;
$encrypted = [System.Convert]::ToBase64String($encbytes)
$aes.Dispose();
$sha.Dispose();
$encrypted
```
{% endcode %}

Decrypting

{% code overflow="wrap" %}
```powershell
using namespace System.Security.Cryptography
$encrypted = "75z5B42nn1Ig6JDw"

$k = "burmat1234!";
$sha = New-Object SHA256Managed

$aes = New-Object AesManaged
$aes.Mode = [CipherMode]::CBC
$aes.Padding = [PaddingMode]::Zeros
$aes.BlockSize = 128
$aes.KeySize = 256
$aes.Key = $sha.ComputeHash([System.Text.Encoding]::UTF8.GetBytes($k))
$encbytes = [System.Convert]::FromBase64String($encrypted)
$aes.IV = $encbytes[0..15]
$decryp = $aes.CreateDecryptor()
$bytes = $decryp.TransformFinalBlock($encbytes, 16, $encbytes.Length - 16)
$decypted = [System.Text.Encoding]::UTF8.GetString($bytes).Trim([char]0)
$aes.Dispose()
$decypted
```
{% endcode %}

### Loading PowerShell ActiveDirectory Module Manually

No administrative access is required.

{% file src="../.gitbook/assets/Import-ActiveDirectory.ps1" %}

### PowerShell Terminal Profile

In the works, goes in your `~/WindowsPowerShell` directory.

{% code overflow="wrap" %}
```powershell
Start-Transcript -OutputDirectory C:\WindowsPowerShell\TranscriptionLogs
#Set-ExecutionPolicy Bypass

Import-Module PSReadLine
Import-Module Get-ChildItemColor

Set-Alias -Name "python3" -Value "python"
Set-PSReadLineOption -HistoryNoDuplicates
Set-PSReadLineOption -HistorySearchCursorMovesToEnd
Set-PSReadLineOption -HistorySaveStyle SaveIncrementally
Set-PSReadLineOption -MaximumHistoryCount 4000

# History substring search
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward

# Tab completion
Set-PSReadLineKeyHandler -Chord 'Shift+Tab' -Function Complete
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete

function l {
    Get-ChildItem -Force
}

function tools {
    set-location "C:\Tools"
}

function projects {
    Set-Location "C:\Projects"
}
```
{% endcode %}

## Bash

### SMTP

{% code overflow="wrap" %}
```shell
## base64 the username and password and pass the values through telnet

# echo -n "username" | openssl enc -base64
dXNlcm5hbWU=

# echo -n "Welcome1" | openssl enc -base64
V2VsY29tZTE=

# openssl s_client -connect 169.133.88.123:25 -starttls smtp -quiet -crlf
<-- SNIP --> 
250 XRDST
# AUTH LOGIN
334 VXNlcm5hbWU6
# Z2R1YXJ0ZUBoZXJzaGV5cy5jb20=
334 UGFzc3dvcmQ6
# V2VsY29tZTE=
535 5.7.3 Authentication unsuccessful << will be successful, if you are able to

# EHLO target.com
# MAIL FROM: victim@target.com
# RCPT TO: <outside@user.com>  NOTIFY=success,failure
# DATA
# Subject: Test Email

body of the message
.

EXIT
```
{% endcode %}

### Comment / Uncomment Lines

When you need to comment out or uncomment out a line, sed makes it pretty easy. Just change the word pattern to something identifiable:

{% code overflow="wrap" %}
```bash
sed -i '/pattern/s/^/#/g' apache2.conf # comment
sed -i '/pattern/s/^#//g' apache2.conf # uncomment
```
{% endcode %}

### Loops

#### Reading a File

{% code overflow="wrap" %}
```sh
while read ip; do echo "$ip" done < affected_hosts.txt
```
{% endcode %}

#### Loop an Array

{% code overflow="wrap" %}
```sh
l=( "one" "two" "three" ); for i in "${l[@]}"; do echo $i; done;
```
{% endcode %}

### Resolving Hostnames

{% code overflow="wrap" %}
```sh
cat hosts.txt | xargs -n1 -P100 dig +short +retry=3 | grep -E '^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' > ip_list.txt
```
{% endcode %}

### Rudementary Port Scanner

{% code overflow="wrap" %}
```sh
for ip in {1..254}; do
	for port in {22,80,443,3306,3389}; do
		(echo >/dev/tcp/10.1.1.$ip/$port) &> /dev/null && echo "10.1.1.$ip:$port is open";
	done;
done;
```
{% endcode %}

### Network Time Protocol (NTP)

Because setting the clock is fucking painful:

{% code overflow="wrap" %}
```bash
## set our time to match the system time of the DC:
curl -s --stderr - -v <http://sauna.htb> | grep "Date"
timedatectl set-timezone GMT
timedatectl set-time $(curl -s --stderr - -v <http://sauna.htb> | grep "Date" | cut -d ' ' -f 7)

timedatectl && echo "server time:\\n" && curl -s --stderr - -v <http://sauna.htb> | grep "Date"
               Local time: Fri 2020-02-21 10:30:39 GMT
           Universal time: Fri 2020-02-21 10:30:39 UTC
                 RTC time: Fri 2020-02-21 10:30:46
                Time zone: GMT (GMT, +0000)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
server time:

< Date: Fri, 21 Feb 2020 10:30:47 GMT
```
{% endcode %}

### NTP Mode 6 Check:

* `ntpq -c rv <IP>`

If there is output, it is susceptible to this: [https://scan.shadowserver.org/ntpversion/](https://scan.shadowserver.org/ntpversion/)

### DNS Host Discovery

{% code overflow="wrap" %}
```sh
for i in $(cat hosts.txt); do host $i; done
```
{% endcode %}

### Installing Ruby

{% code overflow="wrap" %}
```sh
apt-get install ruby-full
apt-get install git curl autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm6 libgdbm-dev libdb-dev
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash

echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(rbenv init -)"' >> ~/.zshrc
source ~/.zshrc
rbenv install -l
rbenv install 2.5.3
```
{% endcode %}

### Certificate Fingerprint

{% code overflow="wrap" %}
```sh
echo -n | openssl s_client -connect 107.154.75.199:443 2>/dev/null | openssl x509 -noout -fingerprint -sha1
```
{% endcode %}
