# Shodan

## 📖 Basic Usage Scenarios and Queries:

Shodan is a great source to begin looking for hosts that the parent organization owns. Shodan aggregates this information by banner grabbing and collecting information being leaked by insecure devices. The discovered information is then categorized and stored in a large database that users can search by type of device, brand name, operating system, location, and more. It is important to note that to use Shodan, you must have an account.

### ⛹️‍♂️ Basic Usage

For this example, we will leverage Shodan’s search engine. Our first search is a generic organization search to see what we can find on the organization using the following command:

* `org:"Organization_Name"`

This usually only returns a hand full of results, but it is still important to mention. To take it a step further, we will leverage Shodan’s SSL certificate searching function. To minimize the potential for stale records, we will have Shodan only return IP’s that respond with a 200 status code using the following command:

* `ssl:"Organization_Name" 200`

To show the amount of noise you get when searching without the 200 status code included, I queried Shodan against the organization “burmat.co”. Without specifying the required 200 status code, the query returned 152 results. With the required 200 status code, it returned only 7 results.

### 🔬 Shodan Queries

Fill in queries!

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

#### Exposed C2 hosts

```bash
metasploit C2: https://www.shodan.io/search?query=ssl%3A%22MetasploitSelfSignedCA%22
cobalt strike C2: https://www.shodan.io/search?query=product%3A%22Cobalt+Strike+Beacon%22
covenant: https://www.shodan.io/search?query=ssl%3A%E2%80%9DCovenant%E2%80%9D%20http.component%3A%E2%80%9DBlazor%E2%80%9D
mythic c2: https://www.shodan.io/search?query=ssl%3AMythic+port%3A7443
brute ratel C4: https://www.shodan.io/search?query=http.html_hash%3A-1957161625
```

***

## 🛠 Shodan Command Line Tools Installation

You can get your API key after signing into your account via browser and clicking on account in the top right corner.

* `pip3 install shodan`
* `shodan init <Your API Key>`

***

### 🍼 Basic Command Line Usage

[https://github.com/lothos612/shodan](https://github.com/lothos612/shodan)

* `shodan <ipaddr>`
* `shodan domain <domain.com>`
* `shodan search ssl:"<domain.com>"`
* `shodan search net:"63.64.65.66/24"`

***

### ➰ Looping Hosts with Shodan CLI

Good way to quickly pull host data, port data, and CVE data.

* `while read ip; do shodan host "$ip" >> shodan_results.txt; sleep 1; done < discovered_ips.txt`

***
