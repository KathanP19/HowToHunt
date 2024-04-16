## RATE LIMIT FLAWS
This flaw leveraged by malicious actors to perform DDoS, brute force, and bot attacks on APIs. Although it's more than that.
##### NOTE: Some organisation `keep rate-limit bug as OOS`, So check their policy before testing.
## Rate-limit Checks
1 - Rate limit on Forget password
2 - Rate limit on Sign-up Page
3 - Rate limit on Login Page
4 - Rate limit on Invite user normal
5 - Rate limit on Invite user using MACROS
6 - Rate limit on 2FA
7 - Rate-limit on Comment and sent messages
8 - Use your own brain somewhere

## Bypass-Techniques
#### 1 - Append NULL characters at the end of the request : 
`%00, %0d%0a, %0d, %0a, %09, %0C, %20, ( )space`

    POST /signup/new/1337 HTTP/1.1
    HOST: api.target.com
    ...
    email=hacker%40gmail.com&password=12345678%00
#### 2 - Append NULL characters at the end of the Path : 
`%00, %0d%0a, %0d, %0a, %09, %0C, %20, ( )space`
`POST /profile/post/like%00 HTTP/2`

#### 3) Using Custom `HTTP headers`
	X-Originating-IP: 127.0.0.1
	X-Forwarded-For: 127.0.0.1
	X-Remote-IP: 127.0.0.1
	X-Remote-Addr: 127.0.0.1
	X-Client-IP: 127.0.0.1
	X-Host: 127.0.0.1
	X-Forwared-Host: 127.0.0.1

---
	X-Originating-IP: 127.0.0.2
	X-Forwarded-For: 127.0.0.2
	X-Remote-IP: 127.0.0.2
	X-Remote-Addr: 127.0.0.2
	X-Client-IP: 127.0.0.2
	X-Host: 127.0.0.2
	X-Forwared-Host: 127.0.0.2
---
	X-Originating-IP: 127.0.1
	X-Forwarded-For: 127.0.1
	X-Remote-IP: 127.0.1
	X-Remote-Addr: 127.0.1
	X-Client-IP: 127.0.1
	X-Host: 127.0.1
	X-Forwared-Host: 127.0.1

#### 4 - Changing the value of `User-Agent:`
    UserAgent: 'CHANGED_USERAGENT'

#### 5 - Adding Custom `parameter` in GET request
`GET /accout/passwordreset/?test=test`

#### 6 - Change request body, (`JSON -> XML`) or vice versa
   Use Burp Extension --> `Content Type Converter`

#### 7 - Changing API version , 
`/api/v2/user/reset_pw --> /api/v1/user/reset_pw or /api/v3/user/reset_pw`

#### 8 - Bypass through Exploiting Logic flaw on Login page, 
   - Take Attacker and Victim account
   - Identify how many enough login attempts in application
   - For-eg. if application gives only 3 attempts, then
   - By using burp macros, send the attackers login request 1 time and victim login request 2 time, or alternatively 
   - If NOT blocked, Repeat the process until we get victim's password
#### 9 - Try to find `Origin IP` of the Application
   - Shodan
   - Censys
   - Visit the application with it's IP address 
   - Do your own research