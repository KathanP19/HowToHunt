# Password Reset Token Leakage

Reset tokens are bearer credentials: whoever holds them holds the account. Leakage means the token escapes to a place the victim doesn't control — third-party scripts, the response body, the email itself, or logs. Each leak vector turns a phishing-grade primitive into direct ATO. Severity: typically P1–P2 when leakage to a third-party domain is proven with a real token.

Referrer-header leakage with full steps is documented once in `Top_5_Password_Reset_Bugs.md` §1 — this file adds the analytics/3rd-party-domain check and the remaining leak surfaces.

## Hunting Checklist

1. Trigger a reset; open the emailed link with Burp intercept **on** and a second browser tab ready.
2. Walk every outbound request from the reset page and grep them all for the token string from the URL.
3. Inspect the token-bearing request/response bodies for the same token or its source.
4. Open the reset email as text and check for token disclosure beyond the link (plain text, QR, API endpoints).
5. Check redirect handling from email links (token re-appearing on the final page's resource requests).
6. Test with the *token from the link* — do not paste live production tokens of other users.

## Test Cases & Examples

### 1. Token Leaked to Third-Party Domains (analytics/social trackers)
Steps (original method):
1. Send a password-reset request via "forgot password".
2. Check your email.
3. Copy the reset-page link, open it in another tab, and keep Burp intercept **on**.
4. Inspect every request leaving the page for a token equal to the one in the reset URL, aimed at a third-party domain such as `bat.bing.com` or `facebook.com`.
5. If found, the reset token is leaked to a third party — leakage confirmed (a victim clicking through from search/social leaks their token the same way).

Example of what to grep for:
```http
GET /action/event?...&ref=https%3A%2F%2Ftarget.com%2Freset%3Ftoken%3D<same_token_as_email_link> HTTP/1.1
Host: bat.bing.com
```
```bash
# quick manual grep while walking the flow
grep -r "token=<value-from-reset-url>" burp_requests/
```

### 2. Token in the Reset Email Body
- Read the email as raw/source: token may appear in a "copy-paste this code" line, a `List-Unsubscribe`-style URL, or a QR image, widening exposure to inbox scanners/Mail Rules that forward body content.

### 3. Token/OTP in API Responses
- Intercept responses of the request/reset endpoints: token or OTP echoed in JSON (pattern: `{"otp":"123456"}`) — same root cause as `Account_Takeovers_Methodologies/Account_Takeovers_Methods.md` §6.

### 4. Referrer-Based Leakage
- Full reproduction steps in `Top_5_Password_Reset_Bugs.md` §1; the durable check is: any external navigation from the reset page must not carry the token.

## Bypass Techniques

- If the reset page sets `Referrer-Policy`, look for leaks via other channels instead: meta-refresh to a tracker, `<img>` pixels with the token in the query string, shared-link/unfurling services, and error pages reflecting the token.
- Tokens in `Location:` redirects you don't control (open-redirect chaining on the reset flow) count as leakage — check every redirect hop.

## Tools & Resources

- Burp Suite Proxy history (search across all requests for the token value), browser dev-tools Network tab, `grep`/`rg` over exported request sets.
- [10 Password Reset Flaws](https://anugrahsr.github.io/posts/10-Password-reset-flaws/)

## Report Tips

- Redact everything *except* the token in screenshots, and state the exact third-party domain + request line that received it.
- Use a token from **your own** account to prove the mechanism, then explain the victim path (search click, email forward, tracker).
- Severity framing: "bearer token valid T minutes exfiltrated to domain X" — give T (expiry) so triage can rate the window.

## Credits

- [@Virdoex_hunter](https://twitter.com/Virdoex_hunter) — leakage method.
