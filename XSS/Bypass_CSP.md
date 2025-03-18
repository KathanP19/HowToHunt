# Content Security Policy (CSP)

## What is CSP?

Content Security Policy (CSP) is a security mechanism that defines which resources can be fetched or executed by a web page. It acts as a security policy that controls which scripts, images, and iframes can be executed on a specific page and from which sources. CSP is implemented using response headers or meta elements within an HTML page. Once implemented, the browser enforces the policy and actively blocks any violations detected.

---

## How Does CSP Work?

CSP works by restricting the sources from which active and passive content can be loaded. Additionally, it enforces security policies such as preventing the execution of inline JavaScript, disabling the use of `eval()`, and limiting resource loading to specific origins.

---

## Defining CSP Rules

The following example illustrates a CSP configuration:

```plaintext
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

---

## Key CSP Directives

Below are some important CSP directives and their functions:

1. **script-src:** Defines allowed sources for JavaScript execution, including inline scripts and external script files.
2. **default-src:** Sets the default policy for resource loading when specific fetch directives are not defined.
3. **child-src:** Controls allowed sources for web workers and embedded frames.
4. **connect-src:** Restricts URLs used in interfaces such as `fetch`, `WebSocket`, and `XMLHttpRequest`.
5. **frame-src:** Defines allowed sources for `<frame>` and `<iframe>` elements.
6. **frame-ancestors:** Specifies which sources can embed the current page in `<frame>`, `<iframe>`, `<object>`, or `<embed>` elements.
7. **img-src:** Defines allowed sources for loading images.
8. **manifest-src:** Specifies allowed sources for application manifest files.
9. **media-src:** Defines sources for loading media files such as audio and video.
10. **object-src:** Restricts the sources for `<object>`, `<embed>`, and `<applet>` elements.
11. **base-uri:** Specifies allowed base URLs that can be loaded using the `<base>` element.
12. **form-action:** Lists valid endpoints for form submissions using `<form>` elements.
13. **plugin-types:** Restricts the MIME types of plugins that can be invoked.
14. **sandbox:** Applies various security restrictions on loaded resources, such as preventing pop-ups, restricting script execution, and enforcing the same-origin policy.
15. **upgrade-insecure-requests:** Instructs browsers to upgrade HTTP requests to HTTPS.

---

## CSP Bypass Techniques

### 1. CSP Misconfiguration

One of the most common reasons for CSP bypass is the use of insecure values in policy definitions. Consider the following CSP header:

```plaintext
default-src 'self' *;
```

- The `default-src` directive acts as a fallback policy for all fetch directives.
- The wildcard `*` allows loading resources from any source.
- This effectively nullifies the security benefits of CSP.

Another example:

```plaintext
script-src 'unsafe-inline' 'unsafe-eval' 'self' data: https://www.google.com http://www.google-analytics.com/gtm/js  https://*.gstatic.com/feedback/ https://accounts.google.com;
```

- The `script-src` directive allows JavaScript execution from the specified sources.
- The presence of `'unsafe-inline'` permits inline JavaScript execution, which can lead to Cross-Site Scripting (XSS).
- The presence of `data:` allows loading JavaScript from `data:` URLs, making it easier for attackers to inject malicious scripts.

Example exploit:

```html
<iframe src="data:text/html,<svg onload=alert(1)>"></iframe>
```

---

### 2. JSONP-Based CSP Bypass

JSONP (JSON with Padding) is a technique used to bypass the Same-Origin Policy (SOP) by injecting JavaScript payloads into API responses. If a JSONP endpoint is included in the `script-src` policy, it can be exploited to inject malicious scripts.

Example JSONP endpoint:

```plaintext
https://accounts.google.com/o/oauth2/revoke?callback=alert(1337)
```

If a CSP policy includes `accounts.google.com` in the `script-src` directive, an attacker can exploit it as follows:

```plaintext
something.example.com?vuln_param=https://accounts.google.com/o/oauth2/revoke?callback=alert(1337)
```

This allows JavaScript execution from an external source, effectively bypassing CSP.

---

### 3. CSP Injection

CSP injection occurs when user-controlled input is reflected in the CSP header. Consider the following vulnerable URL:

```plaintext
example.com?vuln=something_vuln_csp
```

If the value of `vuln` is directly inserted into the CSP header, an attacker can manipulate the policy:

```plaintext
script-src something_vuln_csp;
object-src 'none';
base-uri 'none';
require-trusted-types-for 'script';
report-uri https://csp.example.com;
```

By modifying the `script-src` directive, an attacker can include a malicious domain, allowing external JavaScript execution.

---

## Author

For further information or discussions, feel free to reach out to:

- **[@harsha0x01](https://twitter.com/harsha0x01)**

---

*Enhanced and reformatted for HowToHunt repository by [remonsec](https://x.com/remonsec)*
