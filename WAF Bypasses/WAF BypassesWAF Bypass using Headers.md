# WAF Bypass using Headers(Password reset poisoning)
  For waf bypass, and similar
  ```
	X-Forwarded-Host
	X-Forwarded-Port
	X-Forwarded-Scheme
	Origin: 
	nullOrigin: [siteDomain].attacker.com
	X-Frame-Options: Allow
	X-Forwarded-For: 127.0.0.1
	X-Client-IP: 127.0.0.1
	Client-IP: 127.0.0.1
  ```
  
 # Authors
* [Virdoex_hunter](https://twitter.com/Virdoex_hunter)
