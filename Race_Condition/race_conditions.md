# Race Conditions

Race conditions occur when a website processes requests concurrently without proper verification, so multiple threads interact with the same data at the same time and "collide," producing unintended behavior. They are closely related to [business logic flaws](https://portswigger.net/web-security/logic-flaws) and matter because they break the assumptions behind limits, one-time actions, and uniqueness guarantees. Typical impact: limit overruns (invites, votes, redemptions), duplicated payments, account takeover via email-change races, and rate-limit bypass.

## Hunting Checklist

1. Inventory every feature with a one-time or capped effect: gift cards, coupons, promo codes, referral/invite limits, withdrawals, free trials, CTF flag submission, likes/follows/subscribes/votes, group joins, email/phone change, password reset.
2. For each, ask: does the check happen before the action is committed, and can two requests pass the check before either write lands?
3. Single-request race: send N copies of the same request simultaneously (Burp Repeater → create a tab group → **Send Parallel (single-packet attack)**; requires Burp 2023.9.x+, or write your own script).
4. Multi-endpoint race: chain the whole user flow (e.g., `/product` → `/cart` → `/cart/checkout`) as parallel tabs and fire together.
5. Second-order race: if functionality requires email verification and a new value overwrites the stored old one in the DB *before* confirmation, race the two overwriting requests.
6. Time-sensitive race: fire two related requests (e.g., two forgot-password requests) and analyze whether tokens/state collide or response times overlap.
7. For rate-limit bypass, use Turbo Intruder with a single-packet HTTP/2 engine rather than raw parallelism.
8. Repeat attempts many times — races are probabilistic; one failure is not evidence of absence.
9. Compare server-side effects (not just HTTP status): re-fetch balances/limits/member lists to confirm the double-application.

## Test Cases & Examples

### Limit-overrun race (logic flaw)

Targets: invite user, joining a group, like/subscribe/follow/unfollow/vote — anything with a required limit.

Example single-action request to replay in parallel:

```http
POST /api/coupons/apply HTTP/1.1
Host: example.com
Content-Type: application/json
Cookie: session=<attacker-session>

{"code":"SAVE10"}
```

Method (easiest — Burp 2023.9.x or higher; you can also create your own script):

1. Send the request to Repeater N times.
2. Create a tab group for all those requests and choose **Send Parallel (single-packet attack)**.
3. Hit send — if the application is vulnerable, you'll see the magic (limit applied once, action executed N times).

### Rate-limit bypass via RC (Turbo Intruder)

1. Select the parameter to brute-force (e.g., password) and send the request into Turbo Intruder.
2. Copy your wordlist to the clipboard and use the script below:

```python
def queueRequests(target, wordlists):

    # as the target supports HTTP/2, use engine=Engine.BURP2 and concurrentConnections=1 for a single-packet attack
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )

    # assign the list of candidate passwords from your clipboard
    passwords = wordlists.clipboard

    # queue a login request using each password from the wordlist
    # the 'gate' argument withholds the final part of each request until engine.openGate() is invoked
    for password in passwords:
        engine.queue(target.req, password, gate='1')

    # once every request has been queued
    # invoke engine.openGate() to send all requests in the given gate simultaneously
    engine.openGate('1')

def handleResponse(req, interesting):
    table.add(req)
```

3. Hit attack and see the magic.

### Multi-endpoint race conditions

When one functionality chains multiple requests, e.g., buying a product on an e-commerce app:

```http
POST /product HTTP/1.1
Host: example.com
```

```http
POST /cart HTTP/1.1
Host: example.com
```

```http
POST /cart/checkout HTTP/1.1
Host: example.com
```

1. Send all the required requests into Burp Repeater for the target product, in sequence, and create a tab group.
2. Select **Send Parallel (single-packet attack)** and hit send — one checkout may satisfy N orders (e.g., zero-discount applied N times, or stock/limit checked once).

### Single-endpoint race condition (email change → ATO)

If a functionality edits an older object and requires email verification, test there (e.g., email change):

- Account A (attacker): `attacker@email.com`
- Account B (victim): `victim@email.com`

1. The app's email-change feature updates the requested email over the old one in the DB and sends a confirmation link to the *stored* address.
2. Fire a **Send Parallel (single-packet attack)** of the change-email request for both `attacker@email.com` and `victim@email.com`.
3. Because the requests arrive faster than the server processes them, when the app generates the confirmation for `attacker@email.com` the `victim@email.com` request is already mid-handling — the app gets confused about which to prioritize and sends both confirmation links to the same email.
4. Impact: this can lead to full account takeover.

### Time-sensitive vulnerability (forgot-password)

1. Send two parallel forgot-password requests for the attacker's account (Account A).
2. If both password-reset links contain the **same token**, test for ATO.
3. Send both requests again, changing the victim's username/email in one of them.
4. Analyze response times: if both requests have the same response time, there may be a chance of ATO.

## Bypass Techniques

- **Latency kills races**: prefer HTTP/2 single-packet attack (all requests share one connection, last byte flushed at once via gate) — Turbo Intruder's `Engine.BURP2` with `concurrentConnections=1`.
- **Distribute across worker threads**: try many parallel copies and vary counts (5/10/20/50) to find the window where two handlers overlap.
- **Strip noise**: remove browser-injected headers/CSRF churn only if the endpoint tolerates it; keep sessions valid so you don't lose the race to an auth check.
- **Unique-key extraction**: if the app rejects duplicate payloads, vary non-semantic fields (whitespace, param order) so WAFs/parsers treat requests as distinct while the logic still collides.
- **H2 vs H1 behavior differs** — test the same endpoint on both protocol versions.

## Tools & Resources

- Burp Repeater group + Send Parallel (single-packet attack) — Burp 2023.9.x+
- Turbo Intruder (BApp store) — https://github.com/PortSwigger/turbo-intruder
- PortSwigger race condition research/labs: https://portswigger.net/web-security/race-conditions
- Real-world reports:
  - Race condition in flag submission (H1): https://hackerone.com/reports/454949 — same CTF flag submitted multiple times, inflating points.
  - Race condition on invite-user (Omise): https://hackerone.com/reports/1285538 — invite the same member multiple times to one team.
  - Duplicated retest payments (H1): https://hackerone.com/reports/429026 — multiple simultaneous confirm-requests paid the retester several times.
  - Un-deletable group member (H1): https://hackerone.com/reports/604534 — user couldn't be removed by admin after a joined-in-race state.
  - Following a user twice (every.org): https://hackerone.com/reports/927384 — follow the same user multiple times instead of getting an error.
  - Popular reports feature (H1): https://hackerone.com/reports/146845 — up/downvote a report multiple times, boosting its rank.
  - CTF group join (H1): https://hackerone.com/reports/1540969 — join the same CTF group multiple times; duplicate member entries.
  - Invitation-limit bypass (Keybase): https://hackerone.com/reports/115007 — 7 invites sent despite a 3-invite limit.
  - Gift-card multi-redemption (Reverb): https://hackerone.com/reports/759247 — redeem one gift card repeatedly to buy multiple times.

## Report Tips

- Prove server-side state changed twice (before/after screenshots of balance, counter, member list) — not just 2× HTTP 200.
- Quantify the business impact: money duplicated, invites/gifts over-issued, or ATO path — financial/coupon/gift-card races are usually high/critical.
- State the concurrency primitive used (Repeater parallel / Turbo Intruder gate, request count, HTTP/2) so the team can reproduce reliably.
