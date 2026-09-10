# Rate Limit Bypass

Rate limiting should throttle repeated abuse of sensitive endpoints (login, OTP, password reset, invitations), but implementations frequently key the counter on something the client controls — IP derived from a header, HTTP method, exact path, or request body format. Sending the same request in a slightly different shape resets the counter, turning a "protected" endpoint into an unlimited brute-force, spam, or enumeration vector. Impact mirrors missing rate limits: credential stuffing/ATO, email/SMS cost abuse, and promo/OTP code guessing.

## Hunting Checklist

1. Trigger the legitimate limit first (e.g. 3–10 wrong logins) so you know what "blocked" looks like (status code, message, headers like `X-RateLimit-Remaining`).
2. Determine the rate-limit key: change one identity signal at a time (IP header, User-Agent, session cookie, account) and see which change resets the counter.
3. Work through the bypass categories below, one variable per test: HTTP method → spoofing headers → path/payload special characters → parameters/body format → API version → client identity (IP rotation) → logic abuse.
4. For each variant, send the *next* request that should have been blocked — a success means the limit is bypassed, not just inconsistent.
5. Confirm end-to-end abuse at scale (e.g. 50 more attempts through the bypass) before reporting.
6. If the limit only applies to the exposed API gateway, look for the origin IP and test it directly.

## Test Cases & Examples

### 1. Customizing HTTP methods

- If the request goes on `GET`, try changing it to `POST`, `PUT`, etc.
- If you want to bypass the rate limit on APIs, try the `HEAD` method.

```http
POST /api/v1/login HTTP/1.1
Host: target.com
Content-Type: application/json

{"email":"victim@x.com","password":"guess"}
```

```http
HEAD /api/v1/login?email=victim@x.com&password=guess HTTP/1.1
```

### 2. Adding headers to spoof the IP

Use the following headers just below the `Host` header:

```http
X-Forwarded-For: IP
X-Forwarded-IP: IP
X-Client-IP: IP
X-Remote-IP: IP
X-Originating-IP: IP
X-Host: IP
X-Client: IP
```

Or use a double `X-Forwarded-For` header (backends often trust the last value):

```http
X-Forwarded-For:
X-Forwarded-For: IP
```

Full header set collected for spoofing IPs and evading detection:

```http
X-Forwarded: 127.0.0.1
X-Forwarded-By: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Forwarded-For-Original: 127.0.0.1
X-Forwarder-For: 127.0.0.1
X-Forward-For: 127.0.0.1
Forwarded-For: 127.0.0.1
Forwarded-For-Ip: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Forwared-Host: 127.0.0.1
```

Rotate the value on every request to keep minting "new clients":

```http
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
```

### 3. Special characters in payloads and paths

- Adding a null byte (`%00`) at the end of the email can sometimes bypass the rate limit — the validator sees a "different" value while the backend may still normalize/deliver it.
- Try adding a space character after the email (not encoded).
- Common characters that help bypass rate limits: `%0d`, `%2e`, `%09`, `%20`, `%0`, `%00`, `%0d%0a`, `%0a`, `%0C`
- Appending a slash (`/`) at the end of the API endpoint can also bypass the limit: `domain.com/v1/login` → `domain.com/v1/login/`

Append null characters at the end of the request body:

```http
POST /signup/new/1337 HTTP/1.1
Host: api.target.com
Content-Type: application/x-www-form-urlencoded

email=hacker%40gmail.com&password=12345678%00
```

Append them at the end of the path:

```http
POST /profile/post/like%00 HTTP/2
Host: target.com
```

### 4. Changing the User-Agent

```http
User-Agent: CHANGED_USERAGENT
```

Try changing the User-Agent, the cookies... anything that could identify you — some limiters key the counter on the fingerprint rather than the IP.

### 5. Adding a custom parameter to a GET request

```http
GET /accout/passwordreset/?test=test HTTP/1.1
Host: target.com
```

An unexpected parameter can make the request hash/path "new" to the limiter while the endpoint still processes it.

### 6. Changing the request body format (JSON ⇄ XML)

```http
POST /api/v1/login HTTP/1.1
Content-Type: application/xml

<login><email>victim@x.com</email><password>guess</password></login>
```

Use the Burp extension **Content Type Converter** to flip formats quickly — limits often count only one serialization.

### 7. Changing the API version

```http
/api/v2/user/reset_pw   →   /api/v1/user/reset_pw
/api/v2/user/reset_pw   →   /api/v3/user/reset_pw
```

### 8. Exploiting the limiter's logic (login example)

If the app gives only 3 attempts per *account*:

1. Take attacker and victim accounts.
2. Identify how many login attempts are allowed.
3. Using Burp macros, send the attacker's login request 1 time (burning 1 of the shared quota) and the victim's login request 2 times — repeat.
4. If NOT blocked, repeat the process until you get the victim's password.

### 9. Bypassing the edge entirely — origin IP

If they are limiting to 10 tries per IP, every 10 tries change the IP inside the header; also try reaching the app itself:

1. Find the origin IP of the application via Shodan / Censys.
2. Visit the application by its IP address directly.
3. Do your own research (DNS history, certificate transparency, mail headers).

## Bypass Techniques (summary)

| Category | Trick |
|---|---|
| Verb | GET↔POST↔PUT↔HEAD |
| Identity headers | IP-spoofing headers, rotating their values |
| Client fingerprint | User-Agent / cookie changes |
| Path | trailing slash, `%00` and other control/space chars |
| Payload | `%00`, space after email, common chars (`%0d`, `%2e`, `%09`, `%20`, `%0d%0a`, `%0a`, `%0C`) |
| Request shape | extra custom parameter, JSON ⇄ XML |
| Route | old/new API version |
| Logic | burn quota with attacker account via macros |
| Network | IP rotation extension, direct origin-IP access |

## Tools & Resources

- **IP Rotate (Burp extension)** — rotates egress IPs so per-IP limits never trigger; works well in many cases (requires Jython installed). Everything you need: <https://github.com/PortSwigger/ip-rotate>
- **Content Type Converter (Burp BApp)** — JSON ⇄ XML switching.
- **Burp macros + Turbo Intruder** — logic-abuse and high-volume confirmation.
- Shodan / Censys — origin IP discovery.
- Writeups: [Bypassing Rate Limit Like a Pro](https://medium.com/bugbountywriteup/bypassing-rate-limit-like-a-pro-5f3e40250d3c), [Methods to Bypass Rate Limit (with screenshots)](https://medium.com/@huzaifa_tahir/methods-to-bypass-rate-limit-5185e6c67ecd)

References:
- <https://twitter.com/m4ll0k2/status/1294983599943540738/photo/1>
- <https://twitter.com/SalahHasoneh1/status/1287366496432332800>
- <https://twitter.com/SMHTahsin33/status/1295054667613757441> (all-in-one, must check)

## Report Tips

- Reproduce the full cycle: "blocked at N attempts → applied <bypass> → N more attempts succeeded" — the delta proves the bypass, and the final successful login/OTP/invite proves impact.
- Name what the bypass protects (login, 2FA, password reset, SMS/email sending) so triage sees the credential-stuffing or cost impact, not just a header trick.
- If IP-spoofing headers are trusted app-wide, mention that the same headers may also poison logs, audits, and IP-based allowlists (larger blast radius than rate limits).

## Authors

- [Keshav Malik](https://www.linkedin.com/in/keshav-malik-22478014a)
- [0xd3vil](https://linkedin.com/in/0xd3vil)
- [Virdoex_hunter](https://twitter.com/Virdoex_hunter)
- [@0xCyberPirate](https://twitter.com/0xCyberPirate)
