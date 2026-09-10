# 403 Forbidden Bypass

403/401 responses often come from a path-matching layer (WAF, nginx/Apache rule, framework filter) that parses the URI differently than the backend serving it — so "forbidden" endpoints like `/admin`, `/secret.txt`, or actuator/debug routes can still be reached with path, protocol, or header trickery. This class of bug is a direct improper-access-control finding and frequently leaks admin functionality or sensitive files.

Comprehensive header list for this attack family: [WAF_Bypass_Using_headers.md](../WAF_Bypasses/WAF_Bypass_Using_headers.md) · Index/mind maps: [README.md](./README.md)

## Hunting Checklist

1. Note *which layer* returns the 403 (WAF page vs. app page vs. server default) — headers/status/body shape tell you where to attack.
2. Path-shape tricks: for directories without trailing slash try `/*` and `/./`; for files try a trailing slash and encoded-separator prefixes.
3. Protocol trick: request the same path over `http://` when `https://` is blocked (and vice-versa).
4. Verb trick: retry the blocked path as `POST`, `HEAD`, `OPTIONS`, `PUT`, or `PATCH`.
5. Header tricks: add `X-Forwarded-For: 127.0.0.1` and friends (`Client-IP`, `X-Real-IP`, `X-Custom-IP-Authorization`) for IP-gated rules; `X-Original-Url`/`X-Rewrite-Url` for routing gaps (see Host-Header doc).
6. Case and encoding noise: `/Admin`, `/%61dmin`, `/admin.`, `/admin;`, `/..;/admin`, trailing `.`.
7. Automation: run 4-ZERO-3 / 403bypasser over your 403 list, then **verify each hit manually** — compare against a known-200 baseline page to be sure it's the real resource, not an error template.
8. Always keep a PoC screenshot/video: 403 bypasses live or die on reproducible before/after evidence.

## Test Cases & Examples

### Directory based (no trailing slash)

```text
site.com/secret     => 403
site.com/secret/*   => 200
site.com/secret/./  => 200
```

### File based (no trailing slash)

```text
site.com/secret.txt      => 403
site.com/secret.txt/     => 200
site.com/%2f/secret.txt/ => 200
```

### Protocol based

Sounds wired, but check the example:

```text
https://site.com/secret => 403
http://site.com/secret  => 200
```

### Path payload list

```text
/
/*
/%2f/
/./
./
/*/
```

Applied:

```http
GET /secret/* HTTP/1.1
Host: site.com
```

### Header-based (IP-restriction bypass)

Reference: [Bypassing 403 to access an admin console endpoint](https://observationsinsecurity.com/2020-08-09/bypassing-403-to-get-access-to-an-admin-console-endpoints/)

```http
GET /admin-console HTTP/1.1
Host: site.com
X-Forwarded-For: 127.0.0.1
```

### Method switch

```bash
curl -s -o /dev/null -w '%{http_code}\n' -X POST https://site.com/secret
```

## Bypass Techniques

- **Parser mismatch** (the whole game): WAF/proxy normalizes away `*`, `.` segments, `%2f`, `;` — the backend resolves them to the protected resource (all directory/file payloads above exploit this).
- **IP allowlist trust headers:** `X-Forwarded-For`, `X-Real-IP`, `Client-IP`, `X-Originating-IP`, `X-Remote-IP`, `X-Custom-IP-Authorization` — try them individually; one trusted hop can flip the rule (full list in the WAF headers doc).
- **Routing headers:** `X-Original-Url: /admin`, `X-Rewrite-Url: /admin/login` (Host-Header doc, item 9).
- **HTTP verb/protocol surface:** rules often bind to `GET` + `https` only; re-test with other methods/schemes.
- **Extension/content-type dressing:** `/admin.json`, `/admin.css`, `/admin..;/` confuse suffix-matching ACLs (framework-dependent).

## Tools & Resources

- [403bypasser](https://github.com/yunemse48/403bypasser) · [4-ZERO-3](https://github.com/Dheerajmadhukar/4-ZERO-3)
- Payload wordlists: PayloadsAllTheThings [Bypass 404 – 403 – 401](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Bypass%20404-403-401)
- Mind maps & Twitter tip galleries: [README.md](./README.md)
- PoC videos (403 & improper access control): YouTube [Mehedi Hasan Remon](https://www.youtube.com/channel/UCF_yxU7acxUojiGiOAMafQQ/videos?view_as=subscriber)
- Authors: **[@remonsec](https://twitter.com/remonsec)**, **[@KathanP19](https://twitter.com/KathanP19)** ("shared from personal experience — no official references for some tips")

## Report Tips

- Prove it's the *same protected resource*: show 403 baseline vs. bypassed 200 with identical content (hash/screenshot diff), plus one sensitive element (admin form, data field) to kill "custom error page" pushback.
- State the bypass class (path normalization, trust header, method) and the layer at fault — it maps to a one-line fix (normalization at WAF, exact-match allowlist) and reads as a proper access-control finding.
- If multiple protected endpoints are reachable via one rule gap, list them all with codes; breadth of access is the severity driver.
