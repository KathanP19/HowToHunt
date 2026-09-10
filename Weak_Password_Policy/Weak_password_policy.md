# Weak Password Policy

A weak password policy increases the probability of an attacker succeeding with brute-force and dictionary attacks against user accounts. An attacker who can determine user passwords can take over those accounts and access sensitive data in the application. Severity is typically medium, escalating to high when combined with a missing/weak login rate limit or when the policy applies to privileged/admin accounts.

## Hunting Checklist

1. Test the policy at **every** password-setting surface: account creation, password reset, and "change password" in account settings — applications usually have restrictions while creating an account, but you must check all these cases again when resetting/changing, since validation is often implemented inconsistently.
2. Check if you can use the password same as the Email Address.
3. Check if you can use the password same as the Username.
4. Check if you can use obvious weak passwords such as `123456`, `111111`, `abcabc`, `qwerty123`.
5. Test minimum length enforcement end-to-end (UI message vs. actual server acceptance).
6. Test whether complexity requirements are inverted: short-but-complex (`Aa1!`) accepted while long-but-weak (`passw0rdpassw0rd`) rejected — a policy that fails to block long weak passphrases is still weak.
7. Check whether the policy is client-side only: strip the validation JS (or replay the request directly in Burp) and see if the server still accepts the password.
8. Try the username/email-derived families: `<user>123`, `Welcome1`, company/product name + year, reversed username.
9. Test sequential/repeating patterns (`abcdef`, `aaaaaa`) and keyboard patterns (`qwer1234`) if the app claims to block them.
10. Check whether a leaked/breached-password check exists (e.g. reject of known-breached passwords) and whether default/first-login admin passwords are forced to change.

## Test Cases & Examples

### Server-side acceptance checks (repeat on signup / reset / change)

```http
POST /api/register HTTP/1.1
Host: target.com
Content-Type: application/json

{"email":"tester1@example.com","username":"tester1","password":"tester1@example.com"}
```

```http
POST /api/register HTTP/1.1
Host: target.com
Content-Type: application/json

{"email":"tester2@example.com","username":"tester2","password":"tester2"}
```

```http
POST /api/register HTTP/1.1
Host: target.com
Content-Type: application/json

{"email":"tester3@example.com","username":"tester3","password":"123456"}
```

```http
POST /api/reset-password HTTP/1.1
Host: target.com
Content-Type: application/json

{"token":"<reset_token>","password":"qwerty123"}
```

If any of these are accepted server-side, the policy is weak. The same payloads work through the UI — confirm by actually logging in with the weak password, because "the UI complained but the API stored it anyway" is a classic client-side-only validation finding.

### Bypassing a client-side-only length rule

```bash
# The form enforces minlength in JS; call the endpoint directly:
curl -sk 'https://target.com/api/change-password' \
  -H 'Content-Type: application/json' \
  -H "Cookie: session=<cookie>" \
  --data '{"current":"OldPass123!","new":"1234"}'
```

### Bulk check with a weak-password list

```bash
# Example: confirm several trivial passwords are accepted (low volumes, your own test accounts only)
for p in 123456 111111 abcabc qwerty123 password Password1; do
  n="$(tr -dc A-Za-z </dev/urandom | head -c6)"
  curl -sk -o /dev/null -w "%{http_code}  $p\n" 'https://target.com/api/register' \
    -H 'Content-Type: application/json' \
    --data "{\"email\":\"hunt+$n@example.com\",\"password\":\"$p\"}"
done
```

Use a curated weak-password wordlist (SecLists top-10k, or the public "best1050" style lists) for broader coverage of what the policy tolerates.

## Bypass Techniques

- **Validation present on one flow only:** the strongest finding is usually password *reset* or admin-invited account setup where signup rules don't apply.
- **Client-side rules:** edit/remove the JS check or bypass the browser entirely with Burp/replay.
- **Server-side regex gaps:** rules that only look at "has a digit and a symbol" allow `1111aaaa@#$`-style garbage.
- **Whitespace tricks:** test `" 123456"` or `"123456 "` acceptance where the login normalizes but the policy check doesn't (same class of input-handling gap as rate-limit `%00` tricks).

## Tools & Resources

- [SecLists](https://github.com/danielmiessler/SecLists) — `Passwords/` for common/weak password lists.
- Burp Suite Repeater/Intruder for driving each password-setting endpoint.
- Have I Been Pwned password range API concept — reference for what "block breached passwords" mitigation should look like.
- [OWASP ASVS - V2.1 Password Security](https://owasp.org/www-project-application-security-verification-standard/) — the yardstick for a strong policy.
- References:
  - [All About Weak Password Policy](http://applicationsecurity.io/appsec-findings-database/weak-password-policy/)
  - [OWASP Guide for Weak Passwords](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/07-Testing_for_Weak_Password_Policy)

## Report Tips

- Prove exploitability, not just policy text: "the account was created with `123456` and login with it succeeded" (note the flow used: signup vs. reset).
- State which flows enforce nothing — "password **reset** accepts passwords disallowed at signup" shows the inconsistency and is the more compelling fix argument.
- If you show offline brute-force relevance, pair with login rate limiting status: a weak policy *plus* no lockout = credential-stuffing/ATO path, which raises severity.
- Recommend concrete mitigation: minimum length aligned with OWASP guidance, blocklist of breached/common passwords, server-side enforcement on every flow.
