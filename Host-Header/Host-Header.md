# Summary For Host Header
![https://pbs.twimg.com/media/ET39wJOWoAAfTBb?format=jpg&name=small](https://pbs.twimg.com/media/ET39wJOWoAAfTBb?format=jpg&name=small)

# Also Check This Things While Testing
1. Add two `HOST:` in Request.
2. Try this Headers
    ```      
       X-Original-Url:
       X-Forwarded-Server:
       X-Host:
       X-Forwarded-**Host**:
       X-Rewrite-Url:
    ```
3. If you come across `/api.json` in any AEM instance during bug hunting, try for web cache poisoning via following  
  `Host: , X-Forwarded-Server , X-Forwarded-Host:`
   and or simply try https://localhost/api.json HTTP/1.1
4. Also try `Host: redacted.com.evil.com`
5. Try Host: evil.com/redacted.com
[https://hackerone.com/reports/317476](https://hackerone.com/reports/317476)
6. Try this too `Host: example.com?.mavenlink.com`
7. Try `Host: javascript:alert(1);` Xss payload might result in debugging mode.
[https://blog.bentkowski.info/2015/04/xss-via-host-header-cse.html](https://blog.bentkowski.info/2015/04/xss-via-host-header-cse.html)
8. Host Header to Sqli
[https://blog.usejournal.com/bugbounty-database-hacked-of-indias-popular-sports-company-bypassing-host-header-to-sql-7b9af997c610](https://blog.usejournal.com/bugbounty-database-hacked-of-indias-popular-sports-company-bypassing-host-header-to-sql-7b9af997c610)
9. Bypass front server restrictions and access to forbidden files and directories through `X-Rewrite-Url/X-original-url:` 
   `curl -i -s -k -X 'GET' -H 'Host: <site>' -H 'X-rewrite-url: admin/login' 'https://<site>/'.`


Author:[@KathanP19](https://twitter.com/KathanP19)
