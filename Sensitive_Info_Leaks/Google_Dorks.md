# Google Dorks — Juicy Content via Search Engines

Search engines index things nobody meant to expose: directory listings, log files, `.env` configs, SQL dumps, and even password-reset tokens. Google dorks turn `site:`/`inurl:`/`intitle:` operators into a passive scanner for sensitive content. Typical impact: credential/PII disclosure and exposed admin content — and what Google indexes, it cached, so removal takes real effort.

## Hunting Checklist

1. **Enumerate in-scope domains** first (see [Recon workflow](../Recon/Workflow.md)) — dorks are per-domain.
2. **Run the dork list below** for each domain, swapping `example.com`.
3. **Rotate search engines** — besides Google, try these dorks on DuckDuckGo, Bing, Yandex, etc. (indexing differs; one engine's hit is another's miss, and Google may block automated queries).
4. **Check cached/archived copies** — if the live page is gone, Google cache or the Wayback Machine may still prove exposure.
5. **Assess what's actually leaked** (credentials? PII? tokens?) before reporting, and don't redistribute third-party data found in listings.

## Dork List

```
inurl:example.com intitle:"index of"
inurl:example.com intitle:"index of /" "*key.pem"
inurl:example.com ext:log
inurl:example.com intitle:"index of" ext:sql|xls|xml|json|csv
inurl:example.com "MYSQL_ROOT_PASSWORD:" ext:env OR ext:yml -git
inurl:example.com intitle:"index of" "config.db"
inurl:example.com allintext:"API_SECRET*" ext:env | ext:yml
inurl:example.com intext:admin ext:sql inurl:admin
inurl:example.com allintext:username,password filetype:log
site:example.com "-----BEGIN RSA PRIVATE KEY-----" inurl:id_rsa
site:*.gov.* "responsible disclosure"
```

![t](https://miro.medium.com/max/550/1*N9W6DfGA6wxgKTiywV9aUA.png)

Reference: [How recon helped Samsung protect their production repositories](https://blog.usejournal.com/how-recon-helped-samsung-protect-their-production-repositories-of-samsungtv-ecommerce-estores-4c51d6ec4fdd)

#### Other than Google, try these dorks on various search engines such as DuckDuckGo, Bing, etc.

## Bypass Techniques

- **Google rate-limits/CAPTCHAs fast scripted queries:** lower your query rate, or use Bing/DuckDuckGo/Yandex for the same operators (`site:`, `inurl:`, `intitle:` work nearly everywhere).
- **Page removed but still sensitive?** Chase the evidence via cache and web.archive.org snapshots.
- **Indexing side-effects worth knowing for impact:** token-bearing URLs (e.g. password-reset links) can be indexed through `X-Robots-Tag`-less responses, referrer leakage, and crawlers following links from exposed pages — this is why reset-token reports below are accepted.

## Tools & Resources

- [Google Hacking Database (ExploitDB)](https://www.exploit-db.com/google-hacking-database) — largest public dork collection for more inspiration
- Multi-engine query runners (any dork-search CLI that supports `site:`/`inurl:` operators on Bing/DuckDuckGo/Yandex)
- [Wayback Machine](https://web.archive.org) / Google cache — historical proof
- Related: [Shodan CVE dorks](Shodan_cve_dorks.md), [GitHub dorks](Github-dorks.md)

## Reports (HackerOne)

### Resolved

- [Securing "Reset password" pages from bots](https://hackerone.com/reports/43807)
- [Private Grab Messages on Android App can be accessed and cached by Search Engines](https://hackerone.com/reports/221558)

### Informative

- [Information disclosure through search engines (password reset token)](https://hackerone.com/reports/322988)

### N/A

- [Research papers on yelp are getting indexed by google bots.](https://hackerone.com/reports/207435)

## Report Tips

- Show the *sensitive thing* indexed (token, credentials, private content), not just that a page exists in search results — that's what separated the resolved reports from the informative/N/A ones above.
- Include the dork used and the exact indexed URL + timestamp so triage can reproduce.
- Recommend the fix in report terms: `noindex`/`X-Robots-Tag`, robots + auth on directories, and purging cached URLs.

## Author

- [Keshav Malik](https://twitter.com/g0t_rOoT_)
- [Naveen Prakaasham](https://twitter.com/NPrakaasham)
- [@klaus](https://twitter.com/klaus_dev)
- [Fani Malik](https://twitter.com/fanimalikhack)
