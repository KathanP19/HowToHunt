# 2FA Bypass

Two-Factor Authentication (2FA/MFA) adds a second verification step (TOTP/SMS/app OTP, backup codes) after credentials. When the second factor is enforced only client-side, or its state is not tied to the server-side session, attackers can skip or defeat it entirely. A working bypass of mandatory 2FA typically removes the last defense in front of the account and is rated as full Account Takeover (P1/P2).

> Visual reference: [2FA Bypass Mindmap by Harsh Bothra](https://www.mindmeister.com/1736437018?t=SEeZOmvt01)

## Hunting Checklist

1. Log in with 2FA enabled and map every request in the flow: credential check, challenge request, code verification, "remember me", disable page, backup codes.
2. On the verification step, inspect the server response for trust signals the client acts on (`success`, `verified`, `status`, OTP echoed back).
3. Try client-side defeats: response manipulation, status-code manipulation, replaying a previous successful response.
4. Try input-side defeats: `null`, empty, `000000`, alphabetic values instead of digits, repeated (HPP/batch) code parameters.
5. Brute-force check: does the endpoint rate-limit or lock out after N wrong codes? Note the code length (4–6 digits).
6. Reusability check: request a code, verify it, then re-verify with the same code; request a new code and test if the old one still works.
7. Integrity check: enroll 2FA on Account A, get a valid code, and try to complete verification on Account B (and cross-substituting request params/cookies/user IDs).
8. Check whether 2FA is enforced on every entry point: password reset, OAuth/social login, API/mobile endpoints, session cookie issued before the challenge.
9. Check lifecycle flaws: password reset disabling 2FA, enabling 2FA without expiring active sessions, backup codes (leakage/reuse/no-expiry).
10. Check the 2FA disable page for CSRF and clickjacking (no re-auth required).
11. Analyze "remember me"/memorization: identify the cookie/localStorage value/IP binding and test its strength and scope.
12. Grep exposed JS bundles for OTP logic, hardcoded/static codes.

## Test Cases & Examples

### 1. Response Manipulation
Intercept the verification response (Burp match-and-replace / Break on response) and flip the trust flag:
```json
{ "success": false }
```
```json
{ "success": true }
```
Also seen in the wild:
```json
{"code":"invalid_credentials"}  →  {"code":"valid_credentials"}
{"verify":"false"}              →  {"verify":"true"}
```
If the client trusts the response body, access is granted without a valid code.

### 2. Status Code Manipulation
Wrong code returns a 4xx error that the frontend only checks the status of. Change the status line in the intercepted response:
```http
HTTP/1.1 401 Unauthorized
```
```http
HTTP/1.1 200 OK
```

### 3. Replaying a Previous Successful Response
```
Enter correct OTP → capture the successful response → logout
→ login again with WRONG OTP → replace the response with the captured
successful one → logged in
```

### 4. 2FA Code Leakage in Response
On the request that triggers the code delivery, check the API response:
```json
{ "otp": "123456" }
```
If present, use the leaked code directly.

### 5. JavaScript File Analysis
Grep exposed bundles for OTP logic or static codes:
```javascript
var otp = "123456";
```

### 6. Missing Brute-Force Protection
```http
POST /api/2fa/verify HTTP/1.1
Host: target.com
Content-Type: application/json
Cookie: session=<attacker_session>

{"code":"000000"}
```
Burp Intruder → clusterbomb/sniper on `code`, Numbers payload `000000–999999` (match length). Weak validation lets you guess the live code.

### 7. Missing Code Integrity Validation (Cross-Account)
- Get a valid OTP for Account A; submit it on the 2FA challenge of Account B.
- Variant: if the verify request carries an account/session-bound identifier (user ID, form ID, cookie), substitute Account B's values into a request from Account A's session, verify with A's code, and reload — 2FA on Account A may disappear.

### 8. CSRF on 2FA Disable
No CSRF token / no re-auth on the disable endpoint:
```html
<form action="https://victim-site.com/disable-2fa" method="POST">
    <input type="hidden" name="disable" value="true">
    <input type="submit" value="Click to win a prize!">
</form>
```
POC: two attacker-controlled accounts → capture disable request from Account 1 → fire it while logged in as Account 2 → 2FA disabled.

### 9. Clickjacking on 2FA Disable Page
If disabling 2FA requires no password/OTP re-check and the page is framable, a transparent `<iframe>` overlay can trick the victim into clicking "Disable".

### 10. Password Reset Disables 2FA
- With 2FA enabled on the account, complete a password reset.
- Log in with the new password: if the 2FA challenge is skipped or the factor is silently removed, the reset flow defeated the second factor.

### 11. Backup Code Abuse
- Are backup codes weak/short or brute-forceable (same checks as #6)?
- Do they survive password resets / 2FA re-enrollment / being "consumed"?
- Are they displayed in full anywhere in-app (API response, account page) without re-auth?

### 12. Enabling 2FA Does Not Expire Active Sessions
- Log in without 2FA in Browser A; in Browser B enable 2FA.
- Refresh Browser A: if the old session remains valid, a hijacked cookie keeps access despite 2FA being "on" (persistence vector).

### 13. `null`, `000000`, Empty
```http
POST /api/2fa/verify HTTP/1.1

{"code":null}
```
Also try empty string, `000000`, `123456`.

### 14. Wrong Type: Alphabetic Instead of Numeric
```http
POST /api/2fa/verify HTTP/1.1

{"code":"abcdefgh"}
```
Weak type handling can make comparisons succeed or errors leak state.

### 15. HPP / Batch Request Bypass
If the code parameter is validated once but consumed twice (or the framework merges arrays), repeat it:
```http
POST /api/2fa/verify HTTP/1.1
Content-Type: application/x-www-form-urlencoded

code=000000&code=<real_otp>
```
Same idea against batch-mode REST APIs: send the same sub-request multiple times with different codes.

### 16. "Remember Me" / Memorization Bypass
Identify what makes a device "remembered" (cookie, localStorage/sessionStorage key, IP binding):
- Replay that artifact from a different browser/machine/IP — does it grant the post-2FA state?
- Is it long-lived, not bound to the user, or forgeable (e.g., `remember=1` sent by the client)?

### 17. OAuth/Login-Path Bypass (rare)
```
Site.com requests OAuth token → provider verifies user → callback code → Site.com logs user in
```
If the OAuth callback marks the session authenticated *without* the 2FA step, log in via the social button instead of the password+OTP form.

### 18. Code Reusability / No Expiry
- Verify a code, log out, log in again, re-submit the *same* code — if accepted, OTPs are not one-time.
- Request a new code and test the previous one; request a reset and test old tokens.

## Bypass Techniques (quick list)

- Check fallback flows first: backup codes, "lost device" flows, security questions, support/social-engineering paths that skip the factor.
- Test every state transition for the "skip a step" pattern: direct forced browsing to the page that loads after verification.
- Race/duplicate submission: send the same verification request twice in parallel.
- Look for unauthenticated or differently-authenticated API/mobile variants of the verify endpoint lacking the checks.

## Tools & Resources

- Burp Suite (Repeater, Intruder, Match-and-Replace for response manipulation)
- [Testing Two-Factor Authentication – NCC Group](https://research.nccgroup.com/2021/06/10/testing-two-factor-authentication/)
- [2FA Bypass Mindmap – Harsh Bothra](https://www.mindmeister.com/1736437018?t=SEeZOmvt01)
- Example writeups: [Fani Malik 1](https://twitter.com/FaniMalikHack/status/1395042756108505092), [Fani Malik 2](https://twitter.com/FaniMalikHack/status/1402627994833805313)

## Report Tips

- Frame impact as "attacker with only the victim's password fully bypasses the mandatory second factor", i.e., 2FA provides no protection — usually P1/P2.
- Include a clean video POC logging in with a wrong/absent code and landing in the victim's session.
- If the bypass only works on one entry point (e.g., OAuth or reset), still report it — partial-factor enforcement is a real auth bypass.

## Credits

- Original lists: Harsh Bothra, Vishal Saini, Fani Malik ([@FaniMalikHack](https://twitter.com/FaniMalikHack)); merged & reformatted for HowToHunt by remonsec.
