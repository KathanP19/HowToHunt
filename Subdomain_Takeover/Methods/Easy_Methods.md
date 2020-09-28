#  1. Method by [@Virdoex_hunter](https://twitter.com/Virdoex_hunter)
Easy Subdomain Takeover Method
```
Step:

1:Grab all subdomains of target. i.e, subfinder -d flaws.cloud | tee -a domains.txt
			
2:Run this one liner
			
3:cat domains.txt | while read domain;do dig  $domain;done | tee -a digs.txt
			
4::Grab all the CNAME Entries i.e, cat digs.txt | grep CNAME
			
5:Find a domain that is pointed to third party domain like sub.exampple.com CNAME x.aws.com
			
6:Check wheather the main subdomain is down
			
7:Go to host provider where the domain is pointed to and register that domain if you registered congrats you have takeover the subdomain.
			
```

# 2. Method by [@WhoIs1nVok3r](https://twitter.com/WhoIs1nVok3r)
```
Step-1:- First of all collect all subdomain of the target using assetfinder,subfinder,chaos(needs API key).

Step-2:- Next sort out duplicate URLs using -- cat unresolved | sort -u | tee -a resolved

Step-3:- Pass it to subzy,subjack or other subdomain-takeover tool -- using subzy tool  --  subzy -targets resolved , or use subjack

Step-4:- We can also use nuclei templates but we need to first use httpx -- cat resolved | httpx | tee -a hosts

Step-5:- Next use nuclei-templates -- cat hosts | nuclei -t nuclei-templates/vulnerabilites -o nuclei.txt -v 

Tools Used:-

https://github.com/projectdiscovery/nuclei
https://github.com/projectdiscovery/subfinder
https://github.com/projectdiscovery/httpx
https://github.com/projectdiscovery/nuclei-templates
https://github.com/projectdiscovery/chaos-client
https://github.com/haccer/subjack
https://github.com/LukaSikic/subzy
```

## Author 
* [@Virdoex_hunter](https://twitter.com/Virdoex_hunter)
* [@WhoIs1nVok3r](https://twitter.com/WhoIs1nVok3r)
