# Open Redirect — Hunting Guide

Open redirection occurs when a URL parameter (or header/JS sink) sends the user to an attacker-controlled external site without validation. On its own most programs rate it low/informational, but it is a powerful chain piece: phishing with a trusted domain in the address bar, OAuth token theft via a poisoned `redirect_uri`, and (rarely) XSS escalation — so finding the *hidden* redirect parameters others miss pays off.

Filter-evasion payloads: [Open_Redirection_Bypass.md](./Open_Redirection_Bypass.md)

## Hunting Checklist

1. Fuzz obvious parameters across the app: `url`, `redirect`, `redirect_uri`, `redirectUrl`, `next`, `retUrl`, `return`, `returnurl`, `continue`, `target`, `dest`, `r`, `go`, `image`, `path`.
2. **Login-redirect trick** (find redirect params you didn't know existed):
   1. If the application has a Sign-In/Sign-Up feature, register a user and log in.
   2. Go to your user profile page, e.g. `samplesite.me/accounts/profile`.
   3. Copy the profile page's URL.
   4. Logout, clear all cookies, and go to the site's homepage.
   5. Paste the copied profile URL into the address bar.
   6. If the site prompts for login, check the address bar — you may find a login page with a redirect parameter:
      - `https://samplesite.me/login?next=accounts/profile`
      - `https://samplesite.me/login?retUrl=accounts/profile`
   7. Try to exploit the parameter by adding an external domain and load the crafted URL.
   8. If it redirects to the evil domain, that's your open redirection bug.
   9. Try to leverage it to XSS (see Test Cases).
3. Check every unauthenticated→authenticated transition (login, password reset, logout, SSO/OAuth flows), pricing/promo hops, language/locale switches, and email-tracking/unsubscribe links.
4. Mine historical URLs for redirect-shaped parameters:

```bash
waybackurls target.com | grep -Ei '(\?|&)(url|next|redirect|returl|return|continue|target|dest)=' 
```

5. Test client-side redirects too: grep JS for `location.href=`, `location.replace(`, `window.location=` fed from query params (`?url=` patterns reach `location` unsafely).
6. When direct external URLs are blocked, move to [Open_Redirection_Bypass.md](./Open_Redirection_Bypass.md).

## Test Cases & Examples

Direct exploitation of a discovered parameter:

```text
https://samplesite.me/login?next=https://evil.com/
```

Regex-beating credential form (domain looks legit, real host is `evil.com`):

```text
https://samplesite.me/login?next=https://samplesite.me@evil.com/
```

XSS escalation attempt (executes only where the value lands in `href`/`location:` JS sinks, and is blocked in modern address-bar navigations):

```text
https://samplesite.me/login?next=javascript:alert(1);//
```

Request showing the vulnerable 302:

```http
GET /logout?redirect=https://evil.com HTTP/1.1
Host: target.com
```

```http
HTTP/1.1 302 Found
Location: https://evil.com
```

## Bypass Techniques

Whitelists that only pattern-match (contain, ends-with, subdomain prefix) are all breakable — the curated payload list, including the `targetweb.com.attackersite.com` subdomain-prefix trick, is in [Open_Redirection_Bypass.md](./Open_Redirection_Bypass.md).

## Tools & Resources

- Recon: waybackurls / gau + `grep`, Burp parameter grep, Arjun for hidden params
- Payload references: OWASP [Unvalidated Redirects and Forwards](https://owasp.org/www-community/attacks/Unvalidated_Redirects_and_Forwards), PortSwigger Web Security Academy *Location hijacking via redirect parameter*, PayloadsAllTheThings [Open Redirect](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Open%20Redirect)
- Original trick author: **[febinrev](https://twitter.com/febinrev)** ("I apply this every time while testing web applications and found many Open Redirects — and even an XSS — using this trick!")

## Report Tips

- Don't stop at "302 to evil.com": show the *context* (login/logout/OAuth/email link) where the redirect makes the destination look trusted to victims.
- Highest-value framings: OAuth/SSO `redirect_uri` poisoning → token theft → account takeover; or a redirect used in a password-reset email/link.
- If only `javascript:`/`data:` schemes fire, demonstrate actual script execution in the victim browser (it's XSS, not a redirect) and state the browser/version where it works.
