# Password Reset Flaws — Expanded Catalog

Common security flaws in password reset functionality, compiled from Twitter, writeups, and disclosed reports. This file is the **index of flaw classes**; the full payloads and step-by-step exploitation for the core five live in `Top_5_Password_Reset_Bugs.md`, and adjacent behaviors are distributed across the files linked inline. Use it to plan one pass per flaw class.

## Hunting Checklist

1. Run `Top_5_Password_Reset_Bugs.md` items 1–5 first (they cover the highest-hit-rate flaws):
   - Token leak via `Referer`
   - Email array/param injection
   - Reset OTP brute force
   - Change-password API accepting another user's identity
   - Client-trusted error responses
2. Then sweep this catalog for the remaining classes below (enumeration, entropy, expiry, auth-scope, redirect, multi-step skipping).
3. For each attempt, record both the request and response — most reset findings are proven from the delta.

## Test Cases & Examples

### 1–5. Core Five
Full exploitation steps and payloads: see `Top_5_Password_Reset_Bugs.md` §§1–5 (same numbering). Quick recall of the *signal* to look for:
- #1: token appears in an outbound `Referer` header.
- #2: backend accepts `{"email_address":["victim","attacker"]}`.
- #3: OTP endpoint has no rate limit/lockout.
- #4: `/api/changepass` accepts `email` from the body.
- #5: SPA advances flow when the proxied response is rewritten to success.

### 6. Username/Account Enumeration via Reset
Compare responses of a reset request for an existing vs non-existing address:
```http
POST /api/forgot-password HTTP/1.1
Content-Type: application/json

{"email":"doesnotexist@example.com"}
```
```json
{"error":"No account found for this email"}
```
Distinct status/body/timing ⇒ valid-email list an attacker can feed into #3/#5 and credential attacks.

### 7. Token Predictability / Low Entropy
Collect several tokens for different accounts/times and look for structure:
```
2026-09-10 + userid + md5 → sequential, short, base32-only, or decodable JWT with alg=none
```
Predictable tokens make reset links stealable without any other bug (test against your own account only).

### 8. Token Sent to the Wrong/Requester-Controlled Address
Request a reset, then tamper for a second "to" address:
```http
POST /api/forgot-password HTTP/1.1
Content-Type: application/json

{"email":"victim@example.com","cc":"attacker@evil.com","forward_to":"attacker@evil.com"}
```
Same class: CRLF header injection in the email param (payloads in `Account_Takeover_By_Password_Reset_Functionality.md`).

### 9. Multi-Step Flow Skipping / Broken Object Reference
If step N+1 is a plain endpoint:
```http
GET /reset/step2?token=<yours> HTTP/1.1
```
- Try skipping earlier steps (visit final form directly).
- Swap `token`, `user_id`, `uuid` between two of *your* accounts to test object-level binding.

### 10. Redirect After Reset Not Validated / Token Forwarded
The reset endpoint completes then redirects; if the next URL is attacker-controlled, the token can be carried away:
```http
GET /reset/confirm?token=<token>&redirect=https://evil.com HTTP/1.1
```
Also test `?next=`, `?url=`, `?returnurl=` on every step.

### 11. HTTP Verb & Status Trust Variants
- Retry failed steps with `PUT/PATCH/DELETE` or `_method=` override.
- Rewrite `4xx` reset responses to `200 OK` for clients that check status only (pattern of #5).

## Bypass Techniques

- Token *expiry/invalidation* gaps (old token still valid after use, after re-request, after email change): procedures in `Broken_Auth_And_Session_Management/Session_based_bugs.md` §§6–8.
- Token *leakage* channels beyond referrer (analytics/3rd-party domains, API responses): `Password_Reset_Token_Leakage.md`.
- Host-header poisoning of emailed links: `Account_Takeovers_Methodologies/Account_Takeovers_Methods.md` §3.
- Email-param HPP/pipe/CRLF variants: `Account_Takeover_By_Password_Reset_Functionality.md`.

## Tools & Resources

- Burp Suite (Intruder, Repeater, Match-and-Replace), browser dev tools (Network) for third-party requests on the reset page.
- [Password reset flaws mindmap](https://twitter.com/N008x/status/1302515523557548032/photo/1) + [source blog](https://anugrahsr.github.io/posts/10-Password-reset-flaws/) — see `README.md` in this folder.

## Report Tips

- Work the catalog as a matrix (step × flaw class); note negative results — triage likes "tested, not present".
- Enumeration (#6) is usually P4 alone, but report it chained: enumeration + no rate limit (#3) = mass takeover path.
- Keep one PoC video per flaw class; don't mix token leakage and parameter pollution in the same demo.

## Credits

- [Sm4rty](https://twitter.com/Sm4rty_) — original flaw list; expanded catalog for HowToHunt.
