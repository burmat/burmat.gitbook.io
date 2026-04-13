# Shodan

## Basic Usage Scenarios and Queries

Shodan is a search engine that aggregates information by banner grabbing and collecting data leaked by internet-connected devices. The discovered information is categorized and stored in a searchable database by device type, brand name, operating system, location, and more. An account is required to use Shodan effectively.

### Basic Usage

For this example, we will leverage Shodan's search engine. Start with a generic organization search to see what can be found:

* `org:"Organization_Name"`

This usually only returns a handful of results, but it is still important. To take it further, leverage Shodan's SSL certificate searching function. To minimize the potential for stale records, have Shodan only return IPs that respond with a 200 status code:

* `ssl:"Organization_Name" 200`

To show the amount of noise you get when searching without the 200 status code included, I queried Shodan against the organization "burmat.co". Without specifying the required 200 status code, the query returned 152 results. With the required 200 status code, it returned only 7 results.

### Shodan Queries

#### Search Query Indexes

Lists that probably have more queries than this page or your notes. Take a look through if you want some ideas. That’s primarily what this page is for! Use these arguments to build your own as-needed.

* [https://github.com/jakejarvis/awesome-shodan-queries](https://github.com/jakejarvis/awesome-shodan-queries)
* [https://www.osintme.com/index.php/2021/01/16/ultimate-osint-with-shodan-100-great-shodan-queries/](https://www.osintme.com/index.php/2021/01/16/ultimate-osint-with-shodan-100-great-shodan-queries/)

#### Favicon Hashes

Below is a link toe a list of favicon hashes. You can use these in a query like this: http.favicon.hash:81586312 to find specific hosts (this example is Jenkins).

* [https://github.com/sansatart/scrapts/blob/master/shodan-favicon-hashes.csv](https://github.com/sansatart/scrapts/blob/master/shodan-favicon-hashes.csv)

#### Queries:

* `http.html:"xoxb-"` - Search for Slack tokens
* `http.favicon.hash:81586312` - Search for Jenkins by favicon hash
* `http.title:"Grafana"` - Publicly exposed Grafana
* `vuln:ms17-010` - Eternal Blue hosts
* `vuln:CVE-2014-0160` - Search for CVE
* `ssh port:22,3333` - Search for SSH on specific ports only
* `proftpd port:21` - Search for proftpd on port 21 only
* `ssh -port:22` - Search for SSH on any port BUT 22 (non-standard ports)
* `os:"Windows 10 Home 19041"` - Checking for vulnerable operating system
* `country:"IN" os:"windows 7"` - Search by OS and country

#### Exposed C2 Hosts

```bash
# Metasploit C2
https://www.shodan.io/search?query=ssl%3A%22MetasploitSelfSignedCA%22

# Cobalt Strike C2
https://www.shodan.io/search?query=product%3A%22Cobalt+Strike+Beacon%22

# Covenant C2
https://www.shodan.io/search?query=ssl%3A%E2%80%9DCovenant%E2%80%9D%20http.component%3A%E2%80%9DBlazor%E2%80%9D

# Mythic C2
https://www.shodan.io/search?query=ssl%3AMythic+port%3A7443

# Brute Ratel C4
https://www.shodan.io/search?query=http.html_hash%3A-1957161625
```

#### Exposed Infrastructure

```bash
# Docker registries
product:"Docker Registry"

# Kubernetes dashboards
http.title:"Kubernetes Dashboard"

# Redis (no auth)
product:"Redis" -authentication

# MongoDB (no auth)
product:"MongoDB" -authentication "port:27017"

# Elasticsearch (open)
product:"Elasticsearch" http.status:200

# Exposed etcd
product:"etcd"

# Grafana with default creds
http.title:"Grafana" http.favicon.hash:1485257654
```

***

## Shodan Command Line Tools Installation

You can get your API key after signing into your account via browser and clicking on account in the top right corner.

* `pip3 install shodan`
* `shodan init <Your API Key>`

***

### Basic Command Line Usage

[https://github.com/lothos612/shodan](https://github.com/lothos612/shodan)

* `shodan <ipaddr>`
* `shodan domain <domain.com>`
* `shodan search ssl:"<domain.com>"`
* `shodan search net:"63.64.65.66/24"`

***

### Looping Hosts with Shodan CLI

Good way to quickly pull host data, port data, and CVE data.

* `while read ip; do shodan host "$ip" >> shodan_results.txt; sleep 1; done < discovered_ips.txt`

***
