# Hunting for Bugs in the Sign-Up Feature

The registration flow is the one place where attackers get to be *unauthenticated yet write to the database* — creating users, picking usernames, injecting payloads before any session exists. Bugs here seed takeover, phishing, spam, and path-hijacking issues, so signup deserves its own mini-methodology. Typical impact spans stored XSS, account collision takeover, enumeration, and resource abuse.

Sign-up generally comes in two flavors: **Manual** (name/email/password forms) and **Social/OAuth**. This document focuses on manual signup; OAuth signup bugs are covered in `OAuth/`.

## Hunting Checklist

1. Enumerate every signup input (username, email, phone, name, DOB, password fields, free-text "how did you hear about us") and note where each value later renders.
2. Test uniqueness of identity fields: register with the same email/username/phone again; try case and whitespace variants (`Abc@gmail.com`, `abc @gmail.com`, unicode look-alikes).
3. Test duplicate-account *takeover*: after a second signup with the same email, log in with each credential set and see which one wins.
4. Fuzz length limits: submit very long strings (≈100k chars) in name/password to probe hashing/service DoS.
5. Inject HTML/JS payloads into every field and check reflection in-app **and in emails** (welcome mail, admin notifications).
6. Check rate limiting on the signup endpoint (mass fake-account creation).
7. Audit email verification: can you reach settings or privileged pages while unverified? Can a stale link verify a changed email? (Full matrix in `Broken_Auth_And_Session_Management/Session_based_bugs.md`.)
8. If profiles live at `target.tld/{username}`, register reserved/system filenames as usernames (path overwrite/hijack).
9. Inspect the signup request/response for extra parameters worth tampering with (role, verified, plan) and secrets leaked in responses.
10. Verify weak password policy + username enumeration feedback ("account exists", different error strings).

## Test Cases & Examples

### 1. Duplicate Registration / Overwrite Existing User
1. Create an account with `abc@gmail.com` + Password-1.
2. Log out and register again with `abc@gmail.com` + Password-2 (try case variants: `Abc@gmail.com`).
3. If creation succeeds, log in with each password — observe which account/identity is actually reached.
- Impact: two profiles sharing one identity, or silent overwrite of the existing user's credentials → ATO.
- Further read: [H1 #187714](https://hackerone.com/reports/187714), [Duplicate Registration: The Twinning Twins](https://shahjerry33.medium.com/duplicate-registration-the-twinning-twins-883dfee59eaf), [UAO via Signup](https://blog.securitybreached.org/2020/01/22/user-account-takeover-via-signup-feature-bug-bounty-poc/)

### 2. DoS on Name/Password Fields (Long-String)
A 100,000-character string can exhaust CPU/memory in a weak hashing implementation when the form is submitted.
1. Fill the signup form; paste a very long string into the password (and name) field.
2. Submit — a `500 Internal Server Error` (or multi-second hang) indicates the hash step is DoS-able.
```http
POST /signup HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

name=AAAA…(100000 chars)&email=x@x.io&password=BBBB…(100000 chars)
```
```bash
python3 -c "print('A'*100000)"   # generate the payload string
```
- Further read: [Long String DoS](https://shahjerry33.medium.com/long-string-dos-6ba8ceab3aa0), [H1 #738569](https://hackerone.com/reports/738569), [H1 #223854](https://hackerone.com/reports/223854)

### 3. (Stored) XSS in Registration Fields
Inject into username/email fields and watch where they render (profile pages, admin panels, emails):
```html
<svg/onload=confirm(1)>
```
```text
"><svg/onload=confirm(1)>"@x.y
```
- Further read: [H1 #196989](https://hackerone.com/reports/196989), [H1 #470206](https://hackerone.com/reports/470206), [H1 #119090](https://hackerone.com/reports/119090); also `HTML_Injection/HTML_Injection_on_password_reset_page.md`.

### 4. No Rate Limit at Signup
Mass fake accounts pollute the database, abuse trials/referral programs, and enable spam.
1. Capture the signup request, send to Intruder.
2. Payload a fresh email (use `payloads`/gagarin-style addresses) per request.
3. Fire — sustained `200 OK` creations = missing rate limit.
```http
POST /signup HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

email=fake+1@mailinator.io&username=user1&password=Str0ngPass!
```
- Further read: [H1 #905692](https://hackerone.com/reports/905692), [H1 #97609](https://hackerone.com/reports/97609), [H1 #262830](https://hackerone.com/reports/262830)

### 5. Insufficient Email Verification
Applications that don't verify the email (or weakly) can usually be bypassed via:
1. **Forced browsing** — navigate directly to post-verification pages/endpoints.
2. **Response/status-code manipulation** — flip a `403` "verify your email" response to `200`.
3. Stale-link reuse / email round-trip tricks (detailed in `Broken_Auth_And_Session_Management/Session_based_bugs.md` §4–5).

POC — verification bypass after signup:
1. Sign up as `attacker@mail.com`; receive the confirmation email, **don't** open the link.
2. Check whether the app still lets you reach the account-settings page while unverified.
3. In settings, change the email to `victim@mail.com`; you're asked to confirm the new address.
4. Instead of opening the *new* link, open the *old* confirmation link from the attacker inbox.
5. If that marks `victim@mail.com` as verified, that's an email-verification bypass.
- Further read: [H1 #1040047](https://hackerone.com/reports/1040047), [H1 #617896](https://hackerone.com/reports/617896), [H1 #737169](https://hackerone.com/reports/737169)

### 6. Path Overwrite / Profile Page Hijack
If profiles are served at `target.tld/{username}`, register system-reserved filenames:
```http
POST /signup HTTP/1.1

username=index.php
```
```text
https://target.tld/index.php     ← attacker's profile occupies the app page
https://target.tld/login.php     ← imagine the login page "taken over"
```
- Further read: [Logical Flaw Resulting in Path Hijacking](https://infosecwriteups.com/logical-flaw-resulting-path-hijacking-dd4d1e1e832f)

## Bypass Techniques

- Email normalization to duplicate accounts: `+`, dots, case, trailing whitespace, IDN homophones.
- If signup blocks a payload, try storage transforms: double URL-encode, overlong UTF-8, or split the payload across first/last name.
- Weak/missing password policy: pair with login rate-limit absence per `Account_Takeovers_Methodologies/Account_Takeovers_Methods.md` §2.

## Tools & Resources

- Burp Suite (Intruder + `payloads` lists), browser dev tools for reflection checks, mailinator/1secmail for throwaway inboxes.
- Repo images: `Sign_Up_Functionality/Signup_Mindmap.png`.
- Related: `XSS/Xss.md`, `Authentication_Bypass/OTP_Bypass.md`, `Weak_Password_Policy/Weak_password_policy.md`.

## Report Tips

- For duplicate registration, prove which login wins and what data of the first user is exposed — "registered twice" alone gets debated.
- Signup XSS is far stronger when shown in an **admin/victim** context (welcome email render, back-office user list) than self-XSS.
- Rate-limit findings need counts: "created 500 accounts in 4 minutes with no CAPTCHA/throttle/referral abuse".

## Credits

- [Kushagra Sarathe](https://twitter.com/kushagrasarathe) / ([GitHub](https://github.com/kushagrasarathe)), [Sm4rty](https://twitter.com/Sm4rty_) / ([LinkedIn](https://www.linkedin.com/in/sm4rty)); source tip: [tweet](https://twitter.com/kushagrasarathe/status/1385111472385060867).
