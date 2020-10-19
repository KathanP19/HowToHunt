**Identifying a WAF**
```
dig +short example.com
curl -s https://ipinfo.io/IP | jq -r '.org'
```

-  With AWS, you can often identify a load balancer with the presence of "AWSLB" and "AWSLBCORS" cookies

**Identifying the source**

- Use https://dnsdumpster.com to generate a map.

- Next, make a search using Censys and save the IP's that look to match your target in a text file.
Example: https://censys.io/ipv4?q=0x00sec.org

- Another way you can find IP's tied to a domain is by viewing their historical IPs. You can do this with SecurityTrails DNS trails. 
https://securitytrails.com/domain/0x00sec.org/dns

	-	Here we can see what A records existed and for how long. It is so common for an administrator to switch to a WAF solution after X amount of years of using it bare-metal, and do you think they configure whitelisting? No of course not, it works fine!
	-	you can just copy the entire table(Select full table and copy paste it in a txt file) body and use awk to filter the IP's out.
		
		`grep -E -o "([0-9]{1,3}[\\.]){3}[0-9]{1,3}" tails.txt | sort -u | tee -a ips.txt`
		
**DNS Enumeration**
		
	If you enumerate your targets DNS, you may find that they have something resembling a dev.example.com or staging.example.com subdomain, and it may be pointing to the source host with no WAF. 
		
	- Get all the subdomains.
		`subfinder -silent -d 0x00sec.org | dnsprobe -silent | awk  '{ print $2 }'  | sort -u | tee -a ips.txt`
		
**Checking IP's for hosts**


```
for ip in $(cat ips.txt) # iterate through each line in file
do 
	org=$(curl -s <https://ipinfo.io/$ip> | jq -r '.org') #  Get Org from IPInfo
  title=$(timeout 2 curl -s -k -H "Host: 0x00sec.org" <https://$ip/> | pup 'title text{}') # Get title
	echo "IP: $ip Title: $title Org: $org" # Print results
done 
```
in one line, same command:
`for ip in $(cat ips.txt); do org=$(curl -s <https://ipinfo.io/$ip> | jq -r '.org'); title=$(timeout 2 curl --tlsv1.1 -s -k -H "Host: 0x00sec.org" <https://$ip/> | pup 'title text{}'); echo "IP: $ip Title: $title Org: $org"; done`


- What we have now is a quick overview of which IP's respond to which Host header, and we can view the title
- We went through each host, requested the IP directly with the host header, and we have our source IP!

**Setting the Host Header manually**
`curl -s -k -H "Host: 0x00sec.org" https://<ip address>/`

or set Host Header in burp.

**CloudFail** 

```
git clone <https://github.com/m0rtem/CloudFail.git>
cd CloudFail
pip install -r requirements.txt
python3 cloudfail.py -t 0x00sec.org
```

**But first, Recon!**
- The idea is to start your normal recon process and grab as many IP addresses as you can (host, nslookup, whois, ranges…), then check which of those servers have a web server enabled (netcat, nmap, masscan). 
- Once you have a list of web server IP, the next step is to check if the protected domain is configured on one of them as a virtual host.

**Censys**
-  Choose “Certificates” in the select input, provide the domain of your target, then hit \<enter\>
-  You should see a list of certificates that fit to your target
-  Click on every result to display the details and, in the “Explore” menu at the very right, choose “IPv4 Hosts”.
-  You should be able to see the IP addresses of the servers that use the certificate
-  From here, grab all IP you can and, back to the previous chapter, try to access your target through all of them.
example: 
`curl -s -k -H "Host: 0x00sec.org" https://<ip address>/`

**Mail headers**
- The next step is to retrieve the headers in the mails issued by your target: Subscribe the newsletter, create an account, use the function “forgotten password”, order something… in a nutshell do whatever you can to get an email from the website you’re testing 
- Once you get an email, check the source, and especially the headers. Record all IPs you can find there, as well as subdomains, that could possibly belong to a hosting service. And again, try to access your target through all of them.

The value of header Return-Path worked pretty well

Tool: https://github.com/christophetd/CloudFlair
This tools works on censys data.

References:
https://delta.navisec.io/a-pentesters-guide-part-5-unmasking-wafs-and-finding-the-source/
https://blog.detectify.com/2019/07/31/bypassing-cloudflare-waf-with-the-origin-server-ip-address/

# Authors
* [@maverickNerd](https://twitter.com/maverickNerd)
