# Account Ban / Suspension Bypass

Ban/suspension enforcement is usually implemented per-entry-point, so a "banned" identity often still works somewhere else (another subdomain, API, or flow). It matters because it lets a blocked or permanently-banned attacker keep accessing the platform — a broken access-control / auth issue. Impact ranges from ban evasion (P4/P3) to full auth-state bypass when session issuance is involved.

## Hunting Checklist

1. Get banned on a test account (or use a program-acknowledged fake ban via support) — never ban a real user.
2. Enumerate every surface that consumes the same credential store: web app, `forms.`/`careers.`/`api.` subdomains, mobile API endpoints, admin panels, SSO.
3. Try logging in with the banned account on each surface and compare enforcement.
4. Check where enforcement lives: client-side (response manipulation possible) vs server-side vs gateway/WAF only.
5. Test the bypass flows around the ban: password reset, re-verification email, OAuth/social login for the same email, session issued *before* the ban, ban expiry via cached responses.
6. Test scope: is the ban per-service or global across the SSO estate?

## Test Cases & Examples

### 1. Same Account, Different Subdomain (original finding)
```
If you got banned from xyz.com, check other domains that use the same
account (forms, status pages, dashboards). You may be able to log in
from forms.xyz.com even though the main site blocks you.
(Found on a HackerOne private program.)
```
```http
POST /login HTTP/1.1
Host: forms.xyz.com
Content-Type: application/x-www-form-urlencoded

email=banned%40example.com&password=Passw0rd!
```

### 2. Enumerate the Shared-Credential Estate
```bash
subfinder -d xyz.com -silent | httpx -status-code -title
```
Then run the banned login attempt against each hit.

### 3. API / Mobile Endpoint Inconsistency
Web UI blocks the ban flag but the underlying auth API doesn't:
```http
POST /api/v1/auth/login HTTP/1.1
Host: xyz.com
Content-Type: application/json

{"email":"banned@example.com","password":"Passw0rd!"}
```
If a token/session is returned, the ban is cosmetic only.

### 4. Client-Side Ban Check → Response Manipulation
If the "account suspended" state is decided from an API response the frontend trusts, intercept the login response and flip the status field:
```json
{"status":"banned","access":false}   →   {"status":"active","access":true}
```
```http
HTTP/1.1 403 Forbidden    →    HTTP/1.1 200 OK
```

### 5. Pre-Ban Session Persistence
1. Log in normally in Browser A, then trigger the ban from Browser B.
2. Refresh Browser A — if the session is not invalidated, the ban never applies to active sessions.

### 6. Ban Bypass via Alternative Auth Flows
- Password-reset a banned account and log in through the reset link.
- Sign in with OAuth/social login for the banned email — some flows skip the ban-state check.
- "Reactivate/verify account" emails or appeal links that directly restore access.

## Bypass Techniques

- Compare enforcement matrix: main app vs subdomains vs API vs mobile endpoints vs GraphQL — bans are commonly set on only one path.
- Check case/normalization: `Banned@Example.com` vs the stored banned email may slip past exact-match blocklists.
- Cached pages: CDN/browser cache may still serve an authenticated page after ban or logout (no `Cache-Control` on authed responses).

## Tools & Resources

- Burp Suite (Repeater, Match-and-Replace), subfinder/amass/httpx for surface enumeration.
- Related: `Status_Code_Bypass/403Bypass.md`, `Broken_Auth_And_Session_Management/Session_based_bugs.md`.

## Report Tips

- Show both sides: same credentials blocked on the main app, accepted on the other surface — that contrast is the bug.
- Clarify you used a self-owned test account for the ban.
- If a session survives the ban, state persistence impact: banned users retain access indefinitely.
