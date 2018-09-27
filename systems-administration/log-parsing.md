---
description: One of the greatest responsibilities of a sysadmin - Reading logs.
---

# Log Parsing

### APACHE HTTP LOGS

#### Get Visitor IP's by Count

Get IP address hits by count:

```text
cat access.log | cut -d " " -f 1 | sort | uniq -c | sort -urn
```

#### Get Page Requests by IP

Get the pages hit by a given IP address:

```text
cat access.log | grep '172.11.1.123' | cut -d "\"" -f 2 | uniq -c
```



