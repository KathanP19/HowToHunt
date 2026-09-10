# Host Header Attacks

`Host` (and forwarding headers like `X-Forwarded-Host`) are used by apps, frameworks, and reverse proxies for routing, URL generation, cache keys, and virtual-host resolution — but the value is client-controlled. Mishandling it enables password-reset poisoning, web cache poisoning, cache deception, access-control bypass to forbidden paths, and even SSRF/SQLi/XSS in misconfigured stacks. Impact ranges from low (debug echo) to critical (account takeover via reset-link poisoning or poisoned cache of auth pages).

Closely related header-fuzzing set for reset poisoning: [WAF_Bypass_Using_headers.md](../WAF_Bypasses/WAF_Bypass_Using_headers.md)

## Hunting Checklist

1. Baseline: replay a normal request, note how `Host` appears (or not) in responses, generated links, and cache behavior.
2. Send two `Host:` headers (duplicate) — different backends pick first vs. last, revealing front-server/app mismatch.
3. Fuzz forwarding headers one at a time: `X-Original-Url`, `X-Forwarded-Server`, `X-Host`, `X-Forwarded-Host`, `X-Rewrite-Url` (full list in the companion doc).
4. Test domain-confusion values: `Host: redacted.com.evil.com`, `Host: evil.com/redacted.com`, `Host: example.com?.mavenlink.com`.
5. On AEM instances, if you see `/api.json`, try web cache poisoning via `Host` + `X-Forwarded-Server` + `X-Forwarded-Host`, and simply request `https://localhost/api.json`.
6. Look for reflection of the header in debug pages/errors — also try `Host: javascript:alert(1);` (some debug modes echo it → XSS).
7. Check whether values land in SQL queries (search errors) — Host-header SQLi happens in sloppy logging.
8. Try front-server restriction bypass on forbidden paths using `X-Rewrite-Url` / `X-Original-Url` path rewrites.
9. Pivot checks: reset-email link poisoning (send reset with poisoned header), cache poisoning (`Cache-Control` + `Vary` inspection), vhost enumeration.

## Test Cases & Examples

Duplicate `Host` headers (front server vs. backend parse differently):

```http
GET / HTTP/1.1
Host: target.com
Host: evil.com
```

Forwarding-header fuzz set:

```http
X-Original-Url: /admin
X-Forwarded-Server: evil.com
X-Host: evil.com
X-Forwarded-Host: evil.com
X-Rewrite-Url: /admin
```

Domain-confusion variants:

```http
Host: redacted.com.evil.com
```

```http
Host: evil.com/redacted.com
```

HackerOne reference: https://hackerone.com/reports/317476

```http
Host: example.com?.mavenlink.com
```

AEM `/api.json` cache-poison probe:

```http
GET /api.json HTTP/1.1
Host: localhost
X-Forwarded-Server: evil.com
X-Forwarded-Host: evil.com
```

Debug-mode XSS probe:

```http
Host: javascript:alert(1);
```

Reference: [XSS via Host header (debug echo)](https://blog.bentkowski.info/2015/04/xss-via-host-header-cse.html)

Host-header SQLi (watch for query errors/time-based behavior in logging): reference write-up: [bypassing Host header to SQLi](https://blog.usejournal.com/bugbounty-database-hacked-of-indias-popular-sports-company-bypassing-host-header-to-sql-7b9af997c610)

Front-server restriction bypass / forbidden path access:

```bash
curl -i -s -k -X 'GET' -H 'Host: <site>' -H 'X-rewrite-url: admin/login' 'https://<site>/'
```

## Bypass Techniques

- **Front server vs. app parsing gap:** `X-Rewrite-Url` / `X-Original-Url` make the WAF/proxy see a harmless path while the app serves the real one (step 9 above).
- **Header name/case variants and duplicates:** repeat a header, try `X-Fowarded-Host`-style typosquatting handled by permissive proxies, tab/space LWS padding.
- **URL-shape confusion:** `?`, `#`, `@`, `/` and encoded separators in the Host value slip past `endsWith` style domain checks (same logic as [CORS origin bypasses](../CORS/CORS_Bypasses.md)).
- **Cache validation gaps:** if the poisoned response is cacheable and `Vary` ignores `Host`/`X-Forwarded-Host`, the attacker URL persists for other users — that's full web cache poisoning (see PortSwigger research).
- Combine with password reset: one poisoned reset request swaps the link host in the victim's email → token theft → ATO (see [WAF_Bypass_Using_headers.md](../WAF_Bypasses/WAF_Bypass_Using_headers.md)).

## Tools & Resources

- Mind map: https://pbs.twimg.com/media/ET39wJOWoAAfTBb?format=jpg&name=small
- Burp Suite Repeater (raw header editing), Turbo Intruder for duplicate-header/cache tests
- References: hackerone.com/reports/317476 · PortSwigger "Web cache poisoning" research series · blog.bentkowski.info Host-header XSS post · usejournal Host→SQLi write-up
- Author: **[@KathanP19](https://twitter.com/KathanP19)**

## Report Tips

- Rank findings by what the header *changes*: reset-link host (ATO — critical), poisoned cache served to other users (high), auth-page access (high), reflected echo/debug (low–medium) — state which class you proved.
- Show before/after evidence: clean response vs. same request with the injected header (different link in email, 200 vs 403 path, cached artifact fetched anonymously).
- For cache poisoning include response headers (`age`, `cache-status`, `vary`) proving other users receive it; for path bypass include the exact curl/Repeater request verbatim.
