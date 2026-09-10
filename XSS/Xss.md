# Cross-Site Scripting (XSS)

XSS lets an attacker inject JavaScript that runs in another user's browser within the target's origin, making it one of the most common and impactful web vulnerabilities. It enables session hijacking, account takeover, credential/phishing attacks, defacement, and actions performed on behalf of victims. Reflected/Stored XSS with reachable victims is typically medium–high severity; XSS that steals tokens or leads to account takeover should be reported as high/critical.

> Companion docs (no overlap kept here): [Filter/WAF bypasses →](./XSS_Bypass.md) · [CSP bypasses →](./Bypass_CSP.md) · [postMessage/DOM XSS →](./post_message_xss.md) · [Automation pipelines →](./Automated_XSS.md)

## Hunting Checklist

1. Enumerate every input that can reach a response body or another user: query parameters, POST bodies, path segments, JSON fields, headers, forms, error pages.
2. **Burp (manual):** install the **Reflection** and **Sentinel** plugins, walk/spider the site, inspect the *reflected parameters* tab, then send candidates to Sentinel or verify manually.
3. **Wayback-driven:** collect URLs with [Gau](https://github.com/lc/gau) / [WaybackURLs](https://github.com/tomnomnom/waybackurls), filter parameters with `grep "="` or **GF patterns**, scan with [Gxss](https://github.com/KathanP19/Gxss) / [Bxss](https://github.com/ethicalhackingplayground/bxss/) / [Dalfox](https://github.com/hahwul/dalfox) — full pipelines in [Automated_XSS.md](./Automated_XSS.md).
4. **Google dorks:** `site:target.com`, `site:target.com inurl:".php?"`, `site:target.com filetype:php`; more in [Top 100 XSS Dorks](https://www.openbugbounty.org/blog/devl00p/top-100-xss-dorks/).
5. **Hidden variables in source:** search Page Source / JS files for `var=`, `=""`, `=''`, then append discovered parameters to the URL (e.g. `https://example.com?hiddenvariablename=xss`). Use [Arjun](https://github.com/s0md3v/Arjun) to discover hidden parameters.
6. Check **error pages (404, 403, etc.)** for reflected values; trigger a 403 by requesting the `.htaccess` file. Test **all** reflected parameters.
7. **Identify the firewall** with [WhatWaf](https://github.com/Ekultek/WhatWaf); if payloads are blocked, work through [XSS_Bypass.md](./XSS_Bypass.md) and [Awesome WAF Bypass](https://github.com/0xInfection/Awesome-WAF).
8. **Stored XSS:** enumerate WAF rules first, then test Username, Address, Email, profile-picture **filename and metadata**, comments/reviews/feedback, and every input that reflects data to other users; register an account with a payload in the **name field**.
9. **Blind XSS:** inject callback payloads ([XSS Hunter](https://xsshunter.com/), Burp Collaborator, Ngrok) into contact forms, admin dashboards, user-input logs, e-commerce checkout fields, **User-Agent** headers, log viewers, chat apps, moderation panels.
10. **DOM XSS:** manual detection is hard — use Burp Suite PRO or [RA2 DOM XSS Scanner](https://github.com/dpnishant/ra2-dom-xss-scanner); trace sources (`location.*`, `postMessage`) into sinks (`innerHTML`, `eval`, `src`, `href`); for message listeners see [post_message_xss.md](./post_message_xss.md).
11. If execution is blocked by response headers, pivot to [Bypass_CSP.md](./Bypass_CSP.md).

## Test Cases & Examples

Confirm reflection with a harmless unique marker, then adapt to the context (HTML body, attribute, JS string, URL).

Reflected probe (break out of HTML context):

```html
"><img src=x onerror=alert(document.domain)>
```

Entity-injection probe for stored-XSS filters (if any payload executes, refine and escalate):

```html
<a href=#>test</a>
```

Source-code hunting — search Page Source / JS files for `var=`, `=""`, `=''`, then append discovered parameters:

```bash
curl -sk https://target.com/page | grep -Eo "var [a-zA-Z0-9_]+ ?="
curl -sk https://target.com/page | grep -Eo '[a-zA-Z0-9_]+="[^"]*"' | sort -u
```

Wayback + GF triage one-liner (details in [Automated_XSS.md](./Automated_XSS.md)):

```bash
waybackurls target.com | grep '=' | qsreplace '"><img src=x onerror=alert(1)>' | head -200 | dalfox pipe
```

Trigger a 403 error page that may reflect input:

```http
GET /.htaccess?q=xss%22%3E%3Cimg%20src%3Dx%20onerror%3Dalert(1)%3E HTTP/1.1
Host: target.com
```

## Bypass Techniques

Kept out of this doc to avoid duplication:

- Filter/WAF evasion (case alternation, encoding, recursive filters, polyglots, whitespace tricks, `alert()`/`onerror` alternatives) → [XSS_Bypass.md](./XSS_Bypass.md)
- Content-Security-Policy evasion (misconfigurations, JSONP gadgets, CSP injection) → [Bypass_CSP.md](./Bypass_CSP.md)
- `postMessage`-driven DOM XSS → [post_message_xss.md](./post_message_xss.md)

## Tools & Resources

- Plugins: Reflection, Sentinel (Burp) · Scanners: Dalfox, Gxss, Bxss, RA2 · Recon: gau, waybackurls, GF + patterns, Arjun, WhatWaf
- Callbacks: XSS Hunter, Burp Collaborator, Ngrok
- Mind map: [Reflected XSS Mindmap](https://github.com/A9HORA/Reflected-XSS-Mindmap) by **[@A9HORA](https://twitter.com/A9HORA)**
- Videos: [Reflected XSS Automation](https://www.youtube.com/watch?v=wuyAY3vvd9s) · [Practical XSS Hunting](https://www.youtube.com/watch?v=GsyOuQBG2yM) · [Blind XSS Hunting](https://www.youtube.com/watch?v=uHy1x1NkwRU) · [Understanding DOM XSS](https://www.youtube.com/watch?v=gBqzzhgHoYg)
- [Hunting Checklist](https://github.com/heilla/SecurityTesting/blob/master/HuntingCheckList.md)
- Credits: **[@KathanP19](https://twitter.com/KathanP19)**, **[@harsha0x01](https://twitter.com/harsha0x01)**; thanks to **[The XSS Rat](https://www.youtube.com/channel/UCjBhClJ59W4hfUly51i11hg)** and **[@sratarun](https://twitter.com/sratarun)**; write-up: [How I Found My First Stored XSS](https://medium.com/@fatin151485/how-i-found-my-first-stored-xss-on-popular-eboighar-com-6bd497b0bb96)

## Report Tips

- Show the payload executing in the victim's browser (not just reflection) and name the context; attach a short PoC video/screenshot plus exact URL and steps.
- Escalate impact concretely: session/cookie theft, account takeover via token or reset-link exposure, or actions performed as the victim; for stored/blind XSS prove another user (ideally admin — use the XSS Hunter callback) executed it.
- State which filter/WAF/CSP you bypassed and whether the payload survives at rest; mention self-XSS limitations honestly and only claim chains you actually demonstrated.
