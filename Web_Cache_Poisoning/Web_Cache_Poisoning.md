# Web Cache Poisoning and Web Cache Deception

## What is a Web Cache?

A web cache sits between the user and the origin server (for example a CDN like Cloudflare, Akamai, Fastly, or a reverse proxy like Varnish / Nginx). When a response is cacheable, the cache stores it and serves the **same stored response** to other users, without asking the origin server again.

The cache decides whether two requests are "the same" using a **cache key** — usually the URL path + query string, and sometimes a few selected headers. Anything that is **not** part of the cache key but still changes the response is dangerous. That gap is the root cause of both attacks in this guide.

---

## Poisoning vs Deception — What is the Difference?

| | Web Cache Poisoning | Web Cache Deception |
|---|---|---|
| **Idea** | Attacker puts a *malicious* response into the cache | Attacker tricks the cache into storing a *victim's sensitive* response |
| **Who is harmed** | Every user who later requests the same URL | The victim whose private page got cached |
| **Attacker needs an account?** | Usually no | Yes — the victim must be logged in |
| **Typical result** | Stored XSS, redirect to phishing site, defacement | Account data leak (profile, API keys, tokens) readable by anyone |
| **Root cause** | Unkeyed input (header/parameter) reflected in the response | Path confusion — cache and origin disagree on what the URL means |

---

## Impact

1. **Stored XSS for every visitor** — one poisoned response is served to all users of that URL, no victim interaction needed.
2. **Open redirect / phishing** — poison the cached `Location` header or a script URL and send real users to an attacker site.
3. **Sensitive data exposure (deception)** — a logged-in user's `/account` page is cached and then readable by an unauthenticated attacker.
4. **DoS** — poison a page or a JS file with an error response and break it for everyone until the cache expires.

---

## Step 0: Test Safely (Important)

Cache bugs affect **other real users**, so always test like this:

- Work on a **unique, unpopular URL** of your own (your profile page, a blog post nobody visits) — never the homepage of a live site.
- Always add a **cache buster** while experimenting, e.g. `?cb=axe12345`, and use a harmless unique marker (like the string `axepoison123`) instead of a real payload, until you have confirmed the behaviour.
- Confirm poisoning with a **second clean request** (new browser / curl, no special header). If the marker is still there, it was served from cache.
- Stop once you have proof for your report. Do not leave a real malicious payload sitting in a production cache.

---

## Step 1: Find Out If the Page Is Cached

Send the same request twice and compare these response headers:

```
Cache-Control: public, max-age=3600     -> cacheable
Age: 142                                -> seconds since the response was cached
X-Cache: HIT  /  CF-Cache-Status: HIT   -> served from cache (MISS = from origin)
Vary: Accept-Encoding, User-Agent       -> these headers ARE part of the cache key
```

Quick check with curl:

```bash
curl -s -D - https://example.com/page?cb=test1 -o /dev/null
curl -s -D - https://example.com/page?cb=test1 -o /dev/null
# If the 2nd response shows Age > 0 or X-Cache: HIT, the page is cached.
```

Good hunting grounds: homepages, blog posts, product pages, login pages, and shared JS/CSS files — anything public and frequently visited.

---

## Step 2: Web Cache Poisoning — Find Unkeyed Inputs

An **unkeyed input** is anything the cache ignores when building the key, but the origin server uses when building the response. If that input is *reflected* in the response, you can poison it.

### Common unkeyed inputs to try

**Headers** (send one at a time, with a cache buster):

```
X-Forwarded-Host: axepoison123.example
X-Forwarded-Scheme: http
X-Forwarded-Proto: http
X-Original-URL: /axepoison123
X-Rewrite-URL: /axepoison123
X-Host: axepoison123.example
Forwarded: host=axepoison123.example
Origin: https://axepoison123.example
```

**Query parameters** — many caches ignore marketing parameters, but the app may reflect them:

```
?utm_source=axepoison123
?callback=axepoison123        (JSONP-style endpoints)
?lang=axepoison123
```

### Confirm the bug

1. Send `GET /page?cb=axe1` **with** the suspicious header and your unique marker.
2. Send `GET /page?cb=axe1` again **without** the header.
3. If your marker still appears in the 2nd response → the cache stored your input. **Poisoning confirmed.**

Example:

```bash
# Request 1 - with the unkeyed header
curl -s https://example.com/page?cb=axe1 -H "X-Forwarded-Host: axepoison123.example"

# Request 2 - clean request, no header
curl -s https://example.com/page?cb=axe1
# axepoison123.example in this response = served from cache = vulnerable
```

### Escalate it

- Marker reflected inside a **script URL / canonical link / redirect** → replace it with your domain → XSS or phishing for every visitor.
- Marker reflected **unescaped in the HTML body** → direct stored XSS via the cache.
- Nothing reflected? Try **fat GET** (GET request with a body), **parameter cloaking** (`;` instead of `&`, e.g. `?cb=axe1;utm_source=marker` — some caches treat everything after `;` as a path parameter and ignore it), and duplicate parameters.

### Tool that does this automatically

**Param Miner** (Burp BApp Store extension) — it guesses unkeyed headers and parameters for you and flags reflections. Run it on the target page, then verify every finding manually with the cache-buster method above.

---

## Step 3: Web Cache Deception — Steal a Cached Private Page

Here the cache is tricked into storing a page it should never cache. The trick is a URL that the **origin** treats as the private page, but the **cache** treats as a static file.

### The classic payloads

If the private page is `/account/profile`, try:

```
/account/profile/axepoison.css
/account/profile%0aaxepoison.css
/account/profile%3baxepoison.css
/account/profile..;/axepoison.css
/account/profile/..%2faxepoison.css
```

Why it works: many caches cache **by file extension** (`.css`, `.js`, `.png` are always cached), while the origin ignores the extra path part (path parameter `;`, encoded newline `%0a`, or a non-existent sub-path) and returns the real profile page.

### Confirm the bug

1. While **logged in as yourself**, open `https://example.com/account/profile/axepoison.css`.
2. Check the response headers — if it says `X-Cache: HIT` / `CF-Cache-Status: HIT` and the body is your profile page, it was cached.
3. Open the **exact same URL in a private/incognito window (logged out)**.
4. If you can still see your private data → **deception confirmed.** Anyone with that link can read it until the cache expires.

Test only with **your own account** — never trick a real victim into clicking such a link while you are still just confirming the bug for a report.

---

## Hunting Checklist

- [ ] Identify cached pages (`Age`, `X-Cache`, `CF-Cache-Status` headers)
- [ ] Note the `Vary` header — everything NOT listed there is a potential unkeyed input
- [ ] Try common unkeyed headers one by one, always with a fresh cache buster
- [ ] Try reflected query parameters (`utm_*`, `callback`, `lang`)
- [ ] Confirm with a clean second request — marker must come from the cache
- [ ] Try parameter cloaking (`;`) and fat GET if simple attempts fail
- [ ] Run Param Miner and verify its findings manually
- [ ] For deception: append `.css` / `.js` with `;`, `%0a`, `%3b`, `/` tricks to a private page
- [ ] Confirm deception by opening the URL logged-out
- [ ] Report with both requests (poisoning request + clean request) as proof

---

## Tools

1. [Param Miner](https://portswigger.net/bappstore/17d2949a985c4e531c56bc1422a93e33) — Burp extension to find unkeyed headers/parameters automatically
2. [Web Cache Vulnerability Scanner (wcvs)](https://github.com/Hackmanit/Web-Cache-Vulnerability-Scanner) — automated cache poisoning/deception checks
3. Burp Repeater + curl — for the manual cache-buster confirmation method above

## References

- [PortSwigger — Web Cache Poisoning](https://portswigger.net/web-security/web-cache-poisoning)
- [PortSwigger — Web Cache Deception](https://portswigger.net/web-security/web-cache-deception)
- [James Kettle — Practical Web Cache Poisoning](https://portswigger.net/research/practical-web-cache-poisoning)
- [Omer Gil — Web Cache Deception Attack](https://omergil.blogspot.com/2017/02/web-cache-deception-attack.html)

---

### Author

[Asad Faizee](https://github.com/Faizee-Asad)

###### If you think something was missed, feel free to add/modify/delete it :)
