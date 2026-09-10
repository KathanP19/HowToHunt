# Broken Authentication & Session Management Bugs

Session-management bugs let credentials, sessions, tokens, or verification states survive or leak across lifecycle events (logout, password change, email change, reset). They rarely look critical alone, but each one is a bridge to Account Takeover when combined with XSS, phishing, or stolen devices. Typical impact: session hijacking, verification bypass, P3–P1 depending on the chain proven.

## Hunting Checklist

1. Run every sensitive action twice across two browsers/profiles and observe what the other session can still do after: password change, email change, logout, account delete, 2FA enable, reset completion.
2. Test logout semantics: paste the pre-logout cookies back after logging out (session invalidation), and use the Back button after logout (cache on authed pages).
3. For password-reset tokens: test single-use, expiry after successful reset, expiry when a new token is requested, and expiry after email change (see also `Password_Reset_Functionality/`).
4. For email-verification states: never click the first link; change email mid-flow and see which pending link verifies which address.
5. Replay captured authenticated requests *after logout* to test server-side auth checks on state changes.
6. Note every behavior where an unverified or lower-privilege state can reach privileged pages (forced browsing).

## Test Cases & Examples

### 1. Old Session Does Not Expire After Password Change
1. Create an account; log in in two browsers (Chrome + Firefox/incognito).
2. Change the password in Chrome.
3. Refresh the Firefox session — if it is still logged in, sessions are not invalidated on credential change (classic ATO persistence vector).

### 2. Session Hijacking via Non-Invalidated Cookies (Intended-Behavior Check)
1. Log in; copy all target cookies with a cookie-editor extension.
2. Log out.
3. Paste the old cookies back and refresh — if you are logged in again, old session cookies were never revoked; a stolen cookie survives logout.
```http
GET /dashboard HTTP/1.1
Host: target.com
Cookie: session=<copied_before_logout>
```

### 3. Cache Control Missing on Authed Pages
1. Log in, navigate through sensitive pages, then log out.
2. Press Back (`Alt + ←`): if pages render from cache, sensitive data is exposed to the next person at the machine (PC-cafe scenario).
- Root cause: authed responses lacking `Cache-Control: no-store`.

### 4. Email Verification Bypass via Round-Trip (P3/P4)
```
Category: Broken Authentication and Session Management →
Failure to Invalidate Session → On Password Reset and/or Change
```
1. Create an account; receive the verification email — **do not click it**.
2. Notice the app grants reduced privileges until verified.
3. Change the email to Email B, click the B verification link.
4. Change the email back to the original address.
5. The profile now shows "verified" even though the *original* signup link was never clicked → verification state survived email round-trips; attacker reaches full-privilege state without completing the intended check.

### 5. Email Verification Bypass via Stale Link (P3/P4)
1. Create an account with your own email; leave the verification link unclicked.
2. Change the account email to the victim's address.
3. Open **your own** original verification link.
4. If the victim's email is now marked verified, the pending token validated the *new* address — an attacker who controls an unverified signup can get arbitrary emails marked as verified.

### 6. Password-Reset Token Valid After Password Change
1. Request a reset link, use it, change the password, log in.
2. Re-open the **old** reset link and try to change the password again.
3. Success ⇒ used tokens are not single-use (token reuse ⇒ delayed ATO).

### 7. Old Reset Token Not Revoked When a New One Is Issued (sometimes P4)
1. Request reset link #1, then request link #2 without using #1.
2. Complete reset using link #1.
3. Success ⇒ old tokens are not invalidated on reissue. Note: some programs accept this as valid, others don't — report the reuse chain with an expiry claim.

### 8. Reset Token Survives Email Change (Insecure Configurability)
1. Request a forget-password token and **don't** open the link.
2. Log in and change the account email to another address.
3. Use the token that went to the **old** email — if it still resets the password, the pending token wasn't bound to the current address.

### 9. State Change Accepted After Logout (Insufficient Process Validation)
1. Create an account; open the profile edit page.
2. Fill name/email changes, enable Burp intercept on Save, capture the update request, send to Repeater.
3. **Log out** of the site.
4. Click "Go" in Repeater:
```http
POST /api/profile/update HTTP/1.1
Host: target.com
Content-Type: application/json
Cookie: session=<post_logout_copy>

{"name":"attacker","email":"attacker@evil.com"}
```
5. If it returns 200 and the change persists at next login, unauthenticated/stale sessions can still mutate account state.

## Bypass Techniques

- If direct logout replay is blocked, replay before the server sees logout (parallel/queued requests — race).
- For cached-session bugs, combine with `Clear-Site-Data`-less logout flows and shared machines as the impact path.
- Verification-state bugs often also reproduce via password-reset "skip email step" flows — try both orders (verify-then-change, change-then-verify).

## Tools & Resources

- Burp Suite (Repeater, Match-and-Replace), Cookie Editor / EditThisCookie, browser dev-tools Application tab for storage scope checks.
- OWASP Testing Guide: Session Management Testing (ST); PortSwigger Web Security Academy: "Testing other application defenses", "Session management".
- Related files: `Password_Reset_Functionality/`, `Account_Takeovers_Methodologies/Account_Takeovers_Methods.md`, `Sign_Up_Functionality/Hunting_for_bugs_in_signup_feature.md`.

## Report Tips

- Always finish with a demonstrated consequence: logged-in state from a revoked cookie, or an email marked verified that never was — not just "session still works".
- Map each finding to lifecycle event (logout / password change / email change / reset) and say which sessions should have died.
- Low-severity-looking ones (stale reset links) get better ratings when you chain them: "old token + no expiry = ATO window of N days".

## Credits

- [Virdoex_hunter](https://twitter.com/Virdoex_hunter), [Chirag Agrawal](https://www.linkedin.com/in/chirag-agrawal-770488144/) ([@Raiders](https://twitter.com/ChiragA15977205)), [Fani Malik](https://twitter.com/fanimalikhack), [Suprit Pandurangi](https://www.linkedin.com/in/suprit-pandurangi-a90526106/).
