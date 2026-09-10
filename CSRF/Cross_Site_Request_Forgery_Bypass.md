# CSRF Bypass Cheat List

A field-tested checklist of ways to defeat CSRF protections when a state-changing endpoint *appears* token-protected. Each item corresponds to a common implementation mistake; full worked PoCs for the token/Referer scenarios live in [CSRF.md](CSRF.md).

**Where to look:** login, logout, password reset, change password, add-to-cart, like, comment, profile change, user details change, balance transfer, subscription, etc.

## Hunting Checklist

1. Pick a state-changing request and note where the CSRF token lives (body / query / header / cookie).
2. Replay the request through each quick test below until one is accepted.
3. If the token is accepted but tied to *your* session, move to the cookie-injection chains (CRLF/XSS) in CSRF.md.
4. If everything server-side holds, check the client side: is the endpoint's page clickjackable?
5. Confirm impact by performing the action on a secondary victim account.

## Test Cases & Quick Bypasses

```text
- Change request method [POST => GET]
- Remove the total token parameter
- Remove the token, and give a blank parameter
- Send null as the token value
- Copy an unused valid token by DROPPING the request, and use that token
- Use your own CSRF token to feed it to the victim (not tied to session)
- Replace the token value with one of the SAME LENGTH
- Change/delete the last or first character of the token
- Reverse-engineer the token (algorithm/pattern, forge a valid one)
- Extract the token via HTML injection
- Switch content type: from non-form `Content-Type: application/json` /
  `application/x-www-form-urlencoded` to `multipart/form-data`
- Change/remove the Referer ("Change referrer to Referrer")
- Bypass the Referer regex:
    if the site looks for "bank.com" in the referer URL, try
    "bank.com.attacker.com" or "attacker.com/bank.com"
- Remove the Referer header completely (see payload below)
- Clickjacking the same endpoint
```

Suppress the Referer header from your PoC page:

```html
<meta name="referrer" content="no-referrer">
```

Blank / null token forms:

```html
<form method="POST" action="https://target.com/email/change">
  <input type="hidden" name="email" value="pwned@evil-user.net">
  <input type="hidden" name="csrf" value="">
</form>
<script>document.forms[0].submit();</script>
```

### Clickjacking as a CSRF bypass

Exploiting clickjacking on the same endpoint **bypasses all CSRF protection**, because technically the request *is* originating from the legitimate site. If the page hosting the vulnerable endpoint is clickjackable, all CSRF defenses become irrelevant and you achieve the same result as a CSRF attack — with a bit more effort (framed page + transparent button overlay).

- OWASP background: [Clickjacking](https://owasp.org/www-community/attacks/Clickjacking)

## Tools & Resources

- [PortSwigger Web Security Academy — CSRF](https://portswigger.net/web-security/csrf) (worked labs per validation flaw)
- [Intro to CSRF](https://medium.com/swlh/intro-to-csrf-cross-site-request-forgery-9de669df03de)
- [Attacking sites using CSRF](https://medium.com/swlh/attacking-sites-using-csrf-ba79b45b6efe)
- [Bypassing CSRF protection](https://medium.com/swlh/bypassing-csrf-protection-c9b217175ee)
- Detailed methodology + PoCs: [CSRF.md](CSRF.md) · index: [README.md](README.md)

## Report Tips

- Pair each accepted bypass with the *specific* validation flaw name (presence-only check, not session-bound, length-only check, Referer regex, content-type tolerance) — helps devs fix the root cause.
- Even "self-XSS-ish" low-value actions (add-to-cart, like) stack up when chained to account-relevant endpoints; report the highest-impact action you reproduced.

**Authors:** [@SMHTahsin33](https://twitter.com/SMHTahsin33), [@Virdoex_hunter](https://twitter.com/Virdoex_hunter), [@remonsec](https://twitter.com/remonsec), [@tamimhasan404](https://twitter.com/tamimhasan404)
