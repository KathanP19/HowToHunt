# Drupal

Drupal powers many government, education, and enterprise sites behind distinctive URL conventions (`/node/`, `/user/`, `/entity/`, `/core/`). Its default install quietly exposes enumeration primitives — content IDs, usernames, installed modules, version markers — that chain into recon for stronger bugs, and unpatched or misconfigured instances (open redirects via `destination`, exposed REST/JSON:API, public `files/` directories) are frequent reportable findings. Severity is usually Low–Medium for disclosure, up to Critical for unpatched core/plugin RCE.

## Hunting Checklist

1. **Detect Drupal:** Wappalyzer/whatweb; manual tells — `/core/misc/drupal.js` (D8+) or `/misc/drupal.js` (D7), `/sites/default/files/`, the `X-Generator` header (if not removed), `/user/login` markup, and `/.htaccess` (contains Drupal-version-specific rewrite rules).
2. **Pin the major version** before testing anything version-bound (CHANGELOG.txt on D7, `/core/misc/drupal.js` markers, error page paths).
3. **Fuzz content nodes** for hidden/dev pages not linked anywhere (core quick win — see Test Case 1).
4. **Enumerate users** via `/user/<n>` and compare anonymous views.
5. **Enumerate installed modules/themes** from `/modules/`, `/themes/`, `/profiles/` paths (see Tools), then check each contrib module's published advisories for the detected core version — verify, never guess a CVE from memory.
6. **Test the `destination` parameter** on `/user/login` / `/user/logout` for open redirects.
7. **Check for exposed APIs:** `/entity/node?_format=json`, `/jsonapi/node/article`, `/rest/...` readable anonymously when Web Services/JSON:API are misconfigured.
8. **Check file exposure:** browsing `/sites/default/files/` and generated styles directories, backup dumps left by migration modules.
9. Note historical context: Drupal core has had critical public RCEs (e.g. Drupalgeddon2, CVE-2018-7600, affecting old 7.x/8.x) — only relevant if version checks prove the site is that old and unpatched.

## Test Cases & Examples

### 1. Node fuzzing (hidden pages)

Original tip: if you hunt on a Drupal website, fuzz with Intruder on `/node/$` where `$` is a number (1 to 500 for example) — you could find hidden pages (test, dev) which are not referenced by the search engines. ([@adrien_jeanneau](https://twitter.com/adrien_jeanneau/status/1273952564430725123?t=SUinUf09jxjRXu1yF9AQDg&s=19))

```http
GET /node/1 HTTP/1.1
Host: target.com
```

```bash
ffuf -u 'https://target.com/node/FUZZ' -w <(seq 1 500) -mc 200,403
```

Compare `403` vs `404` — a 403 on a node means the content exists but is hidden: check `/node/<id>/edit` and preview routes.

### 2. User & entity enumeration

```http
GET /user/1 HTTP/1.1
Host: target.com

GET /entity/node?_format=json HTTP/1.1
Host: target.com

GET /jsonapi/node/article HTTP/1.1
Host: target.com
```

### 3. Open redirect via destination

```http
HTTP/1.1 GET /user/logout?destination=https://evil.example
Host: target.com
```

Also try on `/user/login?destination=` and `/node/1?destination=`.

### 4. Version/config markers

```http
GET /CHANGELOG.txt HTTP/1.1
Host: target.com

GET /.htaccess HTTP/1.1
Host: target.com
```

## Bypass Techniques

- Trailing-slash and case variants (`/Node/1`, `/node/1/`) to slip naive ACL/filter checks; `?q=node/1` on D7 when clean URLs are filtered.
- `%20` / `//` prefix noise on entity routes (`/%20entity/node?_format=json`).
- When `/jsonapi` is blocked at the edge, request with `Accept: application/vnd.api+json` on `/index.php/jsonapi/...` or `?_format=json` overrides.
- `X-Forwarded-For`/`X-Original-URL` style tricks for geo/IP-blocked `/admin/...` routes (shared misconfig pattern — see Default Credentials guide).

## Tools & Resources

- [droopescan](https://github.com/droopescan/droopescan) — fingerprint + enumerate modules/themes/users/nodes
- whatweb / Wappalyzer; nuclei Drupal templates; Burp Intruder for `/node/$` fuzzing
- [Drupal security advisories](https://www.drupal.org/security/advisories) — the authoritative place to verify any module/core issue before testing it

## Report Tips

- Hidden dev/test nodes found by fuzzing are Low/info disclosure — sell impact only if they leak internal data, credentials, or unreleased features (show the content, not just the 200).
- For exposed JSON:API/REST read access, include the smallest record that proves anonymous reads and the number of accessible entities.
- Never fire an old Drupalgeddon-style payload at a live site: prove version + missing patch metadata, or report "unpatched core" with advisory reference and let maintainers confirm.
