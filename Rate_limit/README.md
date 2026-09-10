# Rate Limit Flaws

Missing or bypassable rate limits are leveraged by malicious actors to perform DDoS, brute-force, and bot attacks on APIs — and to burn money on email/SMS providers. Although it's more than that.

> **NOTE:** Some organisations keep plain rate-limit bugs as out-of-scope, so check their policy before testing.

## Contents

- [Missing Rate Limit Scenarios](Missing_Rate_Limit_Scenarios.md) — scenario playbook: forgot-password, invite user, verify-email, verify-phone (SMS), promo codes; reproduction steps, impact, mitigations.
- [Rate Limit Bypass](RateLimitBypass.md) — techniques to defeat implemented limits: HTTP methods, IP-spoofing headers, special characters/paths, body & version tricks, IP rotation, logic abuse, origin IP.
