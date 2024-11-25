# amass

## Basic Usage

A common sub-domain enumeration command:

`amass enum -v -src -ip -brute -min-for-recursive 2 -oA amass_results -d domain1.com,domain2.com`

* `-src` will print data sources for discovered names
* `-ip` will print the corresponding IP address
* `-brute` will perform a brute-force subdomain check
* `-min-for-recursive` is the number of subdomain levels seen before recursive brute-forcing (e.g. idk what this means, but the default is `1`)

the "database" file is located here, in case you want to clear it out:

* `~/.config/amass/amass.json`

User Guide:

[OWASP/Amass](https://github.com/OWASP/Amass/blob/master/doc/user\_guide.md)

## Comparing Discovered Subdomains to Existing IP Scope

The client provided domains that they would like their subdomains discovered for, as well as the \~9K IPs they provided.

* `amass enum -v -src -ip -brute -min-for-recursive 2 -oA amass_results -d domain1.com,domain2.com,domain3.net,domain4.co,domain5.ne`

You can get a rough idea of the hosts with:

* `cat amass_results.json | jq '.name, .addresses[] .ip'`

But because a lot of these were already in scope for the assessment (defined in the IP scope of 9K hosts), I used the following Python code to parse out the IP addresses that were NOT located in the defined scope and wrote the subdomains to a new file:

{% code overflow="wrap" %}
```python
# - data.json is the output from amass (you have to comma seperate the lines and surrond with [])
# - for each entry, grab the associated IP addresses
# - check to see if the IP address is already listed in our IP scope
# - if it is NOT, echo out the hostname so we scan it
import json
f = open('data.json')
data = json.load(f)
for d in data:
	subdomain = d['name']
	ips = d['addresses']
	for i in ips:
		r = open('ip_scope.txt', 'r').read().find(i['ip'])
		if r == -1:
			print(subdomain)

f.close()
```
{% endcode %}

(make sure you de-dupe the resulting subdomain list)

## **Chaining Tools via File Output**

We can output the results to a text file with the `-o` flag. This is useful because we can save this information for later and feed this output into other tools/scripts.

We are going to use docker's `-v` switch to share the "/shared" directory between the host environment and the container's environment, hence anything we store in the "/shared" directory will still be accessible even after the docker container has finished running:

```sh
amass -o /shared/results_subdomains_amass.txt -d lizardblue.com
```

We can now view the content of the text file with the cat command, even though the docker container using amass is no longer running.

```sh
cat /shared/results_subdomains_amass.txt
```
