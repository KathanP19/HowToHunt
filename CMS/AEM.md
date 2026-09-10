# Adobe Experience Manager (AEM)

AEM (formerly Day Communique/CQ) is an enterprise CMS built on a JCR content repository (CRX) fronted by the Dispatcher cache. Its architecture has a very recognizable URL model — `/content/...` pages with `.html` suffixes, plus internal trees (`/libs`, `/apps`, `/etc`, `/var`) and `.json` node endpoints — and when the Dispatcher ACLs are misconfigured those internals leak: information disclosure, SSRF, CSRF, and (via exposed consoles) RCE. Impact is commonly High–Critical.

## Hunting Checklist

1. **Confirm AEM:** pages served as `/content/<site>/...html`; HTML references to `/etc.clientlibs/` or `/etc/designs/`; Wappalyzer/whatweb AEM/CQ signatures.
2. **Run the dedicated scanners first** (they automate most of the list below):

```bash
python aemscan.py --url https://target.com
python aem-hacker.py --domain target.com --enum all
```

3. **Fuzz the wordlist** for AEM-sensitive paths ([aem.txt](https://raw.githubusercontent.com/clarkvoss/AEM-List/main/paths)).
4. **Probe restricted trees** directly: `/apps/`, `/etc/`, `/var/`, `/content/` listings and `/bin/` endpoints.
5. **Test JSON node exposure:** append `.json`, `.nocache.json`, `.infinity.json` to pages and `/content/...` nodes.
6. **Try the author-mode tells:** `/editor.html`, `/sites.html`, `/crx/de/index.jsp` (CRXDE Lite), `?wcmmode=disabled` on a normal page, `/system/console/` (OSGi/Felix web console), `/bin/groovyconsole` (RCE if reachable without auth).
7. **Check request-surface bugs the tools flag:** SSRF/CSRF via replication & dispatcher-flush endpoints, stored XSS in authoring components, `.html.html`/selector path confusion.
8. **Separate environments in recon:** author vs publish hosts (`author.target.com`, `publish.target.com`) frequently differ wildly in exposure.

## Test Cases & Examples

```http
GET /apps/ HTTP/1.1
Host: target.com

GET /etc/ HTTP/1.1
Host: target.com

GET /content/geodypSites/geodyp/global/en.json HTTP/1.1
Host: target.com

GET /content/target-page.infinity.json HTTP/1.1
Host: target.com
```

Author/UI probes:

```http
GET /editor.html HTTP/1.1
Host: target.com

GET /crx/de/index.jsp HTTP/1.1
Host: target.com

GET /content/page.html?wcmmode=disabled HTTP/1.1
Host: target.com

GET /system/console/bundles HTTP/1.1
Host: target.com
```

CLI fuzzing:

```bash
ffuf -u 'https://target.com/FUZZ' -w paths.txt -mc 200,403
aem-hacker.py --domain target.com --file <subdomain> --outputdir output/
aem-hacker.py --domain target.com --checkpath /etc/ --addcustomheader "X-Forwarded-For: 127.0.0.1"
```

## Bypass Techniques

- **Selector/extension confusion** — Dispatcher ACLs often match only the exact extension; retry blocked paths with added selectors/extensions (`.html.html`, `.json.jsp`, `.custom.css.html`, `.infinity.xml` style variants) — the scanners automate the known set.
- **Encoding noise** — case changes, trailing `//`, `%2e%2e`, tab/newline in the path to slip naive prefix rules.
- **Header spoofing to reach author endpoints on publish** — `X-Forwarded-For: 127.0.0.1` (as above) and similar edge-client-IP headers.
- Cache tricks — request the JSON node with a unique query param to force an origin (non-cached) fetch.

## Tools & Resources

**Tools:**

- [aem-hacker](https://github.com/0ang3el/aem-hacker) — enumerates AEM sensitive paths, JSON node exposure, SSRF/CSRF vectors
- [aemscan](https://github.com/Raz0r/aemscan) — AEM instance/path scanner

**Wordlist for fuzzing:**

- [aem.txt](https://raw.githubusercontent.com/clarkvoss/AEM-List/main/paths)

**Resources:**

- [Approaching AEM (Bugcrowd webinar)](https://www.bugcrowd.com/resources/webinar/aem-hacker-approaching-adobe-experience-manager-web-apps/)
- [Securing AEM (slides by 0ang3el)](https://www.slideshare.net/0ang3el/securing-aem-webapps-by-hacking-them)

## Report Tips

- An exposed internal tree or `.infinity.json` dump is High on its own (config, credentials, design internals) — attach only the sensitive fragments, not the whole dump.
- `/bin/groovyconsole` or reachable OSGi console without auth = Critical: prove with a read-only action (list bundles / echo a value), never deploy persistence.
- Always state author vs publish host and the exact path that bypassed the Dispatcher — it tells the fix owner which ACL layer broke.
