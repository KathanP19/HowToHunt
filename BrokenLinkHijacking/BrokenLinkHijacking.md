# Broken Link Hijacking

Applications reference external resources — social pages, widgets, SDK snippets, CDN scripts — that eventually go dead (404). If the vanished destination can be *registered by an attacker* (a GitHub org, Twitter/LinkedIn company page, Slack subdomain, or a public npm/CDN package path), the target effectively hands control of that slot to whoever claims it. Impact ranges from brand-defacing content to stored XSS/cookie theft and full site takeover when the link is a script include.

## Hunting Checklist

1. Crawl every page in scope and collect **external** links and resource includes (JS/CSS/iframes/widgets).
2. Flag every dead one: HTTP 4xx/5xx responses and DNS-not-found especially on third-party hosts.
3. For each dead link, determine whether the resource is **registrable** (account name, org name, package name, URL shortener keyword).
4. Check whether the company actually owns the page now — if not, register it as the company/claim the username *with the program's permission where required*, or just demonstrate the takeover path.
5. Prioritize **script includes over profile links** — a hijacked `<script src>` = JS execution on the target; a dead social link = brand issue.
6. Also scan source/bundles for *commented-out or stale* CDN references that only execute on some paths.

## Test Cases & Examples

### Manual + automated link crawl

1. Manually find and click external links on the target site (e.g. links to social media accounts or external media).
2. In parallel, run [broken-link-checker](https://github.com/stevenvachon/broken-link-checker) in the background in your terminal:

```bash
blc -rof --filter-level 3 https://example.com/
```

Output looks like:

```
─BROKEN─ https://www.linkedin.com/company/ACME-inc-/ (HTTP_999)
```

3. Check whether the company still owns that page — if not, register it as the company or obtain that username/URL.

### Alternate (online) steps

1. Go to [Online Broken Link Checker (Ahrefs)](https://ahrefs.com/broken-link-checker), [Dead Link Checker](https://www.deadlinkchecker.com/), or [Alternative Broken Link Checker](https://brokenlinkcheck.com/).
2. Input the domain name and review dead external links.

### What to check for hijackability

```bash
# Dead GitHub org/user links
curl -sI https://github.com/dead-org-name        # 404 = org available to create

# Dead Twitter/X company handle
curl -sI https://twitter.com/dead_handle

# Missing CDN package paths referenced by the site (classic sub-resource hijack)
# e.g. <script src="https://cdn.jsdelivr.net/npm/<pkg>@<ver>/lib.js"> → 404 + unclaimed <pkg> on npm
```

Common hijackable destinations: GitHub (orgs/repos), Twitter/X, LinkedIn company pages, Slack subdomains, Bitly/custom shorteners, Gumroad, Heroku apps, unpkg/jsdelivr/CDN paths backed by unclaimed npm packages, and custom 404-vanity URLs on other platforms.

## Bypass Techniques

- **Filter-level 3 in `blc`** hides 403/429 "false alive" pages — cross-check suspicious statuses manually; many sites rate-limit crawlers (HTTP_999 = LinkedIn blocking you, not a dead page — verify by opening in a browser).
- **Robots/excluded paths:** crawlers miss links behind JS/SPAs — also grep page source and JS bundles for `http` URLs rather than trusting crawl results alone.
- CSP doesn't help you spot dead links, but remember hijacks work only when the page *includes* the resource — confirm the include still executes (browser devtools: Network tab, blocked/failed requests).

## Tools & Resources

- [broken-link-checker (`blc`)](https://github.com/stevenvachon/broken-link-checker) — CLI crawler used above
- [lychee](https://github.com/lycheeverse/lychee) — fast link checker for CI-scale crawling
- [waybackpy](https://github.com/rikisoni/waybackpy) / [web.archive.org](https://web.archive.org) — confirm when the external resource died (evidence of exposure window)
- Reference write-ups: [EdOverflow — Broken Link Hijacking](https://edoverflow.com/2017/broken-link-hijacking/), [How I took over the company's LinkedIn page](https://medium.com/@bathinivijaysimhareddy/how-i-takeover-the-companys-linkedin-page-790c9ed2b04d)

## Report Tips

- Severity depends on what executes: **dead script/CDN include = "stored XSS/account-takeover potential" (high/critical)**; a dead profile link = "brand/impersonation" (low/informative) — frame accordingly.
- PoC best practice: register the missing resource (with program rules in mind), show it loads *in the context of the target page*, and include a console `alert(document.domain)`-style proof for script hijacks — then release the account.
- Include crawl evidence: the exact page + line referencing the URL, the 404/not-owned proof, and the first-seen timestamp (Wayback).

## Author

- [@KathanP19](https://twitter.com/KathanP19)
- [@cyph3r_asr](https://twitter.com/cyph3r_asr)
- [@v3daxt](https://twitter.com/v3daxt)
