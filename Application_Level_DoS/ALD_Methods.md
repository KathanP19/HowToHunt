# Application-Level Denial of Service (ALDoS)

Application-Level DoS is a logic-level bug class: instead of flooding bandwidth, the attacker drives cheap, feature-level actions that consume disproportionate server resources (hashing, lookups, email provider reputation) or lock legitimate users out. It matters because it survives infrastructure DDoS protection and can brick real business functions (no emails sent, login broken, endpoints hanging). Severity usually ranges from medium (temporary self/limited DoS) down to low, unless permanent or affecting the whole user base.

## Hunting Checklist

1. **Email bounce DoS**: find invite / "send email to" functionality; submit invalid, non-existent, or typo'd addresses; check the mail headers to identify the Email Service Provider (AWS SES, HubSpot, Campaign Monitor, ...).
2. Look up that provider's hard-bounce threshold; repeatedly trigger bounces *within scope* and confirm the company risks being blocked by the ESP.
3. **Long password DoS**: registration usually restricts password length — check Forgot-Password and Change-Password (authenticated) fields, where the limit is often missing.
4. Submit a ~150–200 character password to detect the absence of a length restriction; if none, go much longer and watch response time / app stability.
5. Use the repo's `Password.txt` sample as the oversized input (see Tools) — ⚠️ do not use more than 5000 characters as a password during testing.
6. **Long string DoS**: put very long strings (e.g., 1000+ characters) into stored fields (username, address, profile picture filename) and trigger expensive consumers of that data (search user A from account B).
7. **Account-lockout abuse**: test whether repeated wrong logins lock accounts, how long the lock lasts, and whether old sessions are invalidated.
8. Observe measurable degradation (timeouts, 500s, lockouts) — a slower-but-fine response is not a finding.

## Test Cases & Examples

### 1. Email bounce DoS

```http
POST /api/invites HTTP/1.1
Host: example.com
Content-Type: application/json

{"email":"nonexistent-random-1a2b3@example.tld","role":"member"}
```

- Send invites to invalid email accounts via the invite functionality.
- Identify the ESP from email headers.
- Known hard-bounce tolerances: HubSpot's hard-bounce limit is 5% (many ISPs prefer <2%); AWS SES escalates from 2–5% up to 5–10%.

**Impact**: once hard-bounce limits are reached, the ESP blocks the company — no more emails reach its users.

### 2. Long password DoS

Passwords are hashed before being stored; hashing cost grows with input length, and unlimited-length fields let one cheap request burn CPU.

```http
POST /api/change-password HTTP/1.1
Host: example.com
Content-Type: application/json

{"current_password":"OldPass1!","new_password":"<200,000 x 'A'>"}
```

```bash
# generate the long input
python3 -c "print('A'*200000)" > long.txt
```

Where to test: Forgot-Password and authenticated Change-Password endpoints (Registration is usually restricted but the others usually aren't). Watch for multi-second responses or the application hanging/crashing briefly.

### 3. Long string DoS

```bash
python3 -c "print('A'*1000)"   # then use as username/address/filename value
```

Set a 1000+ character string in a stored field (username, address, or profile picture name), then trigger its consumer — e.g., search account A's long username from account B's session. Either the search spins for a long time or the application crashes (500 error).

### 4. Account lockout — permanent DoS to a victim

This is not classic ALDoS but a permanent DoS against one victim, abusing the lockout feature itself:

1. On `example.com` login, enter a valid account email + wrong password 10–20 times (Burp Repeater/Intruder).
2. If the account gets blocked, measure the blocking period; > 30 minutes is reportable.
3. Confirm no CAPTCHA is present (otherwise the attack can't be automated/looped).
4. Check whether old sessions remain valid after the lock.

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

email=victim@example.com&password=wrongpass
```

## Bypass Techniques

- Length/character restrictions on one field rarely apply to its siblings (reset vs change vs register vs username/address fields) — move laterally across the same value's endpoints.
- If inputs are trimmed server-side, pad with data the consumer must still process (multibyte characters in string-search DoS) rather than trivial repeated spaces.
- Lockout loops: rotate source IPs only if the lock is IP-independent (account-based) — test both single-IP and distributed behavior.

## Tools & Resources

- `Password.txt` (long-password sample): bundled in this repo at [Application_Level_DoS/Password.txt](./Password.txt) / https://raw.githubusercontent.com/KathanP19/HowToHunt/master/Application_Level_DoS/Password.txt
- Burp Repeater / Intruder for looping lockout and long-password requests
- References:
  - Email Bounce: https://medium.com/bugbountywriteup/an-unexpected-bounty-email-bounce-issues-b9f24a35eb68
  - Long Password DoS: https://www.acunetix.com/vulnerabilities/web/long-password-denial-of-service/ , https://hackerone.com/reports/738569 , https://hackerone.com/reports/167351
  - Long String DoS: https://medium.com/@shahjerry33/long-string-dos-6ba8ceab3aa0 , https://hackerone.com/reports/764434
  - Lockout DoS: https://youtu.be/5drIMXCQuNw

## Report Tips

- Permanent victim lockout after N failed attempts = P2; temporary lockout = P3/P4. Frame it as "an attacker can permanently block any user account by looping this request with small intervals."
- ESP bounce findings should name the provider and its threshold to show the outage is real, not theoretical.
- For long-password DoS, attach timing deltas (normal vs oversized input) as proof of resource consumption; keep testing at low volume.

---
*Authors: [Keshav Malik](https://twitter.com/g0t_rOoT_), [Fani Malik](https://twitter.com/fanimalikhack)*
