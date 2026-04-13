# Adobe Experience Manager (AEM)

Adobe Experience Manager is a comprehensive content management solution commonly used by enterprises. AEM instances are frequently exposed to the internet and can contain various security misconfigurations that lead to information disclosure, remote code execution, or full system compromise.

Source: [https://twitter.com/Alra3ees/status/1410062155248979968?s=20\&t=IJ1SKGHYSxj6h1e94oFkdQ](https://twitter.com/Alra3ees/status/1410062155248979968?s=20\&t=IJ1SKGHYSxj6h1e94oFkdQ)

## Find AEM Instances

Discover AEM instances from a list of URLs:

[https://github.com/0ang3el/aem-hacker](https://github.com/0ang3el/aem-hacker)

`python3 aem_discoverer.py --file urls.txt`

## Scan for Vulnerabilities

Automated vulnerability scanner for AEM:

[https://github.com/Raz0r/aemscan](https://github.com/Raz0r/aemscan)

`aemscan https://target.burmat.co`

## Wordlists

AEM-specific wordlists for content discovery:

[https://github.com/emadshanab/Adobe-Experience-Manager](https://github.com/emadshanab/Adobe-Experience-Manager)

## Nuclei Templates

Scan with Nuclei for known AEM vulnerabilities:

`nuclei -l hosts -tags AEM -t /root/nuclei-templates`

## Common Attack Vectors

### Default Credentials

Many AEM instances use default credentials:

* `admin:admin`
* `author:author`
* `replication-receiver:replication-receiver`

### Exposed Servlets

Check for exposed servlets that may leak information or allow code execution:

{% code overflow="wrap" %}
```bash
# Query builder servlet (information disclosure)
curl https://target.burmat.co/bin/querybuilder.json?path=/content

# User enumeration
curl https://target.burmat.co/libs/granite/security/currentuser.json

# Default GET servlet
curl https://target.burmat.co/content.infinity.json

# POST servlet (potential RCE)
curl -X POST https://target.burmat.co/bin/receive
```
{% endcode %}

### SSRF via AEM Externalizer

AEM's externalizer can be abused for SSRF attacks:

`curl -X POST https://target.burmat.co/system/console/jmx/com.adobe.granite.maintenance.impl%3Atype%3DPurgeTask`
