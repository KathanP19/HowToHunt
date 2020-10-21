# HTTP Desync or Request Smuggling:
- Basics:
"HTTP request smuggling is a technique for interfering with the way a web site processes sequences of HTTP requests that are received from one or more users. Request smuggling vulnerabilities are often critical in nature, allowing an attacker to bypass security controls, gain unauthorized access to sensitive data, and directly compromise other application users. " -Portswigger  


 ## Where ?:  

 - Any Endpoint might be Vulnerable to HTTP Desync attack.  
 
 - You can Find the Vulnerability on Non-endpoints as well, But impact is always much higher on Sensitive Endpoints ;)
 ---
 ### Step 1:  

 * Go To Repeater tab, and try various Timing based payloads to confirm the bug. More Explaination here:  

[Finding the Vulnerability](https://portswigger.net/web-security/request-smuggling/finding)

### Step 2:  

* Once you have successfully discovored the bug, you can chain it with various bugs eg. Account Takeover by stealing session IDs, Cross side Scripting Attacks in User-Agent Header,etc. More Description here:  

[Exploiting the Vulnerability](https://portswigger.net/web-security/request-smuggling/exploiting)  

---
## Tools:  

1. [defparam`s_smuggler.py](https://github.com/defparam/smuggler)  

`Usage:`  
* Smuggler.py :

    `cat alive_urls.txt | python3 smuggler.py -m GET/POST #either GET or POST ` 
    
    OR
    
    ` python3 smuggler.py -u https://example.com -m GET/POST  `
    
2. [Burp_smuggler](https://github.com/PortSwigger/http-request-smuggler) (also available in BApp store)  

## More Info:  

### Topics  

https://paper.seebug.org/1049/ (Recommended !)  

[Portswigger Topic](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn)  

[Portswigger Lab](https://portswigger.net/web-security/request-smuggling)  

### Reports (Hackerone):  

[Report 1](https://hackerone.com/reports/737140)  

[Report 2](https://hackerone.com/reports/867952)  

[Report 3](https://hackerone.com/reports/498052)  

[Report 4](https://hackerone.com/reports/526880)

[Report 5](https://hackerone.com/reports/771666)  

[Report 6](https://hackerone.com/reports/753939)  

[Report 7](https://hackerone.com/reports/648434 )  

[Report 8](https://hackerone.com/reports/740037)  

## Writeups (Medium.com):  

[Article 1](https://medium.com/@ricardoiramar/the-powerful-http-request-smuggling-af208fafa142)  

[Article 2](https://medium.com/cyberverse/http-request-smuggling-in-plain-english-7080e48df8b4)  

[Article 3](https://medium.com/@cc1h2e1/write-up-of-two-http-requests-smuggling-ff211656fe7d)  

[Article 4](https://medium.com/bugbountywriteup/crossing-the-borders-the-illegal-trade-of-http-requests-57da188520ca)  

## Extra:  

[A Brief Video About Req. Smuggling](https://youtu.be/gzM4wWA7RFo)

### Author:
[Neutron__](https://twitter.com/Neutron__)
###### If you think something was missed, feel free to add/modify/delete it :)
