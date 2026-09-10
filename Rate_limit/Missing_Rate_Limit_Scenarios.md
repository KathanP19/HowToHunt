# Missing / Inadequate Rate Limiting — Scenario Playbook

Rate limiting caps how often a user can repeat an action within a timeframe (logins, OTP checks, invites, code redemptions). When it is missing on user-triggered side effects, attackers can spam emails/SMS, brute-force valid codes, or enable credential stuffing and bot abuse against APIs. Impact is usually financial loss (per-message billing, exhausted provider quotas), service/mailbox disruption, coupon abuse, and user enumeration — severity typically medium, higher on payment or ATO-adjacent flows. Note that some bug bounty programs mark plain rate-limit findings out-of-scope, so check policy before testing.

## Hunting Checklist

Endpoints/flows to verify a rate limit exists on:

1. Forgot / reset password (email triggering).
2. Sign-up page.
3. Login page.
4. Invite user (plain, and re-triggering invites using Burp macros).
5. 2FA / OTP verification.
6. Verify-email / resend verification email.
7. Verify-phone-number / resend SMS code.
8. Promo / coupon / offer code redemption.
9. Comments and sent messages (spam vectors).
10. Use your own brain somewhere: any endpoint that spends money, sends a notification, or validates a secret.

Generic reproduction pattern:

1. Navigate to the feature and complete one legitimate action.
2. Intercept the request in Burp Suite.
3. Send to Intruder and clear § positions (repeat the request unchanged).
4. Set payload type to **Null payloads** with a count of ~100 and start the attack (tune threads; add a unique marker if the app de-duplicates).
5. Observe: do all requests succeed, or does the server throttle/block after N tries?
6. Check whether the limit is per-IP, per-account, per-email/phone, or per-session — then confirm it actually stops the abuse (see RateLimitBypass.md for defeating weak implementations).
7. Quantify the abuse: count of delivered messages, codes guessed, money moved per unit time.

## Test Cases & Examples

### Scenario 1 — No rate limit on forgot/reset password (email triggering)

1. Navigate to `https://abc.target.com/forgot-password` (it could also be `https://abc.target.com/reset-password`).
2. Enter the victim's email.
3. Intercept the request in Burp Suite.
4. Send to Intruder and clear payload positions.
5. Use Null payloads and set the payload count to 100.
6. Start attack.

```http
POST /forgot-password HTTP/1.1
Host: abc.target.com
Content-Type: application/x-www-form-urlencoded

email=victim@example.com
```

### Scenario 2 — No rate limit on invite user (email triggering)

1. Go to `https://target.com/` and navigate to the **Invite User** option.
2. Enter the victim's email and send the invite.
3. Intercept the request.
4. Send to Intruder and clear payload positions.
5. Set payload type to Null payloads, payload count to 100.
6. Click Start attack after applying the threads.
7. The victim receives a huge number of invite emails.

```http
POST /api/invite HTTP/1.1
Host: target.com
Content-Type: application/json

{"email":"victim@example.com","role":"member"}
```

Variant: rotate the **victim email** instead (Null payloads = spam one user; list payloads = spam-anything/enumeration), and use Burp macros to re-fire while a session or CSRF token expires.

### Scenario 3 — No rate limit on verify-email (unlimited email triggering)

1. Navigate to `https://abc.target.com/verify-email`.
2. Intercept the request in BurpSuite.
3. Send to Intruder and clear the payload positions.
4. Use Null payloads as the payload type and set the payload count to 100.
5. Start attack.

```http
POST /verify-email HTTP/1.1
Host: abc.target.com
Content-Type: application/json

{"email":"victim@example.com","resend":true}
```

### Scenario 4 — No rate limit on verify phone number (unlimited SMS triggering)

1. Open `https://target.com/phone-number-verify`.
2. Enter the victim's mobile number.
3. Intercept the request and send it to Intruder.
4. Use payload type NULL payloads, set the payload count, and start the attack.

```http
POST /phone-number-verify HTTP/1.1
Host: target.com
Content-Type: application/json

{"phone":"+15550001234","action":"send_otp"}
```

### Scenario 5 — No rate limit on promo/coupon code endpoint

1. Go to `https://abc.target.com/product/121/checkout/promo`.
2. Navigate to the **Offer/Promo/Coupon code** field.
3. Enter a random value.
4. Intercept the request and send it to Intruder.
5. Apply payload (e.g. Numeric generator or a coupon-pattern list like `SUMMER{4}`, `SAVE{5}-{3}`) and Start attack.

```http
POST /api/checkout/apply-promo HTTP/1.1
Host: abc.target.com
Content-Type: application/json

{"product_id":"121","code":"SAVE10"}
```

Watch responses for the difference between "invalid code" and "code valid but expired/already-used" — that is a valid-code oracle you can grind without limits.

### Impact (email/SMS triggering scenarios)

If the company uses an email/SMS service API (such as AWS, GCP, etc.) or a purchased tool for the emails sent on the support domain, the missing rate limit results in **financial loss** and can **slow down services**: huge/mass mails lead to disruption of data that original users might send, or the purchased quota may be exhausted. For promo codes: **financial loss** — an attacker can easily brute-force all promo/coupon/offer codes.

### Mitigation (standard recommendations)

1. IP-based blocking/throttling.
2. Captcha (proof-of-work) on the trigger.
3. Firewall / WAF request limits.
4. Reducing the number of allowed API requests (per-IP **and** per-account/per-target-email limits).

## Bypass Techniques

A limit that "exists" is often trivially bypassed — always confirm whether it survives the tricks in [RateLimitBypass.md](RateLimitBypass.md) (IP-spoofing headers, method changes, trailing slashes, null bytes, HTTP parameter pollution, body format switching, API version fallback). For example, test whether a per-IP limit resets when the same email is submitted through different header values, or whether an aliased/param-polluted email value (`victim+1@x.com`, `victim@x.com%00`) circumvents per-account counters.

## Tools & Resources

- Burp Suite Intruder — Null payloads (clusterbomb/nulldit to repeat requests unchanged), Numeric/Regex payloads for code brute force.
- Burp macros — keep sessions/tokens valid across long attacks (invite-user macro check in the program scope).
- Turbo Intruder (PortSwigger BApp) — high-throughput confirmation that no limit exists.
- OWASP [Testing for Rate Limiting](https://owasp.org/www-project-web-security-testing-guide/) guidance in the session management / API chapters.
- Companion doc: [RateLimitBypass.md](RateLimitBypass.md) — techniques for defeating implemented limits.

## Report Tips

- Show raw evidence: "100/100 requests succeeded with 200 + delivered email/SMS, zero throttling" with a trimmed Intruder screenshot/log — volume + success rate is the finding.
- Monetize the impact: name the provider class (SES/Twilio/AWS/GCP-style billing), per-message cost, and messages/hour achievable; note that an attacker can also spend the org's quota to deny legitimate notifications (mail/SMS DoS).
- For promo brute force, redeem one real discovered code to a small amount (or show a valid-code oracle response pair) instead of a raw wordlist dump — demonstrated abuse beats theoretical.
- If enumeration is possible via different responses (registered vs unregistered email/phone), state it as an account-enumeration impact alongside the spam.
