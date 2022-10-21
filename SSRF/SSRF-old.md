# SSRF ( Server-Side-Request-Forgery)
* What's SSRF ??
   * SSRF is a type of exploit where an attacker abuses the functionality of a server causing it to access or manipulate information in the realm of that server that would otherwise not be directly accessible to the attacker.

## Where to look for ??

1. If you got Open Redirect try escalating it to SSRF.

2. gf SSRF to grep parameters may vulnerable to SSRF.

3. SSRF's are more in API's so crawl the whole web app with burp proxy turned on and search for keywords like., eg :
```
?url=
?uri=
?req= 
etc.....
```
4. Sign up with an Email like blabla.collaborator.net. If u receive HTTP req. in collaborator then its SSRF. But if there's no impact Don't Report it :) DNS and SMTP req. Doesn't matters.

## AWS Metadata
Most of the sites use AWS nowadays...

* AWS localhost is 169.254.169.254 so don't use 127.0.0.1 there!

* If you found an SSRF vulnerability that runs on EC2, try requesting :
```
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/user-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/IAM_USER_ROLE_HERE
http://169.254.169.254/latest/meta-data/iam/security-credentials/flaws/
```
* Source: https://twitter.com/ADITYASHENDE17/status/1305051512335298562

## Escalation

* SSRF can be Escalated to RCE :) [Impact High] 
* `<os cmd>`.collaborator.net (thehackerish has a good video in it :)
* If there's no impact! on your SSRF rather than a redirect try to escalate it to XSS.

## Resources 💯
### Youtube
* https://www.youtube.com/watch?v=U0bPPw6uPgY&t=1s
* https://www.youtube.com/watch?v=324cZic6asE
* https://www.youtube.com/watch?v=o-tL9ULF0KI
* https://www.youtube.com/watch?v=324cZic6asE&t=751s
* https://youtu.be/m4BxIf9PUx0
* https://youtu.be/apzJiaQ6a3k
* [A New Era of SSRF](https://www.youtube.com/watch?v=R9pJ2YCXoJQ) by [Orange Tsai](https://blog.orange.tw/)

### Hackerone Reports
* https://hackerone.com/hacktivity?order_field=popular&filter=type%3Apublic&querystring=SSRF
* https://hackerone.com/reports/737161
* https://hackerone.com/reports/816848
* https://hackerone.com/reports/398799
* https://hackerone.com/reports/382048
* https://hackerone.com/reports/406387
* https://hackerone.com/reports/736867
* https://hackerone.com/reports/517461
* https://hackerone.com/reports/508459
* https://hackerone.com/reports/738553
* https://hackerone.com/reports/514224
* https://www.hackerone.com/blog-How-To-Server-Side-Request-Forgery-SSRF
* https://hackerone.com/reports/341876
* https://hackerone.com/reports/793704
* https://hackerone.com/reports/386292
* https://hackerone.com/reports/326040
* https://hackerone.com/reports/310036
* https://hackerone.com/reports/643622
* https://hackerone.com/reports/885975
* https://hackerone.com/reports/207477
* https://hackerone.com/reports/514224

### Blogs
* https://medium.com/@madrobot/ssrf-server-side-request-forgery-types-and-ways-to-exploit-it-part-1-29d034c27978
* https://medium.com/@kapilvermarbl/ssrf-server-side-request-forgery-5131ffd61c3c
* https://medium.com/@zain.sabahat/exploiting-ssrf-like-a-boss-c090dc63d326
* https://medium.com/@chawdamrunal/what-is-server-side-request-forgery-ssrf-7cd0ead0d95f
* https://medium.com/swlh/ssrf-in-the-wild-e2c598900434
* https://medium.com/@briskinfosec/ssrf-server-side-request-forgery-ae44ec737cb8
* https://medium.com/@GAYA3_R/vulnerability-server-side-request-forgery-ssrf-9fe5428184c1
* https://medium.com/@gupta.bless/exploiting-ssrf-for-admin-access-31c30457cc44
* https://medium.com/bugbountywriteup/server-side-request-forgery-ssrf-f62235a2c151
* https://medium.com/@dlpadmavathi.us/ssrf-attack-real-example-a7279256abee
* https://blog.securityinnovation.com/the-many-faces-of-ssrf
* https://www.netsparker.com/blog/web-security/server-side-request-forgery-vulnerability-ssrf/
* http://www.techpna.com/uptzh/blind-ssrf-medium.html
* https://blog.appsecco.com/finding-ssrf-via-html-injection-inside-a-pdf-file-on-aws-ec2-214cc5ec5d90
* http://institutopaideia.com.br/journal/blind-ssrf-medium-cfa769
* https://www.reddit.com/r/bugbounty/comments/cux2zs/ssrf_in_the_wild_the_startup_medium/
* https://www.sonrn.com.br/blog/5a44cc-blind-ssrf-medium
* https://ssrf-bypass-medium.thickkare.pw/
* https://hackerone.com/reports/326040
* https://www.zerocopter.com/vulnerabilities-price-list-printable
* https://medium.com/swlh/intro-to-ssrf-beb35857771f
* https://medium.com/poka-techblog/server-side-request-forgery-ssrf-attacks-part-1-the-basics-a42ba5cc244a
* https://medium.com/@madrobot/ssrf-server-side-request-forgery-types-and-ways-to-exploit-it-part-3-b0f5997e3739
* https://medium.com/bugbountywriteup/server-side-request-forgery-ssrf-testing-b9dfe57cca35
* https://medium.com/@madrobot/ssrf-server-side-request-forgery-types-and-ways-to-exploit-it-part-2-a085ec4332c0
* https://medium.com/bugbountywriteup/tagged/ssrf
* https://medium.com/seconset/all-about-ssrf-524f41ab96df
* https://blog.cobalt.io/from-ssrf-to-port-scanner-3e8ef5921fbf
* https://portswigger.net/web-security/ssrf
* https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery
* https://krevetk0.medium.com/ssrf-vulnerability-via-ffmpeg-hls-processing-f3823c16f3c7
* https://docs.google.com/presentation/d/1JdIjHHPsFSgLbaJcHmMkE904jmwPM4xdhEuwhy2ebvo/htmlpresent

### Github Repos
* https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery
* https://github.com/jdonsec/AllThingsSSRF

### Author:
* [@0xCyberPirate](https://twitter.com/0xCyberPirate)
* [0xrtt](https://twitter.com/0xrtt)
