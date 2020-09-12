
Easy Subdomain Takeover Method


		Step:
			
			1:Grab all subdomains of target. i.e, subfinder -d flaws.cloud | tee -a domains.txt
			
			
			2:Run this one liner
			
			
			
			3:cat domains.txt | while read domain;do dig +short $domain;done | tee -a digs.txt
			
			
			
			
			4::Grab all the CNAME Entries i.e, cat digs.txt | grep CNAME
			
			
			
			
			5:Find a domain that is pointed to third party domain like sub.exampple.com CNAME x.aws.com
			
			
			
			
			6:Check wheather the main subdomain is down
			
			
			
			
			
			7:Go to host provider where the domain is pointed to and register that domain if you registered congrats you have takeover the subdomain.
			
			
			
			
			
			
			
