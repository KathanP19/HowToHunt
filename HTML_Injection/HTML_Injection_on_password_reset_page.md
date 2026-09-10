# HTML Injection on the Password Reset Page

HTML injection means user-controlled data is rendered as raw HTML in a context the victim trusts — here, the password reset **email**, which is one of the highest-trust messages a user reads. Apps that allow tags/special characters in the account name often interpolate it unescaped into the reset email. Impact: convincing phishing from the target's own domain, credential harvesting, and an escalation ramp to stored XSS.

## Hunting Checklist

1. Find every field that later appears in emails or the reset page: name, username, email display name, org/company.
2. Edit the profile name to a benign visible tag and save.
3. Trigger a password reset and inspect the email **as rendered HTML** (not plain text) — "show original" alone isn't enough; view the formatted message.
4. If tags execute, escalate step by step: styled text → link (`<a href>`) → controlled-domain phish → previously-found XSS page.
5. Also test the in-app reset page itself for the same reflection (name shown above "Hi, {name}").
6. Verify whether sanitization strips only `<script>` (leave HTML) vs all tags — try event handlers and case variants later.

## Test Cases & Examples

### 1. Basic Tag Execution in the Reset Email
1. Create an account.
2. Edit the name to either of:
```html
<h1>attacker</h1>
```
```html
"abc><h1>attacker</h1>
```
3. Save, request a password reset, and open the email — the `<h1>` renders executed → HTML injection confirmed.

### 2. Severity Escalation via Controlled Link
HTML injection alone is usually low/medium; serving an attacker link flips it into phishing:
```html
<h1>attacker</h1><a href="https://your-controlled-domain">Click here</a>
```
- Redirect the victim to a fake password-reset page on your domain → credential theft.
- Or point the link at a page hosting a **previously reported XSS** to chain into cookie theft/session hijack.

### 3. Reset-Page Context Reflection
Same payload injected into the name field, then triggered through the reset flow's confirmation page:
```http
GET /reset HTTP/1.1
Host: target.com

```
Check whether the greeting/name area renders the injected markup.

## Bypass Techniques

- If `<h1>` is filtered, rotate: `<b>`, `<i>`, `<img src=x>`, uppercase tags (`<H1>`), nested/obfuscated encodings, or split markers across two fields.
- If angle brackets are stripped, test double-encoding (`%253C`) and attribute contexts (`" onmouseover=...`) in other name fields.
- CSP only blocks script execution, **not** HTML injection — pure markup/phishing often lands untouched.

## Tools & Resources

- Burp Suite (intercept the profile-update request; Repeater for variants), email "view source" to confirm the markup reaches the client.
- Related: `XSS/Xss.md` (escalation chaining), `Password_Reset_Functionality/` (the flow being poisoned).

## Report Tips

- Lead with the phishing scenario: victim receives a *genuine, in-domain* reset email whose visible text/links are attacker-controlled.
- Include the rendered screenshot of the actual email (headers visible) — proof of context, not just injection.
- If chained to a known XSS page for cookie theft, attach that chain as "additional impact" — HTML injection reports with escalation are triaged higher.

## Credits

- [@C1pher15](https://twitter.com/C1pher15)
