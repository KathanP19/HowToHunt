# Easy CVES using Researching
  
### Tools
* Google
* Twitter
* Nuclei
  
## Steps:
```
    1.Grab all the subdomains i.e, subfinder -d domain.com | tee -a domains.txt
    2.Grap all alive domains i.e,  cat domains.txt | httpx -status-code | grep 200 | cut -d " " -f1 | tee -a alive.txt
    3.Run nuclei basic-detection,panels,workflows,cves templates differently and store results in different file. i.e, cat alive.txt | nuclei -t nuclei-templates/workflows | tee -a workflows.
    4.Read each output carefully with patience.
    5.Find interest tech used by target. i.e, jira
    6.put that link into browser check the version used by target.
    7.Go on google search with jira version exploit.
    8.grep the cves
    9.Go to twitter in explore tab search CVE(that you found from google) poc or CVE exploit
    10.Go to google and put cve or some details grab from  twitter for a better poc read writeups related to that.
    11.Try all cves if success report it.:)
 ```   

### Authors
* [@Virdoex_hunter](https://twitter.com/Virdoex_hunter)
    
    
    
