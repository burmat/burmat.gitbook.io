# jq

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## Parsing BloodHound Data

***

`@ippsec` [put out a pretty cool video](https://youtu.be/o3W4H0UfDmQ) on parsing large amounts of data by downloading the BH graph as a Json file to parse with `jq`. Below are the queries he demonstrates to get useful data from large query results, which should give you a framework and methodology to experiment:

### **User Object Queries with `jq`**

#### **To list out the keys inside of a JSON object:**

* `cat users.json | jq '. | keys'`

#### **Go into the list based on the key (we only care about data here):**

* `cat users.json | jq '.data[]'`

#### **Dump all of the usernames:**

* `cat users.json | jq '.data[].Properties | .name'`

#### **Dump all users that are currently enabled:**

* `cat users.json | jq '.data[].Properties | select( .enabled == true ) | .name'`

#### **Show all usernames and descriptions for disabled/enabled users&#x20;**_**(check for passwords in the description field!)**_**:**

* `cat users.json | jq '.data[].Properties | select( .enabled == false ) | .name + " " + .description'`

#### **Enabled users, where the description is populated with something:**

* `cat users.json | jq '.data[].Properties | select( .enabled == true) | select( .description != null ) | .name + " " + .description'`

#### **If you need to convert something from a number to a string, use** `(.value|tostring)`**.**

* `cat users.json | jq '.data[].Properties | select( .enabled == true) | select( .description != null ) | .name + " " + (.lastlogontimestamp|tostring)'`

#### **Show all enabled users and their** `lastlogin` **value&#x20;**_**(if it’s**_ `1`_**, the account hasn’t been logged into and you should target it for password spraying!)**_**:**

* `cat users.json | jq '.data[].Properties | select( .enabled == true) | select( .description != null ) | .name + " " + (.lastlogin|tostring)'`

**Show enabled users that have a passwords last set that is greater than the last logon of the computer&#x20;**_**(password reset but never logged in? password spray it!!!)**_**:**

* `cat users.json | jq '.data[].Properties | select( .enabled == true) | select( .pwdlastset > .lastlogontimestamp ) | .name + " " + (.lastlogin|tostring)'`

**Kerberoastable Users:**

* `cat users.json | jq '.data[].Properties | select( .serviceprincipalnames != [] ) | .name'`

### **Computer Object Queries with `jq`**

#### **Gather hostnames and their operating system:**

* `cat computers.json | jq '.data[].Properties | select(.operatingsystem != null) | .name + ":" + .operatingsystem'`

#### **Show all hosts that are not running Windows 10 Pro:**

* `cat computers.json | jq '.data[].Properties | select(.operatingsystem != null) | select(.operatingsystem != "Windows 10 Pro") | .name + ":" + .operatingsystem'`

#### Unsupported Operating Systems

* `cat computers.json | jq '.data[].Properties | select(.operatingsystem != null) | select(.operatingsystem | contains("2008","2003","XP","NT","2012")) | .name + ":" + .operatingsystem'`

#### **Every machine has a password in AD that corresponds to the computer object.**

* It’s rotated every 30 days. The `lastlogontimestamp` value represents the last time a computer was started.

If the epoch timestamp is from a long time ago, this is a good way of detecting machines that are currently turned off, therefore they will be missed by automated scans and tools. Below is a query to show these systems:

* `cat computers.json | jq '.data[].Properties | .name + " " + (.lastlogontimestamp|tostring)'`
* `cat computers.json | jq '.data[].Properties | select(.lastlogontimestamp > 1646237212) | .name'`
